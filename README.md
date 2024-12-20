# 📘 Práctica 4

Este proyecto configura dos servidores web independientes (`perfectweb` y `pedro`) en una máquina virtual utilizando Vagrant y Nginx. Ambos sitios usan autenticación básica.

## 📑 Descripción General

Este proyecto proporciona un entorno web configurado en una máquina virtual usando Vagrant para la gestión y Nginx como servidor.

- **Sitio `pedro`**: Un sitio web clonado desde un repositorio de GitHub.
- **Sitio `perfectweb`**: Un sitio web estático con contenido alojado localmente.

Ambos sitios están protegidos con autenticación básica, y el acceso al sitio `perfectweb` está restringido a una IP específica (`192.168.57.1`). Este README describe paso a paso cómo configurarlos y solucionar problemas.

---

## 📌 Objetivos del Proyecto

1. Instalar y configurar Nginx en una máquina virtual Debian.
2. Configurar dos sitios web:
   - `pedro`: Clonado desde un repositorio Git.
   - `perfectweb`: Contenido estático protegido y con restricciones de IP.
3. Configurar autenticación básica para ambos sitios con `.htpasswd`.
4. Asignar los dominios `pedro` y `perfectweb` a la IP `192.168.57.103`.
5. Verificar el correcto funcionamiento accediendo desde un navegador.

---

## 📝 Requisitos

- **Software**:
  - Vagrant instalado en la máquina anfitriona.
  - VirtualBox u otro proveedor compatible con Vagrant.
- **Archivos necesarios**:
  - Carpeta `html/` con el contenido del sitio `perfectweb`.
  - Archivo `.htpasswd` para la autenticación básica.
  - Archivos de configuración `pedro` y `perfectweb` para Nginx.

---

## 🗃️ Estructura del Proyecto

```plaintext
proyecto/
├── Vagrantfile            # Configuración de Vagrant para la VM con Nginx
├── README.md              # Documentación del proyecto
├── .htpasswd              # Archivo de autenticación para ambos sitios
├── pedro                  # Configuración del sitio pedro en Nginx
├── perfectweb             # Configuración del sitio perfectweb en Nginx
└── html/                  # Carpeta con los archivos del sitio perfectweb
    ├── fonts/
    ├── css/
    ├── js/
    ├── images/
    ├──  php/
    ├── about.html
    ├── contact.html
    ├── index.html
    ├── services.html
```

# 🗂️ Archivos importantes

Configuración de Nginx
Archivo para pedro: `/etc/nginx/sites-available/pedro`

```
server {
    listen 80;
    server_name pedro;

    root /var/www/pedro/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        auth_basic "Restricted Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

Archivo para perfectweb:
`/etc/nginx/sites-available/perfectweb`

```
server {
    listen 80;
    server_name perfectweb;

    root /var/www/perfectweb/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        auth_basic "Restricted Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
        allow 192.168.57.1;
        deny all;
        try_files $uri $uri/ =404;
    }
}
```

Archivo /etc/hosts en la máquina anfitriona:

```
127.0.0.1    localhost
192.168.57.103 pedro
192.168.57.103 perfectweb
```

📝 Comprobaciones y Pruebas
✔️ Verificación de los sitios web
Acceso a pedro: Abre http://pedro en un navegador y usa las credenciales configuradas en .htpasswd.

Acceso a perfectweb: Abre http://perfectweb desde la IP permitida (192.168.57.1). Asegúrate de introducir las credenciales correctas.

# ✅ Comprobación de Logs

Para verificar los logs de acceso y errores, revisa estos archivos en la máquina virtual:

- Log de accesos: `/var/log/nginx/access.log`
- Log de errores: `/var/log/nginx/error.log`

# ❌ Solución de Problemas

## El sitio no carga:

1. Verifica los enlaces simbólicos:
   `ls -l /etc/nginx/sites-enabled/`

Si los archivos `pedro` o `perfectweb` no están enlazados, crea los enlaces simbólicos:

```
sudo ln -s /etc/nginx/sites-available/pedro /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/perfectweb /etc/nginx/sites-enabled/
```

2. Comprueba el estado de Nginx:

```
sudo systemctl status nginx
```

3. Valida la configuración de Nginx:

```
sudo nginx -t

