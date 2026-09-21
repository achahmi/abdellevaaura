# abdellevaaura
Apuntes de abdellah chahmi

---

# 1. El ecosistema DNS (OSINT y Web)

## 1.1 Investigación de jerarquía

### ¿Qué organismo coordina y asigna los parámetros a nivel global del sistema de nombres de dominio e IPs?

La **ICANN** (Corporación para la Asignación de Nombres y Números en Internet). Es una organización que no busca dinero la se encarga de que Internet funcione de forma ordenada en todo el mundo. Entre otras cosas, coordina los nombres de dominio y las direcciones IP para que no haya dos iguales. Dentro de la ICANN está la función **IANA**, que es la que reparte los bloques de direcciones IP y mantiene la lista oficial de los dominios como .es .cat .edu .ifp.es
### ¿Qué empresa u organismo gestiona (Registry) cada dominio de nivel superior?

| Dominio | Quién lo gestiona (Registry) |
| --- | --- |
| **.es** | **Red.es**, una entidad pública del Gobierno de España. Lo gestiona a través de su web Dominios.es (antes conocida como ESNIC). |
| **.cat** | **Fundació puntCAT**, que ahora funciona bajo el nombre **Accent Obert**. Es el dominio pensado para la lengua y la cultura catalanas. |
| **.edu** | **EDUCAUSE**, una asociación estadounidense de tecnología para la educación superior. Trabaja con **Verisign** para la parte técnica. Solo pueden tenerlo universidades y centros de EE. UU. |
| **.ifp.es** | Aquí hay un detalle: `ifp.es` **no es un dominio de nivel superior**, es un dominio normal que está dentro de `.es`. Por eso el registry sigue siendo **Red.es**. Quien lo tiene registrado y lo administra es su propietario, **iFP** (Innovación en Formación Profesional, un centro de FP que ahora se llama Planeta FP). |

## 1.2 Herramientas OSINT (Whois y DNS Lookup)

### ¿Qué información da una consulta Whois sobre un dominio?

Es como la "ficha" del dominio. Normalmente se puede ver:

- Quién es el dueño, aunque muchas veces sale oculto por temas de privacidad.
- Qué **registrador** lo tiene gestionado.
- Las fechas importantes: cuándo se **creó**, cuándo **caduca** y cuándo se modificó por última vez.
- Los **servidores DNS** que usa el dominio.
- El **estado** del dominio (activo, bloqueado, caducado…) y algunos datos de contacto.

### Diferencia entre Registry y Registrar

- El **Registry** (registro) es el que manda sobre una terminación entera y guarda la base de datos oficial de todos sus dominios. Por ejemplo, Red.es para los `.es`.
- El **Registrar** (registrador) es la empresa donde tú compras y gestionas tu dominio, como GoDaddy o Dinahosting. Se encarga de hacer los trámites con el Registry por ti.

Dicho de forma sencilla: el Registry es el "mayorista" que guarda la lista oficial, y el Registrar es la "tienda" a la que vas tú a comprar.

### ¿Qué es DNSSEC y qué problema de seguridad intenta resolver?

De normal, el DNS funciona con confianza: cuando pregunto por una web, me creo la respuesta sin comprobar que sea de verdad. Eso permite que un atacante pueda **falsificar la respuesta** y llevarme a una página falsa (por ejemplo, una que imita a mi banco) sin que yo me entere.

**DNSSEC** soluciona esto añadiendo **firmas digitales** a las respuestas DNS. Así, mi equipo puede comprobar que la respuesta viene realmente del dueño del dominio y que nadie la ha cambiado por el camino. Ojo: DNSSEC no cifra nada, solo sirve para comprobar que la respuesta es auténtica.

## 1.3 Rendimiento DNS (GRC DNS Benchmark)

Esta prueba mide cuánto tarda cada servidor DNS en responder desde mi conexión. Cuanto menos tarda, más rápido es. Pasos que seguí:

