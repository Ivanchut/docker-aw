# RA3 · Práctica 3 · Importar y exportar contenidos en distintos formatos

**Módulo:** Implantación de Aplicaciones Web (0376) · ASIR 2º
**Resultado de aprendizaje:** RA3 · Administración de gestores de contenidos (CE e)
**Rama:** `sesion16`

## Objetivo

Mover contenidos y usuarios entre sitios WordPress usando cuatro formatos (WXR, CSV, JSON y SQL) y saber cuándo conviene cada uno.

| Formato | Qué lleva | Para qué sirve |
|---|---|---|
| **WXR** (XML de WordPress) | Entradas, páginas, categorías, etiquetas, comentarios, autores | Pasar contenido de un WordPress a otro |
| **CSV** | Una tabla, una fila por elemento | Listados en hoja de cálculo e importación de usuarios |
| **JSON** (API REST) | Datos estructurados | Que otras aplicaciones lean o escriban en el sitio |
| **SQL** | La base de datos completa | Copias de seguridad y migraciones completas |

## Entorno

- Stack Docker Compose con tres contenedores: `db` (MariaDB 11), `wordpress` (WordPress 7.1.2, PHP 8.3-FPM) y `nginx` (puerto `8080`).
- WP-CLI mediante el atajo `./wp` (contenedor `wpcli`).
- Carpeta `datos/` para los ficheros de entrada y `exportaciones/` para las salidas (esta última está en `.gitignore`).

```bash
sudo docker-compose up -d
docker-compose ps
mkdir -p exportaciones
```

## Paso 2 · Exportar el sitio en WXR

```bash
./wp export --stdout > exportaciones/sitio_$(date +%Y%m%d).xml
grep -c '<item>' exportaciones/sitio_*.xml
grep -o '<wp:post_type>.*</wp:post_type>' exportaciones/sitio_*.xml | sort | uniq -c
```

**Resultado:** 18 elementos exportados.

| Tipo | Nº |
|---|---|
| `flamingo_contact` | 8 |
| `page` | 3 |
| `post` | 3 |
| `llar_digest_day` | 1 |
| `wpcf7_contact_form` | 1 |
| `wp_global_styles` | 1 |
| `wp_navigation` | 1 |

## Paso 3 · Importar contenidos de ejemplo (WXR)

Se copió `contenidos_asir.xml` a `datos/` (hizo falta `sudo cp` por permisos en la carpeta).

```bash
./wp plugin install wordpress-importer --activate
./wp import /datos/contenidos_asir.xml --authors=create
./wp post list --post_type=post,page --fields=ID,post_type,post_title,post_author
./wp term list category --fields=name,count
```

**Resultado:** importados 8 elementos (6 entradas y 2 páginas). Mensaje final: `Success: Finished importing from '/datos/contenidos_asir.xml' file.`

| ID | Tipo | Título |
|---|---|---|
| 101 | post | Jornada de puertas abiertas del ciclo ASIR |
| 102 | post | Charla: contenedores en producción |
| 103 | post | Nuevo armario de comunicaciones en el aula |
| 104 | post | Práctica: el stack de WordPress en contenedores |
| 105 | post | Empresas colaboradoras para la formación en empresa |
| 106 | post | Hackathon de ciberseguridad |
| 107 | page | Horario del ciclo |
| 108 | page | Normas del aula técnica |

Categorías tras la importación:

| Categoría | Entradas |
|---|---|
| Eventos | 3 |
| Noticias | 3 |
| Prácticas | 1 |
| Sin categoría | 3 |

## Paso 4 · Importar usuarios desde CSV

```bash
head -3 datos/usuarios.csv
./wp user import-csv /datos/usuarios.csv
./wp user list --fields=user_login,display_name,roles
```

**Resultado:**

- Actualizados (ya existían): `coord.web`, `profe.redes`, `profe.so`, `alumna.fct`, `familia.garcia`.
- Creados: `profe.bd`, `alumno.fct`, `exalumno.2025`.
- Total final: 11 usuarios (roles: administrator, editor, author, contributor, subscriber y `revisor`).

El CSV no incluye contraseñas: WordPress genera una aleatoria para cada usuario nuevo. Es lo correcto, porque nunca se deben pasar contraseñas en un fichero de texto.