```

## Errores en los permisos:

1. Configura los permisos del directorio del sitio web:

```
sudo chown -R www-data:www-data /var/www/pedro/html
sudo chmod -R 755 /var/www/pedro/html
```

2. Verifica el archivo `index.html`: Si el archivo `index.html` no existe, crea uno básico:

```
echo "<h1>Hello, World!</h1>" > /var/www/pedro/html/index.html
```

## Problemas de red:

Modifica el archivo hosts en la máquina anfitriona: Asegúrate de que el dominio pedro esté asignado a la IP 192.168.57.103.

En Windows:

```
C:\Windows\System32\drivers\etc\hosts
```

Prueba la conectividad:

```
ping 192.168.57.103
```

Para verificar los logs de acceso y errores, revisa los siguientes archivos en la máquina virtual:

Log de accesos: `/var/log/nginx/access.log`

```
192.168.57.1 - - [22/Nov/2024:21:51:13 +0000] "GET / HTTP/1.1" 401 581 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:51:17 +0000] "GET / HTTP/1.1" 403 186 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:51:18 +0000] "GET /favicon.ico HTTP/1.1" 403 186 "http://pedro/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:37 +0000] "GET / HTTP/1.1" 200 5428 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:37 +0000] "GET /css/bootstrap.min.css HTTP/1.1" 200 140421 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:37 +0000] "GET /css/pogo-slider.min.css HTTP/1.1" 200 41279 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:37 +0000] "GET /css/style.css HTTP/1.1" 200 40138 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:37 +0000] "GET /css/responsive.css HTTP/1.1" 200 8069 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:37 +0000] "GET /css/custom.css HTTP/1.1" 200 35 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:37 +0000] "GET /images/loader.gif HTTP/1.1" 200 44094 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:37 +0000] "GET /images/logo.png HTTP/1.1" 200 12631 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/search_icon.png HTTP/1.1" 200 1584 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/i1.png HTTP/1.1" 200 10376 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/popper.min.js HTTP/1.1" 200 20495 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/bootstrap.min.js HTTP/1.1" 200 50676 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/i2.png HTTP/1.1" 200 10105 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/jquery.min.js HTTP/1.1" 200 86659 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /css/animate.css HTTP/1.1" 200 56693 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /css/font-awesome.min.css HTTP/1.1" 200 31000 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /css/timeline.css HTTP/1.1" 200 75853 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /css/flaticon.css HTTP/1.1" 200 908 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /css/magnific-popup.css HTTP/1.1" 200 7782 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /css/responsiveslides.css HTTP/1.1" 200 490 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/jquery.pogo-slider.min.js HTTP/1.1" 200 27645 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/jquery.magnific-popup.min.js HTTP/1.1" 200 20216 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/smoothscroll.js HTTP/1.1" 200 7217 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/slider-index.js HTTP/1.1" 200 381 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/form-validator.min.js HTTP/1.1" 200 6055 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/contact-form-script.js HTTP/1.1" 200 1600 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/isotope.min.js HTTP/1.1" 200 35324 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/images-loded.min.js HTTP/1.1" 200 5565 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/i3.png HTTP/1.1" 200 7851 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /js/custom.js HTTP/1.1" 200 2964 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/i4.png HTTP/1.1" 200 23168 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/i7.png HTTP/1.1" 200 1502 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/p2.png HTTP/1.1" 200 216828 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/i5.png HTTP/1.1" 200 1366 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/p3.png HTTP/1.1" 200 165555 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/img6.png HTTP/1.1" 200 852475 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/i6.png HTTP/1.1" 200 1301 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/img2.png HTTP/1.1" 200 424189 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/p1.png HTTP/1.1" 200 243562 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/img10.png HTTP/1.1" 200 847972 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/img7.png HTTP/1.1" 200 1081715 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/heading_main_border.png HTTP/1.1" 200 1148 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/img8.png HTTP/1.1" 200 340121 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/footer_logo.png HTTP/1.1" 200 5737 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/img9.png HTTP/1.1" 200 264861 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /images/banner_img.png HTTP/1.1" 200 1638220 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:51:38 +0000] "GET /fonts/fontawesome-webfont.woff2?v=4.7.0 HTTP/1.1" 200 77160 "http://perfectweb/css/font-awesome.min.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:51:39 +0000] "GET /images/gb.png HTTP/1.1" 401 581 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:51:39 +0000] "GET /images/wave.svg HTTP/1.1" 401 581 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:51:39 +0000] "GET /images/slider-01.jpg HTTP/1.1" 401 581 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/bootstrap.min.css HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/pogo-slider.min.css HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/style.css HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/custom.css HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/responsive.css HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/loader.gif HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/logo.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/search_icon.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/i1.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/i2.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/jquery.min.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/popper.min.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/bootstrap.min.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/jquery.magnific-popup.min.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/animate.css HTTP/1.1" 304 0 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/font-awesome.min.css HTTP/1.1" 304 0 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/magnific-popup.css HTTP/1.1" 304 0 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/responsiveslides.css HTTP/1.1" 304 0 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/timeline.css HTTP/1.1" 304 0 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /css/flaticon.css HTTP/1.1" 304 0 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/jquery.pogo-slider.min.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/slider-index.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/contact-form-script.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/isotope.min.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/smoothscroll.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/form-validator.min.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/images-loded.min.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /js/custom.js HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/i3.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/i4.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/i5.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/i6.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/i7.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/img2.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/p1.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/p3.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/p2.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/img6.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/img7.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/img8.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/img9.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/img10.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/footer_logo.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:17 +0000] "GET /images/banner_img.png HTTP/1.1" 304 0 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:18 +0000] "GET /images/heading_main_border.png HTTP/1.1" 304 0 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:21:53:18 +0000] "GET /fonts/fontawesome-webfont.woff2?v=4.7.0 HTTP/1.1" 304 0 "http://perfectweb/css/font-awesome.min.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:53:19 +0000] "GET /images/gb.png HTTP/1.1" 401 581 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:53:19 +0000] "GET /images/wave.svg HTTP/1.1" 401 581 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:53:19 +0000] "GET /images/slider-01.jpg HTTP/1.1" 401 581 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:54:22 +0000] "GET / HTTP/1.1" 401 581 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET / HTTP/1.1" 200 5428 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/bootstrap.min.css HTTP/1.1" 200 140421 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/pogo-slider.min.css HTTP/1.1" 200 41279 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/style.css HTTP/1.1" 200 40138 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/responsive.css HTTP/1.1" 200 8069 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/custom.css HTTP/1.1" 200 35 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/loader.gif HTTP/1.1" 200 44094 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/logo.png HTTP/1.1" 200 12631 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/search_icon.png HTTP/1.1" 200 1584 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/i1.png HTTP/1.1" 200 10376 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/i2.png HTTP/1.1" 200 10105 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/jquery.min.js HTTP/1.1" 200 86659 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/popper.min.js HTTP/1.1" 200 20495 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/bootstrap.min.js HTTP/1.1" 200 50676 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/jquery.magnific-popup.min.js HTTP/1.1" 200 20216 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/jquery.pogo-slider.min.js HTTP/1.1" 200 27645 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/slider-index.js HTTP/1.1" 200 381 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/smoothscroll.js HTTP/1.1" 200 7217 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/form-validator.min.js HTTP/1.1" 200 6055 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/contact-form-script.js HTTP/1.1" 200 1600 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/isotope.min.js HTTP/1.1" 200 35324 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/images-loded.min.js HTTP/1.1" 200 5565 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /js/custom.js HTTP/1.1" 200 2964 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/animate.css HTTP/1.1" 200 56693 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/font-awesome.min.css HTTP/1.1" 200 31000 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/magnific-popup.css HTTP/1.1" 200 7782 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/responsiveslides.css HTTP/1.1" 200 490 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/timeline.css HTTP/1.1" 200 75853 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /css/flaticon.css HTTP/1.1" 200 908 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/i3.png HTTP/1.1" 200 7851 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/i4.png HTTP/1.1" 200 23168 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/i5.png HTTP/1.1" 200 1366 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/i6.png HTTP/1.1" 200 1301 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:29 +0000] "GET /images/i7.png HTTP/1.1" 200 1502 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/img2.png HTTP/1.1" 200 424189 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/p1.png HTTP/1.1" 200 243562 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/p2.png HTTP/1.1" 200 216828 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/p3.png HTTP/1.1" 200 165555 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/img9.png HTTP/1.1" 200 264861 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/img8.png HTTP/1.1" 200 340121 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/img6.png HTTP/1.1" 200 852475 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/img7.png HTTP/1.1" 200 1081715 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/footer_logo.png HTTP/1.1" 200 5737 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/img10.png HTTP/1.1" 200 847972 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/banner_img.png HTTP/1.1" 200 1638220 "http://perfectweb/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /images/heading_main_border.png HTTP/1.1" 200 1148 "http://perfectweb/css/style.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - gallegos [22/Nov/2024:21:54:30 +0000] "GET /fonts/fontawesome-webfont.woff2?v=4.7.0 HTTP/1.1" 200 77160 "http://perfectweb/css/font-awesome.min.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:54:42 +0000] "GET / HTTP/1.1" 403 186 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - - [22/Nov/2024:21:54:43 +0000] "GET /favicon.ico HTTP/1.1" 403 186 "http://pedro/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
```

Log de errores: `/var/log/nginx/error.log`

```
2024/11/22 21:48:21 [notice] 1738#1738: using inherited sockets from "5;6;"
2024/11/22 21:51:17 [error] 1774#1774: *3 access forbidden by rule, client: 192.168.57.1, server: pedro, request: "GET / HTTP/1.1", host: "pedro"
2024/11/22 21:51:18 [error] 1774#1774: *3 access forbidden by rule, client: 192.168.57.1, server: pedro, request: "GET /favicon.ico HTTP/1.1", host: "pedro", referrer: "http://pedro/"
2024/11/22 21:54:42 [error] 1775#1775: *26 access forbidden by rule, client: 192.168.57.1, server: pedro, request: "GET / HTTP/1.1", host: "pedro"
2024/11/22 21:54:43 [error] 1775#1775: *26 access forbidden by rule, client: 192.168.57.1, server: pedro, request: "GET /favicon.ico HTTP/1.1", host: "pedro", referrer: "http://pedro/"
```

## Error al intentar acceder a la página de 'pedro'

![Error](img/forbidden.png)

## Acceso a la página de 'perfectweb'

![Access](img/concedido.png)

# 👤 Autor

Pedro Rodríguez Gallegos
