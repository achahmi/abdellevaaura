# Confinamiento de Nginx con AppArmor

## 1. Objetivo

En esta práctica voy a instalar y comprobar Nginx en Linux, crear un perfil de seguridad con AppArmor, observar su comportamiento en modo **complain** (aprendizaje), ajustar los permisos necesarios y activarlo en modo **enforce** para comprobar que el servidor web sigue funcionando.

## 2. Entorno utilizado

- Sistema operativo: **Kali Linux**
- Máquina: **Virtual**
- Servidor web: **Nginx**
- Herramienta de seguridad: **AppArmor**
- Dirección IP utilizada: **192.168.111.58/24**

Comandos para consultar el entorno:

```bash
cat /etc/os-release
ip -br a
nginx -v
sudo aa-status
```

<img width="696" height="539" alt="image" src="https://github.com/user-attachments/assets/7f9b8c73-baeb-48e6-816d-28e1d26c9f88" />


## 3. Instalación y configuración inicial de Nginx

Primero instalé las herramientas necesarias y dejé Nginx arrancado para poder comprobar que respondía correctamente antes del confinamiento.

```bash
sudo apt update
sudo apt install -y nginx apparmor apparmor-utils
sudo systemctl enable --now apparmor
sudo systemctl enable --now nginx
sudo systemctl status nginx --no-pager
curl -I http://127.0.0.1
```

**Captura 2 — Nginx funcionando:** <img width="946" height="778" alt="image" src="https://github.com/user-attachments/assets/6660a1c6-ce77-49d5-8877-d79fccb3698c" />


## 4. Creación del perfil de AppArmor

Para generar el perfil del ejecutable de Nginx utilicé la herramienta `aa-genprof`. Esta herramienta permite ejecutar el servicio, revisar los accesos que realiza y decidir qué permisos necesita.

```bash
sudo aa-genprof /usr/sbin/nginx
```

Mientras la herramienta estaba abierta, desde **otra terminal** generé peticiones al servidor:

```bash
curl -I http://127.0.0.1
curl http://127.0.0.1/ > /dev/null
```

Después volví a `aa-genprof`, analicé los eventos con la opción de escanear los registros, acepté solo los permisos que necesitaba Nginx y guardé el perfil generado.

**Captura 3 — Generación del perfil:** 
<img width="536" height="470" alt="image" src="https://github.com/user-attachments/assets/f63a3adc-d2aa-4d04-b44e-ba52909a6bba" />




## 5. Modo complain y revisión de los registros

El modo **complain** permite que el programa siga funcionando mientras AppArmor registra las acciones que el perfil no autorizaría. Es útil para detectar los accesos necesarios sin bloquear el servidor durante las pruebas.

```bash
sudo aa-complain /usr/sbin/nginx
sudo aa-status
curl -I http://127.0.0.1
sudo journalctl -k --no-pager | grep -i apparmor | tail -n 30
```

En los registros busqué entradas relacionadas con `nginx` y `apparmor`, especialmente eventos con `apparmor="ALLOWED"` en modo complain. Si no aparecían eventos, revisé los registros después de hacer más peticiones: **no se deben inventar registros**.

**Captura 4 — Modo complain:** 
<img width="653" height="399" alt="image" src="https://github.com/user-attachments/assets/469c27de-91f8-4016-ba32-1aeddbb9766b" />


**Captura 5 — Registros reales:** 
<img width="936" height="747" alt="image" src="https://github.com/user-attachments/assets/76472e24-0ec6-4689-b362-ed6b091f6cfe" />


**Eventos observados:** 

nov 16 15:24:26 kali kernel: audit: type=1400 audit(1794839066.938:219): apparmor="ALLOWED" operation="open" class="file" profile="/usr/sbin/nginx" name="/etc/nginx/sites-enabled/" pid=8641 comm="nginx" requested_mask="r" denied_mask="r" fsuid=0 ouid=0

nov 16 15:24:32 kali kernel: audit: type=1400 audit(1794839072.002:280): apparmor="ALLOWED" operation="accept" class="net" profile="/usr/sbin/nginx" pid=8646 comm="nginx" lport=80 family="inet" sock_type="stream" protocol=6 requested_mask="accept" denied_mask="accept"

