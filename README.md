# Menu Club YAMA - Pedidos de comedor

Un solo `index.html` + Firestore via REST, hospedado en GitHub Pages.
URL: https://comedor-yama.grupoproesa.mx/ · Firebase project: `menu-proesa-yama`. Mismo esquema de seguridad que Menu LANS.

## Seguridad
- Todo acceso a Firestore requiere token de Firebase Auth.
- **Empleados: entran desde el Gestor de Personal** (seccion Comedor). La Cloud Function `ssoCallback` valida
  el token firmado que manda el Gestor y regresa a la app con un custom token
  (`#ct=...`) que trae el claim `numEmp` (mayusculas, sin ceros a la izquierda, igual que el id de
  `orders/{id}`). El numero queda fijo en pantalla y las reglas solo dejan leer, pedir, modificar o
  cancelar `orders/{numEmp}` propio. Solo el admin lista todos los pedidos.
- Quien abre el link sin pasar por el Gestor entra como anonimo: ve el menu y un aviso con boton
  "Ir al Gestor de Personal", pero no puede pedir. El anonimo queda para el acceso de admin con PIN.
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

## Entrada desde el Gestor de Personal - issue #3
Los colaboradores entran desde la seccion **Comedor** del Gestor de Personal (opcion 2 acordada con TI).
El Gestor redirige a la Function `ssoCallback` con `?t=<JWT HS256>` firmado con `HANDOFF_SECRET`
(`iss` = `gestor-personal`, `aud` = `comedor-yama`, trae `numEmp`, `nombre`, `correo`). La Function lo valida y
regresa a la app con un custom token (`#ct=...`) con el claim `numEmp`.
Archivos: `functions/index.js` (Node 22, 2nd gen), `firebase.json`, `.firebaserc`.

**Deploy (Cloud Shell o tu maquina, con firebase CLI):**
```
firebase use menu-proesa-yama
(cd functions && npm install)
firebase functions:secrets:set HANDOFF_SECRET   # pega el secreto que te de TI; NUNCA en el repo
firebase deploy --only functions:ssoCallback
```
URL: `https://us-central1-menu-proesa-yama.cloudfunctions.net/ssoCallback`. El mismo `HANDOFF_SECRET` lo tiene TI en el
`.env` del Gestor; si lo rotan, se vuelve a correr `secrets:set` (y contestar Y para re-desplegar).
El `GESTOR_TOKEN` del flujo anterior (portal) ya no se usa: se puede borrar con
`firebase functions:secrets:destroy GESTOR_TOKEN`.

**Permiso para firmar custom tokens (una sola vez, ya hecho):** la cuenta de servicio de la Function
(`<NUMERO_DE_PROYECTO>-compute@developer.gserviceaccount.com`) necesita el rol **Service Account Token Creator**:
```
gcloud projects add-iam-policy-binding menu-proesa-yama \
  --member="serviceAccount:<NUMERO_DE_PROYECTO>-compute@developer.gserviceaccount.com" \
  --role="roles/iam.serviceAccountTokenCreator"
```

**Orden para publicar:** (1) Function con `HANDOFF_SECRET`; (2) merge a `main` del `index.html` nuevo (funciona
tambien con las reglas actuales); (3) prueba con un colaborador real entrando desde el Gestor; (4) publicar
[`firestore.rules`](firestore.rules) (consola o `firebase deploy --only firestore:rules`). No publicar las reglas
antes del `index.html`: el anterior lista todos los pedidos y con las reglas nuevas eso ya no se permite.
Desde el paso 4 solo se puede pedir entrando desde el Gestor.

**API key:** la Web API key ya esta restringida por dominio (paso 6 de abajo); `signInWithCustomToken` usa
Identity Toolkit, que ya esta en la lista de APIs permitidas. No hay que cambiar nada.

**Errores que muestra la app** (`?error=` del callback): `sin_token` (se abrio la Function sin token),
`sin_numero` (el token no trae numero de empleado), `sso` (token invalido, expirado, de otro comedor o error al firmar).

**Pruebas de reglas** (emulador de Firestore, necesita Java 11+):
```
cd tests && npm install && npm test
```

## Dominio
https://comedor-yama.grupoproesa.mx/ (GitHub Pages, repo `Grupo-PROESA/Menu-club-YAM`, CNAME a
`grupo-proesa.github.io`, Enforce HTTPS). El poster `qr_poster.html` genera el QR con esa URL.

Equipo compartido: el link "No soy yo" junto al numero cierra la sesion en ese navegador.
