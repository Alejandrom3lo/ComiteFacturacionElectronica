# Comité Mensual de Facturación Electrónica — FE/RE Sinco

App del comité conectada a Cloud Firestore (proyecto `comitemensual`), con sincronización en tiempo real.

## Publicar en GitHub Pages

1. Crea un repositorio en https://github.com/new (por ejemplo `comite-fe`). Debe ser **público** para usar GitHub Pages en el plan gratuito.
2. Sube el archivo `index.html` de esta carpeta (botón "Add file" → "Upload files" → Commit).
3. Ve a **Settings → Pages**.
4. En "Build and deployment": Source = **Deploy from a branch**, Branch = **main**, carpeta **/ (root)** → **Save**.
5. Espera 1-2 minutos. Tu app quedará en:

   `https://TU_USUARIO.github.io/comite-fe/`

## Notas

- La configuración de Firebase (`firebaseConfig` dentro de `index.html`) es un identificador público del proyecto; la seguridad la dan las **reglas de Firestore**.
- Reglas recomendadas (Consola Firebase → Firestore → Reglas):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /comitesFE/{doc} {
      allow read, write: if true;
    }
  }
}
```

- Cualquier persona con el enlace puede leer y escribir en la agenda. Para restringirlo al equipo, agregar Firebase Authentication (inicio de sesión con Google) y ajustar las reglas.
- Para actualizar la app: edita/reemplaza `index.html` en el repositorio y GitHub Pages se redespliega solo.
