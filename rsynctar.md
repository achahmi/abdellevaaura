# RSYNC - TAR

| | |
| --- | --- |
| **Módulo** | M0378 - Seguridad |
| **Ciclo** | ASIX |
| **Actividad** | Copias de seguridad en Linux con rsync y tar.gz |
| **Nombre alumno** | *(escribe aquí tu nombre)* |
| **Dedicación** | 2 horas |

---

## Parte escrita

### Copias de seguridad

#### ¿Qué son las copias de seguridad? ¿Por qué debemos hacer copias?

Una copia de seguridad es una réplica de nuestros datos guardada en otro sitio, para poder recuperarlos si el original se pierde o se estropea. Hacerlas es cuestión de tiempo: tarde o temprano un disco falla, alguien borra algo sin querer, entra un ransomware o se inunda la sala de servidores, y sin copia no hay marcha atrás

#### ¿Cuáles son los criterios de clasificación de la información que permiten determinar el tipo de datos a salvaguardar?

Lo normal es mirar cuánto vale esa información para la organización y qué pasaría si se pierde. Los criterios más habituales son la criticidad (si el negocio puede seguir sin ello), la confidencialidad (datos personales, contratos, nóminas), las obligaciones legales como el RGPD o la conservación de facturas, la frecuencia con la que cambia y cuánto cuesta volver a generarla. Con eso se decide qué se copia siempre, qué se copia de vez en cuando y qué ni merece la pena guardar

#### ¿De qué depende el tiempo de duración de una copia de seguridad completa?

Sobre todo de cuántos datos hay y de lo rápido que se pueden leer y escribir. Influyen la velocidad del disco de origen y del destino, la red si la copia va a otra máquina, si se comprime o se cifra (que gasta CPU) y hasta el tipo de archivos, porque miles de ficheros pequeños tardan mucho más que un único fichero grande del mismo tamaño total. También cuenta si el equipo está en uso mientras se copia

#### ¿Qué tipos de discos pueden utilizarse para los respaldos? ¿Influyen en el tiempo de respaldo? ¿Cuándo es más habitual hacer los respaldos y restauraciones de la información en una organización?

Se pueden usar discos duros (HDD), discos de estado sólido (SSD), discos externos por USB, cintas LTO, NAS y también almacenamiento en la nube. Sí que influyen: un SSD es mucho más rápido que un HDD, y una cinta escribe muy bien en secuencial pero restaurar un solo fichero es lento. Los respaldos se suelen hacer por la noche o en fin de semana, cuando casi nadie trabaja y el sistema está libre. Las restauraciones, en cambio, no se eligen: se hacen cuando ocurre una incidencia, aunque conviene probarlas de forma periódica para asegurarse de que funcionan

#### ¿Qué es una copia incremental y qué diferencias tiene con la diferencial?

La incremental guarda solo lo que ha cambiado desde la última copia, sea completa o incremental. La diferencial guarda lo que ha cambiado desde la última copia completa. La incremental es más rápida y ocupa menos, pero para restaurar necesitas la completa y todas las incrementales en orden. La diferencial va creciendo cada día, pero para restaurar solo necesitas la completa y la última diferencial, así que es más cómoda a la hora de recuperar

#### ¿Dónde podemos almacenar nuestra copia de seguridad?

En un disco externo, en un NAS, en otro servidor de la empresa, en cintas, en otra sede de la organización o en la nube. Lo importante es que no esté en la misma máquina que los datos originales, porque si falla esa máquina se pierde todo junto

#### ¿En qué consiste la estrategia 3-2-1 de las copias de seguridad?

Es una regla fácil de recordar: tener 3 copias de los datos (el original más dos copias), guardadas en 2 tipos de soporte distintos (por ejemplo disco y nube) y con 1 de ellas fuera de la sede. Así, aunque pase algo gordo como un incendio o un ataque, siempre queda una copia a salvo

#### ¿Qué tipo de herramientas de cifrado nos permiten proteger las copias de seguridad?

