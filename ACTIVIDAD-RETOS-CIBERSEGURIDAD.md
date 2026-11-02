# Actividad individual: retos de mínimo privilegio en Linux (retos.sh)

Laboratorio de hacking ético inicial en Kali Linux, basado en la guía *"7 retos de mínimo privilegio"* de Planeta FP. En cada reto, el script `retos.sh` configura mal a propósito una característica del sistema. Como `becario` (usuario sin privilegios) reproduzco el ataque, y luego, como administrador, lo corrijo y compruebo que la corrección funciona sin romper el uso legítimo.

## Ideas clave antes de empezar

- **UID efectivo:** Linux decide qué puede hacer un proceso mirando el UID con el que se ejecuta. El UID `0` es el de `root`, que no tiene restricciones.
- **Escalada de privilegios:** pasar de un usuario normal a root aprovechando una mala configuración. Es lo que se intenta en cada uno de estos retos.
- **Mínimo privilegio:** cada usuario, programa o servicio debe tener solo los permisos que necesita para su trabajo, ni más ni menos.

## Preparación del laboratorio

1. Copié el script `retos.sh` a mi máquina Kali Linux (VMware).
2. Hice una instantánea (snapshot) de la VM antes de empezar, por si hay que volver atrás.
3. Le di permiso de ejecución: `chmod +x retos.sh`.
4. Cada reto se prepara con `sudo bash retos.sh preparar N` (N = 1 a 7).

---

## Reto 1 · Permisos de ficheros (rwx)

**La característica:** cada fichero de Linux tiene permisos para tres grupos: el dueño, el grupo y el resto de usuarios (otros). Cada uno puede leer, escribir o ejecutar.

**El fallo:** el fichero de nóminas (`nominas.csv`) tenía permisos `666`, es decir, cualquier usuario de la máquina podía leerlo y modificarlo, aunque no fuera de RR. HH.

### 1. Evidencia de la preparación

`ls -l /srv/empresa/nominas.csv` (mala configuración), y `cat` + `echo ... >>` como becario (el ataque funciona).

**Captura:**

_(pega aquí la captura)_

### 2. Instrucciones que resuelven el reto

```bash
chmod 660 /srv/empresa/nominas.csv
```
`chmod 660` deja lectura y escritura solo para el dueño (`root`) y el grupo (`rrhh`), y quita todos los permisos a "otros" (`becario` no está en ese grupo).

**Captura:**

_(pega aquí la captura)_

### 3. Prueba que confirma la solución

**a) El ataque ya no funciona** (como becario, `cat` y `echo >>` sobre el fichero):

**Captura:**

_(pega aquí la captura)_

**b) El uso legítimo sigue funcionando** (`sudo -u ana_rrhh cat /srv/empresa/nominas.csv`):

**Captura:**

_(pega aquí la captura)_

**Lección:** un permiso demasiado abierto rompe a la vez la confidencialidad y la integridad de los datos; hay que dar acceso solo al grupo que de verdad lo necesita.

---

## Reto 2 · El grupo sudo

**La característica:** cualquier usuario del grupo `sudo` puede ejecutar cualquier comando como root, con solo su propia contraseña.

**El fallo:** el usuario `becario` estaba metido en el grupo `sudo` "para que ayude con todo".

### 1. Evidencia de la preparación

`id` (mala configuración), y `sudo -i` + `whoami` como becario (el ataque funciona).

**Captura:**

_(pega aquí la captura)_

### 2. Instrucciones que resuelven el reto

```bash
gpasswd -d becario sudo
```
Saca a `becario` del grupo `sudo`, quitándole el permiso de administración total que no necesita para su trabajo.

**Captura:**

_(pega aquí la captura)_

### 3. Prueba que confirma la solución

**a) El ataque ya no funciona** (`id` sin el grupo sudo, y `sudo -i` fallando):

**Captura:**

_(pega aquí la captura)_

**b) El uso legítimo sigue funcionando** (`becario` sigue pudiendo iniciar sesión y trabajar, por ejemplo `whoami`):

**Captura:**

_(pega aquí la captura)_

**Lección:** dar acceso al grupo `sudo` da el control total del sistema, no solo "una ayuda puntual"; si roban esa contraseña, roban la máquina entera.

---

## Reto 3 · sudo con find

**La característica:** en `/etc/sudoers.d/` se pueden dar permisos de root limitados a un solo comando, en vez de a todos.