1. Descargué la aplicación desde la web de GRC (no necesita instalación) y la abrí.
2. Ejecuté el test (*Run Benchmark*) y esperé a que terminara.
3. Miré la lista ordenada por velocidad y me quedé con los 3 primeros.

**Captura del resultado:**
<img width="581" height="461" alt="image" src="https://github.com/user-attachments/assets/d8885e5f-51a9-4ac5-a83b-b523de12912b" />

**Los 3 servidores DNS más rápidos:**

| Puesto | IP | Empresa |
| --- | --- | --- |
| 1 | _(4.2.2.3)_ | _(Level 3 Parent, LLC - Louisiana, se dedica a la prestación de servicios de telecomunicaciones por línea fija (wireline), redes basadas en IP, fibra óptica y soluciones de conectividad empresarial)_ |
| 2 | _(1.0.0.1)_ | _(Cloudflare, se dedica a ofrecer infraestructura, seguridad y optimización de rendimiento para sitios web, aplicaciones y redes en Internet)_ |
| 3 | _(1.1.1.1)_ | _(Cloudflare, se dedica a ofrecer infraestructura, seguridad y optimización de rendimiento para sitios web, aplicaciones y redes en Internet)_ |

Estos son los servidores que usaré en la siguiente fase.

---

# 2. Configuración y Caché

## 2.1 Cambio de servidores DNS

### ¿Cómo ver por consola qué servidores DNS tengo asignados?

- **Windows:** con `ipconfig /all` y buscando la línea *Servidores DNS*. También sirve `Get-DnsClientServerAddress` en PowerShell.

**Captura:** 
<img width="1000" height="468" alt="image" src="https://github.com/user-attachments/assets/419dd675-ff7e-4bae-857e-f8d70d76d31c" />


### Cambiar los DNS de mi equipo por los del Benchmark

Puse como DNS primario y secundario los dos primeros servidores que salieron en el benchmark de la fase 1.
<img width="530" height="700" alt="image" src="https://github.com/user-attachments/assets/2a52a1c8-e097-4849-b916-7b1bf289fe6e" />
<img width="835" height="752" alt="image" src="https://github.com/user-attachments/assets/6df857ed-33f0-4c0f-bd79-15dd33835155" />

### ¿Dónde se pueden forzar unos DNS en el móvil para una red Wi-Fi?

- **iPhone (iOS):** Ajustes → Wi-Fi → pulsar la "i" de la red → **Configurar DNS** → cambiar de Automático a **Manual** → añadir los servidores.

## 2.2 Gestión de la caché DNS

La caché DNS es una "libreta" donde el equipo apunta las direcciones que ya ha consultado, para no tener que preguntar otra vez cada vez que entra en la misma web.

### Ver la caché

- **Windows:** `ipconfig /displaydns`

**Captura:**

<img width="591" height="981" alt="image" src="https://github.com/user-attachments/assets/ca9f0e10-518a-4cf2-b139-b2d669d70a19" />
<img width="674" height="930" alt="image" src="https://github.com/user-attachments/assets/6fa27e13-c333-4314-b32f-ee5e2349814a" />



### Vaciar la caché

- **Windows:** `ipconfig /flushdns`

**Captura:**

<img width="508" height="135" alt="image" src="https://github.com/user-attachments/assets/8f897d2f-384a-46c5-b9e8-62b3c51154ad" />


**¿Para qué sirve en el día a día de un administrador?**

Sirve para asegurarse de que el equipo pregunta de nuevo y no usa datos antiguos. Por ejemplo, si cambiamos una web de servidor y la IP nueva no se ve porque el ordenador sigue recordando la antigua, vaciamos la caché y así comprobamos si el cambio funciona. También ayuda cuando una web no carga o lleva a un sitio equivocado por un dato viejo o erróneo guardado, y es un paso rápido a probar antes de buscar otros problemas.

---

# 3. Administración - Troubleshooting con DIG y CLI

