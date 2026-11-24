# Examen - Network Chuck

Lo primero que hice fue abrir el archivo quetengoaqui.pcapng con Wireshark y revisar el tráfico de red. Al seguir la secuencia TCP encontré un listado de posibles contraseñas que se habían transmitido por la red

[Captura 1: listado de contraseñas en Wireshark]

Después utilicé Hydra desde Kali para comprobar las contraseñas en el servidor SSH del entorno del examen. Conseguí encontrar una contraseña válida para el usuario de la máquina Debian

[Captura 2: resultado de Hydra]

Una vez encontrada la contraseña, inicié sesión mediante SSH en la máquina Debian

[Captura 3: conexión por SSH]

Finalmente entré en la carpeta SEGiAD dentro de Documents y utilicé nano para abrir un archivo con mi nombre

[Captura 4: acceso a la carpeta SEGiAD y comando nano]

Con esta actividad he podido comprobar lo importante que es proteger los datos que se transmiten por la red y utilizar contraseñas seguras, ya que una captura de tráfico puede contener información que facilite el acceso a un equipo
