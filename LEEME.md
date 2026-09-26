# Betancourt Propiedades — Panel de administración

## Publicación automática (GitHub → Firebase)

- Cada cambio que se acepta en `main` se publica solo en el sitio
  `betancourt-propiedades` (https://betancourt-propiedades.web.app).
- Cada PR publica una vista previa temporal y deja el enlace como comentario.
- La publicación automática sube **solo las páginas** (`public/`). Las reglas
  (`firestore.rules`, `storage.rules`) se comparten con los otros sitios del
  proyecto y se publican a mano desde la consola de Firebase.
- Los avisos, artículos y fotos que se suben desde el panel viven en
  Firestore/Storage: publicar la página no los borra.

## Qué hay en esta carpeta

- `public/index.html` — tu sitio público, ya liviano (52 KB) y conectado a Firestore.
- `public/admin.html` — el panel de administración (login + gestión de propiedades y blog).
- `public/images/` — tus 51 fotos reales de avisos, ya extraídas y nombradas.
- `firestore.rules` — reglas de seguridad de la base de datos.
- `storage.rules` — reglas de seguridad de las fotos.

## Pasos para dejarlo funcionando

### 1. Consigue tu configuración de Firebase
Ve a Firebase Console → ⚙️ Configuración del proyecto → pestaña "Tus apps".
Si no tienes una "app web" creada, créala ahí (es gratis, distinto de Hosting).
Copia el objeto `firebaseConfig` que te muestra.

### 2. Pega esa configuración en DOS archivos
- `public/admin.html` — busca `const firebaseConfig = {` y reemplaza los valores.
- `public/index.html` — busca lo mismo, más abajo en el archivo, y reemplaza igual.
(Deben quedar IDÉNTICOS en ambos archivos.)

### 3. Copia estos archivos a tu proyecto real
Reemplaza tu carpeta `public/` actual por el contenido de esta carpeta `public/`.
Copia también `firestore.rules` y `storage.rules` a la raíz de tu proyecto
(junto a tu `firebase.json`).

### 4. Conecta las reglas de seguridad en firebase.json
Agrega estas líneas dentro de tu `firebase.json` (junto a "hosting"):

```json
"firestore": {
  "rules": "firestore.rules"
},
"storage": {
  "rules": "storage.rules"
}
```

### 5. Sube las reglas y el sitio
Desde la terminal, en la carpeta del proyecto:

```
firebase deploy --only firestore:rules,storage:rules,hosting:betancourt
```

### 6. Crea tu usuario admin (si no lo hiciste ya)
Firebase Console → Authentication → Users → Add user → tu correo y contraseña.

### 7. Entra al panel
Ve a tu-sitio.web.app/admin.html, inicia sesión, y empieza a cargar
propiedades reales — aparecerán automáticamente en tu página pública.

## Notas importantes

- Las propiedades de ejemplo (Til Til, Bellavista, etc.) siguen en el código
  como respaldo por si algún día falla la conexión a internet del visitante.
  Las reales que subas desde el panel siempre aparecen primero.
- Cada foto que subas desde el panel pesa hasta 10 MB máximo (protección
  configurada en storage.rules) y se guarda en Storage con respaldo de Google.
- Solo tú puedes crear/editar/borrar (login obligatorio). Cualquiera puede
  ver la página pública sin necesitar cuenta.
