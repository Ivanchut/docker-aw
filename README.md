# Copias de seguridad
**Alumno/a:** <nombre>
**Rama:** `ra3-admin`
**Fecha de realización:** <AAAA-MM-DD>
**Entorno:** <mi equipo>

---

## 1. Qué se copia y qué no

| Pieza | Dónde está | ¿Se copia? |
|---|---|---|
| Base de datos (entradas, usuarios, ajustes, roles) | Volumen `db_data` | Sí: volcado SQL (`db_*.sql.gz`) |
| `wp-content` (temas, plugins, imágenes subidas) | Volumen `wp_data` | Sí: `wp-content_*.tar.gz` |
| Núcleo de WordPress | Volumen `wp_data` | No: lo trae la imagen |
| `wp-config.php` | Volumen `wp_data` | No: lo genera el compose |
| `docker-compose.yml`, `nginx/`, `scripts/` | Repositorio | Ya está en GitHub |
| `.env` | Mi equipo | Aparte, nunca en GitHub |

Con el repositorio, el `.env`, un volcado de la base de datos y una copia de `wp-content` se reconstruye
el sitio entero en cualquier máquina con Docker.

---

## 3. Copia manual de un volumen (técnica general)

```bash
docker volume ls
```

```text
DRIVER    VOLUME NAME
local     b1df958f5cce49fbd1ce6f1d46092ee49c39c2bfb398bff6410b4c1aa3f2d5cc
local     cdff8803186c8aed749e58e8858ee5e5ea1b3b9ae252592ef6a69827a1b3a4ad
local     docker-aw_db_data
local     docker-aw_wp_data
local     docker-compose_db_data
local     docker-session10_db_data
local     docker-session10_wp_data
```

Nombre de mi volumen de WordPress: `docker-aw_wp_data`

```bash
mkdir -p backups
docker run --rm \
  -v <NOMBRE>_wp_data:/origen:ro \
  -v "$PWD/backups":/destino \
  alpine tar -czf /destino/volumen_wp_data.tar.gz -C /origen .
ls -lh backups/
```

```text
latest: Pulling from library/alpine
881fc932939e: Already exists 
c35a7d82b8fa: Download complete 
86ff264fe568: Download complete 
Digest: sha256:f59abe656b692d901e184be3445e084786602108599d2a51d229c0f5a48c71bf
Status: Downloaded newer image for alpine:latest
total 48M
-rw-r--r-- 1 root root 48M nov 17 16:38 volumen_wp_data.tar.gz
```

**Explicación con mis palabras:** <Un volumen no se puede leer directamente; se monta en un contenedor
alpine desechable (en solo lectura) junto con la carpeta backups y se empaqueta con tar.>

Tamaño obtenido de `volumen_wp_data.tar.gz`: <48 MB>

---

## 4. Volcado manual de la base de datos

```bash
docker compose exec -T db sh -c \
  'mariadb-dump --single-transaction --no-tablespaces -u"$WP_DB_USER" -p"$WP_DB_PASSWORD" "$WP_DB_NAME"' \
  | gzip > backups/prueba.sql.gz
zcat backups/prueba.sql.gz | head -40
zcat backups/prueba.sql.gz | grep -c 'CREATE TABLE'
```

Primeras líneas del volcado:

```text
/*M!999999\- enable the sandbox mode */ 
-- MariaDB dump 10.20-11.8.9-MariaDB, for debian-linux-gnu (x86_64)
--
-- Host: localhost    Database: wp_db
-- ------------------------------------------------------
-- Server version	11.8.9-MariaDB-ubu2404

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*M!100616 SET @OLD_NOTE_VERBOSITY=@@NOTE_VERBOSITY, NOTE_VERBOSITY=0 */;

--
-- Table structure for table `wp_cmplz_cookiebanners`
--

DROP TABLE IF EXISTS `wp_cmplz_cookiebanners`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8mb4 */;
CREATE TABLE `wp_cmplz_cookiebanners` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `title` text NOT NULL,
  `banner_version` int(11) NOT NULL,
  `default` int(11) NOT NULL,
  `position` text NOT NULL,
  `checkbox_style` text NOT NULL,
  `use_logo` text NOT NULL,
  `logo_attachment_id` text NOT NULL,
  `close_button` text NOT NULL,
  `revoke` text NOT NULL,
  `manage_consent_options` text NOT NULL,
  `header` text NOT NULL,
  `dismiss` text NOT NULL,
  `save_preferences` text NOT NULL,
18
```