`dig` es la herramienta que se usa en Linux para hacer consultas DNS y ver qué responde cada servidor. He usado el dominio **aliexpress.com**.

## 3.1 Consultas de registros

### Registro A: `dig aliexpress.com`

**Captura:**
<img width="558" height="280" alt="image" src="https://github.com/user-attachments/assets/818847b7-b76a-4cfa-9b63-9baafe34f838" />


En la *ANSWER SECTION* aparece la respuesta a la pregunta. Cada línea tiene el nombre del dominio, el **TTL** (los segundos que se puede guardar la respuesta en la caché), la palabra `IN` (Internet), el tipo `A` y, al final, la **dirección IP** del dominio. Si sale más de una IP, es porque la web se reparte en varios servidores. Además, arriba se ve el estado de la consulta (`NOERROR` significa que todo fue bien) y, abajo, qué servidor DNS respondió y cuánto tardó.

**Lo que me salió a mí:** 
<img width="523" height="81" alt="image" src="https://github.com/user-attachments/assets/09077e0f-8533-4fe1-9aa1-98ce790cda39" />


### Formato corto: `dig +short aliexpress.com`

**Captura:**
<img width="394" height="62" alt="image" src="https://github.com/user-attachments/assets/6febca69-ab25-40b2-8607-b82d5d218161" />


Con `+short` solo aparece la respuesta (la IP), sin nada más. Es útil en los scripts de Bash porque el resultado se puede guardar directamente en una variable o usar en otro comando, sin tener que limpiar todo el texto extra. Por ejemplo: `IP=$(dig +short aliexpress.com)`.

### Registro MX: `dig MX aliexpress.com`

**Captura:**

<img width="613" height="342" alt="image" src="https://github.com/user-attachments/assets/3a6d3a9c-bce5-4ef4-aee5-776084279f1a" />


Los registros MX indican qué servidores reciben el correo del dominio. Junto a cada servidor hay un número, la **prioridad** (*preference*). **Cuanto más bajo es el número, más prioridad tiene**: el correo se intenta entregar primero al servidor con el número más bajo, y los demás quedan como reserva por si ese falla.

**Lo que me salió a mí:** _(anota aquí los servidores y sus prioridades)_

### Registro NS: `dig NS aliexpress.com`

**Captura:**

<img width="649" height="355" alt="image" src="https://github.com/user-attachments/assets/f565fd07-f49d-4aec-9c49-fb456c25eec0" />


Los registros NS muestran los servidores que tienen la **autoridad** sobre el dominio, es decir, los que guardan la información oficial y son los que dan la respuesta definitiva sobre él.

## 3.2 Autoridad y Caché (TTL)

### Diferencia entre SOA y NS

- **NS:** es una lista con los servidores que se encargan de responder por el dominio. Dice *quién* responde.
- **SOA:** es como la "ficha técnica" de la zona del dominio. Indica cuál es el servidor principal, el correo del administrador, un número de versión (*serial*) y varios tiempos que controlan cada cuánto se sincronizan los servidores y cuánto se guardan los datos. Dice *cómo se gestiona* la zona.

### Prueba del TTL

Hice una consulta a un dominio y anoté el TTL. A los 5 segundos volví a hacer la misma consulta.

| | TTL |
| --- | --- |
| Primera consulta | 600 |
| A los 5 segundos | 594 |

**Captura:**
<img width="555" height="121" alt="image" src="https://github.com/user-attachments/assets/89a0ded4-f098-4782-af34-76bb93cbb967" />


El TTL **ha bajado** (unos 5 segundos menos). Eso demuestra que la respuesta no vino directamente del servidor oficial, sino de la **caché** de un servidor intermedio, que va descontando el tiempo que le queda al dato. Si la respuesta viniera del servidor autoritativo, siempre saldría el valor completo del TTL. Cuando el TTL llega a 0, la caché borra el dato y vuelve a preguntar al servidor oficial.

## 3.3 Trazabilidad completa (Trace)