**El fallo:** se le dio permiso a `becario` para ejecutar `find` como root sin contraseña, pensando que era inofensivo.

### 1. Evidencia de la preparación

`sudo -l` (mala configuración), y `sudo find /tmp -maxdepth 0 -exec /bin/bash \;` + `whoami` (el ataque abre una shell de root).

**Captura:**

_(pega aquí la captura)_

### 2. Instrucciones que resuelven el reto

```bash
visudo -f /etc/sudoers.d/reto3
```
Dejando la regla como:
```
becario ALL=(root) NOPASSWD: /usr/local/sbin/limpiar_tmp.sh
```
Limita el permiso de root al único script que `becario` necesita, en vez de a un comando genérico como `find` que permite escapar a una shell.

**Captura:**

_(pega aquí la captura)_

### 3. Prueba que confirma la solución

**a) El ataque ya no funciona** (`sudo find ... -exec /bin/bash` fallando):

**Captura:**

_(pega aquí la captura)_

**b) El uso legítimo sigue funcionando** (`sudo /usr/local/sbin/limpiar_tmp.sh`):

**Captura:**

_(pega aquí la captura)_

**Lección:** dar permiso de root sobre un programa "inofensivo" es arriesgado, porque muchos comandos del sistema permiten escapar a una shell (ver GTFOBins); lo seguro es dar permiso solo sobre el script exacto necesario.

---

## Reto 4 · El bit SUID

**La característica:** un ejecutable con el bit SUID activado se ejecuta con los permisos de su dueño, no de quien lo lanza.

**El fallo:** una copia de `cat` (`verlog`) tenía el bit SUID de root, "para que todos pudieran ver los logs".

### 1. Evidencia de la preparación

`find / -perm -4000 -type f` + `ls -l /usr/local/bin/verlog` (mala configuración), y `verlog /etc/shadow` (el ataque funciona).

**Captura:**

_(pega aquí la captura)_

### 2. Instrucciones que resuelven el reto

```bash
chmod u-s /usr/local/bin/verlog
```
Quita el bit SUID, para que `verlog` se ejecute con los permisos de quien lo lanza, no con los de root.

**Captura:**

_(pega aquí la captura)_

### 3. Prueba que confirma la solución

**a) El ataque ya no funciona** (`verlog /etc/shadow` fallando):

**Captura:**

_(pega aquí la captura)_

**b) El uso legítimo sigue funcionando** (`verlog /etc/hostname`):

**Captura:**

_(pega aquí la captura)_

**Lección:** una copia de un programa del sistema con SUID es tan peligrosa como el original, porque conserva su función pero con privilegios de root; el bit SUID solo debe estar en los binarios imprescindibles.

---

## Reto 5 · Capabilities

**La característica:** Linux reparte el poder de root en piezas sueltas llamadas capabilities, que se pueden dar a un binario sin hacerlo SUID completo.

**El fallo:** una copia de `python3` tenía la capability `cap_setuid`, "para un script de mantenimiento".

### 1. Evidencia de la preparación

`getcap -r /` (mala configuración), y `pyherramienta -c 'import os; os.setuid(0); os.system("id")'` (el ataque funciona).

**Captura:**

_(pega aquí la captura)_

### 2. Instrucciones que resuelven el reto

```bash
setcap -r /usr/local/bin/pyherramienta
```
Retira todas las capabilities del binario, dejándolo sin ese poder especial.

**Captura:**

_(pega aquí la captura)_

### 3. Prueba que confirma la solución

**a) El ataque ya no funciona** (el mismo comando de `setuid` fallando):

**Captura:**

_(pega aquí la captura)_

**b) El uso legítimo sigue funcionando** (`pyherramienta -c 'print("hola mundo")'`):

**Captura:**

_(pega aquí la captura)_

**Lección:** algunas capabilities, como `cap_setuid`, equivalen en la práctica a dar root entero, así que hay que tratarlas con el mismo cuidado que el bit SUID.

---

## Reto 6 · Tareas programadas (cron)

**La característica:** cron ejecuta tareas con el usuario indicado en `/etc/cron.d/`; si es root, la tarea corre con todos sus privilegios.

**El fallo:** un script que cron ejecutaba cada minuto como root tenía permisos `777`, editable por cualquiera.

### 1. Evidencia de la preparación