## Paso 5 · Exportar listados en CSV y JSON

```bash
./wp post list --post_type=post --fields=ID,post_title,post_date,post_status --format=csv > exportaciones/entradas.csv
./wp user list --fields=user_login,user_email,roles --format=json > exportaciones/usuarios.json
cat exportaciones/entradas.csv
python3 -m json.tool exportaciones/usuarios.json | head -20
```

**Resultado:**

- `entradas.csv`: cabecera `ID,post_title,post_date,post_status` y 9 entradas, todas en estado `publish`.
- `usuarios.json`: un objeto por usuario con `user_login`, `user_email` y `roles`.

El mismo dato sale en distinto formato según quién lo vaya a usar (`--format=table|csv|json|yaml|count`).

## Paso 6 · JSON desde la API REST

```bash
curl -s "http://localhost:8080/wp-json/wp/v2/posts?per_page=3&_fields=id,title,link" | python3 -m json.tool
curl -s "http://localhost:8080/wp-json/wp/v2/users?_fields=id,name" | python3 -m json.tool
```

**Incidencia:** ambas órdenes devolvieron `Expecting value: line 1 column 1 (char 0)`, es decir, `/wp-json/` no respondía JSON porque los enlaces permanentes no están activos.

**Solución:** usar la ruta alternativa `?rest_route=`:

```bash
curl -s "http://localhost:8080/?rest_route=/wp/v2/posts" | python3 -m json.tool
```

Con ella la API devolvió las entradas en JSON (campos `id`, `title`, `content`, `author`, `categories`, `_links`…).

## Paso 7 · SQL: exportar una tabla y entender el riesgo

```bash
docker compose exec -T db sh -c \
 'mariadb-dump --no-tablespaces -u"$WP_DB_USER" -p"$WP_DB_PASSWORD" "$WP_DB_NAME" wp_users' \
 > exportaciones/wp_users.sql
grep -o "'\$wp\$[^']*'" exportaciones/wp_users.sql | head -3
```

**Resultado:** el volcado contiene las contraseñas cifradas con bcrypt (cadenas que empiezan por `$wp$2y$10$…`). No se pueden descifrar, pero sí atacar por fuerza bruta si alguien obtiene el fichero.

**Por qué `backups/` y `exportaciones/` están en `.gitignore`:** para que estos ficheros nunca se publiquen en el repositorio. Se comprobó con `git status`: ninguna de las dos carpetas aparece.

## Paso 8 · Intercambio con la pareja

> **Pendiente de documentar.** Este paso se hace desde el panel y no aparece en la salida de comandos.

1. Crear una entrada en la categoría **Prácticas** que explique en cinco líneas para qué sirve cada formato.
2. Exportarla desde *Herramientas → Exportar → Entradas → Categoría: Prácticas*.
3. Pasar el fichero a la pareja.
4. Importar el fichero de la pareja desde *Herramientas → Importar → WordPress*.

## Control de versiones

```bash
git add .
git commit -m "practica 3 importar e exportar"
git push origin sesion16
```

Commit `cb62f2d` en la rama `sesion16` con `datos/contenidos_asir.xml` y `datos/usuarios.csv`.

## Incidencias y soluciones

| Problema | Causa | Solución |
|---|---|---|
| `cp: Permiso denegado` al copiar a `datos/` | La carpeta pertenece a otro usuario (contenedor) | `sudo cp ...` |
| `Expecting value` con `/wp-json/` | Enlaces permanentes no activos | Usar `/?rest_route=/wp/v2/posts` |
| `git commit` sin cambios | Los ficheros no estaban añadidos al índice | `git add .` antes del commit |
| `git push`: `Connection timed out` (puerto 22) | Fallo de red puntual | Repetir el `push`, que funcionó al segundo intento |

## Conclusiones

- **WXR** es el formato adecuado para mover contenido (entradas, páginas, términos) entre WordPress.
- **CSV** es cómodo para usuarios y listados, pero solo representa una tabla.
- **JSON** permite integrar el sitio con otras aplicaciones y es de lectura pública para el contenido publicado.
- **SQL** es la copia más completa, y por eso la más delicada: contiene los hashes de contraseñas y debe protegerse y excluirse del repositorio.
