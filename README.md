# Panel de Administración — BETANCOURT Propiedades

Sistema de administración para gestionar las propiedades de la web, usando **Firebase** (plan Blaze).

## Arquitectura

- **Firebase Authentication**: login de administrador (email/password)
- **Cloud Firestore**: datos de las propiedades (título, precio, ubicación, descripción, etc.)
- **Firebase Storage**: almacenamiento de las fotos de cada propiedad
- **Firebase Hosting** (opcional): para publicar la web y el panel

## 1. Crear proyecto en Firebase (plan Blaze)

1. Ve a [https://console.firebase.google.com](https://console.firebase.google.com)
2. Crea un nuevo proyecto (o usa uno existente)
3. Actualiza a plan **Blaze** (pago por uso). Necesario para Storage y Hosting ilimitado.
4. En el proyecto:
   - Activa **Authentication** → método **Correo electrónico/contraseña**
   - Crea un usuario administrador (Authentication → Users → Add user)
   - Activa **Firestore Database** (modo producción)
   - Activa **Storage**
   - (Opcional) Activa **Hosting**

## 2. Configuración de seguridad (importante)

### Firestore Rules
Ve a Firestore → Rules y pega:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /betancourt_properties/{propertyId} {
      allow read: if true;                    // público puede leer
      allow write: if request.auth != null;   // solo admin autenticado
    }
  }
}
```

### Storage Rules
Ve a Storage → Rules y pega:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /betancourt/properties/{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null
                   && request.resource.size < 10 * 1024 * 1024
                   && request.resource.contentType.matches('image/.*');
    }
  }
}
```

## 3. Obtener la configuración web

1. En la consola Firebase → Project settings (engranaje) → Your apps → Web (</>)
2. Registra una app web
3. Copia el objeto `firebaseConfig`
4. Pégalo en:
   - `admin.html` (busca `YOUR_FIREBASE_CONFIG`)
   - `index.html` (si usas la versión con Firebase)

## 4. Cómo usar el panel

1. Abre `admin.html` (puedes servirlo localmente con `npx serve` o subirlo a Hosting)
2. Inicia sesión con el usuario admin que creaste
3. Desde el panel puedes:
   - Ver listado de propiedades
   - **Agregar** nueva propiedad (con múltiples fotos)
   - **Editar** cualquier campo y las fotos
   - **Eliminar** propiedades
   - Subir/reemplazar imágenes (se guardan en Storage y las URLs en Firestore)

## 5. Conectar la web pública (index.html)

En el archivo `index-firebase.html` se muestra cómo reemplazar el array hardcodeado por datos en tiempo real de Firestore.

Pasos recomendados:
1. Sube las fotos actuales desde el panel (o con un script de migración)
2. Copia los datos de cada propiedad al panel
3. Reemplaza tu `index.html` por la versión que lee de Firestore
4. Publica todo con Firebase Hosting

## 6. Publicar con Firebase Hosting (opcional)

```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# selecciona el proyecto, carpeta public = .
firebase deploy
```

Estructura sugerida:

```
/
├── index.html          # web pública
├── admin.html          # panel de administración
├── firebase.json
└── .firebaserc
```

## Modelo de datos (colección `properties`)

```js
{
  type: "casa" | "departamento" | "oficina" | "terreno" | "industrial" | "otro",
  label: "Casa",                    // texto del badge
  title: "Casa en arriendo...",
  loc: "Peñalolén, Región Metropolitana",
  price: "$1.060.000 / mes (UF 25,94)",
  specs: "4 dorm · 2 baños · 90 m² · 1 estac.",
  images: ["https://firebasestorage.../foto1.jpg", "..."],
  coord: null,                      // o string si usas coordenadas
  details: [
    ["Superficie total", "90 m²"],
    ["Dormitorios", "4"],
    // ...
  ],
  description: "Texto largo de la ficha...",
  createdAt: Timestamp,
  updatedAt: Timestamp
}
```

## Costos estimados (Blaze)

Para un volumen bajo-medio de propiedades y visitas:
- Firestore: gratis hasta 50k lecturas/día
- Storage: gratis hasta 5 GB + 1 GB/día de descarga
- Auth: gratis
- Hosting: gratis hasta 10 GB/mes

Normalmente se mantiene dentro del free tier de Blaze si no hay tráfico masivo.

## Soporte

Si necesitas que migre automáticamente las 11 propiedades actuales (con sus fotos base64) a Firestore + Storage, avísame y te preparo el script.

## 7. Migrar las propiedades actuales (script automático)

Ya está preparado el script en la carpeta `migration/`.

### Pasos:

1. **Descarga la clave de cuenta de servicio**
   - Firebase Console → ⚙️ Project settings → pestaña **Service accounts**
   - Click en **Generate new private key**
   - Guarda el archivo como `serviceAccountKey.json` **dentro de la carpeta `migration/`**

2. **Instala dependencias y ejecuta** (necesitas Node.js 18+):

```bash
cd migration
npm install
node migrate.js
```

El script:
- Lee las 11 propiedades de tu HTML original
- Sube todas las fotos (base64 y URLs de Unsplash) a Storage
- Crea los documentos en la colección `properties` de Firestore

Al terminar verás un resumen con los IDs creados. Luego abre `admin.html`, inicia sesión y ya estarán todas las propiedades listadas.

> **Importante:** Nunca subas `serviceAccountKey.json` a internet ni a Hosting. Es una clave privada.

## Soporte

Si algo falla en la migración (error de permisos, Storage bucket, etc.), revisa:
1. Que el proyecto esté en plan Blaze
2. Que Storage y Firestore estén activados
3. Que el nombre del bucket coincida (normalmente `TU_PROJECT_ID.appspot.com`)
4. Que las reglas de Storage permitan escritura a la cuenta de servicio (por defecto sí)
