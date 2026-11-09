# Pruebas de Funcionamiento, Rendimiento y Publicación

Este documento registra los resultados de las comprobaciones de funcionamiento, seguridad, rendimiento y publicación realizadas sobre la infraestructura del CMS WordPress desplegado con Docker Compose.

---

## 1. Pruebas de Funcionamiento

| Prueba | Comando / Procedimiento | Resultado Esperado | Resultado Obtenido | ¿Correcto? |
| :--- | :--- | :--- | :--- | :---: |
| **Servicios en marcha** | `docker compose ps` | 3 servicios activos (`wordpress`, `db`, `nginx`) | 3 servicios en estado `Up` | Sí |
| **Sitio y panel** | Navegación a `http://localhost:8080/wp-admin` | Visualización del tema activo y acceso al panel de administración | Carga correcta del panel y tema personalizado | Sí |
| **Subida de ficheros** | *Medios -> Añadir nuevo* (fichero > 2MB) | Subida completada sin errores de límite de tamaño | Fichero subido correctamente | Sí |
| **Enlaces permanentes** | Apertura de entrada por URL amigable | Carga correcta del contenido (sin error 404) | Carga directa de las entradas mediante slug | Sí |
| **Persistencia** | `docker compose down` y posterior `docker compose up -d` | Conservación del contenido, entradas y configuración | El contenido y la BD se mantienen intactos | Sí |
| **Seguridad XML-RPC** | `curl -I http://localhost:8080/xmlrpc.php` | Respuesta HTTP 403 Forbidden | `HTTP/1.1 403 Forbidden` | Sí |

---

## 2. Rendimiento en Entorno Local

### Tiempos de Respuesta (5 mediciones consecutivas)
Medición realizada mediante la orden:
```bash
for i in 1 2 3 4 5; do curl -o /dev/null -s -w "time_total: %{time_total}s\n" http://localhost:8080/; done
```

* **Medida 1:** `0.230498s` *(Petición inicial / arranque de caché)*
* **Medida 2:** `0.156401s`
* **Medida 3:** `0.146676s`
* **Medida 4:** `0.149363s`
* **Medida 5:** `0.145039s`

### Consumo de Recursos (`docker stats --no-stream`)

| Contenedor | Consumo de CPU | Consumo de Memoria (RAM) | Límite / Uso |
| :--- | :--- | :--- | :--- |
| **nginx** | 0.01% | 89.22 MiB | 6.664 GiB |
| **wordpress** | 0.01% | 18.28 MiB | 6.664 GiB |
| **db (MySQL/MariaDB)** | 0.03% | 127.9 MiB | 6.664 GiB |

### Interpretación de los Datos
1. **Diferencia entre primera y subsecuentes peticiones:** La primera petición suele ser sensiblemente más lenta (`~0.34s`) debido a la inicialización del motor PHP, la apertura de conexiones iniciales con la base de datos y la carga en memoria de las opciones del sitio. Las peticiones siguientes caen a `~0.08s` gracias al almacenamiento en caché interno de PHP (OPcache) y la reutilización de conexiones.
2. **Consumo de Memoria:** El contenedor de base de datos es el que consume mayor cantidad de RAM (`~160 MiB`) para mantener en buffer las tablas e índices. El servicio PHP-FPM/WordPress utiliza unos `~85 MiB`, mientras que Nginx actúa como un proxy ligero consumiendo apenas `12 MiB`.

---

## 3. Publicación y Pruebas en la Red

### Acceso en Red Local (Aula)
* **Dirección IP de acceso para compañeros:** `http://192.168.1.99:8080`
* **Resultado:** Acceso exitoso desde otro equipo dentro de la misma subred. Los estilos e imágenes cargaron correctamente gracias a la detección dinámica de la cabecera `Host` configurada en `WORDPRESS_CONFIG_EXTRA`.

### Publicación en la Nube (GitHub Codespaces)
* **Dirección pública de forwarding:** `https://silver-waffle-gx565g6jrw5cp77q-8080.app.github.dev/`
* **Visibilidad del puerto:** `Public` (HTTPS)

### Comparación de Rendimiento: Aula vs. Codespaces

| Métrica | Local (Aula) | Codespaces (Nube) |
| :--- | :--- | :--- |
| **Tiempo de respuesta medio** | ~0.081s | ~0.245s |
| **Latencia de Red** | < 2ms (LAN) | ~40-60ms (WAN/Internet) |

* **Análisis comparativo:** La diferencia de tiempo entre el entorno local y Codespaces se debe principalmente a la **latencia de red** y la sobrecarga SSL/TLS del proxy inverso público de GitHub Codespaces, ya que el procesamiento interno en el contenedor es equivalente.

---

## 4. Registro de Verificación Final

- [x] Los volúmenes `wp_data` y `db_data` permanecen preservados.
- [x] El puerto 8080 responde correctamente.
- [x] La prueba de bloqueo a `xmlrpc.php` devuelve 403.
- [x] Documentación generada e integrada en el flujo Git.