Hay bastantes según el caso. GPG sirve para cifrar archivos sueltos o un tar.gz, LUKS y VeraCrypt cifran discos o volúmenes enteros, 7-Zip permite proteger ficheros con AES-256 y OpenSSL también vale para cifrar archivos desde la terminal. Además, herramientas de backup como restic o BorgBackup ya cifran las copias de serie

#### ¿Qué debemos considerar a la hora de hacer copias de seguridad en la nube?

Lo primero, cifrar los datos antes de subirlos para que el proveedor no pueda leerlos. Después mirar dónde están físicamente los servidores (por el RGPD), cuánto cuesta no solo guardar sino también descargar, cuánto tarda la primera subida y cuánto tardaría una restauración completa con nuestra conexión. También hay que revisar cuántas versiones se conservan, activar la verificación en dos pasos en la cuenta y leer bien las condiciones del servicio

#### ¿Qué es un SAN?

Una SAN (Storage Area Network) es una red dedicada solo a almacenamiento, que da a los servidores discos como si estuvieran conectados directamente a ellos (a nivel de bloque). Suele montarse con Fibre Channel o iSCSI y se usa en centros de datos y entornos de virtualización, donde hace falta mucho rendimiento y alta disponibilidad. Es potente pero cara y más compleja de montar

#### ¿Qué es un NAS? Pon ejemplos de pago y libre

Un NAS (Network Attached Storage) es un aparato de almacenamiento conectado a la red que comparte carpetas y archivos con otros equipos mediante protocolos como SMB o NFS. Es mucho más sencillo y barato que una SAN, ideal para pequeñas empresas y casas. Ejemplos de pago: Synology, QNAP o Western Digital My Cloud. Ejemplos libres: TrueNAS, OpenMediaVault o Rockstor

---

### Explica el proceso desarrollado en la parte práctica con rsync y tar.gz

#### Creación y restauración de copias de una carpeta de contenidos en una misma MV

Se crean dos carpetas, `origen` con los ficheros y `destino` donde irá la copia. Con rsync se copia el contenido de una a la otra y, si algo cambia, se vuelve a lanzar el mismo comando y solo se transfiere lo nuevo. Con tar.gz se empaqueta y comprime `origen` en un único archivo que se guarda en `destino`. Para restaurar con rsync se invierten origen y destino, y con tar.gz se descomprime el archivo en la carpeta que queramos

#### Creación y restauración de copias en dos MV (una origen y otra destino)

Es lo mismo pero añadiendo `usuario@IP:` delante de la ruta de la máquina remota, así que hace falta tener SSH funcionando entre las dos MV. Con rsync se copia directamente la carpeta al otro equipo. Con tar.gz se crea el archivo y se manda con `scp`, o se comprime y se envía todo de una vez por SSH. La restauración es igual pero en sentido contrario, trayendo los datos desde la máquina remota

#### Si tengo que desarrollar un script de copias, ¿qué variante utilizo: rsync o tar.gz?

Depende de lo que busque. Si quiero una copia diaria rápida de una carpeta, rsync es lo más cómodo porque solo envía lo que ha cambiado y, si la conexión se corta, puede continuar. Si lo que quiero es guardar un histórico con versiones fechadas (la del lunes, la del martes), tar.gz encaja mejor porque cada copia es un archivo cerrado que se puede guardar, mover o cifrar fácilmente

#### ¿Cuál consideras la mejor opción? Argumenta

Para el día a día me quedo con rsync, porque es más rápido, gasta menos red y menos espacio al no repetir lo que no ha cambiado, y además es muy sencillo de automatizar. Pero no creo que haya que elegir una sola, porque lo ideal es combinarlas: rsync para la sincronización diaria y un tar.gz de vez en cuando como copia fechada que se pueda guardar fuera de la sede

---

### RSYNC

#### ¿Qué es rsync? ¿En qué contexto se puede utilizar?

