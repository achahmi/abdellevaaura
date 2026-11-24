# Wireshark – Amenazas y ataques

En el primer tutorial voy a personalizar las columnas de Wireshark para poder ver mejor las IP, los puertos y los nombres de los servidores. Esto me permitirá identificar las conexiones sin tener que abrir cada paquete por separado

[Captura 1: columnas personalizadas]

En el segundo tutorial voy a utilizar filtros para separar las peticiones web, DNS y las conexiones TCP del resto del tráfico. Lo importante es saber elegir el filtro adecuado para no perder información relevante

[Captura 2: filtro de tráfico web]

[Captura 3: filtro DNS y conexiones TCP]

En el tercer tutorial voy a buscar información de los equipos y usuarios que aparecen en las capturas. Con DHCP puedo relacionar la IP, la MAC y el nombre del equipo, y con otros protocolos como NBNS y Kerberos puedo encontrar más información del sistema y del usuario

[Captura 4: información DHCP]

[Captura 5: información de equipo y usuario en NBNS o Kerberos]

En el cuarto tutorial voy a comprobar cómo Wireshark reconstruye los archivos que se han transmitido por la red. Desde la opción de exportar objetos puedo ver los archivos transferidos por HTTP y SMB. Es importante no ejecutar ni abrir archivos sospechosos

[Captura 6: objetos HTTP]

[Captura 7: objetos SMB]

En el quinto tutorial voy a comprobar la diferencia entre el tráfico HTTPS cifrado y el tráfico que se puede leer al cargar el archivo de claves que acompaña a la captura. Sin las claves correspondientes no puedo ver directamente el contenido cifrado

[Captura 8: tráfico TLS antes de cargar las claves]

[Captura 9: tráfico HTTP después de cargar las claves]

En el último tutorial voy a analizar capturas relacionadas con Trickbot. Buscaré conexiones y peticiones que coincidan con los patrones descritos en el tutorial, como tráfico web por puertos poco habituales. Una sola conexión no es suficiente para confirmar una infección

[Captura 10: tráfico de Trickbot filtrado]

[Captura 11: análisis de una petición sospechosa]

Con esta actividad quiero aprender a utilizar Wireshark para identificar equipos, filtrar tráfico y reconocer posibles amenazas sin tener que revisar todos los paquetes uno por uno

Resultados y conclusiones tras realizar las pruebas: [Completar después de realizar los tutoriales]
