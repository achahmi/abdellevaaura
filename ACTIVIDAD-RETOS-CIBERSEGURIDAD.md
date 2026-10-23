# Actividad: Retos Ciberseguridad — Mínimo privilegio en Linux

Laboratorio de hacking ético inicial en Kali Linux, basado en la guía *"7 retos de mínimo privilegio"* de Planeta FP. En cada reto, el script `retos.sh` configura mal a propósito una característica del sistema. Yo, como `becario` (usuario sin privilegios), ataco aprovechando ese fallo, y luego, como administrador, lo arreglo y compruebo que queda bien.

## Ideas clave antes de empezar

- **UID efectivo:** Linux decide qué puede hacer un proceso mirando el UID con el que se ejecuta. El UID `0` es el de `root`, que no tiene restricciones.
- **Escalada de privilegios:** pasar de un usuario normal a root aprovechando una mala configuración. Es lo que se intenta en cada uno de estos retos.
- **Mínimo privilegio:** cada usuario, programa o servicio debe tener solo los permisos que necesita para su trabajo, ni más ni menos.

## Preparación del laboratorio

1. Copié el script `retos.sh` a mi máquina Kali Linux (VMware).
2. Hice una instantánea (snapshot) de la VM antes de empezar, por si hay que volver atrás.
3. Le di permiso de ejecución: `chmod +x retos.sh`.

El ciclo de trabajo que repito en cada reto es: `preparar` el fallo, `atacar` como el usuario `becario` (contraseña `Becario.2026`), `arreglar` el problema como administrador, y `comprobar` que queda bien resuelto con el propio script.

---

## Reto 1 · Permisos de ficheros (rwx)

**La característica:** cada fichero de Linux tiene permisos para tres grupos: el dueño, el grupo y el resto de usuarios (otros). Cada uno puede leer, escribir o ejecutar.

**El fallo:** el fichero de nóminas (`nominas.csv`) tenía permisos `666`, es decir, cualquier usuario de la máquina podía leerlo y modificarlo, aunque no fuera de RR. HH.

**El ataque:** entré como `becario` y comprobé que podía leer el fichero (`cat`) y también añadirle una línea (`echo ... >>`), dándome de alta yo mismo con un IBAN falso.

**La solución:** cambié los permisos para que solo el dueño y el grupo `rrhh` puedan leer y escribir, y quité todos los permisos a "otros". El comando es `chmod 660 nominas.csv`.

**La comprobación:** `sudo bash retos.sh comprobar 1`. Debe confirmar que `becario` ya no puede leer ni escribir, pero que `ana_rrhh` (del grupo RR. HH.) sigue pudiendo.

**Lo aprendido:** un permiso demasiado abierto (`666`) rompe la confidencialidad (cualquiera lee datos personales) y la integridad (cualquiera los modifica). Es tentador poner permisos amplios "para que no haya problemas", pero eso es justo lo contrario del mínimo privilegio.

**Captura de pantalla:**

<img width="549" height="67" alt="image" src="https://github.com/user-attachments/assets/c656ac87-746e-4b6f-824f-fb33abb24470" />
<img width="598" height="303" alt="image" src="https://github.com/user-attachments/assets/8fc2caf5-fd25-4939-a29e-e8a0128dec40" />
<img width="363" height="51" alt="image" src="https://github.com/user-attachments/assets/7007a353-098c-474b-a55f-e8413a3d8ff9" />

**sudo -u ana_rrhh cat /srv/empresa/nominas.csv** aqui la captura:
<img width="432" height="128" alt="image" src="https://github.com/user-attachments/assets/1e77dae8-fc3c-4943-85f3-489d6bf7e7b6" />


---

## Reto 2 · El grupo sudo

**La característica:** cualquier usuario que esté en el grupo `sudo` puede ejecutar cualquier comando como root, con solo su propia contraseña.

**El fallo:** el usuario `becario` estaba metido en el grupo `sudo` "para que ayude con todo".

**El ataque:** con `id` comprobé que `becario` estaba en el grupo `sudo`, y con `sudo -i` entré como root usando la propia contraseña del becario.

**La solución:** saqué a `becario` del grupo `sudo` con `gpasswd -d becario sudo`.

**La comprobación:** `sudo bash retos.sh comprobar 2`. Comprueba que `becario` ya no pertenece a ningún grupo de administración.