Número de tablas (`CREATE TABLE`): <18>

**Explicación con mis palabras:** <mariadb-dump se ejecuta dentro del contenedor db, escribe SQL por la
salida estándar y gzip lo comprime en mi equipo. Uso el usuario de WordPress y no root (mínimo privilegio).>

Problemas encontrados y cómo los resolví: <ninguno / ...>

---

## 5. Script `scripts/backup.sh`

El script guarda en `backups/` dos ficheros con la misma marca de fecha y conserva solo las últimas 7 copias:

- `db_AAAAMMDD-HHMMSS.sql.gz`: volcado de la base de datos.
- `wp-content_AAAAMMDD-HHMMSS.tar.gz`: temas, plugins y medios subidos.

| Parte | Qué hace |
|---|---|
| `set -euo pipefail` | Detiene el script ante cualquier error, incluso dentro de una tubería |
| `cd "$(dirname "$0")/.."` | Se coloca en la raíz del repositorio, se ejecute desde donde se ejecute |
| `trap ... ERR` | Si algo falla, borra los ficheros incompletos |
| `gzip -t` y `grep -c 'CREATE TABLE'` | Verifica que la copia se descomprime y que el volcado tiene tablas |
| Rotación | Borra las copias que sobrepasan las 7 más recientes |

```bash
chmod +x ../scripts/backup.sh
../scripts/backup.sh
ls -lh backups/
```

```text
== Copia de seguridad 20261117-170034 ==
-> Volcando la base de datos...
-> Empaquetando wp-content...
OK. Copia guardada:
-rw-r--r-- 1 vanxu vanxu 257K nov 17 17:00 backups/db_20261117-170034.sql.gz
-rw-r--r-- 1 vanxu vanxu  27M nov 17 17:00 backups/wp-content_20261117-170034.tar.gz
Copias disponibles: 1 (se conservan 7)
```

- Fichero de base de datos generado: `<db_20261117-170034.sql.gz 257KB>`
- Fichero de wp-content generado: `<wp-content_20261117-170034.tar.gz 27MB>`
- ¿Misma marca de fecha en ambos?: <sí>

---

## 6. Prueba de restauración

### 6.1 Copia justo antes del desastre

```bash
./scripts/backup.sh
```

Marca de fecha de esta copia: `<nov 17 17:00>`

### 6.2 Estado antes del desastre

```bash
./wp post list --post_type=post --fields=ID,post_title
./wp plugin list
```

```text
[+] run 1/1
 ✔ Container docker-aw-db-1 Running                                                  0.0s
Container docker-aw-wpcli-run-482517c12147 Creating 
Container docker-aw-wpcli-run-482517c12147 Created 
ID	post_title
13	profe de redes
8	Mi primera semana en practicas
1	¡Hola, mundo!
[+] run 1/1
 ✔ Container docker-aw-db-1 Running                                                  0.0s
Container docker-aw-wpcli-run-79d200e2a334 Creating 
Container docker-aw-wpcli-run-79d200e2a334 Created 
name	status	update	version	update_version	auto_update
complianz-gdpr	active	none	7.5.7		off
contact-form-7	active	none	6.2.5		off
flamingo	active	none	2.6.4		off
limit-login-attempts-reloaded	active	available	3.3.10	3.4.3	off
simple-history	active	none	5.36.0		off
```

### 6.3 Desastre controlado

```bash
./wp post delete <ID> --force
./wp plugin delete flamingo
```

- Entrada borrada: <ID y título>
- Plugin borrado: flamingo

Comprobación de que se ha borrado:

```text
[+] run 1/1
 ✔ Container docker-aw-db-1 Running                                                                 0.0s
Container docker-aw-wpcli-run-9e728a07a62c Creating 
Container docker-aw-wpcli-run-9e728a07a62c Created 
Deleted 'flamingo' plugin.
Success: Deleted 1 of 1 plugins.
```

