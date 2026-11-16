# M0375B4A3 - Nginx + PHP

## Objetivo
Configurar Nginx con dos webs estáticas que compartan el puerto 80 pero tengan dominios diferentes, una zona protegida con contraseña, una web dinámica con PHP y otra con PHP y MySQL/MariaDB. Comprobar cada apartado y documentar las pruebas.

> **Estado:** guía de realización. Las capturas y resultados se añadirán al hacer la práctica. El documento adjunto pide PHP, base de datos, pruebas e incidencias; también se incluyen los requisitos adicionales de la rúbrica de clase.

## Entorno
- Sistema: Kali Linux (Debian)
- Servidor: Nginx
- PHP: PHP-FPM
- Base de datos: MariaDB (compatible con MySQL)
- Dominios locales de ejemplo: `web1.test`, `web2.test`, `php.test`, `db.test`
- IP: 127.0.0.1 para pruebas en la misma máquina

## 1. Instalar paquetes y comprobar servicios

```bash
sudo apt update
sudo apt install -y nginx php-fpm php-mysql mariadb-server apache2-utils
sudo systemctl enable --now nginx mariadb
sudo systemctl status nginx --no-pager
ls /run/php/php*-fpm.sock
```

Si PHP-FPM no está activo, identificar el servicio con `systemctl list-units 'php*-fpm.service'` y arrancarlo con `sudo systemctl enable --now NOMBRE_DEL_SERVICIO`.

**Captura 1:** Nginx activo, versión/servicio de PHP-FPM y MariaDB.

## 2. Dos webs estáticas, dos dominios, mismo puerto 80

Crear directorios y páginas:

```bash
sudo mkdir -p /var/www/web1 /var/www/web2
echo '<h1>WEB 1 - ESTATICA</h1>' | sudo tee /var/www/web1/index.html
echo '<h1>WEB 2 - ESTATICA</h1>' | sudo tee /var/www/web2/index.html
```

Crear el primer sitio:

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

Crear el segundo sitio:

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

Activar y probar:

```bash
sudo ln -s /etc/nginx/sites-available/web1 /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/web2 /etc/nginx/sites-enabled/
echo '127.0.0.1 web1.test web2.test php.test db.test' | sudo tee -a /etc/hosts
sudo nginx -t
sudo systemctl reload nginx
curl http://web1.test
curl http://web2.test
```

**Explicación:** los dos sitios escuchan en el puerto 80. Nginx elige la web según el nombre de dominio recibido en la petición (`server_name`). Si se accede desde otro equipo, se debe configurar su resolución DNS o archivo hosts con la IP real del servidor.

**Captura 2:** configuraciones `web1` y `web2` con `listen 80`.
**Captura 3:** ambas páginas abiertas en navegador, mostrando dominios distintos.

## 3. Zona protegida con contraseña

Crear un directorio privado en la primera web:

```bash
sudo mkdir -p /var/www/web1/privado
echo '<h1>ZONA PROTEGIDA</h1>' | sudo tee /var/www/web1/privado/index.html
sudo htpasswd -c /etc/nginx/.htpasswd alumno
```

Introducir una contraseña cuando se solicite. **No subir la contraseña ni el archivo .htpasswd al repositorio.**

Editar el sitio:

```bash
sudo nano /etc/nginx/sites-available/web1
```

Añadir dentro de `server { ... }`:

```nginx
location /privado/ {
    auth_basic "Zona privada";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

Probar:

```bash
sudo nginx -t
sudo systemctl reload nginx
curl -I http://web1.test/privado/
```

Debe devolver `401 Unauthorized` sin credenciales. En el navegador abrir `http://web1.test/privado/` e iniciar sesión con el usuario `alumno` y la contraseña creada.

**Captura 4:** petición sin credenciales con 401 o ventana de autenticación.
**Captura 5:** acceso correcto a la zona protegida.

## 4. Web dinámica con PHP

Crear el directorio y archivo:

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

Identificar el socket PHP-FPM:

```bash
ls /run/php/php*-fpm.sock
```