**Lo aprendido:** meter a alguien en el grupo `sudo` le da **todo** el poder de root, no solo lo que necesita. Si le roban la contraseña (por ejemplo con phishing), el atacante tiene control total de la máquina. Por eso, si `becario` tuviera una sesión ya abierta con privilegios de sudo activos, quitarlo del grupo no cerraría esa sesión al instante: habría que forzar el cierre de su sesión o esperar a que sudo pida la contraseña de nuevo.

**Captura de pantalla:**

<img width="585" height="63" alt="image" src="https://github.com/user-attachments/assets/2924f95b-88ce-4c28-a069-447016138751" />
<img width="322" height="112" alt="image" src="https://github.com/user-attachments/assets/64af3cbb-22c7-40db-a790-b6331f4258e4" />
<img width="344" height="101" alt="image" src="https://github.com/user-attachments/assets/c8a127ad-36d1-4f44-800c-941631c3128c" />
<img width="536" height="96" alt="image" src="https://github.com/user-attachments/assets/432ad5e4-37f8-429c-84c6-46cd710daefd" />
<img width="376" height="112" alt="image" src="https://github.com/user-attachments/assets/e020640f-cc84-4680-a7a7-ef2185a031a0" />
<img width="614" height="336" alt="image" src="https://github.com/user-attachments/assets/5cf7656c-322c-40fb-9298-2debc3c36bbf" />


---

## Reto 3 · sudo para un programa "inofensivo"

**La característica:** en `/etc/sudoers.d/` se pueden dar permisos de root limitados a un solo comando, en vez de a todos.

**El fallo:** se le dio permiso a `becario` para ejecutar `find` como root sin contraseña, pensando que era inofensivo ("solo para que busque ficheros").

**El ataque:** con `sudo -l` vi que podía ejecutar `find` como root. Muchos comandos aparentemente inofensivos, como `find`, tienen una opción para lanzar otro programa (`-exec`), y así pude abrir una shell de root: `sudo find /tmp -maxdepth 0 -exec /bin/bash \;`.

**La solución:** cambié la regla de sudoers para que `becario` solo pueda ejecutar el script concreto que necesita (`/usr/local/sbin/limpiar_tmp.sh`), y no el comando `find` entero. Se edita con `visudo -f /etc/sudoers.d/reto3` para evitar errores de sintaxis.

**La comprobación:** `sudo bash retos.sh comprobar 3`. Comprueba que ya no puede ejecutar un comando que abra una shell, pero que sí puede seguir ejecutando el script que necesita para su trabajo.

**Lo aprendido:** dar permiso de root sobre un programa genérico (`find`, `less`, `vim`, `awk`...) casi nunca es seguro, porque muchos de ellos permiten escapar a una shell. La web **GTFOBins** (gtfobins.org) recopila estos casos. Lo seguro es dar permiso solo sobre el script exacto que la persona necesita ejecutar.

**Captura de pantalla:**
<img width="580" height="170" alt="image" src="https://github.com/user-attachments/assets/d786ce77-3ac7-459d-b150-01e713ac37f8" />
<img width="461" height="91" alt="image" src="https://github.com/user-attachments/assets/3411437a-c02c-4e81-86c2-7849bc8d6543" />
<img width="345" height="48" alt="image" src="https://github.com/user-attachments/assets/14241cdc-5586-48d0-98a0-6059c37b47e8" />
<img width="499" height="104" alt="image" src="https://github.com/user-attachments/assets/3dbe46c0-8b45-445f-a15c-bb7e3098d4f7" />
**El ataque ya no funciona:**
<img width="630" height="112" alt="image" src="https://github.com/user-attachments/assets/ed22aa3a-77ef-40f1-8f32-0b3d0cb73b3a" />
<img width="405" height="106" alt="image" src="https://github.com/user-attachments/assets/78cbb7e8-33a0-4114-9a76-1b43a8edc9c1" />


---

## Reto 4 · El bit SUID

**La característica:** un ejecutable con el bit SUID activado (la `s` en `-rwsr-xr-x`) se ejecuta con los permisos de su **dueño**, no de quien lo lanza. Por ejemplo, `passwd` lo necesita para poder cambiar `/etc/shadow`, que es un archivo solo de root.

**El fallo:** se hizo una copia de `cat` (llamada `verlog`) y se le puso el bit SUID de root, "para que todos pudieran ver los logs".

**El ataque:** busqué los binarios con SUID activado (`find / -perm -4000 -type f`) y encontré `verlog`, que no es un programa del sistema. Al ejecutar `verlog /etc/shadow`, pude leer un archivo que normalmente solo puede leer root.

