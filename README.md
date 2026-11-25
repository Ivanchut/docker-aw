# Memoria · RA3 · Práctica 5: Actualizaciones controladas e informes de acceso



---

## Índice

1. [Inventario de versiones](#paso-1--inventario-qué-versiones-tienes)
2. [Copia de seguridad y rama de trabajo](#paso-2--antes-de-tocar-nada-copia-y-commit)
3. [Capa 1: plugins](#paso-3--capa-1-plugins)
4. [Capa 2: imagen (PHP 8.3 → 8.4)](#paso-4--capa-2-la-imagen-php-83--84)
5. [Capa 3: núcleo de WordPress](#paso-5--capa-3-el-núcleo-de-wordpress)
6. [Verificación, fusión o vuelta atrás](#paso-6--verificar-fusionar-o-volver-atrás)
7. [Generación de tráfico](#paso-7--generar-tráfico-para-el-informe)
8. [Lectura de una línea del log](#paso-8--leer-una-línea-del-log)
9. [Informe de accesos](#paso-9--el-informe-scriptsinforme-accessh)
10. [Simple History](#paso-10--el-otro-registro-simple-history)
11. [Cierre: informe y pull request](#paso-11--cierre-del-bloque-informe-y-pull-request)
12. [Conclusiones](#conclusiones)

> **Cómo usar esta plantilla:** debajo de cada orden hay un bloque vacío para pegar la salida real. Donde veas
> `_(…)_` o una pregunta, escribe tu respuesta con tus palabras. Para las capturas, guarda las imágenes en
> `docs/img/` y enlázalas con `![descripción](docs/img/nombre.png)`.

---

## Paso 1 · Inventario: qué versiones tienes

**Objetivo:** conocer las versiones de cada capa antes de actualizar (imágenes, PHP, núcleo, plugins y temas).

### 1.1 Imágenes del stack

```bash
docker compose images
```

```text
CONTAINER               REPOSITORY          TAG                 PLATFORM            IMAGE ID            SIZE                CREATED
docker-aw-db-1          mariadb             11                  linux/amd64         79d59758afc9        450MB               2 months ago
docker-aw-nginx-1       nginx               stable              linux/amd64         9bf97bd7714f        239MB               7 weeks ago
docker-aw-wordpress-1   wordpress           7.1.2-php8.3-fpm    linux/amd64         ce765b661397        1.1GB               7 weeks ago
```

### 1.2 Versión de PHP

```bash
docker compose exec wordpress php -v | head -1
```

```text
PHP 8.3.35 (cli) (built: Oct  6 2026 01:28:43) (NTS)
```

### 1.3 Versión del núcleo de WordPress

```bash
./wp core version --extra
./wp core check-update
```

```text
(pega aquí la salida)
```

### 1.4 Plugins y temas

```bash
./wp plugin list --fields=name,version,update,update_version
./wp theme list --fields=name,version,update
```

```text
(pega aquí la salida)
```

**Comprobación:** PHP 8.3.x, WordPress 7.1.x y columna `update` con `none` o `available`. ☐ Correcto

> 📌 Copia estas versiones en la tabla «Versión antes» del [resumen de versiones](#resumen-de-versiones-sección-7-del-informe).

---

## Paso 2 · Antes de tocar nada: copia y commit

**Procedimiento de actualización controlada:**
1) inventario → 2) copia → 3) rama → 4) actualizar una capa cada vez → 5) verificar → 6) fusionar o volver atrás.

```bash
./scripts/backup.sh
```

```text
(pega aquí la salida)
```

```bash
git status
```

```text
(pega aquí la salida; debe estar limpio)
```

```bash
git switch -c ra3-actualizacion
```

```text
(pega aquí la salida)
```

```bash
ls -lh backups/
```

```text
(pega aquí la salida; debe aparecer la copia nueva)
```

**Nombre de la copia creada hoy:** `AAAAMMDD-HHMMSS` → _______________

**Pregunta:** ¿Por qué no se debe actualizar sin una copia hecha ese mismo día?

_(Tu respuesta)_

---

## Paso 3 · Capa 1: plugins

### 3.1 Instalar una versión anterior de Contact Form 7

> Sustituye `?.?.?` por la versión anterior a la última estable.

```bash
./wp plugin install contact-form-7 --version=?.?.? --force
```

```text
(pega aquí la salida)
```

```bash
./wp plugin list --name=contact-form-7 --fields=name,version,update_version
```

```text
(pega aquí la salida; update_version debe mostrar la versión actual)
```

### 3.2 Simulación de la actualización

```bash
./wp plugin update --all --dry-run
```

```text
(pega aquí la salida)
```

### 3.3 Actualización real

```bash
./wp plugin update --all
```

```text
(pega aquí la salida; tabla con old_version, new_version y Updated)
```

### 3.4 Verificación del formulario de Contacto

- ☐ El formulario de Contacto se muestra correctamente.
- ☐ He enviado un mensaje de prueba.
- ☐ El mensaje aparece guardado en Flamingo.

![Captura del mensaje en Flamingo](docs/img/paso3-flamingo.png)

**Pregunta:** ¿Qué diferencia hay entre `--dry-run` y la actualización normal? ¿Por qué es útil?

_(Tu respuesta)_

---

## Paso 4 · Capa 2: la imagen (PHP 8.3 → 8.4)

### 4.1 Cambios en `docker-compose.yml`

| Servicio | Antes | Después |
|---|---|---|
| wordpress | `wordpress:7.1-php8.3-fpm` | `wordpress:7.1-php8.4-fpm` |
| wp-cli | `wordpress:cli-php8.3` | `wordpress:cli-php8.4` |

```bash
git diff docker-compose.yml
```

```diff
(pega aquí el diff)
```

### 4.2 Descarga y recreación

```bash
docker compose pull wordpress
```

```text
(pega aquí la salida)
```

```bash
docker compose up -d
```

```text
(pega aquí la salida)
```

### 4.3 Comprobación de versiones

```bash
docker compose exec wordpress php -v | head -1
./wp core version
```

```text
(pega aquí la salida: PHP 8.4.x, pero WordPress con la MISMA versión que antes)
```

### 4.4 Explicación (obligatoria)

**Pregunta:** ¿Por qué, tras cambiar la imagen a PHP 8.4, `./wp core version` muestra la misma versión de WordPress que antes?
Explícalo con tus palabras.

_(Tu respuesta. Pista: ¿qué hace la imagen con los ficheros de WordPress y cuándo? ¿Dónde viven esos ficheros ahora?)_

---

## Paso 5 · Capa 3: el núcleo de WordPress

### 5.1 Búsqueda de actualizaciones

```bash
./wp core check-update
```

```text
(pega aquí la salida)
```

### 5.2 Actualización de mantenimiento (solo si hay una 7.1.x nueva)

```bash
./wp core update --minor
```

```text
(pega aquí la salida)
```

### 5.3 Actualización de la base de datos

```bash
./wp core update-db
```

```text
(pega aquí la salida)
```

### 5.4 Verificación de integridad

```bash
./wp core verify-checksums
```

```text
(pega aquí la salida; esperado: Success: WordPress installation verifies against checksums.)
```

**Preguntas:**

1. ¿Qué limita `--minor` y por qué una versión mayor (7.2) se planifica aparte?

   _(Tu respuesta)_

2. Si `verify-checksums` indicase «doesn't verify against checksum» en un servidor real, ¿qué significaría?

   _(Tu respuesta)_

---

## Paso 6 · Verificar, fusionar o volver atrás

### 6.1 Recorrido de verificación

- ☐ Portada
- ☐ Una entrada
- ☐ Página Contacto (mensaje enviado)
- ☐ Panel de administración
- ☐ Feed (`curl -s localhost:8080/feed/ | head`)
- ☐ Revisado `logs/error.log`

```bash
tail -20 logs/error.log
```

```text
(pega aquí la salida)
```

### 6.2 Resultado

☐ Todo funciona → fusionar  ☐ Algo falla → volver atrás

**Si todo va bien:**

```bash
git commit -am "RA3: PHP 8.4 y actualizaciones"
git switch ra3-admin && git merge ra3-actualizacion && git push
```

```text
(pega aquí la salida)
```

```bash
git log --oneline | head
```

```text
(pega aquí la salida; debe verse el commit de la actualización en ra3-admin)
```

**Si algo falla (rellenar solo si ocurre):**

```bash
git restore docker-compose.yml && docker compose up -d
./scripts/restore.sh AAAAMMDD-HHMMSS
```

```text
(pega aquí la salida)
```

**Problema encontrado y cómo lo solucioné:** _(o «Sin incidencias»)_

---

## Resumen de versiones (sección 7 del informe)

| Capa | Componente | Versión antes | Versión después | Método de actualización |
|---|---|---|---|---|
| Imagen | PHP | 8.3.x | 8.4.x | Cambio de etiqueta en `docker-compose.yml` |
| Imagen | nginx | | | |
| Imagen | MariaDB | | | |
| Núcleo | WordPress | 7.1.x | | `wp core update --minor` |
| Plugin | Contact Form 7 | | | `wp plugin update --all` |
| Plugin | (otros) | | | |
| Tema | (tema activo) | | | |

---

## Paso 7 · Generar tráfico para el informe

### 7.1 Tráfico de la pareja

- Dirección desde la que accedió mi pareja: _______________
- Intentos de acceso fallidos realizados (mínimo 3): _____

### 7.2 Tráfico generado con curl

```bash
for i in $(seq 1 30); do curl -s -o /dev/null "http://localhost:8080/?p=$i"; done
curl -s -o /dev/null http://localhost:8080/no-existe
```

```bash
tail -5 logs/access.log
```

```text
(pega aquí la salida)
```

> Si `logs/access.log` está vacío: `docker compose up -d --force-recreate nginx`.

---

## Paso 8 · Leer una línea del log

### 8.1 Formato de log de nginx

```bash
docker compose exec nginx grep -A3 log_format /etc/nginx/nginx.conf
```

```text
(pega aquí la salida)
```

### 8.2 Análisis de una línea real

Línea elegida de mi `logs/access.log`:

```text
(pega aquí una línea real)
```

| Campo | Valor en mi línea | Significado |
|---|---|---|
| `$remote_addr` | | IP que hace la petición |
| `[$time_local]` | | Fecha y hora |
| `"$request"` | | Método, URL y protocolo |
| `$status` | | Código de respuesta |
| `$body_bytes_sent` | | Bytes enviados |
| `"$http_referer"` | | Página de la que viene |
| `"$http_user_agent"` | | Navegador o programa |
| `"$http_x_forwarded_for"` | | IP real si hay proxy delante |

**Pregunta:** ¿Cómo distingues en el log un intento fallido de acceso de uno correcto en `/wp-login.php`?

_(Tu respuesta. Pista: método y código de estado.)_

---

## Paso 9 · El informe: `scripts/informe-accesos.sh`

### 9.1 Preparación

```bash
chmod +x scripts/informe-accesos.sh
ls -l scripts/informe-accesos.sh
```

```text
(pega aquí la salida)
```

### 9.2 Ejecución

```bash
./scripts/informe-accesos.sh
```

```text
(pega aquí la salida completa)
```

### 9.3 Resultados del resumen en terminal

| Dato | Valor |
|---|---|
| Peticiones totales | |
| Direcciones IP distintas | |
| Intentos `POST /wp-login.php` | |
| IP con más intentos de acceso | |

**Códigos de respuesta (resumen):**

| Código | Nº de peticiones | Interpretación |
|---|---|---|
| 200 | | |
| 301 / 302 | | |
| 404 | | |
| otros | | |

### 9.4 Informe HTML de GoAccess

- Fichero generado: `informes/accesos_AAAAMMDD-HHMM.html`

![Panel general de GoAccess](docs/img/paso9-goaccess-general.png)

![Visitantes / URL más pedidas](docs/img/paso9-goaccess-urls.png)

**Interpretación de los datos** (¿qué URL se piden más?, ¿hay 404?, ¿se ven los intentos fallidos?):

_(Tu análisis)_

### 9.5 Explicación del script

**Pregunta:** ¿Qué hacen `awk '{print $9}'`, `sort` y `uniq -c` encadenados en el script?

_(Tu respuesta)_

---

## Paso 10 · El otro registro: Simple History

### 10.1 Inicios de sesión (Escritorio → Historial → filtro «Inicios de sesión»)

![Intentos fallidos en Simple History](docs/img/paso10-logins.png)

- Usuarios que intentó mi pareja: _______________
- Número de intentos fallidos registrados: _____

### 10.2 Plugins (filtro «Plugins»)

![Actualización de plugin en Simple History](docs/img/paso10-plugins.png)

- Plugin: _______________  Versión antigua: _____  Versión nueva: _____

### 10.3 Comparativa nginx vs. Simple History

| | Log de nginx | Simple History |
|---|---|---|
| ¿Qué sabe? | | |
| ¿Qué NO sabe? | | |
| ¿Identifica al usuario de WordPress? | | |
| ¿Identifica la IP? | | |
| ¿Registra cambios (p. ej. versión de un plugin)? | | |

**Pregunta:** ¿Por qué un informe de accesos completo debe usar los dos registros?

_(Tu respuesta)_

---

## Paso 11 · Cierre del bloque: informe y pull request

```bash
git add INFORME_RA3.md scripts/ README.md
git commit -m "RA3: informe de administración"
git push
```

```text
(pega aquí la salida)
```

- ☐ Todas las secciones de `INFORME_RA3.md` completadas
- ☐ Pull request abierta de `ra3-admin` hacia `main`
- ☐ Título: «RA3 · Administración del gestor de contenidos»
- ☐ Descripción con enlace a `INFORME_RA3.md`
- ☐ `backups/`, `logs/`, `informes/` y `exportaciones/` **no** aparecen en los cambios
- ☐ Stack en marcha / codespace disponible para la revisión

**Enlace a la pull request:** _______________________

---

## Conclusiones

### Resumen de lo realizado

_(Qué has actualizado, en qué orden y por qué.)_

### Dificultades encontradas

_(Errores, soluciones aplicadas y qué harías distinto.)_

### Aprendizajes clave

- Actualizar una capa cada vez permite saber qué ha provocado un fallo.
- _(Añade los tuyos)_

---

## Anexo · Chuleta de órdenes del bloque

| Orden | Para qué |
|---|---|
| `./wp user list` | Usuarios |
| `./wp user create U correo --role=R` | Crear usuario |
| `./wp role list` · `./wp cap list R` | Roles y capacidades |
| `./wp plugin list` · `install X --activate` · `delete X` | Plugins |
| `./scripts/backup.sh` | Copia de seguridad |
| `./scripts/restore.sh [FECHA]` | Restaurar |
| `./wp export --stdout > f.xml` | Exportar WXR |
| `./wp import /datos/f.xml --authors=create` | Importar WXR |
| `./wp user import-csv /datos/f.csv` | Importar usuarios |
| `./wp theme list` · `./wp scaffold child-theme …` | Temas |
| `curl -s localhost:8080/feed/` | Ver el canal RSS |
| `./wp core check-update` · `update --minor` | Núcleo |
| `./wp core verify-checksums` | Integridad del núcleo |
| `tail -f logs/access.log` | Visitas en directo |
| `./scripts/informe-accesos.sh` | Informe de accesos |
