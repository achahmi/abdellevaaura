# ACT4 · Investigar ataques DHCP

| | |
| --- | --- |
| **Práctica** | Protocolo DHCP - Seguridad |
| **Módulo** | 037 Seguridad en servicios |
| **Ciclo Formativo** | ASIX |
| **Alumno** | Abdellah Chahmi |

**Objetivo:** análisis e implementación de la seguridad en el servicio DHCP.

---

## 1. Fundamentos y arquitectura del protocolo DHCP

### 1.1 El proceso DORA y el broadcast

**Los cuatro pasos del intercambio DHCPv4 (DORA):**

1. **DHCPDISCOVER:** el cliente, que todavía no tiene IP, manda un mensaje en **broadcast** (a toda la red) preguntando si hay algún servidor DHCP disponible.
2. **DHCPOFFER:** el servidor (o servidores, si hay varios) responde ofreciendo una IP libre del rango, también en broadcast, porque el cliente aún no tiene IP para recibir un mensaje dirigido solo a él.
3. **DHCPREQUEST:** el cliente elige una de las ofertas (normalmente la primera que le llega) y pide esa IP en concreto, otra vez en broadcast, para que el resto de servidores DHCP que también ofrecieron sepan que su oferta no fue la elegida y liberen esa IP.
4. **DHCPACK:** el servidor elegido confirma la asignación y le manda al cliente el resto de los datos de red: máscara, puerta de enlace, DNS, y el tiempo de concesión (*lease time*).

**Implicaciones de seguridad del broadcast:** como todos estos mensajes se envían a toda la red, **cualquier equipo del mismo segmento puede verlos**, aunque no sean para él. Esto le da a un atacante dos puertas abiertas: puede **escuchar** todas las peticiones y ofertas que pasan por la red (sabiendo así qué equipos se están conectando), y puede **responder él mismo** a un DHCPDISCOVER antes que el servidor legítimo, haciéndose pasar por un servidor DHCP falso (*rogue DHCP*) y entregando una configuración de red manipulada, por ejemplo con una puerta de enlace o un DNS bajo su control.

### 1.2 La anatomía del paquete DHCP (Capa 7 vs Capa 2)

**El campo `chaddr`:** dentro del mensaje DHCP (que viaja en Capa 7, dentro de UDP/IP) hay un campo llamado `chaddr` (*Client Hardware Address*), donde el propio cliente **escribe** cuál dice que es su dirección MAC. Es un dato que aporta el cliente dentro del contenido del mensaje.

**Diferencia con la MAC de origen de Ethernet:** la trama Ethernet (Capa 2) también lleva una dirección MAC de origen, pero esa la pone la propia tarjeta de red al construir la trama, no es un dato "declarado" dentro del mensaje. Son dos campos distintos, en dos capas distintas, y lo normal es que coincidan, pero nada obliga a que así sea.

**Por qué el servidor usa `chaddr` y la vulnerabilidad que introduce:** el servidor DHCP trabaja a nivel de aplicación, leyendo el contenido del mensaje DHCP, así que usa el `chaddr` que el cliente ha escrito para decidir qué IP darle y para llevar la cuenta de qué direcciones tiene ya asignadas. El problema es que, como `chaddr` es un simple campo de texto que rellena el propio cliente, un atacante puede **inventarse el valor que quiera** sin necesidad de tocar la tarjeta de red real. Esto abre la puerta al *spoofing*: un mismo equipo, con una sola tarjeta de red, puede fingir ser cientos de clientes distintos solo cambiando el `chaddr` en cada petición, que es exactamente lo que aprovecha un ataque de Starvation.

### 1.3 Carencia de autenticación

**RFC 2131:** el estándar de DHCP **no incluye ningún mecanismo obligatorio de autenticación ni de cifrado**. Cualquier equipo conectado a la red puede mandar mensajes DHCP y nada le impide hacerse pasar por servidor (contestando DHCPOFFER) ni pedir todas las IPs que quiera (mandando muchos DHCPDISCOVER con distintos `chaddr`). El protocolo confía en que todos los equipos de la red se comportan bien, sin comprobar la identidad de nadie.

**RFC 3118 (Autenticación para mensajes DHCP):** esta propuesta añadía una opción al mensaje DHCP con una firma o un código de autenticación (basado en una clave compartida o, en su versión más avanzada, en criptografía de clave pública), para que el cliente y el servidor pudieran comprobar que los mensajes eran legítimos. En la práctica, su adopción ha sido casi nula por varios motivos: requiere que **todos** los equipos de la red (clientes y servidores) lo soporten e implementen correctamente, lo cual es poco realista en redes con muchos dispositivos distintos (móviles, impresoras, IoT...); complica mucho la gestión de las claves compartidas; y, sobre todo, la industria ha preferido resolver el problema de otra forma, **en la infraestructura de red** (switches), con tecnologías como DHCP Snooping, en vez de modificar el protocolo DHCP en sí.

---

## 2. Vectores de ataque: Starvation

### 2.1 Mecánica del DHCP Starvation

**Cómo se agota el pool, paso a paso:**

1. El atacante manda muchísimos mensajes DHCPDISCOVER seguidos, cada uno con un `chaddr` distinto (inventado).
2. El servidor DHCP legítimo, al no poder distinguir que todas esas peticiones vienen del mismo atacante, responde a cada una con una oferta y, si llega el DHCPREQUEST, concede una IP distinta para cada `chaddr` falso.
3. Esto se repite hasta que el servidor se queda sin direcciones libres en su rango (*pool*).
4. A partir de ahí, cualquier cliente legítimo nuevo que llegue a la red no recibirá ninguna oferta, porque ya no quedan IPs disponibles.

**Starvation simple vs. starvation complejo:**
- **Simple:** el atacante solo cambia el `chaddr` dentro del mensaje DHCP (Capa 7), pero sigue usando la misma MAC de origen real de su tarjeta de red en la trama Ethernet (Capa 2).
- **Complejo:** el atacante cambia **también** la MAC de origen de la trama Ethernet en cada envío, para que cada petición parezca venir de un dispositivo físico distinto, no solo de un cliente DHCP distinto.

**Por qué el escenario complejo es más difícil de mitigar para un switch:** muchas defensas a nivel de switch (como Port Security) se basan en contar o limitar **cuántas direcciones MAC de Capa 2** se ven por un mismo puerto físico. Si el atacante solo cambia el `chaddr` (ataque simple), el switch sigue viendo siempre la misma MAC de Ethernet en ese puerto, así que Port Security no detecta nada raro. Si además cambia la MAC de Capa 2 (ataque complejo), el switch empieza a ver muchísimas MACs distintas por el mismo puerto físico, lo cual sí es detectable y limitable con Port Security, pero obliga a configurar esta protección en todos los puertos, y los límites hay que ajustarlos bien para no bloquear situaciones legítimas (como varios equipos detrás de un switch pequeño sin gestionar).

### 2.2 Herramientas de explotación

**Yersinia y Scapy:** Yersinia es una herramienta pensada específicamente para atacar protocolos de Capa 2, con un módulo que automatiza el envío masivo de DHCPDISCOVER con `chaddr` aleatorios para agotar el pool. Scapy, en cambio, es una librería de Python para construir paquetes de red a mano, campo a campo; con ella se puede escribir un script que genere los mensajes DHCP exactamente como se quiera (eligiendo si se cambia solo el `chaddr` o también la MAC de Ethernet, la velocidad de envío, etc.).

**Ventajas de automatizar con un script frente a hacerlo manualmente:** a mano, mandar un solo DHCPDISCOVER con un `chaddr` falso no tiene ningún efecto real; para agotar un pool hace falta repetir el proceso cientos o miles de veces en poco tiempo, algo que una persona no puede hacer a mano con la velocidad y precisión necesarias. Un script permite generar muchas peticiones por segundo, variar automáticamente el `chaddr` (y la MAC) en cada una, y repetir el ataque de forma exacta cuantas veces haga falta para hacer pruebas.

**Ciclo de vida del lease (tiempo de concesión):**
- Si el *lease time* es de **7 días** y el atacante se desconecta justo después de agotar el pool, el efecto del ataque dura mucho tiempo: las IPs quedan "reservadas" para esos clientes falsos (que ya no existen) hasta que se cumplan los 7 días y el servidor las libere por no haberse renovado. Durante todo ese tiempo, ningún cliente nuevo legítimo podrá conseguir IP.
- Si el *lease time* es de **5 minutos**, el efecto es mucho más corto: en cuanto el atacante deja de renovar esas concesiones falsas, pasados los 5 minutos el servidor las da por caducadas y las vuelve a poner disponibles. El ataque solo es efectivo mientras el atacante sigue enviando tráfico de forma continua.

### 2.3 Consecuencias colaterales

**Qué pasa con los nuevos dispositivos:** cuando el pool se agota, un cliente nuevo manda su DHCPDISCOVER pero nadie le responde con una oferta (no hay IPs libres que ofrecer). El cliente se queda sin configuración de red por la vía normal.

**APIPA (169.254.0.0/16):** es un rango de direcciones privado y reservado que los sistemas Windows y Linux se **autoasignan ellos mismos**, sin ayuda de ningún servidor, cuando llevan un tiempo sin recibir respuesta a sus peticiones DHCP. Es una forma de que el equipo tenga al menos una IP y pueda, como mínimo, comunicarse con otros equipos de su misma red local que también se hayan autoconfigurado con APIPA. Sus limitaciones son grandes: no tiene puerta de enlace ni servidor DNS asignados, así que **no hay salida a Internet ni a otras redes**, y solo sirve para comunicarse dentro del mismo segmento local con otros equipos que también tengan una IP APIPA.