`cat /etc/cron.d/reto6` + `ls -l /opt/tareas/limpieza.sh` (mala configuración), y el `echo >>` seguido de `cat /tmp/quien_soy.txt` tras esperar un minuto (el ataque funciona).

**Captura:**

_(pega aquí la captura)_

### 2. Instrucciones que resuelven el reto

```bash
chown root:root /opt/tareas/limpieza.sh
chmod 755 /opt/tareas/limpieza.sh
```
Y quitar del script la línea que añadió el atacante. Cambia el dueño a `root` y los permisos para que solo root pueda escribir en él.

**Captura:**

_(pega aquí la captura)_

### 3. Prueba que confirma la solución

**a) El ataque ya no funciona** (intento de escribir en el script como becario, fallando):

**Captura:**

_(pega aquí la captura)_

**b) El uso legítimo sigue funcionando** (`touch /tmp/prueba.old`, esperar un minuto, y comprobar que `ls /tmp/prueba.old` ya no lo encuentra):

**Captura:**

_(pega aquí la captura)_

**Lección:** no basta con que cron ejecute algo como root; el script y la carpeta donde vive tienen que estar protegidos contra escritura de cualquier otro usuario.

---

## Reto 7 · Servicios ejecutados como root

**La característica:** cada servicio de systemd se ejecuta con el usuario indicado en `User=`; si es root y el servicio falla, el atacante obtiene privilegios de root.

**El fallo:** un servicio web corría como root y publicaba todo el disco (`--directory /`).

### 1. Evidencia de la preparación

Descarga de `/etc/shadow` desde el navegador sin contraseña, y `ps -o user,args -C python3` mostrando que el proceso corre como root (mala configuración y ataque funcionando).

**Captura:**

_(pega aquí la captura)_

### 2. Instrucciones que resuelven el reto

```bash
useradd -r -s /usr/sbin/nologin intranet
```
Y en `/etc/systemd/system/intranet.service`, cambiar `ExecStart` para que use `--directory /srv/web` y `User=intranet`. Después:
```bash
systemctl daemon-reload
systemctl restart intranet
```
Se crea un usuario sin privilegios para el servicio y se limita la carpeta publicada; los dos cambios son necesarios a la vez.

**Captura:**

_(pega aquí la captura)_

### 3. Prueba que confirma la solución

**a) El ataque ya no funciona** (`curl -I http://127.0.0.1:8080/etc/shadow` fallando, y `ps` mostrando el usuario `intranet`):

**Captura:**

_(pega aquí la captura)_

**b) El uso legítimo sigue funcionando** (`curl http://127.0.0.1:8080/` devolviendo el HTML de la intranet):

**Captura:**

_(pega aquí la captura)_

**Lección:** un servicio debe ejecutarse con el usuario mínimo necesario y dar acceso solo a la carpeta que realmente debe publicar; mezclar privilegios de root con una exposición abierta de todo el disco convierte un fallo pequeño en un control total del sistema.

---

## Resumen

| Reto | Característica | Lo que conseguía el atacante |
| --- | --- | --- |
| 1 | Permisos rwx | Leer y alterar datos personales |
| 2 | Grupo sudo | Root con su propia contraseña |
| 3 | Reglas de sudoers | Shell de root |
| 4 | Bit SUID | Leer `/etc/shadow` y cualquier fichero |
| 5 | Capabilities | Convertirse en root |
| 6 | cron | Ejecutar comandos como root cada minuto |
| 7 | Usuario del servicio | Descargar `/etc/shadow` sin contraseña |

Seis de los siete retos acaban dando privilegios de root al atacante. El séptimo acaba en la fuga de datos personales. Esto es la **escalada de privilegios**: pasar de un usuario sin apenas permisos a tener control total (o casi) del sistema, aprovechando una configuración demasiado permisiva.

## Fuentes

- GTFOBins (abuso de binarios: `find`, `less`, `vim`...): gtfobins.org
- Manual de sudoers: sudo.ws/docs/man/sudoers.man
- capabilities(7): man7.org/linux/man-pages/man7/capabilities.7.html
- Python `http.server`, advertencias de seguridad: docs.python.org/3/library/http.server.html
- MITRE ATT&CK (T1548, T1053.003, T1565.001): attack.mitre.org
- `systemd.exec` (`User=`, `NoNewPrivileges=`, `ProtectSystem=`): `man systemd.exec`