**La solución:** quité el bit SUID con `chmod u-s /usr/local/bin/verlog` (o directamente borré el binario, que es lo más seguro si no se necesita).

**La comprobación:** `sudo bash retos.sh comprobar 4`. Comprueba que no quedan binarios con SUID fuera de las carpetas propias del sistema.

**Lo aprendido:** una copia de un programa del sistema con SUID es tan peligrosa como el original, porque sigue haciendo lo mismo pero con permisos de root. Por eso una copia de `bash` con SUID no daría root "directamente" de forma fiable: `bash` (y muchos programas modernos) detecta si el UID real y el UID efectivo no coinciden y, por seguridad, puede ignorar el SUID; en cambio `cat`, al ser un programa simple, sí respeta el SUID y por eso funciona el ataque.

**Captura de pantalla:**

<img width="414" height="131" alt="image" src="https://github.com/user-attachments/assets/3db3ed66-c727-4b7d-ac38-4fa8def87a13" />
<img width="328" height="206" alt="image" src="https://github.com/user-attachments/assets/2f883c9d-b54d-4258-8338-f601c6d5a456" />
<img width="365" height="48" alt="image" src="https://github.com/user-attachments/assets/68f69aa6-6bf7-41af-8e72-3ac3f62e677a" />
<img width="362" height="58" alt="image" src="https://github.com/user-attachments/assets/0b9427e4-6af2-4c0b-a16a-3be7cb426252" />
<img width="237" height="110" alt="image" src="https://github.com/user-attachments/assets/1223555f-2ffa-4eb1-97ab-cb8c0fde42e6" />


---

## Reto 5 · Capabilities

**La característica:** en vez de dar todo el poder de root, Linux permite repartirlo en piezas sueltas llamadas *capabilities* (por ejemplo `cap_net_raw`, `cap_setuid`). Se pueden dar capabilities concretas a un programa sin hacerlo SUID completo.

**El fallo:** se hizo una copia de `python3` y se le dio la capability `cap_setuid`, "para un script de mantenimiento".

**El ataque:** con `getcap -r /` localicé el binario con la capability sospechosa. Con `cap_setuid`, un programa puede cambiar su propio UID a `0` (root) sin que nadie se lo pida. Ejecutando `pyherramienta -c 'import os; os.setuid(0); os.system("id")'` pasé a ser root.

**La solución:** quité la capability con `setcap -r /usr/local/bin/pyherramienta` (o borré el binario).

**La comprobación:** `sudo bash retos.sh comprobar 5`. Comprueba que no queden capabilities peligrosas fuera de las carpetas del sistema.

**Lo aprendido:** las capabilities parecen "más seguras" que el SUID porque dan menos poder, pero algunas, como `cap_setuid`, equivalen en la práctica a dar root entero. Hay que tratarlas con el mismo cuidado que el bit SUID.

**Captura de pantalla:**

<img width="632" height="134" alt="image" src="https://github.com/user-attachments/assets/3675c93c-a19d-47be-8979-cbae1fb13a30" />

<img width="514" height="69" alt="image" src="https://github.com/user-attachments/assets/a3416459-b208-4604-afa6-3a324712208d" />

<img width="593" height="69" alt="image" src="https://github.com/user-attachments/assets/fb277c97-1c88-4ce3-bd07-81d07b4fbcfa" />

<img width="533" height="133" alt="image" src="https://github.com/user-attachments/assets/16729feb-19d8-4b25-b442-63e650ca5309" />

<img width="386" height="73" alt="image" src="https://github.com/user-attachments/assets/9b526227-1a56-4b5a-b931-a011584b5eac" />

---

## Reto 6 · Tareas programadas (cron)

**La característica:** `cron` ejecuta tareas de forma automática con el usuario que se indique en `/etc/cron.d/`. Si se indica `root`, la tarea corre con todos sus privilegios.

**El fallo:** un script que cron ejecutaba cada minuto **como root** tenía permisos `777`, es decir, cualquier usuario podía editarlo.

**El ataque:** comprobé que el script era editable por cualquiera (`ls -l`) y le añadí una línea (`echo 'id > /tmp/quien_soy.txt' >> script`). Al minuto siguiente, cron ejecutó mi línea añadida como root, y el archivo resultante lo demostró.

