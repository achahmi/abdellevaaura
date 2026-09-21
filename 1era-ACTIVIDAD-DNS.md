# abdellevaaura
Apuntes de abdellah chahmi

---

# 1. El ecosistema DNS (OSINT y Web)

## 1.1 Investigación de jerarquía

### ¿Qué organismo coordina y asigna los parámetros a nivel global del sistema de nombres de dominio e IPs?

La **ICANN** (Corporación para la Asignación de Nombres y Números en Internet). Es una organización sin ánimo de lucro que se encarga de que Internet funcione de forma ordenada en todo el mundo. Entre otras cosas, coordina los nombres de dominio y las direcciones IP para que no haya dos iguales. Dentro de la ICANN está la función **IANA**, que es la que reparte los bloques de direcciones IP y mantiene la lista oficial de las terminaciones de dominio (.com, .es, .cat, etc.).

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

- Quién es el **titular** (el dueño), aunque muchas veces sale oculto por temas de privacidad.
- Qué **registrador** lo tiene gestionado.
- Las fechas importantes: cuándo se **creó**, cuándo **caduca** y cuándo se modificó por última vez.
- Los **servidores DNS** que usa el dominio.
- El **estado** del dominio (activo, bloqueado, caducado…) y algunos datos de contacto.

### Diferencia entre Registry y Registrar

- El **Registry** (registro) es el que manda sobre una terminación entera y guarda la base de datos oficial de todos sus dominios. Por ejemplo, Red.es para los `.es`.
- El **Registrar** (registrador) es la empresa donde tú compras y gestionas tu dominio, como GoDaddy o Dinahosting. Se encarga de hacer los trámites con el Registry por ti.

Dicho de forma sencilla: el Registry es el "mayorista" que guarda la lista oficial, y el Registrar es la "tienda" a la que vas tú a comprar.

### ¿Qué es DNSSEC y qué problema de seguridad intenta resolver?

Por defecto, el DNS funciona con confianza: cuando pregunto por una web, me creo la respuesta sin comprobar que sea de verdad. Eso permite que un atacante pueda **falsificar la respuesta** y llevarme a una página falsa (por ejemplo, una que imita a mi banco) sin que yo me entere.

**DNSSEC** soluciona esto añadiendo **firmas digitales** a las respuestas DNS. Así, mi equipo puede comprobar que la respuesta viene realmente del dueño del dominio y que nadie la ha cambiado por el camino. Ojo: DNSSEC no cifra nada, solo sirve para comprobar que la respuesta es auténtica.

## 1.3 Rendimiento DNS (GRC DNS Benchmark)

Esta prueba mide cuánto tarda cada servidor DNS en responder desde mi conexión. Cuanto menos tarda, más rápido es. Pasos que seguí:

1. Descargué la aplicación desde la web de GRC (no necesita instalación) y la abrí.
2. Ejecuté el test (*Run Benchmark*) y esperé a que terminara.
3. Miré la lista ordenada por velocidad y me quedé con los 3 primeros.

**Captura del resultado:**

_(pega aquí la captura del benchmark)_

**Los 3 servidores DNS más rápidos:**

| Puesto | IP | Empresa |
| --- | --- | --- |
| 1 | _(pon aquí la IP)_ | _(qué empresa es y a qué se dedica)_ |
| 2 | _(pon aquí la IP)_ | _(qué empresa es y a qué se dedica)_ |
| 3 | _(pon aquí la IP)_ | _(qué empresa es y a qué se dedica)_ |

Estos son los servidores que usaré en la siguiente fase.