---

## 3. Hardening y mitigaciones

### 3.1 La solución a nivel de infraestructura (Capa 2)

**DHCP Snooping:** es una función de los switches de acceso que filtra el tráfico DHCP según de qué puerto venga.

- Un puerto **trusted** (de confianza) es aquel por donde el switch sabe que puede llegar tráfico de un servidor DHCP legítimo (normalmente el puerto que va hacia el servidor o hacia el router). Por esos puertos se permite el tráfico de servidor (DHCPOFFER, DHCPACK).
- Un puerto **untrusted** (no de confianza) es el resto de puertos, los que van a los equipos normales de usuario. Por esos puertos el switch **bloquea** los mensajes que solo debería mandar un servidor (DHCPOFFER, DHCPACK), y solo deja pasar los mensajes propios de un cliente (DHCPDISCOVER, DHCPREQUEST).

**Cómo evita un servidor DHCP no autorizado (rogue DHCP):** si alguien conecta un servidor DHCP falso en un puerto de usuario (untrusted), sus respuestas (DHCPOFFER) quedarán bloqueadas por el switch antes de llegar al resto de la red, porque ese puerto no tiene permiso para mandar ese tipo de mensajes.

**DHCP Snooping Binding Database:** es una tabla que el switch va construyendo a partir del tráfico DHCP legítimo que observa, y que guarda, por cada cliente, datos como: su dirección MAC, la IP que le ha asignado el servidor, el tiempo de concesión, y el puerto y VLAN por los que ese cliente está conectado. Esta base de datos sirve como referencia para otras protecciones, como comprobar más adelante que un paquete que dice venir de una IP concreta realmente viene del puerto y la MAC que tienen esa IP asignada.

**Port Security:** limita cuántas direcciones MAC de origen distintas puede ver el switch en un mismo puerto físico. Si se fija, por ejemplo, un máximo de una o dos MACs por puerto, un ataque de Starvation que cambie también la MAC de Ethernet en cada petición (el "complejo" del apartado 2.1) se detecta enseguida, porque ese puerto empezaría a mandar tramas con muchas MACs distintas, superando el límite permitido; el switch entonces puede bloquear el puerto o descartar ese tráfico.

### 3.2 La mitigación a nivel de host (Linux)

**El módulo `hashlimit` de iptables:** es un módulo que permite limitar la **tasa** (*rate limit*) de paquetes que llegan, es decir, cuántos paquetes por segundo (o por minuto) se aceptan antes de empezar a descartar el resto. A diferencia del módulo `limit` básico, `hashlimit` puede aplicar ese límite **por cada origen** (por ejemplo, por cada IP o por cada combinación de IP y puerto), en vez de un único límite global para todo el tráfico. Es útil contra DHCP Starvation porque este ataque consiste precisamente en mandar una ráfaga muy rápida de peticiones DHCPDISCOVER: con `hashlimit` se puede decir, por ejemplo, "acepta como máximo tantas peticiones DHCP por segundo", y el resto de la ráfaga se descarta antes de que el servidor DHCP llegue a procesarla y a gastar IPs del pool.

**ebtables:** es la herramienta equivalente a `iptables` pero para **Capa 2** (tramas Ethernet), mientras que `iptables` trabaja en Capa 3 y superiores (paquetes IP). `ebtables` permite filtrar tráfico mirando campos de la propia trama Ethernet, como la MAC de origen o destino.

En nuestro entorno de VirtualBox, sin un switch físico gestionable, se podría usar `ebtables` en el puente de red (bridge) de Linux para comparar la MAC de origen de la trama Ethernet (Capa 2) con el campo `chaddr` que lleva dentro el propio mensaje DHCP (Capa 7). Si ambos valores no coinciden, es una señal clara de que alguien está falsificando uno de los dos, y el tráfico se podría descartar directamente en el bridge antes de que llegue al servidor DHCP.

### 3.3 Lease policies como defensa

**Equilibrio del tiempo de concesión:** un *lease time* muy corto hace que, si ocurre un ataque de Starvation, el efecto dure poco tiempo (las IPs falsas se liberan rápido), pero a cambio obliga a todos los clientes legítimos a renovar su concesión muy a menudo, generando mucho tráfico DHCP de renovación y más carga en el servidor. Un *lease time* muy largo reduce ese tráfico de renovación, pero hace que un ataque de Starvation tenga un efecto mucho más duradero, porque las IPs quedan "ocupadas" durante más tiempo aunque el atacante ya no esté. El administrador tiene que buscar un punto intermedio según el tipo de red: por ejemplo, tiempos más cortos en redes con muchos dispositivos que van y vienen (invitados, móviles) y algo más largos en redes con equipos fijos.

