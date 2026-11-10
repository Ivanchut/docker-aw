# Pruebas de funcionamiento y rendimiento

| Prueba | Resultado | ¿Correcto? |
|---|---|---|
| Servicios en marcha | 3 servicios en marcha (`db`, `nginx`, `wordpress`), todos en estado *Up* (salida completa más abajo) | sí |
| Sitio y panel | Se ve el tema en `/` y se puede entrar en `/wp-admin` | sí |
| Subida de ficheros | Imagen de varios MB subida sin error | sí |
| Enlaces permanentes | La entrada carga por su URL, sin 404 | sí |
| Persistencia (down/up) | Tras `docker compose down` y `up -d` el contenido sigue ahí | sí |
| Seguridad (403) | `curl -I http://localhost:8080/xmlrpc.php` devuelve `403 Forbidden` | sí |

Salida de `docker compose ps`:

```
NAME                    IMAGE                        COMMAND                  SERVICE     CREATED       STATUS          PORTS
docker-aw-db-1          mariadb:11                   "docker-entrypoint.s…"   db          2 weeks ago   Up 15 seconds   3306/tcp
docker-aw-nginx-1       nginx:stable                 "/docker-entrypoint.…"   nginx       2 weeks ago   Up 15 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
docker-aw-wordpress-1   wordpress:7.1.2-php8.3-fpm   "docker-entrypoint.s…"   wordpress   4 days ago    Up 15 seconds   9000/tcp
```

## Rendimiento

- Tiempos de respuesta en el aula (5 medidas):

```
0.503645s
0.037587s
0.030920s
0.031021s
0.031262s
```

- Memoria por contenedor (`docker stats --no-stream`):

```
CONTAINER ID   NAME                    CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O         PIDS
539ae2f5a4bc   docker-aw-wordpress-1   0.00%     55.47MiB / 15.32GiB   0.35%     474kB / 426kB    61.3MB / 0B       3
d47993e440a6   docker-aw-nginx-1       0.00%     10.95MiB / 15.32GiB   0.07%     363kB / 361kB    1.72MB / 0B       13
75c35a4d2f41   docker-aw-db-1          0.02%     125.2MiB / 15.32GiB   0.80%     78.4kB / 462kB   34.3MB / 1.02MB   10
```

- Interpretación:
  - **Primera petición vs. siguientes:** la primera tarda unos 0,50 s y las demás unos 0,03 s (unas 16 veces menos). La primera petición paga el "arranque en frío": PHP-FPM carga y compila los scripts de WordPress (aún no están en la caché de opcode), se abre la primera conexión a MariaDB y se calientan las cachés de la base de datos y del sistema de ficheros. Después todo está en memoria y el tiempo se estabiliza en torno a 31 ms.
  - **Memoria:** el stack completo consume unos 191 MiB (55,5 + 11,0 + 125,2), apenas un 1,2 % de los 15,32 GiB disponibles. El contenedor que más usa es la base de datos (MariaDB, por su buffer pool y cachés), después WordPress/PHP-FPM y, por último, Nginx, que es muy ligero.
  - **CPU:** prácticamente 0 % en los tres contenedores en reposo, así que el sistema tiene mucho margen para este sitio.

## Publicación

- Dirección desde la que accedió mi compañero/a: http://10.20.13.108:8080
- Dirección pública en Codespaces (paso 4, si lo hice): _PENDIENTE (ver comandos abajo)_
- Tiempos desde Codespaces y comparación con el aula: _PENDIENTE (ver comandos abajo)_
