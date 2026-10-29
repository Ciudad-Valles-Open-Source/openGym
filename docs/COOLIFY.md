# Desplegar openGym en Coolify

[Coolify](https://coolify.io/) despliega el `docker-compose.yml` de este repositorio tal cual:
dos servicios (`api` y `web`) y un volumen con los datos. No hace falta base de datos, Redis
ni ninguna cuenta externa.

## Pasos

1. En Coolify, ve a **New Resource > Docker Compose** y conecta tu repositorio de Git (y la
   rama a desplegar). Coolify detecta `docker-compose.yml` automáticamente.
2. En el servicio **`web`**, asigna tu dominio, por ejemplo `https://gym.tudominio.com`.
   nginx escucha en el puerto 80 dentro del contenedor, así que no hace falta añadir puerto.
   Coolify genera el certificado HTTPS con Traefik.
   El servicio `api` **no** lleva dominio: la app y su `/api` tienen que estar en el mismo
   origen para que funcionen las passkeys, y `web` ya le pasa `/api` a `api`.
3. En **Environment Variables**, como mínimo:

   ```dotenv
   RP_ID=gym.tudominio.com
   ORIGIN=https://gym.tudominio.com
   DEFAULT_LANG=es
   ```

   `RP_ID` es solo el nombre del host (sin `https://` ni barra final) y `ORIGIN` la dirección
   completa, exactamente como aparece en el navegador. Si no coinciden con el dominio, el
   registro con passkey falla.
4. Pulsa **Deploy**. Cuando `api` esté *healthy*, `web` arranca.
5. Abre tu dominio y **crea tu perfil antes de compartir la dirección**: con
   `FIRST_USER_ADMIN=1` (el valor por defecto del compose) el primer perfil es el
   administrador. Desde *Ajustes > Cuenta > Panel de administración* puedes generar códigos de
   invitación, ver usuarios y activar el Coach de IA.

## Idioma

La app está traducida al español por completo (interfaz, nombres de ejercicios e
instrucciones). `DEFAULT_LANG=es` hace que la pantalla de inicio de sesión y los perfiles
nuevos empiecen en español; cada persona puede cambiarlo en *Ajustes > Idioma*. Las
descripciones largas de los ejercicios solo existen en inglés y alemán, y se muestran en
inglés.

## Variables opcionales útiles

| Variable | Para qué |
|---|---|
| `INVITE_ONLY=1` | Solo se pueden crear perfiles con un código de invitación |
| `ALLOW_GUEST=0` | Quita el botón "Continuar sin cuenta" |
| `PASSWORD_LOGIN=1` | Permite iniciar sesión con contraseña además de passkey |
| `FIRST_USER_ADMIN=0` | Desactiva la promoción automática del primer perfil a admin |
| `ADMIN_UIDS=id1,id2` | Más administradores (el id está en *Ajustes > Cuenta*) |
| `SESSION_DAYS=30` | Duración de la sesión (90 días por defecto) |
| `API_TARGET=coach` | Imagen con Claude Agent SDK y Codex dentro (~300 MB más). Los proveedores por HTTPS (Anthropic, OpenAI, Gemini, Ollama...) funcionan sin esto |
| `INCLUDE_EXERCISE_MEDIA=false` | Construye la imagen web sin las imágenes y animaciones de ejercicios (ver abajo) |
| `CF_CONNECTING_IP='$http_cf_connecting_ip'` | Solo si Cloudflare está delante (con las comillas simples) |

`.env.example` explica todas las demás.

## Datos y copias de seguridad

Todo vive en el volumen `opengym_data` (`/data` dentro de `api`): usuarios, passkeys,
entrenamientos, el secreto de sesión, las claves de notificaciones y las fotos/vídeos de
ejercicios propios. Es lo único que hay que respaldar; puedes programar copias del volumen
desde Coolify o desde el servidor. Si se pierde el archivo `secret`, todos tendrán que volver a
iniciar sesión.

## Licencias

El código de openGym es AGPL-3.0: puedes usarlo, modificarlo y alojarlo sin restricciones,
siempre que publiques el código de tus cambios si ofreces la app a otras personas por la red.
No hay funciones de pago ni límites por licencia en la app.

Lo único con otra licencia son las imágenes y animaciones de ejercicios (`catalogue/media/`),
de Gym visual, cedidas a openGym a 180 px. Un servidor propio puede servirlas a ese tamaño, que
es lo que hace la imagen `web`; no se pueden extraer para otros proyectos. Si prefieres no
incluirlas, despliega con `INCLUDE_EXERCISE_MEDIA=false`: la app sigue funcionando y muestra un
icono en lugar de la animación. Detalles en [NOTICE.md](../NOTICE.md).

## Problemas frecuentes

- **No se puede crear la passkey**: `RP_ID` u `ORIGIN` no coinciden con el dominio, o la página
  no está en HTTPS.
- **502 en `/api`**: el servicio `api` no está *healthy*; revisa sus registros en Coolify. Suele
  ser el volumen sin permisos de escritura o el disco lleno.
- **Subidas de vídeo rechazadas**: sube `MEDIA_UPLOAD_MAX` y `MEDIA_VIDEO_MAX_MB` juntos.

Más detalle en [SELF_HOSTING.md](SELF_HOSTING.md) (en inglés).
