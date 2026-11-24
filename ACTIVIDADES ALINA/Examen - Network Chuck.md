# Examen - Network Chuck

Lo primero que hice fue abrir el archivo quetengoaqui.pcapng con Wireshark y revisar el tráfico de red. Al seguir la secuencia TCP encontré un listado de posibles contraseñas que se habían transmitido por la red

<img width="645" height="1031" alt="image" src="https://github.com/user-attachments/assets/93a385df-44f7-44fa-add6-d805d389299f" />


Después utilicé Hydra desde Kali para comprobar las contraseñas en el servidor SSH del entorno del examen. Conseguí encontrar una contraseña válida para el usuario de la máquina Debian

<img width="831" height="181" alt="image" src="https://github.com/user-attachments/assets/748fb7b2-9fcb-48b6-ba32-2cbfccc5aff2" />


Una vez encontrada la contraseña, inicié sesión mediante SSH en la máquina Debian

<img width="370" height="58" alt="image" src="https://github.com/user-attachments/assets/eee2a05c-bede-4c67-8403-dd60b0c04791" />


Finalmente entré en la carpeta SEGiAD dentro de Documents y utilicé nano para abrir un archivo con mi nombre

<img width="547" height="51" alt="image" src="https://github.com/user-attachments/assets/f7bcec97-9543-49e1-a18b-96b7d39a8731" />


Con esta actividad he podido comprobar lo importante que es proteger los datos que se transmiten por la red y utilizar contraseñas seguras, ya que una captura de tráfico puede contener información que facilite el acceso a un equipo