Rsync es una herramienta de Linux para copiar y sincronizar archivos entre carpetas o entre equipos. Lo que la hace especial es que solo transfiere las diferencias, es decir, si ya tienes un fichero en el destino y solo han cambiado unas líneas, envía solo esa parte. Se usa para copias de seguridad, para mantener servidores sincronizados, para migrar datos de un equipo a otro y para publicar contenido web

#### Ejemplo de uso entre dos MV diferentes

Desde la máquina origen, enviando la carpeta `origen` al equipo remoto:

```bash
rsync -avzh origen/ punky@192.168.1.78:backups/
```

Y para traerla de vuelta desde el equipo remoto:

```bash
rsync -avzh punky@192.168.1.78:backups/ origen/
```

---

### TAR.GZ

#### ¿Qué es el tar.gz?

Es un archivo que junta muchos ficheros y carpetas en uno solo (eso lo hace `tar`) y luego lo comprime para que ocupe menos (eso lo hace `gzip`). Se queda con permisos, propietarios y fechas, así que es muy práctico para guardar copias

#### ¿Cómo se utiliza el tar.gz para crear copias incrementales y diferenciales?

Se hace con la opción `--listed-incremental`, que usa un fichero de control (el `.snar`) donde tar apunta el estado de los archivos en cada copia. Así sabe qué ha cambiado la próxima vez

Para la incremental, se usa siempre el mismo fichero de control: la primera vez es una copia completa y las siguientes solo guardan lo nuevo desde la última vez

```bash
tar -czvf full.tar.gz --listed-incremental=control.snar origen/
tar -czvf inc1.tar.gz --listed-incremental=control.snar origen/
tar -czvf inc2.tar.gz --listed-incremental=control.snar origen/
```

Para la diferencial, el truco es guardar una copia del fichero de control justo después de la completa y usar esa copia limpia cada vez, para que siempre compare contra la completa

```bash
tar -czvf full.tar.gz --listed-incremental=control.snar origen/
cp control.snar control_full.snar

cp control_full.snar control_dif.snar
tar -czvf dif1.tar.gz --listed-incremental=control_dif.snar origen/

cp control_full.snar control_dif.snar
tar -czvf dif2.tar.gz --listed-incremental=control_dif.snar origen/
```

Para restaurar, se extrae primero la completa y después las incrementales en orden (o solo la última diferencial), siempre con `--listed-incremental=/dev/null`:

```bash
tar -xzvf full.tar.gz --listed-incremental=/dev/null -C restaurado/
tar -xzvf inc1.tar.gz --listed-incremental=/dev/null -C restaurado/
```

> 📸 **Captura:** creación de la copia completa, la incremental y la diferencial con tar

> 📸 **Captura:** restauración de las copias con tar

---

## Parte práctica


Antes de nada, en la maquina destino ponemos el hostname como "Destino"
<img width="570" height="41" alt="image" src="https://github.com/user-attachments/assets/f147ba01-e9a9-44be-870a-dbe4a53644c1" />
Luego, de instalar el ssh y configurarlo todo correctamente, nos hace ping!!:
<img width="619" height="267" alt="image" src="https://github.com/user-attachments/assets/6d9492b7-8444-4371-8d76-14c74888d1a8" />

Además, podemos hacer ssh correctamente:
<img width="733" height="357" alt="image" src="https://github.com/user-attachments/assets/eb875830-c4eb-4dc2-954c-5e7df404ba69" />

### (1) Copia en local con RSYNC y con TAR.GZ

#### Preparar el entorno

Primero creo las dos carpetas y unos cuantos ficheros dentro de origen:

```bash
mkdir origen destino
cd origen
touch file{1..10}
ls -l
```

> 📸 **Captura:** creación de las carpetas y de los ficheros, con el `ls -l` de origen

#### Backup completo con rsync

```bash
rsync -avzh origen/ destino/
```

La primera vez copia todo, así que es el backup completo

> 📸 **Captura:** primera ejecución de rsync copiando los 10 ficheros

#### Backup incremental con rsync

Si lanzo el mismo comando otra vez sin tocar nada, no copia nada. Ahora modifico un fichero y lo ejecuto de nuevo:

```bash
echo "contenido nuevo" > origen/file3
rsync -avzh origen/ destino/
```

Rsync compara los ficheros de las dos carpetas (tamaño y fecha de modificación) y solo transfiere los que han cambiado, en este caso `file3`. Así es como consigue los backups incrementales, sin necesidad de ninguna opción especial

> 📸 **Captura:** segunda ejecución sin cambios (no copia nada)

> 📸 **Captura:** tercera ejecución después de modificar `file3` (copia solo ese fichero)

#### ¿Es lo mismo `origen/` que `origen`?

No es lo mismo. La barra final cambia lo que se copia:

```bash
rsync -avzh carpeta_origen/ carpeta_destino/   # copia el contenido de la carpeta
rsync -avzh carpeta_origen carpeta_destino/    # copia la carpeta entera, dentro de destino
```

Con la barra, los ficheros quedan directamente dentro de `destino/`. Sin la barra, dentro de `destino/` aparece una carpeta llamada `origen` que contiene los ficheros

> 📸 **Captura:** resultado con `origen/` (ficheros directamente en destino)

> 📸 **Captura:** resultado con `origen` sin barra (aparece la carpeta `origen` dentro de destino)

#### Analizar el contenido de la carpeta destino

```bash
ls -la destino/
```

**¿Se mantienen los atributos de los archivos?** Sí, la opción `-a` (archivo) conserva permisos, fechas de modificación y enlaces simbólicos

**¿Se mantienen los usuarios?** Solo si rsync se ejecuta como root. Si lo lanzo como usuario normal, todos los ficheros del destino pasan a ser de ese usuario. Con `sudo` sí se conservan el propietario y el grupo originales

**¿Qué pasa si modifico un fichero en origen?** Al volver a ejecutar rsync, el cambio se refleja en el destino. Si además quiero que lo que borro en origen se borre en destino, hay que añadir `--delete`

> 📸 **Captura:** `ls -la` de origen y destino comparando permisos, fechas y propietarios

> 📸 **Captura:** prueba con `sudo` para ver cómo se mantiene el propietario

#### Recuperación del contenido

Para recuperar se invierten origen y destino. Primero simulo el desastre borrando algo en origen:

```bash
rm origen/file1 origen/file2
rsync -avzh destino/ origen/
```

> 📸 **Captura:** borrado de ficheros en origen y recuperación con rsync

#### Lo mismo con TAR.GZ

Copia completa, comprimiendo la carpeta origen y dejándola en destino:

```bash
tar -czvf destino/backup_full.tar.gz origen/
```

Copias incremental y diferencial (con el fichero de control, como se explica en la parte escrita):

```bash
tar -czvf destino/full.tar.gz --listed-incremental=destino/control.snar origen/
touch origen/file11
tar -czvf destino/inc1.tar.gz --listed-incremental=destino/control.snar origen/
```

Para ver qué hay dentro de un archivo sin descomprimirlo:

```bash
tar -tzvf destino/inc1.tar.gz
```

Recuperación del contenido:

```bash
mkdir restaurado
tar -xzvf destino/backup_full.tar.gz -C restaurado/
```

> 📸 **Captura:** creación del tar.gz completo en destino

> 📸 **Captura:** copia incremental con tar y listado de su contenido

> 📸 **Captura:** restauración del tar.gz en la carpeta `restaurado`

---

### (2) Copia por la red a un equipo remoto con RSYNC y con TAR.GZ

#### Preparar las dos máquinas

Para las pruebas uso dos MV conectadas en modo adaptador puente (también valdría una red solo anfitrión). En la máquina que hace de destino instalo SSH y compruebo su IP:

```bash
sudo apt install openssh-server
ip a
```

Y en la máquina de origen compruebo que llego al otro equipo:

```bash
ping 192.168.1.78
ssh punky@192.168.1.78
```

> 📸 **Captura:** instalación de SSH en la MV destino

> 📸 **Captura:** `ip a` de las dos máquinas, con sus IPs

> 📸 **Captura:** conexión SSH desde la MV origen a la MV destino