### 6.4 Restauración

```bash
./scripts/restore.sh                      # lista las copias disponibles
./scripts/restore.sh <AAAAMMDD-HHMMSS>    # restaura la copia indicada
```

```text
Uso: ./scripts/restore.sh FECHA
Copias disponibles:
20261117-170034
Se va a SOBRESCRIBIR la base de datos y wp-content con la copia 20261117-170034.
Escribe SI (en mayúsculas) para continuar: SI
-> Restaurando la base de datos...
-> Restaurando wp-content...
OK. Restaurada la copia 20261117-170034. Recarga el sitio en el navegador.
```

### 6.5 Comprobación tras restaurar

```bash
./wp post list --post_type=post --fields=ID,post_title
./wp plugin list
```

```text
[+] run 1/1
 ✔ Container docker-aw-db-1 Running                                     0.0s
Container docker-aw-wpcli-run-b3925dcda858 Creating 
Container docker-aw-wpcli-run-b3925dcda858 Created 
ID	post_title
13	profe de redes
8	Mi primera semana en practicas
1	¡Hola, mundo!
[+] run 1/1
 ✔ Container docker-aw-db-1 Running                                     0.0s
Container docker-aw-wpcli-run-664070416b11 Creating 
Container docker-aw-wpcli-run-664070416b11 Created 
name	status	update	version	update_version	auto_update
complianz-gdpr	active	none	7.5.7		off
contact-form-7	active	none	6.2.5		off
flamingo	active	none	2.6.4		off
limit-login-attempts-reloaded	active	available	3.3.10	3.4.3	off
simple-history	active	none	5.36.0		off
```

---

## 7. Automatización con cron

```bash
crontab -e
crontab -l
```

```text
PEGA AQUÍ LA SALIDA de crontab -l
```

Línea añadida (hace copia martes y miércoles a las 13:30):

```cron
30 13 * * 2,3 cd $HOME/<carpeta-del-repo> && ./scripts/backup.sh >> backups/backup.log 2>&1
```

Cron no carga el PATH completo ni se coloca en mi carpeta, por eso la línea empieza por `cd`.

> En Codespaces no se usa cron (la máquina se para). Allí se ejecuta `./scripts/backup.sh` al final de
> cada sesión y se descarga la carpeta `backups` de vez en cuando.

---

## 8. Versionado: el script sí, las copias no

```bash
git status
git add scripts/ .gitignore README.md
git commit -m "SESION15: script de copias y restauración"
git push
```

```text
PEGA AQUÍ LA SALIDA de git status (backups/ NO debe aparecer)
```

---

## 9. Guía rápida

### Hacer una copia

```bash
./scripts/backup.sh
```

Las copias quedan en `backups/` (se conservan las 7 últimas). La carpeta está en `.gitignore`.

### Restaurar una copia

```bash
./scripts/restore.sh                      # lista las copias
./scripts/restore.sh AAAAMMDD-HHMMSS      # restaura esa copia
```

Pide confirmación escribiendo `SI`, porque sobrescribe el estado actual (base de datos y `wp-content`).

### Reconstruir el sitio en otra máquina

1. Clonar el repositorio y cambiar a la rama `ra3-admin`.
2. Copiar el `.env` (se guarda aparte).
3. `docker compose up -d`
4. Copiar los ficheros de `backups/` a la nueva máquina y ejecutar `./scripts/restore.sh <marca>`.

### Problemas frecuentes

| Síntoma | Solución |
|---|---|
| `el servicio 'db' no está en marcha` | `docker compose up -d` |
| `Access denied for user` | La contraseña del `.env` no coincide con la del usuario en MariaDB |
| `falta WP_DB_USER en .env` | Revisar el `.env` (práctica 1, paso 3) |
| Tras restaurar el navegador no muestra los cambios | Recargar sin caché (Ctrl+F5) |
| Tablas creadas después de la copia siguen existiendo | Es normal: el volcado solo recrea las tablas que contiene |