# Confinamiento de Nginx con AppArmor

## 1. Objetivo

En esta práctica voy a instalar y comprobar Nginx en Linux, crear un perfil de seguridad con AppArmor, observar su comportamiento en modo **complain** (aprendizaje), ajustar los permisos necesarios y activarlo en modo **enforce** para comprobar que el servidor web sigue funcionando.

## 2. Entorno utilizado

- Sistema operativo: **[indicar distribución y versión]**
- Máquina: **[virtual o física]**
- Servidor web: **Nginx**
- Herramienta de seguridad: **AppArmor**
- Dirección IP utilizada: **[indicar IP]**

Comandos para consultar el entorno:

```bash
cat /etc/os-release
ip -br a
nginx -v
sudo aa-status
```

**Captura 1 — Sistema y entorno:** [añadir captura donde se vean los datos del sistema y/o el estado de AppArmor]

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

**Captura 2 — Nginx funcionando:** [añadir captura del estado `active (running)` y de la respuesta HTTP obtenida con `curl`]

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

**Captura 3 — Generación del perfil:** [añadir captura de `aa-genprof` con el análisis de accesos o las propuestas de permisos]

## 5. Modo complain y revisión de los registros

El modo **complain** permite que el programa siga funcionando mientras AppArmor registra las acciones que el perfil no autorizaría. Es útil para detectar los accesos necesarios sin bloquear el servidor durante las pruebas.

```bash
sudo aa-complain /usr/sbin/nginx
sudo aa-status
curl -I http://127.0.0.1
sudo journalctl -k --no-pager | grep -i apparmor | tail -n 30
```

En los registros busqué entradas relacionadas con `nginx` y `apparmor`, especialmente eventos con `apparmor="ALLOWED"` en modo complain. Si no aparecían eventos, revisé los registros después de hacer más peticiones: **no se deben inventar registros**.

**Captura 4 — Modo complain:** [añadir captura de `aa-status` indicando el perfil de Nginx en complain]

**Captura 5 — Registros reales:** [añadir captura de las líneas de AppArmor encontradas en `journalctl` o, si no aparecen, del resultado de la comprobación y explicar la ausencia de eventos]

**Eventos observados:** [pegar aquí 2-5 líneas reales de registro o explicar que no se detectaron nuevos eventos]

```text
[PEGAR AQUÍ LOS REGISTROS REALES]
```

## 6. Ajuste y perfil resultante

Después de probar Nginx, revisé los eventos con `aa-logprof` para incluir en el perfil únicamente los permisos necesarios.

```bash
sudo aa-logprof
sudo cat /etc/apparmor.d/usr.sbin.nginx
```

**Captura 6 — Perfil ajustado:** [añadir captura de `aa-logprof` y/o del contenido del perfil]

### Perfil final obtenido

> Sustituir el siguiente bloque por el **contenido real** del archivo `/etc/apparmor.d/usr.sbin.nginx` después de ajustarlo. No se incluye un perfil inventado porque las reglas dependen de los registros y del sistema utilizado.

```apparmor
[PEGAR AQUÍ EL PERFIL REAL GENERADO Y AJUSTADO]
```

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

**Captura 7 — Perfil en enforce:** [añadir captura de `aa-status` en la que se vea Nginx en enforce]

**Captura 8 — Funcionamiento tras el confinamiento:** [añadir captura con Nginx activo y respuesta HTTP correcta tras el reinicio]

Si el reinicio falla, debo revisar los eventos de AppArmor, ajustar el perfil con los permisos estrictamente necesarios y repetir la prueba.

```bash
sudo journalctl -u nginx -n 30 --no-pager
sudo journalctl -k --no-pager | grep -i apparmor | tail -n 30
```

## 8. Diferencia entre complain y enforce

| Modo | Qué hace | Cuándo interesa |
|---|---|---|
| **Complain** | Registra accesos que no encajan con el perfil, pero normalmente no los bloquea | Mientras creamos y ajustamos el perfil |
| **Enforce** | Hace cumplir las reglas y bloquea accesos que no estén permitidos | Cuando el perfil ya está preparado y probado |

## 9. Resultado y reflexión

**Resultado final:** [indicar si el perfil quedó en enforce y si Nginx respondió correctamente, con el código HTTP que apareció]

**Reflexión:** He comprobado que Nginx necesita acceder a sus archivos de configuración, a los archivos que sirve y a algunos recursos del sistema para poder trabajar. La parte más importante ha sido permitir solo los accesos necesarios, en lugar de dar permisos a todo. El modo complain me ha servido para identificar los accesos durante las pruebas y el modo enforce para comprobar que el servidor puede funcionar con restricciones.

**Nota:** Las capturas, los eventos y el contenido del perfil deben añadirse a partir de la ejecución real de la práctica. Este documento es una plantilla de entrega, no una certificación de ejecución.