**La solución:** cambié el dueño del script a `root` y los permisos para que solo root pueda escribir (`chown root:root` y `chmod 755`, o similar), y quité la línea que había añadido el atacante.

**La comprobación:** `sudo bash retos.sh comprobar 6`. Comprueba que el script sigue existiendo, que `becario` ya no puede escribir en él y que la línea añadida ya no está.

**Lo aprendido:** no basta con proteger el archivo del script; también hay que comprobar los permisos de la **carpeta** donde vive. Si la carpeta permite borrar y recrear archivos (por ejemplo con permiso de escritura para todos), un atacante podría borrar el script protegido y poner uno nuevo con su mismo nombre, aunque el archivo en sí estuviera bien protegido. Además, conviene preguntarse si la tarea necesita ejecutarse como root: en este caso, borrar archivos de `/tmp` no requiere privilegios de root.

**Captura de pantalla:**
**Borramos la linea "id > /tmp/quien_soy.txt"**
<img width="456" height="91" alt="image" src="https://github.com/user-attachments/assets/1922de8c-f0c1-4145-9487-6a5346a07a48" />

<img width="571" height="151" alt="image" src="https://github.com/user-attachments/assets/17b16db2-3d82-4432-a4d9-b63c0243998e" />

<img width="410" height="52" alt="image" src="https://github.com/user-attachments/assets/7f72c9a3-313d-45cf-b1ee-87be6a66b24a" />

<img width="357" height="50" alt="image" src="https://github.com/user-attachments/assets/0fc9223b-2735-487f-9691-08fb1dd99ab9" />

---

## Reto 7 · Servicios ejecutados como root

**La característica:** cada servicio gestionado por `systemd` se ejecuta con un usuario concreto (campo `User=`). Si ese usuario es root y el servicio tiene un fallo, el atacante que lo explote obtiene directamente privilegios de root.

**El fallo:** un servicio web (una "intranet") corría como root y, en vez de publicar solo la carpeta de la web, publicaba **todo el disco** (`--directory /`).

**El ataque:** desde el navegador, accediendo a `http://IP:8080/etc/shadow`, pude descargar el archivo de contraseñas del sistema sin ninguna autenticación.

**La solución:** hay que hacer **dos cambios a la vez**, porque uno solo no basta:
1. Crear un usuario sin privilegios para el servicio (`useradd -r -s /usr/sbin/nologin intranet`) y ponerlo en `User=` dentro de `/etc/systemd/system/intranet.service`.
2. Cambiar `--directory /` por `--directory /srv/web`, para que solo publique la carpeta de la intranet y no todo el disco.

Si solo se cambia el usuario, un usuario sin privilegios seguiría pudiendo leer y servir cualquier archivo del disco que tenga permiso de lectura para "otros" (como ocurre con muchos archivos de configuración), así que el problema de fondo seguiría existiendo. Si solo se cambia la carpeta pero el servicio sigue como root, cualquier fallo futuro del programa daría directamente privilegios de root al atacante. Después de editar, hay que recargar systemd: `systemctl daemon-reload` y `systemctl restart intranet`.

**La comprobación:** `sudo bash retos.sh comprobar 7`. Comprueba que el servicio sigue funcionando, que ya no corre como root y que `/etc/shadow` ya no se puede descargar.

**Lo aprendido:** un servicio debe tener solo el usuario y el acceso a disco que necesita para hacer su trabajo. Publicar todo el disco (`/`) y ejecutar como root son dos errores que, juntos, convierten un fallo pequeño en un control total del sistema por parte del atacante.

**Captura de pantalla:**

_(pega aquí la captura)_

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

Seis de los siete retos acaban dando privilegios de root al atacante. El séptimo acaba en la fuga de datos personales. Esto es lo que se llama **escalada de privilegios**: pasar de un usuario sin apenas permisos a tener control total (o casi) del sistema, aprovechando una configuración demasiado permisiva.

## Fuentes

- GTFOBins (abuso de binarios: `find`, `less`, `vim`...): gtfobins.org
- Manual de sudoers: sudo.ws/docs/man/sudoers.man
- capabilities(7): man7.org/linux/man-pages/man7/capabilities.7.html
- Python `http.server`, advertencias de seguridad: docs.python.org/3/library/http.server.html
- MITRE ATT&CK (T1548, T1053.003, T1565.001): attack.mitre.org
- `systemd.exec` (`User=`, `NoNewPrivileges=`, `ProtectSystem=`): `man systemd.exec`