**Detección de conflictos de IP (`ping check` / `conflict-detection`):** antes de entregar una IP a un cliente, el servidor DHCP puede hacer una comprobación (por ejemplo, un ping) a esa dirección para asegurarse de que nadie más la está usando ya en la red. Si recibe respuesta, significa que esa IP ya está ocupada por otro equipo aunque el servidor la tuviera marcada como libre (por ejemplo, porque alguien la configuró a mano sin avisar, o porque el pool está en un estado inconsistente tras un ataque). En ese caso, el servidor marca esa IP como conflictiva y no la ofrece, evitando así asignar direcciones duplicadas aunque la base de datos del pool no esté del todo fiable.

---

## 4. Consideraciones del entorno virtual y seguridad

### 4.1 El modo promiscuo en VirtualBox

**Qué es el modo promiscuo:** una interfaz de red normalmente solo procesa las tramas Ethernet que van dirigidas a su propia dirección MAC (o a broadcast/multicast), y descarta el resto. En **modo promiscuo**, la interfaz acepta y procesa **todas** las tramas que le llegan, vayan dirigidas a ella o no. Los hipervisores como VirtualBox lo tienen desactivado o restringido por defecto porque es justo lo que permite capturar tráfico ajeno (con Wireshark, por ejemplo) o, como en este caso, enviar y recibir tráfico con direcciones MAC que no son la real de la máquina virtual; sin esa restricción, cualquier máquina virtual podría espiar el tráfico de las demás sin que el administrador lo autorizara explícitamente.

**Si Kali no estuviera en modo promiscuo:**
- **Para enviar** paquetes con MACs de origen inventadas, Scapy sí podría construir y enviar la trama igualmente, porque construir y enviar un paquete con una MAC falsa no depende de que la propia interfaz esté en modo promiscuo: ese modo afecta a qué tramas se **aceptan**, no a lo que se puede **mandar**.
- **Para recibir** las respuestas DHCPOFFER que el servidor envía dirigidas a esas MACs falsas, sí sería un problema: sin modo promiscuo, el adaptador de red de Kali descartaría esas tramas porque no van dirigidas a la MAC real de su interfaz, y Scapy nunca llegaría a verlas. Por eso, aunque el ataque de "mandar peticiones" podría funcionar a medias, para completar el DORA y comprobar realmente que se agotan las IPs hace falta el modo promiscuo, que permite a la interfaz recoger también las tramas destinadas a esas MACs inventadas.

### 4.2 Aspectos éticos y legales

**Condiciones legales para realizar estas pruebas:** un ataque de DHCP Starvation solo está permitido cuando se hace sobre una red **propia** o sobre una red de un tercero con **autorización expresa y por escrito** del responsable de esa red (por ejemplo, un contrato de auditoría de seguridad o *pentest* autorizado, con el alcance y las fechas bien definidos). Hacerlo en nuestro laboratorio de VirtualBox está permitido porque es una red interna propia, aislada y creada específicamente para practicar; en una red de producción ajena, sin ese permiso explícito, sería ilegal aunque la intención fuera solo "probar".

**Consecuencias legales en el Código Penal español:** el Código Penal recoge este tipo de conductas principalmente en los artículos sobre daños informáticos:

- El **artículo 264 bis** castiga específicamente a quien, sin autorización y de forma grave, **obstaculice o interrumpa el funcionamiento de un sistema informático ajeno** (lo que encaja directamente con un ataque de denegación de servicio como el Starvation), con pena de **prisión de seis meses a tres años**. Si los hechos perjudican de forma relevante a una empresa, negocio o administración pública, la pena se impone en su mitad superior, pudiendo llegar a la pena superior en grado.
- Si además concurren agravantes (actuar dentro de una organización criminal, afectar a un número elevado de sistemas, perjudicar servicios públicos esenciales o infraestructuras críticas), la pena sube a **prisión de tres a ocho años y multa del triple al décuplo del perjuicio ocasionado** (artículo 264 bis, apartado 2, en relación con el artículo 264.2).
- El **artículo 264 ter** castiga, además, a quien fabrique, consiga o facilite a terceros herramientas (programas o contraseñas) pensadas principalmente para cometer estos delitos, con pena de **prisión de seis meses a dos años o multa de tres a dieciocho meses**.

En resumen, usar herramientas como Yersinia o un script de Scapy para agotar el pool DHCP de una red ajena, sin autorización, puede constituir un delito de daños informáticos / denegación de servicio con penas de prisión de hasta varios años, dependiendo de la gravedad y del tipo de red afectada.
