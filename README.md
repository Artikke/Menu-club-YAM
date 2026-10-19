# Menu Club YAMA - Pedidos de comedor

Un solo `index.html` + Firestore via REST, hospedado en GitHub Pages.
URL: https://comedor-yama.grupoproesa.mx/ · Firebase project: `menu-proesa-yama`. Mismo esquema de seguridad que Menu LANS.

## Seguridad
- Todo acceso a Firestore requiere token de Firebase Auth anonimo.
- Los PIN viven en `config/secrets` (`adminPin`, `superPin`), que nadie puede leer desde el navegador.
- Al entrar, el navegador crea `admin_sessions/{uid}` con el PIN; las reglas lo comparan contra
  `config/secrets` y, si no coincide, la escritura falla. La sesion expira a las 12 h.
- `adminPin`: menu, pedidos, cortesias, historial y buzon. `superPin`: todo lo anterior + catalogo.
- El buzon vive en `comentarios/{id}`. Los comentarios del formato anterior (`config/comentarios`)
  se pasan a la coleccion la primera vez que entra un admin.

## Puesta en marcha (una sola vez, en este orden)
1. **Auth anonimo**: Firebase Console > `menu-proesa-yama` > Authentication > Sign-in method > Anonymous > Habilitar.
2. **Web API key**: Configuracion del proyecto > General > Tus apps > Web. Copiar `apiKey` a `FIREBASE_API_KEY` en `index.html`.
3. **PINs**: Firestore > Datos > coleccion `config` > documento `secrets` con dos campos **string**:
   `adminPin` y `superPin`. Usar PIN nuevos de 6 digitos (los anteriores quedaron expuestos).
4. **Publicar el `index.html`** nuevo (merge a `main`). Funciona tambien con las reglas viejas.
5. **Reglas**: Firestore > Reglas > pegar [`firestore.rules`](firestore.rules) > Publicar.
6. **Restringir la API key** en Google Cloud Console > Credenciales > Browser key:
   referrers `https://comedor-yama.grupoproesa.mx/*` (y `https://artikke.github.io/*` mientras dure el cambio);
   APIs: Identity Toolkit, Token Service, Cloud Firestore.

## Dominio
https://comedor-yama.grupoproesa.mx/ (GitHub Pages, repo `Grupo-PROESA/Menu-club-YAM`, CNAME a
`grupo-proesa.github.io`, Enforce HTTPS). El poster `qr_poster.html` genera el QR con esa URL.

Limitacion conocida (igual que LANS): no hay login de empleado; cualquiera con el link puede
pedir o cancelar con cualquier numero de empleado.