#### Copia con rsync

En la máquina destino creo antes la carpeta `backups` y, desde la máquina origen, envío los datos:

```bash
rsync -avzh origen/ punky@192.168.1.78:backups/
```

Aquí también importa la barra final: con `origen/` envía el contenido, y con `origen` envía la carpeta entera

> 📸 **Captura:** ejecución de rsync hacia el equipo remoto

> 📸 **Captura:** contenido de `backups` en la MV destino

#### Copia con tar.gz

Creo el archivo en local y lo mando con `scp`:

```bash
tar -czvf backup_origen.tar.gz origen/
scp backup_origen.tar.gz punky@192.168.1.78:backups/
```

También se puede hacer todo de una vez, sin crear el archivo en medio, enviándolo directamente por SSH:

```bash
tar -czf - origen/ | ssh punky@192.168.1.78 "cat > backups/backup_origen.tar.gz"
```

> 📸 **Captura:** creación del tar.gz y envío con `scp`

> 📸 **Captura:** archivo tar.gz recibido en la MV destino

---

### (3) Recuperación del contenido a través de la red con RSYNC y con TAR.GZ

#### Con rsync

Ahora el equipo remoto es el origen de los datos y yo los traigo de vuelta:

```bash
rsync -avzh punky@192.168.1.78:backups/ origen/
```

> 📸 **Captura:** recuperación con rsync desde el equipo remoto

> 📸 **Captura:** comprobación de que los ficheros han vuelto a la carpeta origen

#### Con tar.gz

Me traigo el archivo y lo descomprimo:

```bash
scp punky@192.168.1.78:backups/backup_origen.tar.gz .
mkdir restaurado
tar -xzvf backup_origen.tar.gz -C restaurado/
```

O sin pasos intermedios, extrayéndolo directamente desde el equipo remoto:

```bash
ssh punky@192.168.1.78 "cat backups/backup_origen.tar.gz" | tar -xzvf - -C restaurado/
```

> 📸 **Captura:** descarga y descompresión del tar.gz desde el equipo remoto

> 📸 **Captura:** contenido restaurado

---

### (4) Automatizando el proceso de la copia

El problema de las copias por red es que SSH pide la contraseña cada vez, y un proceso automático no puede escribirla. Hay dos maneras de resolverlo

#### Opción 1: sshpass

Es una utilidad que le pasa la contraseña a SSH por nosotros:

```bash
sudo apt install sshpass
sshpass -p 'micontraseña' rsync -avzh -e ssh origen/ punky@192.168.1.78:backups/
```

Funciona y es muy rápida de montar, pero no me convence nada: la contraseña queda escrita en texto plano en el comando (y en el script, y en el historial de la terminal), así que cualquiera con acceso a ese fichero la ve. Sirve para probar o para entornos de laboratorio, pero no la usaría en un servidor real

> 📸 **Captura:** instalación de sshpass

> 📸 **Captura:** copia con rsync usando sshpass sin que pida contraseña

#### Opción 2: claves SSH (pública y privada)

Es la forma correcta. Se genera un par de claves en la máquina origen y se copia la pública al equipo remoto, de modo que este reconoce a la máquina origen sin necesidad de contraseña:

```bash
ssh-keygen -t rsa
ssh-copy-id punky@192.168.1.78
ssh punky@192.168.1.78
```

La clave privada se queda en el origen y nunca se comparte. Si ahora entro por SSH ya no pide contraseña, y lo mismo pasa con rsync y scp. Es mucho más seguro porque no hay ninguna contraseña escrita en ningún sitio

> 📸 **Captura:** generación del par de claves con `ssh-keygen`

> 📸 **Captura:** copia de la clave pública con `ssh-copy-id`

> 📸 **Captura:** conexión SSH sin contraseña

---

### (5) Script de copia de seguridad en un equipo remoto, programado con crontab

Para que los scripts funcionen solos, necesitan que ya estén configuradas las claves SSH del paso anterior

#### Script con RSYNC

