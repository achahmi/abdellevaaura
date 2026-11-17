En esta actividad hemos visto para qué sirve Wireshark y cómo nos puede ayudar a revisar lo que pasa en una red. La idea es poder mirar los paquetes que se envían y reciben y encontrar cosas que no sean normales, por ejemplo conexiones sospechosas o tráfico que pueda venir de un equipo infectado

Una de las primeras cosas que hemos aprendido es que se pueden cambiar las columnas de Wireshark para ver mejor la información que nos interesa, como las direcciones IP, los puertos y los protocolos. Esto viene bien porque cuando hay muchos paquetes cuesta bastante encontrar lo que estás buscando

Aquí pondré una captura de Wireshark con las columnas configuradas

También hemos visto que se pueden usar filtros para mostrar solo determinados paquetes, en vez de tener que revisar todo el tráfico. Por ejemplo, podemos filtrar por IP, por DNS o por HTTP. Creo que esta es una de las cosas más importantes, porque te ahorra mucho tiempo cuando tienes una captura con miles de paquetes

Aquí pondré una captura utilizando filtros en Wireshark

Otra cosa que hemos visto es cómo buscar información sobre un equipo y el usuario a partir de una captura de tráfico. Esto puede servir para descubrir qué ordenador está teniendo un problema o cuál puede estar infectado. Hay que fijarse en las direcciones IP y en otros datos que aparecen en las conexiones, aunque no siempre se puede saber todo solamente mirando los paquetes

Aquí pondré una captura donde se vea información del equipo o de sus conexiones

También hemos aprendido que, en algunos casos, se pueden sacar archivos de una captura pcap para analizarlos aparte. Esto es útil si estamos investigando una descarga sospechosa, pero hay que tener cuidado porque el archivo podría ser peligroso y no deberíamos abrirlo directamente en nuestro ordenador

Aquí pondré una captura de la opción para exportar objetos de Wireshark

Después hemos visto el tema del tráfico HTTPS. Normalmente su contenido está cifrado y no se puede leer directamente en Wireshark. Para verlo hace falta tener las claves correspondientes, como las que se guardan en un archivo de registro de claves cuando se hace la captura. No significa que se pueda descifrar cualquier conexión HTTPS sin más

Aquí pondré una captura del tráfico HTTPS y, si tengo el archivo de claves, otra con el contenido descifrado

Por último, hemos visto cómo se puede investigar una infección de Trickbot mirando el tráfico de red. La idea es buscar conexiones, dominios y comportamientos raros que puedan indicar que el equipo está infectado. También hemos visto que un solo paquete no suele ser suficiente para asegurar que hay un ataque, sino que hay que mirar el conjunto del tráfico

Aquí pondré una captura de Wireshark revisando el tráfico relacionado con Trickbot

En general, lo más importante que me llevo de esta actividad es saber usar bien los filtros, organizar la información para no perderme entre tantos paquetes y entender que Wireshark sirve para investigar lo que ocurre en una red. También hay que tener cuidado al analizar archivos sospechosos y no sacar conclusiones solo por ver una conexión extraña. Creo que es una herramienta bastante útil en ciberseguridad porque permite comprobar con más detalle lo que está haciendo un equipo en la red
