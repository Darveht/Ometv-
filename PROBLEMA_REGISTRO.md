# 🔴 PROBLEMA CRÍTICO: Registro No Funciona

## El Problema

Cuando intentas registrarte, la pantalla se queda en blanco porque **Firebase no está configurado**.

## Causa Raíz

En `index.html` línea ~2199, las credenciales de Firebase son placeholders:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY_HERE",  // ❌ NO ES REAL
  authDomain: "your-app.firebaseapp.com",  // ❌ NO ES REAL
  // ... etc
};
```

## Solución Inmediata

Necesitas configurar Firebase con TUS credenciales reales:

### Paso 1: Crear Proyecto Firebase

1. Ve a https://console.firebase.google.com/
2. Crea un nuevo proyecto o usa uno existente
3. Activa **Authentication** (Email/Password)
4. Activa **Realtime Database**

### Paso 2: Obtener Credenciales

1. En Firebase Console, ve a **Project Settings** (⚙️)
2. Scroll hasta "Your apps"
3. Click en el ícono web `</>`
4. Copia la configuración

### Paso 3: Reemplazar en index.html

Busca la línea ~2199 y reemplaza con tus credenciales:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",  // Tu API Key real
  authDomain: "tu-proyecto.firebaseapp.com",
  databaseURL: "https://tu-proyecto.firebaseio.com",
  projectId: "tu-proyecto",
  storageBucket: "tu-proyecto.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123",
  measurementId: "G-XXXXXXXXX"
};
```

### Paso 4: Configurar Reglas de Firebase

En Firebase Console > Realtime Database > Rules:

```json
{
  "rules": {
    "profiles": {
      "$uid": {
        ".read": true,
        ".write": "$uid === auth.uid"
      }
    },
    "onlineUsers": {
      ".read": true,
      "$uid": {
        ".write": "$uid === auth.uid"
      }
    },
    "calls": {
      ".read": true,
      ".write": true
    },
    "reports": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

## Otros Errores Encontrados

### 1. TURN Server No Configurado
Línea ~2217 - Necesitas un servidor TURN real para WebRTC

### 2. Falta Validación de Elementos
El código intenta acceder a elementos que pueden no existir

### 3. onAuthStateChanged se Ejecuta Antes de Tiempo
Puede causar pantallas en blanco si Firebase no está listo

## Solución Temporal (Para Probar)

Si solo quieres probar sin Firebase, comenta estas líneas:

```javascript
// Línea ~2199
// firebase.initializeApp(firebaseConfig);
// firebase.analytics();
```

Y agrega esto después:

```javascript
// Mock para pruebas
const auth = { 
  onAuthStateChanged: (cb) => cb(null),
  createUserWithEmailAndPassword: () => Promise.reject({code: 'auth/no-config'})
};
const database = {
  ref: () => ({ on: () => {}, once: () => Promise.resolve({exists: () => false}) })
};
```

## Verificación

Abre la consola del navegador (F12) y busca errores como:

- `Firebase: No Firebase App '[DEFAULT]' has been created`
- `auth/invalid-api-key`
- `Failed to get document because the client is offline`

Estos confirman que Firebase no está configurado.
