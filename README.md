# MAFIA — Web, Windows y Android (un solo juego)

Todo sale de la carpeta `www/` (el juego en HTML). No hay que reprogramar nada.

## 1) Web (la que ya tienes)
Sube la carpeta **www** completa a Netlify (arrastrar y soltar). Es lo mismo de siempre.
- `config.json`: ahí van tus datos de relevo (ver LEEME anterior) y los links de descarga:
  `"downloads": { "android": "https://.../Mafia.apk", "windows": "https://.../Mafia-Setup.exe" }`
  Cuando pongas esos links, aparece "Descargar para Android/Windows" en el menú y en Ajustes de la web.

## 2) Windows y Android SIN programar (se compilan solos en la nube, gratis)
1. Crea una cuenta en github.com y un repositorio nuevo (privado está bien).
2. Sube TODO el contenido de esta carpeta al repositorio ("Add file" > "Upload files").
   - Importante: tiene que quedar la carpeta oculta `.github/workflows/build.yml`.
     Si no se subió, crea el archivo a mano: "Add file" > "Create new file", escribe como nombre
     `.github/workflows/build.yml` y pega el contenido de ese archivo.
3. Ve a la pestaña **Actions** > "Compilar apps" > **Run workflow**.
4. En unos 10 a 15 minutos aparecen dos descargas al final de la página:
   - **Mafia-Android-APK** (instalable en Android; hay que permitir "instalar apps de orígenes desconocidos").
   - **Mafia-Windows-instalador** (el .exe para instalar en Windows).
5. Para que la web ofrezca esas descargas: sube los archivos a Netlify (por ejemplo dentro de `www/`) y pon sus
   links en `config.json`.

Si una compilación falla, copia el mensaje de error y mándamelo: es la parte que no pude probar desde aquí.

## 3) Jugar por la misma Wi-Fi SIN internet
- Abre la app de **Windows** (o ejecuta `INICIAR-WIFI-LOCAL.bat`; necesita Node.js instalado).
- Entra a "Online": verás una dirección tipo `http://192.168.1.20:8765` y un QR.
- Los demás (Android, iPhone, otro PC) abren esa dirección en el navegador estando en la misma Wi-Fi. Eso es todo.
- La primera vez Windows pregunta por el Firewall: elige "Permitir" en redes privadas.
- En este modo no hay voz, ni escaneo de QR con cámara (los navegadores las bloquean en direcciones sin https).
- Con internet puedes alternar en el mismo menú: "Usar internet en vez de Wi-Fi local".
- Desde la app de Android también puedes conectarte al PC: Online > "Wi-Fi local sin internet" y escribes la dirección.
- El anfitrión de una partida Wi-Fi local tiene que ser el PC con Windows.

## 4) Alternativa sin cuenta de GitHub
En pwabuilder.com pega el link de tu web de Netlify: genera paquetes para Android y Windows con un clic.
Son versiones que abren tu web (necesitan internet para cargar), no incluyen el modo Wi-Fi local.

## 5) Play Store (más adelante)
Hace falta una cuenta de desarrollador de Google y firmar la app. Cuando quieras publicarla,
te preparo el paso de firma y el formato AAB.

## 6) Cloudflare: relevo TURN + web gratis (para jugar entre redes distintas)
El relevo de Cloudflare da 1.000 GB al mes gratis, mucho más que Metered. Un pequeño servicio tuyo (carpeta `cloudflare/`)
reparte credenciales temporales; el token secreto nunca llega al celular. También puede alojar el juego (carpeta `www`).

1. Crea una cuenta gratis en dash.cloudflare.com.
2. **Llave TURN**: menú *Realtime* > *TURN Server* > crear. Guarda el **Turn Key ID** y el **API Token**
   (el token se muestra una sola vez).
3. **Token para publicar**: *My Profile* > *API Tokens* > *Create Token* > plantilla *Edit Cloudflare Workers*.
   Copia ese token. Anota también tu **Account ID** (aparece en *Workers & Pages*).
4. En GitHub: tu repositorio > *Settings* > *Secrets and variables* > *Actions* > *New repository secret*.
   Crea `CLOUDFLARE_API_TOKEN` (token del paso 3) y `CLOUDFLARE_ACCOUNT_ID`.
5. Pestaña *Actions* > **Publicar en Cloudflare** > *Run workflow*. Si Cloudflare pide elegir un subdominio `workers.dev`,
   elígelo en *Workers & Pages* y vuelve a ejecutarlo. Tu juego queda en `https://mafia-juego.TU-SUBDOMINIO.workers.dev`.
6. En Cloudflare: *Workers & Pages* > mafia-juego > *Settings* > *Variables and Secrets* > agrega dos **secretos**:
   `TURN_KEY_ID` (el Turn Key ID) y `TURN_KEY_API_TOKEN` (el API Token del paso 2).
7. Abre `https://mafia-juego.TU-SUBDOMINIO.workers.dev/api/turn`: debe mostrar un texto JSON con "iceServers".
8. En `www/config.json` pon el endpoint:
   `"turn": { "endpoint": "https://mafia-juego.TU-SUBDOMINIO.workers.dev/api/turn" }`
   (si juegas desde ese mismo dominio basta `"/api/turn"`). Sube el cambio a GitHub y vuelve a ejecutar
   *Publicar en Cloudflare*. Las apps de Windows/Android se vuelven a compilar con *Compilar apps*.
9. En el juego: *Ajustes* > *Probar conexión*. Debe salir ✅ en "Servidor de relevo (TURN)".

Frenos para no pasarte del plan gratis (editables en `cloudflare/wrangler.jsonc`):
- `MAX_PER_DAY`: credenciales máximas por día (3000).  `MAX_PER_IP_HOUR`: por persona y hora (20).
- Al llegar al tope el servicio responde "límite" y el juego sigue con conexión directa (sin relevo).
- No agregues tarjeta de pago a Cloudflare mientras no la necesites.
- Para probar el servicio sin internet: `node cloudflare/test/probar.mjs`.