Comando: `dig +trace aliexpress.com`

**Captura:**

<img width="959" height="473" alt="image" src="https://github.com/user-attachments/assets/bec723b0-2950-472b-9b28-7bd96acd6543" />


Con `+trace`, en vez de preguntar a mi servidor DNS de siempre, dig hace todo el recorrido paso a paso, como haría un servidor DNS por dentro:

1. **Servidores raíz (`.`):** empieza preguntando a uno de los servidores raíz. Ellos no saben la IP de aliexpress.com, pero sí saben quién se encarga de los dominios `.com`, y me dan la lista de esos servidores.
2. **Servidores del TLD (`.com`):** pregunta a uno de ellos. Tampoco tienen la IP final, pero saben qué servidores son los oficiales de aliexpress.com, y me los indican.
3. **Servidores autoritativos de aliexpress.com:** pregunta a uno de estos y este ya sí me da la respuesta final: la dirección IP del dominio.

Es como preguntar por una dirección, primero a alguien que sabe el país, luego a alguien que sabe la ciudad y al final a quien conoce la calle.

---

# 4. Análisis de Tráfico de Red (Wireshark)

## Preparación

1. Abrí **Wireshark** y elegí mi tarjeta de red principal (Wi-Fi o Ethernet) para empezar a capturar.
2. Puse el filtro `dns` en la barra de arriba para ver solo el tráfico DNS.
3. En una terminal ejecuté: `nslookup -type=mx google.com`
4. Detuve la captura y busqué la **petición** (Query) y la **respuesta** (Response).

**Captura general (petición y respuesta):**

_(pega aquí la captura de Wireshark con el filtro aplicado)_

## Capa de transporte

Se usa **UDP**. DNS usa UDP por defecto porque las consultas y las respuestas suelen ser muy pequeñas: una pregunta y una respuesta. UDP es más rápido y ligero, porque no tiene que establecer una conexión antes de enviar los datos como hace TCP. Solo se pasa a TCP cuando la respuesta es demasiado grande o en casos como la copia de zonas entre servidores.

**Captura del panel de detalles:**

_(pega aquí la captura donde se vea UDP)_

## Puertos

- **Puerto de origen (mi equipo):** _(anota aquí el número)_. Es un puerto dinámico, un número alto que el sistema elige al azar en cada consulta.
- **Puerto de destino (servidor DNS):** **53**, que es el puerto conocido del DNS.

**Captura:**

_(pega aquí la captura donde se vean los puertos)_

## Identificador (Transaction ID)

El identificador de la transacción es: _(anota aquí el valor, por ejemplo 0x1a2b)_.

Es un número que se pone en la petición y que el servidor copia en la respuesta. Gracias a eso mi equipo sabe qué respuesta corresponde a cada pregunta, sobre todo cuando hay varias consultas a la vez.

**Captura:**

_(pega aquí la captura donde se vea el mismo ID en la petición y en la respuesta)_

## Flags

En el paquete de respuesta, la opción **Authoritative Answer** normalmente está a **0**, y hay que confirmarlo con mi captura: _(anota aquí el valor que te salió)_.

Que esté a 0 significa que la respuesta no la ha dado el servidor oficial de google.com, sino un servidor intermedio (como el de mi router o el DNS que configuré), que la ha sacado de su caché o la ha consultado por mí. Si estuviera a 1, querría decir que quien responde es directamente el servidor autoritativo del dominio.

**Captura:**

_(pega aquí la captura de la sección Flags)_

## Respuestas (Answers)

En el bloque de respuestas aparecen los servidores de correo de google.com. El que tiene la prioridad más alta es el que tiene el **número más bajo** de *preference*. En mi captura es: _(anota aquí el servidor y su preference)_. Lo habitual es que salga `smtp.google.com` con preference 10, pero hay que comprobarlo con lo que salga en la captura.

**Captura:**

_(pega aquí la captura del bloque Answers)_