Fichero `backup_rsync.sh`:

```bash
#!/bin/bash

ORIGEN="/home/usuario/origen/"
DESTINO="punky@192.168.1.78:backups/"
LOG="$HOME/backup_rsync.log"

echo "=== Inicio: $(date '+%d/%m/%Y %H:%M:%S') ===" >> "$LOG"
rsync -avzh --delete -e ssh "$ORIGEN" "$DESTINO" >> "$LOG" 2>&1
echo "=== Fin: $(date '+%d/%m/%Y %H:%M:%S') ===" >> "$LOG"
```

#### Script con TAR.GZ

Fichero `backup_tar.sh`:

```bash
#!/bin/bash

FECHA=$(date +%Y-%m-%d_%H-%M)
ARCHIVO="/tmp/origen_$FECHA.tar.gz"
DESTINO="punky@192.168.1.78:backups/"
LOG="$HOME/backup_tar.log"

echo "=== Inicio: $(date '+%d/%m/%Y %H:%M:%S') ===" >> "$LOG"
tar -czf "$ARCHIVO" -C /home/usuario origen >> "$LOG" 2>&1
scp "$ARCHIVO" "$DESTINO" >> "$LOG" 2>&1 && rm "$ARCHIVO"
# Borro del equipo remoto las copias de más de 7 días
ssh punky@192.168.1.78 "find backups -name 'origen_*.tar.gz' -mtime +7 -delete" >> "$LOG" 2>&1
echo "=== Fin: $(date '+%d/%m/%Y %H:%M:%S') ===" >> "$LOG"
```

Les doy permisos de ejecución y los pruebo a mano antes de programarlos:

```bash
chmod +x backup_rsync.sh backup_tar.sh
./backup_rsync.sh
./backup_tar.sh
```

> 📸 **Captura:** contenido del script `backup_rsync.sh`

> 📸 **Captura:** contenido del script `backup_tar.sh`

> 📸 **Captura:** ejecución manual de los dos scripts y fichero de log

#### ¿Qué es el crontab?

Es el archivo donde se guardan las tareas programadas con cron, el programador de tareas de Linux. Dentro se escribe una línea por tarea indicando cuándo debe ejecutarse y qué comando o script lanzar, y cron se encarga de hacerlo solo. Cada usuario tiene su propio crontab, y se edita con `crontab -e`. El formato de cada línea tiene cinco campos de tiempo seguidos del comando:

```
minuto  hora  día-del-mes  mes  día-de-la-semana  comando
```

Por ejemplo, esta línea ejecuta un script todos los días a las 19:30:

```
30 19 * * * /home/usuario/backup.sh
```

#### Programar las tareas

Abro el crontab con `crontab -e` y añado estas líneas, una para cada script:

```
30 19 * * * /home/usuario/backup_rsync.sh
0 20 * * 5 /home/usuario/backup_tar.sh
```

La primera lanza la copia con rsync todos los días a las 19:30 y la segunda crea el tar.gz fechado todos los viernes a las 20:00. Para ver las tareas programadas uso `crontab -l`

Para comprobar que funciona sin esperar a la hora, programo temporalmente la tarea para que se ejecute cada minuto (`* * * * *`), miro el log y después la dejo con su horario normal

> 📸 **Captura:** `crontab -e` con las dos tareas programadas

> 📸 **Captura:** `crontab -l` mostrando las tareas

> 📸 **Captura:** log del script confirmando que cron lo ha ejecutado

> 📸 **Captura:** copias recibidas en la MV destino

---

## Links

- https://www.ionos.es/digitalguide/servidores/herramientas/como-crear-copias-de-seguridad-del-servidor-con-rsync/
- https://www.youtube.com/watch?v=t18-C6f9y5Q
- https://www.linuxtotal.com.mx/index.php?cont=rsync-manual-de-uso
- https://sistemas.com.es/rsync-copias-de-seguridad-remotas/
- https://geekflare.com/es/crontab-linux-with-real-time-examples-and-tools/
- https://crontab.guru/#30_20_*_*_*
