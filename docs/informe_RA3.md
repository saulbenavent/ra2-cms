# Informe RA3 - Administración de Gestores de Contenidos

## 1. Gestión de Usuarios, Roles y Capacidades

### Capacidades clave identificadas
* **Publicar entradas:** `publish_posts`
* **Subir ficheros/imágenes:** `upload_files`

### Matriz de permisos comprobados
| Usuario | Rol | Puede hacer | No puede hacer |
| :--- | :--- | :--- | :--- |
| `admin` (tú) | Administrador | Control total: plugins, temas, usuarios, ajustes y todo el contenido. | — |
| `coord.web` | Editor | Crear, editar y publicar entradas/páginas propias y de terceros (incluida la de `alumna.fct`), moderar comentarios y gestionar categorías. | Instalar plugins, cambiar temas, modificar ajustes o gestionar usuarios. |
| `profe.redes` | Autor | Escribir, subir imágenes, publicar y borrar sus propias entradas. | Editar o borrar entradas de otros usuarios, crear páginas o gestionar categorías. |
| `alumna.fct` | Colaborador | Escribir borradores de entradas y enviarlas a revisión. | Publicar directamente ni subir imágenes/archivos multimedia. |
| `familia.garcia` | Suscriptor | Acceder al escritorio para leer y editar su propio perfil. | Crear, editar o publicar ningún tipo de contenido. |
| `revisor.fp` | Revisor (A medida) | Revisar, editar y publicar entradas/páginas propias o de otros usuarios. | Enviar a la papelera o borrar entradas, páginas o gestionar categorías. |

---

## 2. Selección de Módulos (Plugins)

### R1. Formulario de contacto con almacenamiento en BD
* **Opciones comparadas:** *Contact Form 7 + Flamingo* vs *WPForms Lite*.
* **Elección:** **Contact Form 7 + Flamingo**.
* **Justificación:** La versión gratuita de WPForms no guarda los envíos en la base de datos local (requiere licencia Pro), mientras que Contact Form 7 combinado con Flamingo permite almacenar todos los registros en WordPress de forma gratuita y desactivar el envío SMTP (`skip_mail: on`) en entornos de contenedores sin servidor de correo.

### R2. Aviso de cookies (LSSI y RGPD)
* **Opciones comparadas:** *CookieYes – Cookie Banner, Cookie Consent & Cookie Notice for GDPR/CCPA* (`cookie-law-info`, 1.000.000 de instalaciones activas, valoración 96/100, actualizado el 12/11/2026) vs *Complianz GDPR/CCPA Cookie Consent Banner* (`complianz-gdpr`, 1.000.000 de instalaciones activas, valoración 94/100, actualizado el 04/11/2026).
* **Elección:** **CookieYes (`cookie-law-info`)**.
* **Justificación:** Cuenta con un millón de instalaciones activas, mejor valoración (96/100) y mantenimiento muy reciente. Permite configurar el banner en español, mostrar los botones de Aceptar y Rechazar con el mismo peso visual y enlazar la página de Política de cookies cumpliendo la LSSI y el RGPD.

### R3. Registro de actividad y auditoría
* **Opciones comparadas:** *Simple History* vs *WP Activity Log*.
* **Elección:** **Simple History**.
* **Justificación:** Es ligero, gratuito sin restricciones críticas para auditoría básica, integra el visor directamente en el Escritorio de WordPress y registra inicios de sesión exitosos/fallidos, cambios en contenidos y activación de plugins.