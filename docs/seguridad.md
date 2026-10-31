# Seguridad del gestor

| Comprobación | Antes | Después |
|---|---|---|
| Cabecera Server | nginx/1.2X.X (versión expuesta) | nginx (versión oculta) |
| /xmlrpc.php | 200 OK / 405 Method Not Allowed | 403 Forbidden |
| /wp-config.php | 200 OK | 403 Forbidden |
| PHP en uploads | Ejecutaba PHP (Devolvía "2") | 403 Forbidden |
| Edición de ficheros desde el panel | Habilitada | Deshabilitada (DISALLOW_FILE_EDIT) |
| Límite de intentos de login | Inexistente (Permitía fuerza bruta) | Activo (Bloqueo tras intentos fallidos) |

## Medidas aplicadas y por qué

- **server_tokens off:** Oculta la versión exacta de Nginx en las cabeceras HTTP para dificultar el reconocimiento del servidor a posibles atacantes y prevenir exploits dirigidos a versiones específicas.
- **Cabeceras HTTP de seguridad (X-Content-Type-Options y X-Frame-Options):** Protegen contra ataques de sniffing de MIME y mitigan ataques de Clickjacking impidiendo que la web sea embebida en un `iframe` externo.
- **Bloqueo de archivos en Nginx (`xmlrpc.php`, `wp-config.php`, `uploads/*.php`):**
  - `xmlrpc.php`: Desactiva un vector común de ataques de amplificación DDoS y fuerza bruta sobre credenciales.
  - `wp-config.php`: Deniega cualquier acceso directo vía HTTP al fichero sensible que contiene las credenciales de la base de datos y claves de salteado.
  - `uploads/*.php`: Evita la ejecución remota de código (RCE) en caso de que un atacante logre subir una web shell o archivo ejecutable PHP a la carpeta pública de archivos subidos.
- **DISALLOW_FILE_EDIT (`true`):** Elimina el editor de temas y plugins del panel de administración para evitar la inyección de código si una cuenta de administrador se ve comprometida.
- **Separación de roles y limitación de intentos de acceso:** Se crea un usuario con rol *Editor* para el uso diario y se instala un plugin que bloquea las direcciones IP tras varios fallos seguidos en `wp-login.php` para mitigar ataques de fuerza bruta.