Crear el sitio:

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

**Importante:** sustituir `php8.4-fpm.sock` por el nombre exacto mostrado por `ls`; puede variar según la versión instalada.

```bash
sudo ln -s /etc/nginx/sites-available/php /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
curl http://php.test
```

**Captura 6:** configuración PHP-FPM de Nginx y página mostrando la fecha generada por PHP.

## 5. Web PHP + base de datos MariaDB/MySQL

Iniciar la base de datos y entrar:

```bash
sudo systemctl enable --now mariadb
sudo mariadb
```

Ejecutar dentro de MariaDB:

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

Crear la web:

```bash
sudo mkdir -p /var/www/db
sudo nano /var/www/db/index.php
```

```php
<?php
try {
    $pdo = new PDO(
        'mysql:host=localhost;dbname=actividad_nginx;charset=utf8mb4',
        'webapp',
        'CAMBIAR_POR_CONTRASENA_SEGURA',
        [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
    );
    $productos = $pdo->query('SELECT nombre FROM productos')->fetchAll(PDO::FETCH_ASSOC);
    echo '<h1>Productos de la base de datos</h1><ul>';
    foreach ($productos as $producto) {
        echo '<li>' . htmlspecialchars($producto['nombre'], ENT_QUOTES, 'UTF-8') . '</li>';
    }
    echo '</ul>';
} catch (PDOException $e) {
    http_response_code(500);
    echo 'Error al conectar con la base de datos';
}
?>
```

**Seguridad:** usar la misma contraseña segura en SQL y PHP. Este ejemplo la coloca en el archivo para facilitar la práctica; no subir el PHP con credenciales reales a un repositorio público. Para un proyecto real, almacenar la contraseña fuera del directorio web o en variables de entorno.

Crear el sitio:

```bash
sudo nano /etc/nginx/sites-available/db
```

```nginx
server {
    listen 80;
    server_name db.test;
    root /var/www/db;
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

Sustituir el nombre del socket por el que realmente existe.

```bash
sudo ln -s /etc/nginx/sites-available/db /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
curl http://db.test
```

Si la conexión falla, comprobar `sudo systemctl status mariadb`, `sudo systemctl status nginx` y `sudo tail -n 30 /var/log/nginx/error.log`.

**Captura 7:** tabla `productos` con los tres registros desde MariaDB.
**Captura 8:** página `db.test` mostrando los productos obtenidos de la base de datos.

## 6. Comprobaciones finales

```bash
sudo nginx -t
curl -I http://web1.test
curl -I http://web2.test
curl -I http://web1.test/privado/
curl -I http://php.test
curl -I http://db.test
```

Las webs públicas deberían responder con HTTP 200; la zona privada, sin credenciales, debería responder HTTP 401. Abrir cada sitio en el navegador para comprobar el contenido.

**Captura 9:** pruebas finales de las cuatro webs y zona protegida.

## 7. Incidencias

- **Incidencia encontrada:** [escribir el error real, si aparece]
- **Cómo se solucionó:** [comandos y cambios aplicados]
- **Resultado:** [explicar si todo funciona]

## 8. Conclusión

En esta práctica se configuran varias webs en Nginx usando dominios diferentes sobre el mismo puerto 80. También se protege una zona mediante usuario y contraseña, se ejecuta una página PHP con PHP-FPM y se conecta otra página PHP a una base de datos para mostrar información. Las capturas finales servirán para demostrar cada apartado una vez realizado.

## Rúbrica de clase

- [ ] Dos webs estáticas con dominios distintos y puerto 80 — **2 puntos**
- [ ] Acceso a zona protegida — **1 punto**
- [ ] Web con PHP — **2 puntos**
- [ ] Web con PHP + MySQL/MariaDB — **3 puntos**
- [ ] Explicación en GitHub y evidencias — **2 puntos**

> La rúbrica de clase indicada para esta entrega es de 8 puntos prácticos y 2 escritos; el documento adjunto incluye una rúbrica anterior distinta. Las evidencias están pendientes de ejecutar y subir.
