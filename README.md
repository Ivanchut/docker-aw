## Paso 1 · Diagnóstico inicial («antes»)

[vanxu@archie docker-aw]$ curl -I http://localhost:8080/ | grep -i server
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0
Server: nginx/1.30.5


[vanxu@archie docker-aw]$ curl -I http://localhost:8080/xmlrpc.php
HTTP/1.1 302 Found
Server: nginx/1.30.5
Date: Wed, 21 Oct 2026 16:28:53 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/8.3.35
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0, no-store, private
X-Redirect-By: WordPress
Location: http://localhost:8080/wp-admin/install.php

[vanxu@archie docker-aw]$ curl -s http://localhost:8080/wp-json/wp/v2/users
[vanxu@archie docker-aw]$ 


# ¿aparece la versión de nginx? 
Si, aparece la cersion nginx/1.30.5
# ¿responde xmlrpc.php? 
Si
# ¿aparece el nombre de tu usuario en la lista de usuarios pública?

# demostración de por qué importa no ejecutar PHP en la carpeta de subidas.
[vanxu@archie wp_data]$ curl http://localhost:8080/wp-content/uploads/test.php
2[vanxu@archie wp_data]$ 

## Paso 2 · Endurecer nginx

# ustituye nginx/default.conf por esta versión. Los bloques de bloqueo van antes del bloque de PHP: nginx aplica el primero que coincide

# ✔ Verificación nginx -t responde syntax is ok y test is successful.
vanxu@archie docker-aw]$ docker compose exec nginx nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

# Paso 3 · Desactivar la edición de ficheros desde el panel

# Paso 4 · Medidas en WordPress y con los usuarios
 Crea un usuario nuevo con el rol Editor para el trabajo diario. El administrador se usa solo para administrar.

 Comprueba que el nombre público de tu usuario administrador es distinto de su nombre de acceso (Usuarios → Perfil → «Mostrar públicamente como»).

 Instala un plugin de limitación de intentos de acceso de confianza, actívalo y comprueba que, tras varios intentos fallidos en /wp-login.php, te bloquea temporalmente.

 Ejecuta las actualizaciones pendientes (Escritorio → Actualizaciones), si las hay.

## Paso 5 · Diagnóstico final («después»)
# curl -I http://localhost:8080/ | grep -i server
[vanxu@archie ~]$ curl -I http://localhost:8080/ | grep -i server
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0
Server: nginx
# curl -I http://localhost:8080/xmlrpc.php
[vanxu@archie ~]$ curl -I http://localhost:8080/xmlrpc.php
HTTP/1.1 403 Forbidden
Server: nginx
Date: Mon, 26 Oct 2026 23:39:20 GMT
Content-Type: text/html
Content-Length: 146
Connection: keep-alive
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
# curl -I http://localhost:8080/wp-config.php
[vanxu@archie ~]$ curl -I http://localhost:8080/wp-config.php
HTTP/1.1 403 Forbidden
Server: nginx
Date: Mon, 26 Oct 2026 23:39:45 GMT
Content-Type: text/html
Content-Length: 146
Connection: keep-alive
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
# curl -I http://localhost:8080/wp-content/uploads/test.php
[vanxu@archie ~]$ curl -I http://localhost:8080/wp-content/uploads/test.php
HTTP/1.1 403 Forbidden
Server: nginx
Date: Mon, 26 Oct 2026 23:40:05 GMT
Content-Type: text/html
Content-Length: 146
Connection: keep-alive
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN