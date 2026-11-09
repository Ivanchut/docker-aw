# Pruebas de funcionamiento y rendimiento
| Prueba | Resultado | ¿Correcto? |
|---|---|---|
| Servicios en marcha | NAME                    IMAGE                        COMMAND                  SERVICE     CREATED       STATUS          PORTS
docker-aw-db-1          mariadb:11                   "docker-entrypoint.s…"   db          2 weeks ago   Up 15 seconds   3306/tcp
docker-aw-nginx-1       nginx:stable                 "/docker-entrypoint.…"   nginx       2 weeks ago   Up 15 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
docker-aw-wordpress-1   wordpress:7.1.2-php8.3-fpm   "docker-entrypoint.s…"   wordpress   4 days ago    Up 15 seconds   9000/tcp | si |
| Sitio y panel | se ve el tema | si |
| Subida de ficheros | sin error | si |
| Enlaces permanentes | carga | si |
| Persistencia (down/up) | el contenedor persiste | si |
| Seguridad (403) | forbidden 403 | si |
## Rendimiento
- Tiempos de respuesta en el aula (5 medidas):
curl -o /dev/null -s -w "%{time_total}s\n" http://localhost:8080/
done
docker stats --no-stream
0.503645s
0.037587s
0.030920s
0.031021s
0.031262s
CONTAINER ID   NAME                    CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O         PIDS
539ae2f5a4bc   docker-aw-wordpress-1   0.00%     56.61MiB / 15.32GiB   0.36%     474kB / 426kB    61.3MB / 0B       3
d47993e440a6   docker-aw-nginx-1       0.00%     11.93MiB / 15.32GiB   0.08%     363kB / 361kB    1.72MB / 0B       13
75c35a4d2f41   docker-aw-db-1          0.12%     126.7MiB / 15.32GiB   0.81%     78.3kB / 462kB   34.3MB / 1.02MB   12
- Memoria por contenedor (docker stats):
CONTAINER ID   NAME                    CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O         PIDS
539ae2f5a4bc   docker-aw-wordpress-1   0.00%     55.47MiB / 15.32GiB   0.35%     474kB / 426kB    61.3MB / 0B       3
d47993e440a6   docker-aw-nginx-1       0.00%     10.95MiB / 15.32GiB   0.07%     363kB / 361kB    1.72MB / 0B       13
75c35a4d2f41   docker-aw-db-1          0.02%     125.2MiB / 15.32GiB   0.80%     78.4kB / 462kB   34.3MB / 1.02MB   10

## Publicación
- Dirección desde la que accedió mi compañero/a:
- Dirección pública en Codespaces (paso 4, si lo hice):
- Tiempos desde Codespaces y comparación con el aula:
