# CLAUDE.md — Mis Estrellas (tracker de hábitos)

Contexto para asistentes de IA que trabajen en este repo.

## Qué es
App web para gamificar los hábitos diarios de niños/as (pensada para Valeria, 7 años, y su hermano Mateo) con un sistema de estrellas. Objetivo: que el niño la entienda sola y le ayude a cambiar hábitos en verano.

## Principio de diseño clave
**Todo vive en UN ÚNICO archivo `index.html`** (HTML + CSS + JS embebidos, sin build, sin dependencias, sin framework). Debe poder abrirse con doble clic (`file://`) y funcionar, incluida la sincronización en la nube. **No introducir bundlers, npm, ni dividir en varios archivos** salvo petición explícita.

UI orientada a un niño de ~7 años: emojis grandes, colores, animaciones (estrellas, confeti), sonidos con WebAudio, lenguaje sencillo en español. Todo en **español** (es-ES), con tildes correctas.

## Estructura de `index.html`
- `<head>`: estilos en un `<style>` con variables CSS en `:root`. Comentario inicial explicando la sincronización.
- `<body>`: cabecera de marca, hero (saludo + puntuación del día + nivel), secciones (tareas buenas, "cosas a mejorar", semana, gráfico, premio semanal, nivel), y un **dock inferior tipo tab bar** a ancho completo.
- `<script>` (único, NO module): toda la lógica. Funciones globales llamadas desde `onclick` inline.

## Modelo de datos (localStorage)
- Clave `misEstrellas.v2`. Función `migrate()` mantiene compatibilidad con datos antiguos.
- `state = { activeKidId, pin, kids: [...], updatedAt }`. `activeKidId` es **local por dispositivo** (NO se sincroniza: cada dispositivo ve el niño que quiera).
- `kid = { id, name, emoji, color, habits: [...], weekGoal, weekWon, log, _t }`
  - `habit = { id, name, emoji, type: "good"|"bad", points: 1..3 }`
  - `log = { "YYYY-MM-DD": { habitId: true | <conteo> } }` (buenos=`true`; malos=número de veces)
  - `weekGoal = { mode: "stars"|"days", stars, days, minPerDay, prize }`
  - `_t = { meta: ms, days: { "YYYY-MM-DD": { habitId: ms } } }` → marcas de tiempo para el **merge** entre dispositivos (ver Sincronización). `migrate()` lo inicializa.
- Puntuación: buenos suman `points`; malos restan `veces * points`. Los negativos **restan al total** (decisión del usuario); el nivel nunca baja de "Semillita".

## Funciones importantes
- `render()` → orquesta todo: `renderDock`, `renderHabits`, `renderNeg`, `renderWeek`, `renderChart`, `renderWeekPrize`, `renderReward`.
- `save()` → escribe localStorage (`saveLocal`) + sube a la nube (`schedulePush`). Pone `updatedAt = Date.now()`.
- Edición protegida por **PIN de papás** opcional (`askPin`).

## Sincronización (JSONBin.io)
- Sin servidor propio. Config de sync en `localStorage` clave `misEstrellas.sync.v1` = `{ key, binId }` (NO se sincroniza; es por dispositivo).
- API: `jbCreate` (POST crea bin con la Master Key), `jbRead` (GET), `jbUpdate` (PUT). CORS de JSONBin permite cualquier origen (funciona desde `file://`).
- **Código de sincronización** = base64 de `{k:key, b:binId}` (`makeCode`/`parseCode`). Es lo único que se comparte entre dispositivos.
- UX a prueba de errores: un solo campo (`smartSync`) detecta si pegas la **Master Key** (`activateWith` → crea nube) o un **código** (`joinWith` → se conecta). Regla: la clave se usa una vez (1er dispositivo); el resto pegan el código. Pegar la clave en cada dispositivo crearía nubes separadas (error frecuente ya mitigado).
- **Resolución de conflictos = MERGE por celda (no "el último gana con todo el documento")**. Antes se subía/bajaba el `state` entero y ganaba el `updatedAt` mayor → se pisaban cambios concurrentes y los relojes descuadrados liaban la sync. Ahora:
  - `save()` → `stampChanges()`: compara con `lastSnapshot` (foto del último guardado) y sella `_t.meta` (metadatos del niño) y `_t.days[día][habitId]` (cada celda del log) que hayan cambiado. Centralizado en `save()`; no hay que tocar cada función que muta.
  - `mergeStates`/`mergeKid` combinan local + nube: niños por id, metadatos por `_t.meta`, y **cada celda del log por `_t.days`** (días/hábitos distintos se conservan ambos; misma celda a la vez → gana la más reciente; un desmarcado deja "tombstone" en `_t` para que no reviva).
  - `ingestCloud()` mete la nube en el estado local y, si el merge aporta algo que la nube no tenía, reprograma push para **reconverger**. `doPush` hace **leer-combinar-escribir** (GET+merge+PUT) para no pisar lo de otro dispositivo. `adoptCloud` (solo al conectar) toma la nube como fuente de verdad.
  - `canon()` = firma estable (claves ordenadas, sin `activeKidId`) para decidir si algo cambió de verdad y evitar pushes/renders inútiles.
  - Verificado con `/tmp/merge.test.js` (extrae las funciones reales del HTML y simula 2 dispositivos: días/hábitos/niños distintos, concurrencia en la misma celda, desmarcado y convergencia).
- **Sondeo casi en tiempo real**: `startPolling()` hace pull cada `POLL_MS` (15 s) solo con la pestaña visible; se autopausa tras `IDLE_STOP_MS` (90 s) sin interacción ni cambios, y se reactiva con cualquier toque/tecla o al volver el foco. Equilibrio entre inmediatez y no agotar la cuota gratis (10k req/mes). Pull también al abrir y al recuperar el foco.
- **QR / enlace para conectar sin teclear**: el panel de sync (cuando ya está activado) muestra un QR generado en local con la librería `qrcode-generator` (1.4.4, MIT) **vendida inline** en su propio `<script>` antes del de lógica (excepción justificada al "single script", sigue siendo un único archivo sin build). El QR codifica `shareUrl()` = URL de la app alojada (o `location` si corre en http) + `#sync=<código>`. Al abrir un enlace así, `autoJoinFromUrl()` parsea el hash, conecta (`joinWith`) y limpia la URL con `history.replaceState`. Botón "Compartir enlace" usa `navigator.share` (móvil) o copia al portapapeles.

## Hosting
- Repo GitHub público: `git@github.com:vtellez/kidstracker.git` (rama `main`).
- Alojado SIN GitHub Pages vía proxy CDN: `https://raw.githack.com/vtellez/kidstracker/main/index.html`.
- `raw.githack.com/.../main/...` cachea; tras un push, usar recarga forzada o la URL fijada a un commit para ver cambios al instante.
- GitHub Pages es alternativa opcional (Settings → Pages → `main`).

## Flujo de trabajo
- Editar SIEMPRE en este repo: `/Users/vtellez/Projects/kidstracker`. (Existió una carpeta gemela `kidtracker` sin "s" usada al inicio; la fuente de verdad es este repo.)
- Validar sintaxis del JS embebido antes de commitear (extraer el `<script>` y `new Function(...)` con Node).
- Probar visualmente con el preview local (`.claude/launch.json` levanta un `http.server`).
- Commits en español; cerrar con la línea `Co-Authored-By` cuando aplique.

## Pendientes / ideas no implementadas
- Estadísticas mensuales para los padres (cumplimiento, hábito más difícil).
- Autoocultar el dock al hacer scroll.
- Opción de "día redondo = todas las tareas hechas" como criterio del objetivo por días.