nov 16 15:24:32 kali kernel: audit: type=1400 audit(1794839072.002:281): apparmor="ALLOWED" operation="recvmsg" class="net" profile="/usr/sbin/nginx" pid=8646 comm="nginx" laddr=127.0.0.1 lport=80 faddr=127.0.0.1 fport=43460 family="inet" sock_type="stream" protocol=6 requested_mask="receive" denied_mask="receive"


## 6. Ajuste y perfil resultante

Después de probar Nginx, revisé los eventos con `aa-logprof` para incluir en el perfil únicamente los permisos necesarios.

```bash
sudo aa-logprof
sudo cat /etc/apparmor.d/usr.sbin.nginx
```

**Captura 6 — Perfil ajustado:** 
<img width="409" height="271" alt="image" src="https://github.com/user-attachments/assets/2a12feb6-445d-41ce-a77b-f106a37dc437" />


### Perfil final obtenido

 sudo cat /etc/apparmor.d/usr.sbin.nginx
Last Modified: Mon Nov 16 15:34:45 2026
abi <abi/4.0>,

include <tunables/global>

/usr/sbin/nginx flags=(complain) {
  include <abstractions/base>

  capability dac_override,

  /usr/sbin/nginx mr,
  /var/log/nginx/error.log w,

}


### Accesos necesarios y motivo

- **Ejecutable y librerías:** para que Nginx pueda arrancar y cargar sus dependencias
- **Archivos de configuración (`/etc/nginx/`):** para leer la configuración del servidor
- **Archivos de la web (`/var/www/html/`):** para servir las páginas al navegador
- **Registros (`/var/log/nginx/`):** para guardar accesos y errores
- **Archivos de ejecución (`/run/`):** para gestionar procesos y, cuando corresponda, el archivo PID

Las rutas y permisos exactos se deben confirmar con el perfil y los registros de esta práctica.

## 7. Activación en modo enforce

Una vez revisado el perfil, activé el modo **enforce**, que sí aplica las restricciones: cualquier acceso no permitido por las reglas puede bloquearse.

```bash
sudo aa-enforce /usr/sbin/nginx
sudo aa-status
sudo systemctl restart nginx
sudo systemctl status nginx --no-pager
curl -I http://127.0.0.1
```

**Captura 7 — Perfil en enforce:** 

<img width="758" height="350" alt="image" src="https://github.com/user-attachments/assets/93fec805-7a2e-4ded-b201-85189ed66f31" />


**Captura 8 — Funcionamiento tras el confinamiento:**
<img width="907" height="331" alt="image" src="https://github.com/user-attachments/assets/d7647da8-59a2-4e88-90c7-97931e727eb7" />



## 8. Diferencia entre complain y enforce

| Modo | Qué hace | Cuándo interesa |
|---|---|---|
| **Complain** | Registra accesos que no encajan con el perfil, pero normalmente no los bloquea | Mientras creamos y ajustamos el perfil |
| **Enforce** | Hace cumplir las reglas y bloquea accesos que no estén permitidos | Cuando el perfil ya está preparado y probado |

## 9. Resultado y reflexión

**Resultado final:** Bueno, a pesar de que me ha dado bastantes probelemas el nginx al ponerlo en enforce, al final lo he podido solucionar y funciona correctamente:
<img width="952" height="279" alt="image" src="https://github.com/user-attachments/assets/f07b533e-d0a4-489d-af48-d20ddd0773af" />



**Reflexión:** He comprobado que Nginx necesita acceder a sus archivos de configuración, a los archivos que sirve y a algunos recursos del sistema para poder trabajar. La parte más importante ha sido permitir solo los accesos necesarios, en lugar de dar permisos a todo. El modo complain me ha servido para identificar los accesos durante las pruebas y el modo enforce para comprobar que el servidor puede funcionar con restricciones.

**Nota:** Las capturas, los eventos y el contenido del perfil deben añadirse a partir de la ejecución real de la práctica. Este documento es una plantilla de entrega, no una certificación de ejecución.
