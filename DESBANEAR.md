# Cómo Desbanear Usuarios en Firebase

## Opción 1: Desde Firebase Console (Manual)

1. Ve a Firebase Console: https://console.firebase.google.com/
2. Selecciona tu proyecto: `clack-koder`
3. Ve a **Realtime Database**
4. Navega a: `profiles/{userId}`
5. Elimina o edita estos campos:
   - `blocked`: cambiar a `false` o eliminar
   - `blockedUntil`: eliminar
   - `blockReason`: eliminar
   - `offenseCount`: cambiar a `0` o eliminar

## Opción 2: Usando la Consola del Navegador

Abre la consola (F12) en tu app y ejecuta:

```javascript
// Desbanear un usuario específico
const userId = "UID_DEL_USUARIO"; // Reemplaza con el UID real
firebase.database().ref(`profiles/${userId}`).update({
  blocked: false,
  blockedUntil: null,
  blockReason: null,
  offenseCount: 0
}).then(() => {
  console.log("✅ Usuario desbaneado");
}).catch(err => {
  console.error("❌ Error:", err);
});
```

## Opción 3: Desbanear Usuario Actual (si estás logueado)

```javascript
// Si estás logueado con la cuenta bloqueada
const currentUserId = firebase.auth().currentUser.uid;
firebase.database().ref(`profiles/${currentUserId}`).update({
  blocked: false,
  blockedUntil: null,
  blockReason: null,
  offenseCount: 0
}).then(() => {
  console.log("✅ Tu cuenta ha sido desbaneada");
  location.reload(); // Recargar la página
});
```

## Estructura de Datos en Firebase

### `/profiles/{userId}`
```json
{
  "username": "NombreUsuario",
  "blocked": true,
  "blockedUntil": "2024-01-15T10:30:00.000Z",
  "blockReason": "Lenguaje ofensivo detectado: \"palabra\"",
  "offenseCount": 3,
  "lastOffense": 1234567890
}
```

### `/blocks/{userId}/{blockId}`
```json
{
  "reason": "Lenguaje ofensivo detectado: \"palabra\"",
  "detectedWord": "palabra",
  "duration": 3,
  "blockedUntil": "2024-01-15T10:30:00.000Z",
  "timestamp": 1234567890,
  "automatic": true
}
```

## Ver Todos los Usuarios Bloqueados

```javascript
firebase.database().ref('profiles')
  .orderByChild('blocked')
  .equalTo(true)
  .once('value')
  .then(snapshot => {
    const blocked = snapshot.val();
    console.log("👥 Usuarios bloqueados:", blocked);
  });
```

## Desbanear Todos los Usuarios (¡Cuidado!)

```javascript
firebase.database().ref('profiles')
  .orderByChild('blocked')
  .equalTo(true)
  .once('value')
  .then(snapshot => {
    const updates = {};
    snapshot.forEach(child => {
      updates[`profiles/${child.key}/blocked`] = false;
      updates[`profiles/${child.key}/blockedUntil`] = null;
      updates[`profiles/${child.key}/blockReason`] = null;
    });
    return firebase.database().ref().update(updates);
  })
  .then(() => console.log("✅ Todos desbaneados"));
```
