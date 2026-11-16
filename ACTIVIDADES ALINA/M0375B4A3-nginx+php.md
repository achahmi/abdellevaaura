# M0375B4A3 - Nginx + PHP

## Objetivo
En esta actividad voy a montar varias páginas con Nginx. Primero haré dos webs diferentes que utilicen el puerto 80, después pondré una zona con contraseña y por último prepararé una web con PHP y otra que saque información de una base de datos. Iré comprobando cada parte para ver que funciona.


## Entorno que voy a utilizar
- Sistema: Kali Linux (Debian)
- Servidor: Nginx
- PHP: PHP-FPM
- Base de datos: MariaDB (compatible con MySQL)
- Dominios locales de ejemplo: `web1.test`, `web2.test`, `php.test`, `db.test`
- IP: 127.0.0.1 para pruebas en la misma máquina

## 1. Instalación de lo necesario

Para empezar voy a instalar Nginx, PHP y MariaDB, que es la base de datos que utilizaré. También instalaré la herramienta que me permitirá poner una contraseña en una parte de la web.

```bash
sudo apt update
sudo apt install -y nginx php-fpm php-mysql mariadb-server apache2-utils
sudo systemctl enable --now nginx mariadb
sudo systemctl status nginx --no-pager
ls /run/php/php*-fpm.sock
```

Si PHP-FPM no aparece funcionando, miraré el nombre de su servicio con `systemctl list-units 'php*-fpm.service'` y lo arrancaré con `sudo systemctl enable --now NOMBRE_DEL_SERVICIO`.


## 2. Crear dos webs con el mismo puerto 80

Primero voy a crear dos carpetas diferentes y pondré un archivo HTML en cada una para poder distinguir las páginas.

```bash
sudo mkdir -p /var/www/web1 /var/www/web2
echo '<h1>WEB 1 - ESTATICA</h1>' | sudo tee /var/www/web1/index.html
echo '<h1>WEB 2 - ESTATICA</h1>' | sudo tee /var/www/web2/index.html
```

Ahora voy a crear la configuración de la primera web:

```bash
sudo nano /etc/nginx/sites-available/web1
```

```nginx
server {
    listen 80;
    server_name web1.test;
    root /var/www/web1;
    index index.html;
}
```

Después haré lo mismo con la segunda web:

```bash
sudo nano /etc/nginx/sites-available/web2
```

```nginx
server {
    listen 80;
    server_name web2.test;
    root /var/www/web2;
    index index.html;
}
```

Cuando tenga las dos configuraciones, las activaré y añadiré los nombres de las webs al archivo `hosts` para poder abrirlas desde mi propia máquina:

```bash
sudo ln -s /etc/nginx/sites-available/web1 /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/web2 /etc/nginx/sites-enabled/
echo '127.0.0.1 web1.test web2.test php.test db.test' | sudo tee -a /etc/hosts
sudo nginx -t
sudo systemctl reload nginx
curl http://web1.test
curl http://web2.test
```

Las dos webs utilizan el puerto 80, pero tienen nombres diferentes. Nginx sabe cuál tiene que mostrar gracias al `server_name`. Si quisiera abrirlas desde otro ordenador, tendría que configurar allí los nombres para que apunten a la IP del servidor.

<img width="641" height="365" alt="image" src="https://github.com/user-attachments/assets/579c8aa2-b963-49f2-b5f7-3a34590031cb" />

<img width="690" height="303" alt="image" src="https://github.com/user-attachments/assets/b8420837-fb32-4b86-8247-1a3b6ff9a24d" />

## 3. Poner contraseña a una parte de la web

En esta parte voy a crear una carpeta privada dentro de la primera web. La idea es que no se pueda entrar sin poner un usuario y una contraseña.

```bash
sudo mkdir -p /var/www/web1/privado
echo '<h1>ZONA PROTEGIDA</h1>' | sudo tee /var/www/web1/privado/index.html
sudo htpasswd -c /etc/nginx/.htpasswd alumno
```

Cuando me pida la contraseña, pondré una que recuerde. No voy a subirla ni a subir el archivo `.htpasswd` a GitHub.

<img width="865" height="458" alt="image" src="https://github.com/user-attachments/assets/185bfd13-a55f-4e62-a16f-7298841922cf" />

