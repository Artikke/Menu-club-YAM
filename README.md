# Menu Club YAMA - Pedidos de comedor

Un solo `index.html` + Firestore via REST, hospedado en GitHub Pages.
URL: https://comedor-yama.grupoproesa.mx/ · Firebase project: `menu-proesa-yama`. Mismo esquema de seguridad que Menu LANS.

## Seguridad
- Todo acceso a Firestore requiere token de Firebase Auth.
- **Empleados: entran desde el Portal PROESA** (SSO). La Cloud Function `ssoCallback` valida el JWT del
  portal, pide el numero de empleado al Gestor de Personal y regresa a la app con un custom token
  (`#ct=...`) que trae el claim `numEmp` (mayusculas, sin ceros a la izquierda, igual que el id de
  `orders/{id}`). El numero queda fijo en pantalla y las reglas solo dejan leer, pedir, modificar o
  cancelar `orders/{numEmp}` propio. Solo el admin lista todos los pedidos.
- Quien abre el link sin pasar por el portal entra como anonimo: ve el menu y un aviso con boton
  "Ir al Portal PROESA", pero no puede pedir. El anonimo queda para el acceso de admin con PIN.
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

## Login por el portal (SSO) - issue #3
Archivos: `functions/index.js` (Cloud Function `ssoCallback`, Node 22, 2nd gen), `firebase.json`, `.firebaserc`.
Valores: `appCode` = `comedor-yama`; JWKS `https://portal.grupoproesa.mx/.well-known/jwks.json`;
Gestor `https://proesa-team.grupoproesa.mx/api/integracion/v1/empleado?email=...` con header `X-Integracion-Token`.

**Deploy (en tu maquina, con firebase CLI; el proyecto ya esta en plan Blaze):**
```
npm install -g firebase-tools        # si no lo tienes
firebase login
cd Menu-club-YAM
firebase use menu-proesa-yama
(cd functions && npm install)
firebase functions:secrets:set GESTOR_TOKEN   # pega el token que te de TI; NUNCA en el repo
firebase deploy --only functions:ssoCallback
```
El deploy imprime la URL (p. ej. `https://us-central1-menu-proesa-yama.cloudfunctions.net/ssoCallback`);
se la pasas a TI para registrarla como **SSO callback** del portal.

**Permiso para firmar custom tokens (una sola vez):** `createCustomToken` firma con la cuenta de servicio
de la Function (2nd gen usa la de Compute: `<NUMERO_DE_PROYECTO>-compute@developer.gserviceaccount.com`).
En Google Cloud Console > IAM, darle a esa cuenta el rol **Service Account Token Creator**
(`roles/iam.serviceAccountTokenCreator`). O por CLI:
```
gcloud projects add-iam-policy-binding menu-proesa-yama \
  --member="serviceAccount:<NUMERO_DE_PROYECTO>-compute@developer.gserviceaccount.com" \
  --role="roles/iam.serviceAccountTokenCreator"
```
Sin ese rol el callback regresa `?error=sso` y en los logs de la Function sale `iam.serviceAccounts.signBlob`.

**Reglas:** despues del deploy de la Function y de que TI registre el callback, publicar
[`firestore.rules`](firestore.rules) junto con el `index.html` nuevo (merge a `main`), al mismo tiempo: el
`index.html` anterior lista todos los pedidos y con las reglas nuevas eso ya no se permite
(consola o `firebase deploy --only firestore:rules`). Desde ese momento solo se puede pedir entrando por el portal.

**API key:** la Web API key ya esta restringida por dominio (paso 6 de abajo); `signInWithCustomToken` usa
Identity Toolkit, que ya esta en la lista de APIs permitidas. No hay que cambiar nada.

**Errores que muestra la app** (`?error=` que regresa el callback): `sin_ficha` (el correo no tiene
numero de empleado en el Gestor), `gestor` (el Gestor no respondio), `sin_correo`, `sin_token`, `sso`
(JWT invalido/expirado o error al firmar).

**Pruebas de reglas** (emulador de Firestore, necesita Java 11+):
```
cd tests && npm install && npm test
```

## Dominio
https://comedor-yama.grupoproesa.mx/ (GitHub Pages, repo `Grupo-PROESA/Menu-club-YAM`, CNAME a
`grupo-proesa.github.io`, Enforce HTTPS). El poster `qr_poster.html` genera el QR con esa URL.

Equipo compartido: el link "No soy yo" junto al numero cierra la sesion del portal en ese navegador.
