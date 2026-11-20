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

## Planner de tareas

La app tiene dos secciones, que se cambian desde la barra superior:

- **Planner** (`#/planner`, la que abre por defecto): tablero único y continuo de tareas del equipo, a página completa. No depende del mes.
- **Comité** (`#/comite`): el comité mensual, con Agenda, Nuevos temas y Release.

- **Tareas**: texto, responsable, fecha límite y tema de la agenda asociado (opcional). Estados: Por hacer, En proceso, Bloqueado y Hecho.
- **Vistas**: Tablero (Kanban con arrastrar y soltar), Por responsable y Esta semana (vencidas, hoy y próximos 7 días).
- **Desde la agenda**: cada tema tiene un botón "Tarea" para crear tareas ligadas a él; el tema muestra el avance (hechas/total) y la lista de sus tareas.
- **Regla de cierre**: al completar un tema con tareas abiertas se pide confirmación y, si se acepta, esas tareas pasan a Hecho.
- **Cierre de mes**: las tareas se quedan en el tablero. Las pendientes ligadas a un tema que pasa al comité siguiente siguen ligadas a ese tema; si el tema se completó, quedan sin tema.
- **Azure DevOps**: si el texto de una tarea o de un tema contiene `HU 12345`, se enlaza al work item en `dev.azure.com/sincosoftsas/FE`. Las referencias `HD` se resaltan sin enlace.
- **Mis tareas**: el selector "Yo soy" guarda tu nombre solo en el navegador (localStorage) y activa el filtro de tus tareas al abrir el Planner.

### Modelo de datos (dentro del documento `comitesFE/main`)

```
data.miembros        : ["Ximena", "Fabián Núñez", ...]   // responsables disponibles
data.tareas[]        : { id, text, resp, due: "AAAA-MM-DD" | null,
                         hora: "HH:MM" | null,   // obligatoria si hay fecha límite
                         estado: "todo" | "proceso" | "bloqueado" | "hecho",
                         comiteId: <id del comité del tema>,
                         temaId: <id del tema en la agenda de ese comité> | null, ts }
```

Los documentos creados antes del Planner no tienen estas listas; la app las trata como vacías y usa los nombres ya presentes en la agenda como miembros iniciales.

**Migración desde `comite.tareas`**: antes las tareas se guardaban dentro de cada comité. Mientras el documento no tenga `data.tareas`, la app une las de todos los comités (si una tarea se arrastró entre meses, gana la copia del comité más reciente). La primera vez que alguien crea, edita o mueve una tarea se guarda `data.tareas` y desde ahí es la única fuente; las listas `comite.tareas` antiguas quedan en el documento sin usarse.

### Restringir el acceso (pendiente, requiere la consola de Firebase)

Hoy cualquiera con el enlace puede leer y escribir. Para limitarlo al equipo:

1. En la consola de Firebase: **Authentication → Sign-in method → Google → Habilitar** y agregar el dominio de GitHub Pages en "Dominios autorizados".
2. Cambiar las reglas de Firestore a `allow read, write: if request.auth != null && request.auth.token.email.matches('.*@sinco[.]co');`.
3. Agregar en `index.html` el SDK `firebase-auth-compat.js` y un botón de inicio de sesión con `GoogleAuthProvider` antes de suscribirse a Firestore.

Mientras no se haga el paso 1, no debe cambiarse el código: la app dejaría de cargar para todos.
