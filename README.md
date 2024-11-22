# DEAW-DUAL-Practica4

# Estructura del proyecto

## ✅ Comprobación de logs

Para verificar los logs de acceso y errores, revisa los siguientes archivos en la máquina virtual:

Log de accesos: `/var/log/nginx/access.log`

```
192.168.57.1 - pedro [22/Nov/2024:19:07:18 +0000] "GET / HTTP/1.1" 403 186 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:19:07:19 +0000] "GET /favicon.ico HTTP/1.1" 404 187 "http://pedro/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:19:09:19 +0000] "GET / HTTP/1.1" 403 186 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:19:09:21 +0000] "GET / HTTP/1.1" 403 186 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:19:09:24 +0000] "GET / HTTP/1.1" 403 186 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
192.168.57.1 - pedro [22/Nov/2024:19:09:35 +0000] "GET / HTTP/1.1" 403 186 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
```

Log de errores: `/var/log/nginx/error.log`

```
2024/11/22 19:06:38 [notice] 1745#1745: using inherited sockets from "5;6;"
2024/11/22 19:07:18 [error] 1777#1777: *1 directory index of "/var/www/practica2.2/html/" is forbidden, client: 192.168.57.1, server: pedro, request: "GET / HTTP/1.1", host: "pedro"
2024/11/22 19:09:19 [error] 426#426: *1 directory index of "/var/www/practica2.2/html/" is forbidden, client: 192.168.57.1, server: pedro, request: "GET / HTTP/1.1", host: "pedro"
2024/11/22 19:09:21 [error] 426#426: *1 directory index of "/var/www/practica2.2/html/" is forbidden, client: 192.168.57.1, server: pedro, request: "GET / HTTP/1.1", host: "pedro"
2024/11/22 19:09:24 [error] 426#426: *1 directory index of "/var/www/practica2.2/html/" is forbidden, client: 192.168.57.1, server: pedro, request: "GET / HTTP/1.1", host: "pedro"
2024/11/22 19:09:35 [error] 426#426: *1 directory index of "/var/www/practica2.2/html/" is forbidden, client: 192.168.57.1, server: pedro, request: "GET / HTTP/1.1", host: "pedro"
```
