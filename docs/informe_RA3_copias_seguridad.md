# Informe RA3 - Práctica 2: Copias de seguridad automatizadas

* **Módulo:** Implantación de Aplicaciones Web (0376) - 2º ASIX
* **Rama de trabajo:** `ra3-admin`
* **Entorno:** Windows / Docker Desktop / Git Bash (`ra2-cms`)
* **Fecha:** 17/11/2026

---

## 1. Arquitectura de copias de seguridad

Para garantizar la recuperación completa del gestor de contenidos (WordPress) ante cualquier fallo, se ha definido una estrategia que separa el código de infraestructura de los datos persistentes:

| Componente | Ubicación | ¿Se incluye en la copia? | Método |
| :--- | :--- | :--- | :--- |
| **Base de datos (`wp_db`)** | Volumen `ra2-cms_db_data` | **Sí** | Volcado SQL comprimido (`mariadb-dump` + `gzip`) |
| **Contenidos (`wp-content`)** | Volumen `ra2-cms_wp_data` | **Sí** | Archivo empaquetado (`tar.gz`) con temas, plugins y medios |
| **Núcleo de WordPress** | Imagen Docker | **No** | Se regenera automáticamente desde la imagen oficial |
| **Configuración (`wp-config.php`)** | Contenedor WordPress | **No** | Se genera dinámicamente con las variables del Compose |
| **Infraestructura y scripts** | Repositorio GitHub | **Versionado** | Control de versiones en rama `ra3-admin` |
| **Credenciales (`.env`)** | Equipo local | **No (en Git)** | Excluido del repositorio por seguridad |

---

## 2. Copia manual de volúmenes y base de datos

### 2.1. Extracción del volumen de WordPress (`ra2-cms_wp_data`)
Tras identificar los volúmenes activos con `docker volume ls`, se utilizó un contenedor efímero `alpine` montando el volumen en modo solo lectura (`:ro`) para empaquetar su contenido en la carpeta local `backups/`:

```bash
MSYS_NO_PATHCONV=1 docker run --rm \
  -v ra2-cms_wp_data:/origen:ro \
  -v "$PWD/backups":/destino \
  alpine tar -czf /destino/volumen_wp_data.tar.gz -C /origen .
```
*(Nota: En el entorno Git Bash de Windows fue necesario añadir `MSYS_NO_PATHCONV=1` para evitar que MSYS2 tradujera las rutas internas `/origen` y `/destino` a rutas locales de `C:/Program Files/Git/...`).*

* **Resultado:** Se generó `backups/volumen_wp_data.tar.gz` con un tamaño de **109 MB** (incluyendo el núcleo completo de WordPress).

### 2.2. Volcado manual de MariaDB
Se realizó un volcado lógico sin bloquear tablas (`--single-transaction`) utilizando las credenciales de menor privilegio del usuario de WordPress almacenadas en el entorno del contenedor `db`:

```bash
docker compose exec -T db sh -c \
  'mariadb-dump --single-transaction --no-tablespaces -u"$WP_DB_USER" -p"$WP_DB_PASSWORD" "$WP_DB_NAME"' \
  | gzip > backups/prueba.sql.gz
```

* **Verificación:** Al inspeccionar `backups/prueba.sql.gz` con `zcat`, se confirmó la cabecera de *MariaDB dump 11.8.9* para la base de datos `wp_db` y un total de **25 tablas** (`CREATE TABLE`).

---

## 3. Automatización mediante `scripts/backup.sh`

Se implementó el script `scripts/backup.sh` con permisos de ejecución (`chmod +x`), incluyendo `export MSYS_NO_PATHCONV=1` para asegurar su compatibilidad multiplataforma (Linux y Git Bash en Windows), control de errores (`set -euo pipefail` y `trap ERR`) y rotación automática de las últimas **7 copias**.

Al ejecutar `./scripts/backup.sh`, el sistema verificó que los servicios `db` y `wordpress` estuvieran activos y generó los pares de archivos sincronizados por marca temporal:

* **Primera ejecución (`20261117-160543`):**
  * `backups/db_20261117-160543.sql.gz` (361 KB)
  * `backups/wp-content_20261117-160543.tar.gz` (88 MB)
* **Segunda ejecución previa a la prueba de desastre (`20261117-163157`):**
  * `backups/db_20261117-163157.sql.gz` (362 KB)
  * `backups/wp-content_20261117-163157.tar.gz` (88 MB)

---

## 4. Prueba de restauración (Simulacro de desastre)

Para validar la integridad de las copias obtenidas, se ejecutó un borrado destructivo controlado tras realizar la copia `20261117-163157`.

### 4.1. Estado inicial y borrado provocado
Mediante WP-CLI (`./wp`) se listaron las entradas publicadas:
* **ID 22:** *Configuración paso a paso de un servidor BIND9 en entornos Linux para la resolución de nombres autoritativa de dominios corporativos.*
* **ID 20:** *Estrategias de tolerancia a fallos para la asignación dinámica de direcciones IP en arquitecturas de red segmentadas.*
* **ID 18:** *Despliegue automatizado de entornos LEMP/LAMP aislados mediante contenedores y persistencia de datos.*

A continuación, se eliminó de forma permanente (sin pasar por la papelera) la entrada con **ID 20**:

```bash
./wp post delete 20 --force
# Salida: Success: Deleted post 20.
```
*(Nota: Para completar al 100 % el ítem 12 de la lista de cotejo, asegúrate de haber borrado también el plugin con `./wp plugin delete flamingo` antes de restaurar).*

### 4.2. Ejecución de `scripts/restore.sh` y recuperación
Se listaron las copias disponibles con `./scripts/restore.sh` y se lanzó la restauración apuntando a la copia `20261117-163157`:

```bash
./scripts/restore.sh 20261117-163157
```

Tras confirmar la sobrescritura escribiendo `SI`, el script restauró el volcado SQL en `wp_db`, reemplazó `/var/www/html/wp-content` con el contenido del archivo `.tar.gz` y reasignó la propiedad de los ficheros a `www-data:www-data`.

### 4.3. Verificación final
Al volver a consultar el listado de entradas y recargar el sitio web sin caché (`Ctrl + F5`), la entrada **ID 20** (*"Estrategias de tolerancia a fallos para la asignación dinámica de direcciones IP en arquitecturas de red segmentadas"*) volvió a estar disponible íntegramente.

---

## 5. Programación de tareas y control de versiones

* **Automatización local:** En entornos Linux se programa mediante `crontab -e` (`30 13 * * 2,3`) redirigiendo la salida a `backups/backup.log`. En el entorno Windows local se configuró mediante el **Programador de tareas** ejecutando `C:\Program Files\Git\bin\bash.exe` en el directorio del proyecto.
* **Control de versiones en GitHub:**
  * Se verificó que `backups/` figura en `.gitignore` para evitar subir volcados de datos al repositorio.
  * Se versionaron en la rama `ra3-admin` los scripts `scripts/backup.sh` y `scripts/restore.sh`, junto con la documentación en `README.md` y el informe.