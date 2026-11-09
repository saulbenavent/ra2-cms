# Proyecto CMS WordPress con Docker y Nginx (RA2)

Este repositorio contiene la configuración e infraestructura basada en **Docker Compose** para desplegar un sitio web gestionado con **WordPress**, optimizado mediante un servidor web **Nginx** como proxy inverso y respaldado por **MySQL/MariaDB**.

El entorno está diseñado para funcionar tanto en desarrollo local como en entorns cloud (GitHub Codespaces) sin necesidad de modificar variables de entorno de forma manual.

---

## 🛠️ Requisitos Previos

* [Docker Desktop](https://www.docker.com/) (versión 20.10 o superior)
* [Docker Compose](https://docs.docker.com/compose/) v2.x
* [Git](https://git-scm.com/)

---

## 🚀 Despliegue en Entorno Local

Sigue estos pasos para poner en marcha el proyecto desde cero:

### 1. Clonar el repositorio
```bash
git clone https://github.com/TU-USUARIO/TU-REPOSITO.git
cd TU-REPOSITO
```

### 2. Configurar variables de entorno
Copia el archivo de ejemplo y edita las credenciales según corresponda:
```bash
cp .env.example .env
code .env
```

### 3. Levantar la infraestructura
Ejecuta el stack de contenedores en segundo plano:
```bash
docker compose up -d
```

### 4. Verificación de servicios
Comprueba que los tres contenedores (`wordpress`, `db` y `nginx`) están en ejecución:
```bash
docker compose ps
```

Accede al sitio desde tu navegador web:
* **Sitio Web / Inicio:** `http://localhost:8080`
* **Panel de Administración:** `http://localhost:8080/wp-admin`

---

## ☁️ Despliegue en GitHub Codespaces

Este proyecto está optimizado para ejecutarse en GitHub Codespaces con HTTPS automático.

1. Abre el selector de ramas en GitHub y posicionate en `ra2-cms`.
2. Haz clic en **Code -> Codespaces -> Create codespace on ra2-cms**.
3. En la terminal integrada del Codespace, inicializa los contenedores:
   ```bash
   cp .env.example .env
   docker compose up -d
   ```
4. Ve a la pestaña **PORTS** en la barra inferior del editor.
5. Haz clic derecho sobre el puerto `8080` -> **Port Visibility** -> cámbialo a **Public**.
6. Haz clic en la **Forwarded Address** generada (`https://...-8080.app.github.dev`) para acceder a la aplicación.

---

## 📁 Estructura del Repositorio

```text
├── .env.example             # Plantilla de variables de entorno (DB, contraseñas)
├── docker-compose.yml       # Definición de servicios (Nginx, WordPress, MySQL)
├── nginx/
│   └── default.conf         # Configuración del servidor Nginx y reglas de seguridad
├── docs/
│   └── pruebas.md           # Informe de pruebas de funcionamiento, seguridad y rendimiento
└── README.md                # Instrucciones y documentación principal del proyecto
```

---

## 🛡️ Seguridad y Persistencia

* **Persistencia de Datos:** Los datos de la base de datos y los archivos subidos a WordPress se almacenan en volúmenes gestionados por Docker (`db_data` y `wp_data`).
* **Protección contra vulnerabilidades:** Las peticiones al archivo `xmlrpc.php` están denegadas a nivel de servidor web (retornando `403 Forbidden`).
* **Edición de ficheros deshabilitada:** Se restringe la edición de temas/plugins desde el panel mediante `DISALLOW_FILE_EDIT`.

---

## 📄 Enlaces a Documentación

Para ver el informe detallado de pruebas de rendimiento, consumo de memoria y comandos de verificación, consulta:
👉 [Informe de Pruebas y Rendimiento](docs/pruebas.md)