<img width="707" height="236" alt="image" src="https://github.com/user-attachments/assets/fb56f3f0-e12e-4165-98b6-80c3fe54db1c" />

Después abriré la configuración de la primera web:

```bash
sudo nano /etc/nginx/sites-available/web1
```

Dentro del bloque `server` añadiré esto para proteger la carpeta:

```nginx
location /privado/ {
    auth_basic "Zona privada";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

Para comprobarlo, revisaré la configuración y probaré a entrar:

```bash
sudo nginx -t
sudo systemctl reload nginx
curl -I http://web1.test/privado/
```

Si entro sin identificarme, debería aparecer `401 Unauthorized`. Después entraré desde el navegador con el usuario `alumno` y la contraseña que haya creado para comprobar que sí me deja pasar.


## 4. Crear una web con PHP

Ahora voy a hacer una página que utilice PHP. En vez de mostrar solamente HTML, pondré un ejemplo que enseñe la fecha y la hora del servidor.

```bash
sudo mkdir -p /var/www/php
sudo nano /var/www/php/index.php
```

```php
<?php
echo "<h1>Web PHP funcionando</h1>";
echo "<p>Fecha del servidor: " . date("d/m/Y H:i:s") . "</p>";
?>
```

Antes de configurar Nginx miraré qué versión de PHP-FPM tengo instalada:

```bash
ls /run/php/php*-fpm.sock
```

Después crearé la configuración de esta web:

```bash
sudo nano /etc/nginx/sites-available/php
```

```nginx
server {
    listen 80;
    server_name php.test;
    root /var/www/php;
    index index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
    }
}
```

Aquí tendré que cambiar `php8.4-fpm.sock` si en mi máquina aparece otra versión. Utilizaré el nombre que me haya mostrado el comando anterior.

```bash
sudo ln -s /etc/nginx/sites-available/php /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
curl http://php.test
```

<img width="700" height="278" alt="image" src="https://github.com/user-attachments/assets/15b96000-d055-443c-9df0-e4669e422c98" />


## 5. Conectar una web PHP a una base de datos

En esta parte voy a crear una base de datos con unos productos de ejemplo. Después haré que PHP los lea y los muestre en una página.

Primero arrancaré MariaDB y entraré:

```bash
sudo systemctl enable --now mariadb
sudo mariadb
```

Dentro de MariaDB crearé la base de datos, un usuario para consultar los datos y una tabla con tres productos:

```sql
CREATE DATABASE actividad_nginx;
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'CAMBIAR_POR_CONTRASENA_SEGURA';
GRANT SELECT ON actividad_nginx.* TO 'webapp'@'localhost';
USE actividad_nginx;
CREATE TABLE productos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);
INSERT INTO productos (nombre) VALUES ('Teclado'), ('Raton'), ('Monitor');
EXIT;
```
<img width="817" height="706" alt="image" src="https://github.com/user-attachments/assets/a5597377-5ccc-4fd9-96ec-5ef6f1cdf85e" />

Cuando esté lista la tabla, crearé la página PHP:
<img width="726" height="330" alt="image" src="https://github.com/user-attachments/assets/4951f3c0-4188-488a-a9c0-e12e28506b82" />


Igual que antes, cambiaré el nombre del socket de PHP si mi versión es diferente.

```bash
sudo ln -s /etc/nginx/sites-available/db /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
curl http://db.test
```

Si la página no consigue conectar con la base de datos, revisaré los servicios con `sudo systemctl status mariadb` y `sudo systemctl status nginx`, y miraré los errores con `sudo tail -n 30 /var/log/nginx/error.log`.


## 6. Comprobar que todo funciona

Cuando tenga todas las webs preparadas, voy a comprobar que Nginx no tiene errores y que cada dirección responde:

```bash
sudo nginx -t
curl -I http://web1.test
curl -I http://web2.test
curl -I http://web1.test/privado/
curl -I http://php.test
curl -I http://db.test
```

Las páginas normales deberían responder con `200 OK`, mientras que la zona protegida debería devolver `401 Unauthorized` si no pongo contraseña. También abriré las webs en el navegador para ver que muestran el contenido correcto.


## 7. Problemas que me hayan salido

- **Problema:** [aquí explicaré el error que me haya salido]
- **Qué hice para arreglarlo:** [pondré los comandos o cambios que utilicé]
- **Cómo quedó:** [explicaré si finalmente funcionó]


