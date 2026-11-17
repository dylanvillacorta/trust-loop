# Especificación Técnica y Funcional — TrustLoop

> Única fuente de verdad técnica y funcional de **TrustLoop**. El flujo e interfaz funcional están alineados con [wireframe-trustloop.html](wireframe/wireframe-trustloop.html), que prevalece ante cualquier divergencia funcional mientras no exista el diseño visual definitivo. La estética (colores, tipografía, íconos, ilustraciones de los compañeros y grosores) la define el diseñador externo.
>
> Las reglas del juego marcadas con **(S)** son supuestos tomados sin consulta; se listan en la sección final [Supuestos para revisar](#16-supuestos-para-revisar).

---

## 1. Propósito y alcance

**TrustLoop** es una aplicación de escritorio educativa en la que el estudiante juega **partidas** de rondas con **compañeros virtuales**. En cada ronda decide entre **cooperar** o **protegerse**; cada compañero responde según un **estilo** propio que el estudiante debe descubrir. La actividad estimula:

- **La toma de decisiones en situaciones de interacción:** el resultado de cada ronda depende de lo que eligen ambos.
- **La cooperación:** cooperar de forma sostenida produce más puntos para ambos cuando el otro también coopera.
- **El ajuste de la conducta:** reconocer cómo juega cada compañero y adaptar las propias decisiones a ese comportamiento.

El sistema registra automáticamente las decisiones tomadas, el nivel de cooperación, la respuesta ante una traición y el puntaje individual y grupal, y ofrece **retroalimentación al final de cada partida**. Dentro de la partida solo se muestra el resultado de cada ronda.

### Contexto operativo y restricciones de producto

- **Uso:** PC con Windows sin conexión a internet, en aula o laboratorio. Varios estudiantes comparten el mismo equipo mediante perfiles locales aislados.
- **Revisión:** presencial en la propia pantalla junto al docente.
- **Alcance no evaluativo:** herramienta de práctica y reflexión; **no** es una prueba psicológica, no mide la personalidad ni ofrece diagnósticos.
- **Tono e idioma:** el nombre del producto (`TrustLoop`) y la carpeta de datos (`%LOCALAPPDATA%/TrustLoop`) se mantienen siempre en inglés; toda la interfaz, instrucciones y retroalimentación se redactan en español con tono formativo, claro y no punitivo. Ningún texto califica al estudiante (prohibido «egoísta», «ingenuo», «tramposo», «perdedor» o equivalentes); los textos describen decisiones y consecuencias.
- **Sin red ni exportación:** prohibido exportar datos en CSV u otros archivos externos, y prohibido enviar información por red.

### Glosario

| Término | Definición |
| :--- | :--- |
| **Compañero** | Personaje virtual ilustrado por el diseñador, con nombre propio, que juega contra el estudiante. |
| **Estilo** | Regla fija con la que un compañero decide en cada ronda (§5.3). El estudiante no la conoce hasta la retroalimentación. |
| **Cooperar** | Decisión que suma puntos para ambos si el otro también coopera, pero deja al estudiante sin puntos si el otro se protege. |
| **Protegerse** | Decisión que asegura un mínimo de puntos y aprovecha la cooperación del otro, pero reduce el puntaje grupal. |
| **Ronda** | Una decisión simultánea del estudiante y del compañero, con su resultado en puntos. |
| **Encuentro** | Serie de rondas consecutivas con el mismo compañero. |
| **Partida** | Ciclo completo: encuentros con todos los compañeros + retroalimentación final. |
| **Traición** | Ronda en la que el estudiante cooperó y el compañero se protegió. Término técnico de las métricas; en la interfaz se acompaña siempre de su descripción. |
| **Respuesta ante una traición** | Lo que decide el estudiante en la ronda siguiente a una traición con el mismo compañero (§5.5). |

---

## 2. Stack y dependencias aprobadas

Las siguientes dependencias y versiones mayores (alineadas con TheirView, sin escena 3D) quedan **expresamente aprobadas** para este proyecto a efectos de la puerta **G3** de [AGENTS.md](../AGENTS.md):

| Componente | Tecnología / Paquete | Versión mayor aprobada | Rol |
| :--- | :--- | :--- | :--- |
| Runtime de escritorio | `electron` | `^31.0.0` (v31) | Proceso principal, ventana y preload |
| Empaquetado | `electron-builder` | `^24.13.3` (v24) | Instalador NSIS por usuario y portable |
| Interfaz | `react`, `react-dom` | `^18.3.1` (v18) | Vistas y componentes del renderer |
| Bundler | `vite`, `@vitejs/plugin-react` | `^5.3.1` (v5) / `^4.3.1` (v4) | Compilación de renderer y preload |
| Lenguaje | `typescript` | `^5.5.2` (v5, modo estricto) | Tipado estricto en main, preload, lógica y renderer |
| Testing | `vitest`, `jsdom` | `^2.0.0` (v2) / `^24.1.0` (v24) | Pruebas unitarias y de persistencia |
| Tipos auxiliares | `@types/node`, `@types/react`, `@types/react-dom` | `^20.14.0` / `^18.3.x` | Definiciones de tipos |
| Gestor de paquetes | `pnpm` | Único permitido | Prohibido `npm` o `yarn` y sus lockfiles |

TrustLoop **no usa Three.js** ni ninguna escena 3D: toda la parte visual es interfaz e ilustraciones entregadas por el diseñador. Las ilustraciones se empaquetan localmente dentro del bundle, sin red ni CDN.

---

## 3. Estructura de carpetas

```text
trust-loop/
├── build/                      # Ícono oficial y recursos de empaquetado
├── docs/
│   ├── SPECIFICATION.md        # Esta especificación técnica y funcional
│   ├── capturas/               # Capturas PNG generadas por pnpm docs:screenshots
│   └── wireframe/              # Wireframe HTML de referencia funcional y su prompt
├── electron/
│   ├── main.ts                 # Proceso principal, ventana e IPC handlers
│   ├── preload.ts              # contextBridge seguro (window.trustLoopApi)
│   ├── ipcValidation.ts        # Validación estricta de argumentos IPC
│   └── storage/
│       └── atomicStore.ts      # Persistencia atómica (.tmp -> rename) con lock por archivo
├── scripts/
│   ├── dev.mjs                 # Arranque de desarrollo
│   ├── capture-screenshots.mjs # Orquestador de capturas con perfil temporal
│   └── capture-screenshots-main.cjs # Entrada Electron para capturas
├── src/
│   ├── shared/
│   │   ├── types.ts            # Contratos globales de datos e IPC
│   │   ├── ipcChannels.ts      # Constantes de canales IPC
│   │   ├── partners.ts         # Los 5 compañeros (id, nombre, recurso de ilustración)
│   │   ├── payoffs.ts          # Matriz de puntos por ronda
│   │   └── feedbackTexts.ts    # Catálogo de textos de retroalimentación
│   ├── logic/                  # Lógica pura (sin Electron ni DOM): estilos, puntos, métricas, reanudación
│   └── renderer/
│       ├── main.tsx            # Entrypoint React 18
│       ├── App.tsx             # Orquestador de las 8 pantallas
│       ├── assets/             # Ilustraciones de los compañeros y logo entregados por el diseñador
│       ├── audio/              # Motor de efectos sonoros
│       ├── components/         # Componentes reutilizables (p. ej. BackButton, diálogos)
│       ├── hooks/              # Hooks de estado global sincronizado con IPC
│       ├── screens/            # Las 8 pantallas de la aplicación
│       └── styles/             # Tokens CSS y reglas de alto contraste / texto grande
├── tests/                      # Tests unitarios (logic/, storage/, etc.)
├── electron-builder.yml
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
└── vite.config.ts
```

**Archivos compartidos que obligan a secuenciar worktrees:** `package.json`, `pnpm-lock.yaml` y `src/shared/types.ts`.

---

## 4. Flujo y pantallas

La aplicación consta de **8 pantallas** (`ScreenId`). La partida transcurre íntegra en `GAME`, que alterna cuatro fases (`GamePhase`): presentación del compañero, decisión, resultado de la ronda y estilo supuesto.

```text
SPLASH ──(~2.5 s)──▶ USER_SELECT ──(seleccionar / crear)──▶ MENU
                                                              │
       ┌──────────────────┬───────────────────┬───────────────┤
       ▼                  ▼                   ▼               ▼
   [Continuar]         [Jugar]           [Ver avance]     [Ajustes]
   (si hay en curso)      │                   │               │
       │                  ▼                   ▼               ▼
       │      (diálogo si hay partida     RECORDS         SETTINGS
       │       en curso) GUIDE (1.ª vez)                      │
       │                  │                                   │ (Ver guía → GUIDE → SETTINGS)
       │                  ▼
       └──────────▶  GAME: por cada compañero (5):
                       intro ─▶ [decide ─▶ result] × 6 rondas ─▶ guess
                                  │
                                  ▼
                              FEEDBACK ──▶ MENU
```

- **Sin callejones sin salida:** todas las pantallas secundarias (`GUIDE`, `SETTINGS`, `RECORDS`) incluyen un botón `BackButton` arriba a la izquierda para regresar a `MENU` (o a `SETTINGS` cuando `GUIDE` se abrió desde allí). `GAME` ofrece `Pausar y salir al menú` en todas sus fases.

| `ScreenId` | Propósito y comportamiento clave |
| :--- | :--- |
| `SPLASH` | Logotipo centrado de `TrustLoop` con fundido breve (respeta `reduceMotion` y `prefers-reduced-motion`). Sin botones; avanza automáticamente a `USER_SELECT` tras ~2.5 s. En paralelo precarga por IPC usuarios, perfil activo, ajustes e historial; ante cualquier fallo usa valores por defecto sin bloquear el arranque. |
| `USER_SELECT` | Lista perfiles locales aislados (nombre y última actividad) con botón eliminar por ícono y `aria-label`. Campo único «Ingresar o crear perfil» (máx. 32 caracteres; botón deshabilitado si está vacío o inválido) cuyo botón alterna entre `Ingresar` (si coincide) y `Crear perfil` (nuevo). Seleccionar con clic, `Enter` o `Espacio` lleva a `MENU`. Eliminar abre diálogo modal (`role="dialog"`, `aria-modal="true"`, `aria-labelledby`, foco atrapado, cierre con `Escape`); si se elimina el perfil activo, ajustes e historial vuelven a valores por defecto. |
| `MENU` | Botón superior izquierdo `Cambiar usuario` con el nombre del perfil activo (lleva a `USER_SELECT`). Marca `TrustLoop` centrada y **una fila centrada de botones cuadrados** con ícono y nombre corto: `Continuar` (solo si existe partida `in_progress`; primero y destacado, con línea secundaria «Compañero N de 5 · Ronda M»), `Jugar`, `Ver avance` y `Ajustes`. Incluye el aviso de alcance no evaluativo. |
| `GUIDE` | Guía de 3 pasos (`Paso N de 3`) con `Anterior`, `Siguiente` / `Empezar a jugar`, `Omitir` y `BackButton`: (1) cooperar o protegerse y la tabla de puntos, (2) cada compañero tiene un estilo que se descubre observando y al final de cada encuentro se elige el estilo supuesto, (3) durante la partida solo se ve el resultado de cada ronda; la retroalimentación llega al final. Al finalizar u omitir por primera vez marca `hasSeenGuide = true` y crea la partida (`GAME`); si se abrió desde `SETTINGS`, regresa a `SETTINGS`. |
| `SETTINGS` | Ajustes por perfil aplicados y guardados al instante (sin botón guardar): **Alto contraste**, **Tamaño de texto** (`normal` / `large`), **Reducir movimiento**, **Sonido** (interruptor) y **Volumen** (deslizador `0..1` con `aria-label`, deshabilitado cuando el sonido está apagado), **Ver guía de nuevo** (abre `GUIDE`) y **Vista previa en vivo** (§7). Botón `Volver al menú`. No hay ajustes propios de TrustLoop. |
| `RECORDS` | Historial del perfil activo: tabla de partidas con fecha, estado (`COMPLETADA` / `EN CURSO`), puntaje individual, puntaje grupal, nivel de cooperación, respuesta ante una traición y estilos descubiertos (`—` en lo que aún no se jugó o no aplica). Seleccionar una fila (clic, `Enter` o `Espacio`) muestra el detalle por compañero (estilo real, estilo supuesto, cooperación, puntos de cada uno) y por ronda. Una partida `in_progress` muestra solo lo jugado y no revela estilos aún no cerrados. Sin exportación de archivos. |
| `GAME` | Partida de 5 encuentros × 6 rondas (§5). Fases: `intro` (presenta al compañero N de 5 con su ilustración; `Empezar encuentro`, `Enter`/`Espacio`), `decide` (dos botones grandes `Cooperar` [`C`] y `Protegerme` [`P`]; sin límite de tiempo), `result` (revela ambas decisiones y los puntos de esa ronda; `Siguiente ronda`, `Enter`/`Espacio`) y `guess` (tras la ronda 6: «¿Cómo crees que juega <nombre>?» con 5 estilos + `No estoy seguro`; `Siguiente compañero` o `Ver retroalimentación` tras el último). Encabezado fijo con `Compañero N de 5`, `Ronda M de 6` y `Pausar y salir al menú`. Panel lateral con el historial de decisiones del encuentro actual (rondas jugadas y pendientes) y la tabla de puntos. No muestra acumulados, estilos ni valoraciones. |
| `FEEDBACK` | Retroalimentación de la partida completada: cuatro tarjetas (`Puntaje individual`, `Puntaje grupal`, `Nivel de cooperación`, `Respuesta ante una traición`), mensajes formativos del catálogo (§5.6), tabla por compañero (estilo revelado, estilo supuesto, tu cooperación, tus puntos, sus puntos) y detalle por ronda desplegable. Botones `Volver al menú` y `Jugar otra partida`. También se abre desde `RECORDS` para una partida completada. |

---

## 5. Reglas del juego

### 5.1 Estructura de la partida (S)

- **Compañeros por partida:** 5, uno por cada estilo (§5.3).
- **Rondas por encuentro:** 6, siempre visibles como «Ronda M de 6».
- **Total:** 30 rondas por partida (duración estimada de 10 a 15 minutos).
- **Orden y asignación:** al crear la partida se genera una semilla (`seed`). Con ella, la lógica pura baraja el orden de los 5 compañeros y asigna a cada uno un estilo distinto. El mismo personaje puede tener estilos distintos en partidas distintas, para que el estilo no se memorice por la ilustración.
- **Decisión simultánea:** el estudiante elige sin conocer la decisión del compañero de esa ronda; el compañero decide solo con el historial de rondas anteriores del encuentro.
- **Sin límite de tiempo** por ronda: la decisión es reflexiva.

### 5.2 Matriz de puntos por ronda (S)

| Estudiante \ Compañero | Coopera | Se protege |
| :--- | :--- | :--- |
| **Coopera** | Estudiante **3** · Compañero **3** | Estudiante **0** · Compañero **5** (traición) |
| **Se protege** | Estudiante **5** · Compañero **0** | Estudiante **1** · Compañero **1** |

- Cumple la estructura clásica del dilema (5 > 3 > 1 > 0 y 2 × 3 > 5 + 0): protegerse tienta en cada ronda aislada, pero la cooperación mutua sostenida da el mayor puntaje grupal.
- La matriz vive en `src/shared/payoffs.ts` y se muestra siempre en `GAME` (panel lateral) y en el paso 1 de `GUIDE`.

### 5.3 Estilos de los compañeros (S)

| `PartnerStyle` | Nombre en la interfaz | Regla de decisión en la ronda `r` del encuentro |
| :--- | :--- | :--- |
| `reciprocal` | Recíproco | Coopera en la ronda 1; desde la ronda 2 repite la decisión del estudiante en la ronda `r − 1`. |
| `always_cooperate` | Siempre coopera | Coopera en todas las rondas. |
| `always_protect` | Siempre se protege | Se protege en todas las rondas. |
| `random` | Impredecible | Sigue un plan de 6 decisiones generado con la semilla al crear la partida: exactamente 3 cooperaciones y 3 protecciones en orden barajado. No depende de lo que hace el estudiante. |
| `grudger` | Rencoroso | Coopera hasta que el estudiante se protege por primera vez en el encuentro; desde la ronda siguiente se protege en todas las rondas restantes. |

- El plan del estilo `random` se guarda en el registro de la partida (`randomPlan`) para que reanudar una partida reproduzca exactamente las mismas decisiones.
- Las descripciones breves de cada estilo que se muestran en `guess` y en `FEEDBACK` son neutras y describen conducta (por ejemplo, Recíproco: «Responde como lo trataste en la ronda anterior»).

### 5.4 Estilo supuesto

- Al terminar la ronda 6 de cada encuentro, el estudiante elige qué estilo cree que tenía el compañero (`GuessOption`: uno de los 5 estilos o `unsure`). La elección es obligatoria para avanzar, pero `No estoy seguro` siempre es válida.
- El estilo real **no** se revela en ese momento: solo en `FEEDBACK`.
- **Estilos descubiertos:** número de encuentros cuyo estilo supuesto coincide con el real (0 a 5). `unsure` cuenta como no descubierto.

### 5.5 Métricas registradas

Todas se calculan en `src/logic/` a partir de las rondas guardadas; nunca se editan a mano.

- **Decisiones tomadas:** cada ronda guarda la decisión del estudiante y la del compañero, y los puntos de cada uno.
- **Puntaje individual:** suma de los puntos del estudiante en las 30 rondas (rango 0–150).
- **Puntaje grupal:** suma de los puntos del estudiante y de todos los compañeros en las 30 rondas (rango 30–180; 180 si ambos cooperan siempre). Se muestra junto a su máximo posible.
- **Nivel de cooperación:** porcentaje de rondas en que el estudiante cooperó, global y por compañero (`cooperationRate`, `0..1`, mostrado como porcentaje entero).
- **Respuesta ante una traición:**
  - Una **traición** es una ronda `r` en que el estudiante cooperó y el compañero se protegió.
  - Para cada traición con `r < 6`, se observa la decisión del estudiante en la ronda `r + 1` del mismo encuentro: **volvió a cooperar** o **se protegió**.
  - Las traiciones en la ronda 6 se cuentan como recibidas pero no tienen respuesta observable (no entran en el denominador).
  - Se registran `betrayalsReceived`, `betrayalResponses.cooperated` y `betrayalResponses.protected`. En la interfaz se muestra «Tras una traición volviste a cooperar X de N veces» (N = `cooperated + protected`). Con N = 0 se muestra `—` y el texto «No hubo traiciones con respuesta en esta partida».
- **Estilos descubiertos:** según §5.4.

### 5.6 Retroalimentación

- **Durante la partida (fase `result`):** solo el resultado de la ronda, con texto descriptivo y sin valoración. Por ejemplo: «Tú cooperaste y Bruno se protegió. Tú +0 · Bruno +5». Cuatro casos: ambos cooperan, tú cooperas y el otro se protege, tú te proteges y el otro coopera, ambos se protegen. Cada caso se distingue por texto y forma, nunca solo por color.
- **Al final de la partida (`FEEDBACK`):** las cuatro métricas, la tabla por compañero y **tres mensajes formativos** elegidos por reglas deterministas desde `src/shared/feedbackTexts.ts` (sin IA ni texto generado):
  1. **Cooperación** según el nivel global: alto (≥ 67 %), medio (34–66 %) o bajo (≤ 33 %). Cada texto describe la consecuencia y propone una observación, sin calificar (p. ej. alto: «Cooperaste en la mayoría de las rondas. Mira en la tabla con qué compañeros eso sumó puntos para ambos y con cuáles no fue correspondido»).
  2. **Respuesta ante una traición**: mayoría de cooperación, mayoría de protección, empate o sin traiciones con respuesta.
  3. **Ajuste por compañero**: compara la cooperación del estudiante con `always_protect` frente a `always_cooperate` y `reciprocal`; si fue menor con `always_protect`, el texto reconoce que ajustó su conducta; si no, invita a revisar ese encuentro.
- Ningún mensaje usa adjetivos sobre la persona; todos hablan de decisiones, consecuencias y qué observar en la próxima partida.

---

## 6. Perfiles

- **Campo de perfil:** acepta un nombre o un código de estudiante.
- **Validación de formato:**
  - Longitud: `1` a `32` caracteres tras recortar espacios extremos (`trim()`).
  - Caracteres permitidos: letras (incluidas tildes y `ñ`/`Ñ`), dígitos (`0–9`), espacios, guion (`-`) y punto (`.`).
  - Cualquier otro carácter se rechaza antes de crear el perfil, mostrando un mensaje claro al usuario.
- **Coincidencia:** sin distinguir mayúsculas ni minúsculas (case-insensitive) tras recortar espacios extremos. Si coincide con un perfil existente, la acción ingresa a ese perfil; si no existe y es válido, lo crea.
- **Aislamiento y eliminación:** cada perfil tiene su propia carpeta `users/<userId>/` con sus ajustes e historial. Eliminar un perfil exige confirmación en diálogo modal y borra su carpeta; si era el perfil activo, `activeUserId` pasa a `null` y el estado en memoria vuelve a valores por defecto.

---

## 7. Ajustes

Todos los ajustes se aplican al instante en toda la aplicación y se persisten automáticamente en el perfil activo (sin botón guardar):

| Ajuste | Tipo / Valores | Comportamiento |
| :--- | :--- | :--- |
| **Alto contraste** | `boolean` (`data-contrast="high"` en raíz) | Activa la variante de alto contraste definida sobre los tokens CSS. Los colores concretos los define el diseñador. |
| **Tamaño de texto** | `'normal' \| 'large'` | Escala la tipografía de toda la interfaz. |
| **Reducir movimiento** | `boolean` (`reduceMotion`) | Suprime o simplifica el fundido de `SPLASH`, la aparición del resultado de la ronda y las transiciones entre compañeros. Respeta además `prefers-reduced-motion` del sistema. |
| **Sonido y volumen** | `audioEnabled: boolean`, `audioVolume: number` (`0..1`) | Silencia o activa los efectos sonoros y gradúa el volumen; el deslizador se deshabilita cuando el sonido está apagado. |
| **Ver guía de nuevo** | Acción | Abre `GUIDE` y regresa a `SETTINGS` al completar, omitir o pulsar volver. |

- **Vista previa en Ajustes:** tarjeta en vivo con una vista reducida de la fase `decide` (compañero de ejemplo, los dos botones de decisión y una fila de la tabla de puntos) que refleja de inmediato los cambios de alto contraste y tamaño de texto.
- **Sin ajustes propios de TrustLoop:** la dificultad, el número de rondas y los estilos son fijos para que todas las partidas sean comparables.

---

## 8. Persistencia

Toda la persistencia reside exclusivamente en `%LOCALAPPDATA%/TrustLoop/`:

```text
%LOCALAPPDATA%/TrustLoop/
├── users.json
└── users/
    └── <userId>/
        ├── settings.json
        └── matches.json
```

- **`users.json`:** lista de perfiles (`UserProfile[]`) y `activeUserId` (`string | null`).
- **`users/<userId>/settings.json`:** preferencias aisladas del perfil (`UserSettings`).
- **`users/<userId>/matches.json`:** historial de partidas (`MatchRecord[]`) del perfil, incluida la partida `in_progress` si existe (como máximo una).
- **Escritura atómica y concurrencia:** cada escritura crea un archivo temporal en el mismo directorio y ejecuta `rename` atómico, serializado mediante un lock por ruta de archivo para evitar corrupción ante escrituras rápidas consecutivas (cambios en `SETTINGS` o checkpoints).
- **Checkpoint:** se invoca `saveCheckpoint` (a) al crear la partida, (b) al resolver cada ronda (entrada a la fase `result`), (c) al confirmar el estilo supuesto de cada encuentro y (d) al completar la partida (`status = 'completed'`, `completedAt`). Al retomar se entra en el punto que indica `getResumeState` (§10), nunca a mitad de una fase `result`.

---

## 9. Contrato de datos (`src/shared/types.ts`) e IPC

### Tipos compartidos (`src/shared/types.ts`)

```ts
export type ScreenId =
  | 'SPLASH'
  | 'USER_SELECT'
  | 'MENU'
  | 'GUIDE'
  | 'SETTINGS'
  | 'RECORDS'
  | 'GAME'
  | 'FEEDBACK';

export type TextSize = 'normal' | 'large';
export type MatchStatus = 'in_progress' | 'completed';
export type GamePhase = 'intro' | 'decide' | 'result' | 'guess';
export type Choice = 'cooperate' | 'protect';
export type PartnerStyle =
  | 'reciprocal'
  | 'always_cooperate'
  | 'always_protect'
  | 'random'
  | 'grudger';
export type GuessOption = PartnerStyle | 'unsure';

export interface RoundRecord {
  round: number;              // 1..6 dentro del encuentro
  studentChoice: Choice;
  partnerChoice: Choice;
  studentPoints: number;      // 0 | 1 | 3 | 5
  partnerPoints: number;      // 0 | 1 | 3 | 5
  betrayal: boolean;          // estudiante coopera y compañero se protege
}

export interface EncounterRecord {
  index: number;              // 1..5 en el orden de la partida
  partnerId: string;          // id de src/shared/partners.ts
  style: PartnerStyle;
  randomPlan: Choice[] | null; // 6 decisiones solo para 'random'
  guess: GuessOption | null;  // null hasta cerrar el encuentro
  rounds: RoundRecord[];
}

export interface MatchMetrics {
  individualScore: number;
  groupScore: number;
  groupScoreMax: number;      // 6 × rondas jugadas
  cooperationRate: number;    // 0..1 sobre rondas jugadas
  betrayalsReceived: number;
  betrayalResponses: { cooperated: number; protected: number };
  stylesDiscovered: number;   // 0..5, solo encuentros cerrados
}

export interface MatchRecord {
  id: string;
  startedAt: string;
  completedAt: string | null;
  status: MatchStatus;
  seed: number;
  encounters: EncounterRecord[]; // siempre 5, creados al iniciar la partida
  metrics: MatchMetrics;         // recalculadas en cada checkpoint
}

export interface UserProfile {
  id: string;
  name: string;
  createdAt: string;
  lastActiveAt: string;
}

export interface UsersData {
  version: 1;
  activeUserId: string | null;
  users: UserProfile[];
}

export interface UserSettings {
  version: 1;
  highContrast: boolean;
  textSize: TextSize;
  reduceMotion: boolean;
  audioEnabled: boolean;
  audioVolume: number;
  hasSeenGuide: boolean;
}
```

### API IPC (`window.trustLoopApi`) y validación (`electron/ipcValidation.ts`)

Todos los canales expuestos por `preload.ts` validan argumentos, tipos y rangos en `ipcValidation.ts` antes de acceder al sistema de archivos:

| Método en `window.trustLoopApi` | Propósito |
| :--- | :--- |
| `getUsers(): Promise<UsersData>` | Lee `users.json` con la lista de perfiles y `activeUserId`. |
| `getActiveUser(): Promise<UserProfile \| null>` | Resuelve el perfil activo actual o `null`. |
| `selectUser(userId: string): Promise<UserProfile>` | Valida `userId`, actualiza `activeUserId` y `lastActiveAt`. |
| `createUser(name: string): Promise<UserProfile>` | Recorta espacios extremos, valida longitud (`1..32`) y caracteres permitidos, crea `users/<id>/` y archivos iniciales. |
| `deleteUser(userId: string): Promise<UsersData>` | Elimina el perfil y su carpeta `users/<id>/`; si era el activo deja `activeUserId = null`. |
| `getSettings(): Promise<UserSettings>` | Lee `users/<activeUserId>/settings.json` (o valores por defecto si no hay perfil activo). |
| `updateSettings(patch: Partial<UserSettings>): Promise<UserSettings>` | Valida tipos y rangos (`audioVolume` en `0..1`, `textSize` en `'normal' \| 'large'`, booleanos), aplica y persiste atómicamente. |
| `getMatches(): Promise<MatchRecord[]>` | Lee `users/<activeUserId>/matches.json`. |
| `saveCheckpoint(match: MatchRecord): Promise<MatchRecord[]>` | Valida el `MatchRecord` (5 encuentros, estilos distintos, rondas `1..6`, decisiones y puntos coherentes con la matriz, `randomPlan` presente solo en `random`, como máximo una partida `in_progress`) y lo persiste atómicamente. |
| `discardMatch(): Promise<MatchRecord[]>` | Descarta la partida `in_progress` del perfil activo cuando el usuario confirma empezar una nueva, conservando las completadas. |
| `getStoragePath(): Promise<string>` | Devuelve la ruta efectiva de `%LOCALAPPDATA%/TrustLoop`. |

---

## 10. Lógica pura (`src/logic/`)

Todas las reglas del juego, métricas, reanudación, destino inicial y validación de perfil se implementan como funciones puras en `src/logic/` (sin dependencias de Electron ni DOM), con pruebas unitarias:

- **`createMatch(seed, now)`:** genera el orden de los 5 compañeros, asigna un estilo distinto a cada uno y el `randomPlan` del estilo `random` (3 + 3 barajado), todo con un generador pseudoaleatorio determinista a partir de `seed`. Misma semilla → misma partida.
- **`payoff(student, partner)`:** devuelve `{ studentPoints, partnerPoints }` según §5.2.
- **`partnerDecision(style, previousRounds, randomPlan)`:** aplica la regla de §5.3 usando solo las rondas previas del encuentro.
- **`resolveRound(encounter, studentChoice)`:** obtiene la decisión del compañero, los puntos y `betrayal`, y devuelve el `RoundRecord`.
- **`computeMetrics(match)`:** recalcula `MatchMetrics` (§5.5) a partir de las rondas guardadas.
- **`selectFeedback(match)`:** elige los tres mensajes formativos del catálogo según §5.6.
- **Reanudación (`getResumeState(matches)`):** busca la partida `in_progress`. En el primer encuentro no cerrado (`guess === null`): con 0 rondas devuelve fase `'intro'`; con 1 a 5 rondas, fase `'decide'` en la ronda siguiente; con 6 rondas, fase `'guess'`. Si todos los encuentros están cerrados, devuelve `'FEEDBACK'` tras marcar la partida como completada.
- **Primera vez / guía (`getStartDestination(settings)`):** devuelve `'GUIDE'` si `!settings.hasSeenGuide`, o `'GAME'` en caso contrario.
- **Validación de perfil (`validateProfileName(name)`):** reglas de §6.

---

## 11. Convenciones técnicas

- **Tokens CSS:** los colores y variables visuales de la interfaz se consumen exclusivamente mediante tokens CSS (capa de abstracción para soportar `data-contrast="high"`, no para múltiples temas).
- **Frontera estética:** ninguna decisión estética (colores, tipografía, íconos, ilustraciones, grosores) se inventa en el código; proviene del diseñador externo. Mientras no lleguen las ilustraciones, se usa un marcador neutro con el nombre del compañero.
- **Accesibilidad (mouse y teclado):**
  - Todos los controles son operables con teclado. En `decide`, `C` elige Cooperar y `P` elige Protegerme (además de `Tab` + `Enter`); `Espacio` y `Enter` avanzan en presentaciones, resultados y transiciones.
  - Foco visible en todos los elementos interactivos; diálogos modales con `role="dialog"`, `aria-modal="true"`, `aria-labelledby`, foco atrapado y cierre con `Escape`.
  - El resultado de la ronda se anuncia en una región `aria-live="polite"`.
  - Contraste mínimo nivel AA en textos, etiquetas y controles.
  - La información visual nunca depende exclusivamente del color: cooperar y protegerse se distinguen por texto e ícono en botones, historial y tablas; los cuatro casos de resultado se distinguen por texto y forma del aviso.
  - Al ser una aplicación de escritorio operada con mouse y teclado, no existe requisito de área táctil mínima.
- **Seguridad en Electron:** `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true` y validación estricta de todo mensaje IPC en `electron/ipcValidation.ts`.
- **TypeScript estricto:** `strict: true` en `tsconfig.json`, sin usos de `any` injustificados.
- **Sin elevación:** ningún proceso de la aplicación ni del instalador solicita permisos de administrador.

---

## 12. Empaquetado

Configuración en `electron-builder.yml` para Windows (`win.requestedExecutionLevel: asInvoker`), generando artefactos en `release/`:

- **Instalador NSIS por usuario sin elevación:** `perMachine: false`, `allowElevation: false`, `oneClick: true` (`TrustLoop-Setup-${version}.exe`).
- **Portable:** ejecutable autónomo (`TrustLoop-${version}-portable.exe`).
- Ambos comparten la misma carpeta de datos `%LOCALAPPDATA%/TrustLoop`.

Scripts en `package.json`:

| Script | Propósito |
| :--- | :--- |
| `dev` | Arranca Vite y Electron en modo desarrollo |
| `dev:renderer` | Arranca solo el servidor de Vite para el renderer |
| `typecheck` | Verifica tipos de renderer y main sin emitir archivos |
| `test` | Ejecuta la suite de pruebas en Vitest (`vitest run`) |
| `build` | Compila renderer, main y preload |
| `pack` | Compila y genera el paquete sin instalador (`electron-builder --dir`) |
| `dist` | Compila y genera los instaladores NSIS y portable en `release/` |
| `docs:screenshots` | Genera capturas programáticas en `docs/capturas/` |

---

## 13. Capturas programáticas

El comando `pnpm docs:screenshots` (implementado en `scripts/capture-screenshots.mjs` y `scripts/capture-screenshots-main.cjs`, siguiendo el patrón de TheirView) realiza lo siguiente:

1. Verifica que exista la compilación en `dist-electron/`.
2. Crea un directorio temporal en `os.tmpdir()` y redirige `appData` / `userData` de Electron hacia él (nunca toca `%LOCALAPPDATA%/TrustLoop` real).
3. Lanza la aplicación compilada con una semilla fija, recorre las pantallas y estados relevantes (alto contraste, texto grande, las cuatro fases de `GAME`, los cuatro casos de resultado, `FEEDBACK`, historial y diálogos) y guarda los PNG en `docs/capturas/`.
4. Limpia el directorio temporal al finalizar.

---

## 14. Criterios de aceptación

1. Navegación completa entre las 8 pantallas sin callejones sin salida; toda pantalla secundaria cuenta con `BackButton` hacia `MENU` (o `SETTINGS` si `GUIDE` viene de allí) y `GAME` permite pausar y salir en cualquier fase.
2. Creación, selección y eliminación de perfiles por nombre o código de estudiante con validación de caracteres permitidos (`1..32`, letras con tildes/`ñ`, dígitos, espacios, `-` y `.`) y coincidencia sin distinguir mayúsculas.
3. Aislamiento total entre perfiles: cambiar de perfil carga sus propios ajustes e historial sin contaminar otros perfiles.
4. Alto contraste, tamaño de texto, reducir movimiento y sonido con volumen se aplican al instante, se reflejan en la vista previa de `SETTINGS` y persisten tras reiniciar.
5. Una partida tiene 5 encuentros × 6 rondas; cada compañero sigue exactamente la regla de su estilo y los puntos coinciden con la matriz de §5.2 en todas las combinaciones.
6. Durante la partida solo se muestra el resultado de la ronda y el historial de decisiones del encuentro actual: ni acumulados, ni estilos, ni valoraciones. El estilo real se revela solo en `FEEDBACK`.
7. `FEEDBACK` muestra puntaje individual, puntaje grupal (con su máximo), nivel de cooperación, respuesta ante una traición (o `—` sin traiciones con respuesta), estilos descubiertos y los tres mensajes formativos; ningún texto califica al estudiante.
8. Checkpoint y reanudación: cerrar la aplicación en cualquier fase y volver a abrirla ofrece `Continuar` en `MENU` y retoma en el punto de `getResumeState`, con las mismas decisiones del compañero `random`.
9. Empezar una partida nueva existiendo una `in_progress` exige confirmación en diálogo modal e invoca `discardMatch()`.
10. Suite de tests en Vitest cubriendo lógica pura (matriz, cada estilo, `createMatch` determinista, métricas incluidas traiciones en ronda 6, mensajes, reanudación, validación de perfiles), aislamiento entre perfiles y escritura atómica con lock por archivo.
11. Verificación funcional con carpeta de datos temporal en los tres escenarios: (1) desde cero, (2) con datos previos y (3) reingreso con el mismo perfil, además de `pnpm typecheck`, `pnpm test`, `pnpm build` y `pnpm docs:screenshots` en verde.

---

## 15. Decisiones tomadas

1. **Sin 3D ni zona generada por el programa:** toda la parte visual (pantallas, compañeros ilustrados, logo) la entrega el diseñador; el equipo de programación implementa lógica, contenido, datos y textos.
2. **Una sola pantalla de juego (`GAME`) con fases:** presentación, decisión, resultado y estilo supuesto comparten disposición para que la retroalimentación inmediata aparezca en el mismo lugar donde se decidió.
3. **Retroalimentación solo al final:** dentro de la partida se muestra únicamente el resultado de la ronda; las métricas, los estilos y los mensajes formativos se reservan para `FEEDBACK`.
4. **Partida determinista por semilla:** orden, asignación de estilos y plan del estilo `random` se derivan de `seed`, lo que permite reanudar sin cambios y probar con semillas fijas.
5. **Persistencia por checkpoint:** `saveCheckpoint` se invoca al crear la partida, al resolver cada ronda, al cerrar cada encuentro y al completar la partida.
6. **Descarte de partida en IPC (`discardMatch`):** iniciar una partida nueva con otra `in_progress` exige confirmación modal; se conserva como máximo una partida en curso por perfil.
7. **Vista previa en `SETTINGS`:** muestra una vista reducida de la fase de decisión que refleja al instante el contraste y el tamaño de texto.

---

## 16. Supuestos para revisar

Decisiones tomadas sin consulta, elegidas por ser las más conservadoras y coherentes pedagógicamente. Pueden cambiarse sin afectar la arquitectura (salvo donde se indica).

1. **Matriz de puntos** 3/3, 0/5, 5/0, 1/1 (valores clásicos del dilema del prisionero iterado).
2. **5 compañeros por partida**, uno por cada estilo, con **6 rondas** cada uno (30 rondas, ~10–15 min). Cambiar el número de rondas afecta `RoundRecord.round`, `groupScoreMax` y el plan del estilo `random`.
3. **Estilos:** Recíproco, Siempre coopera, Siempre se protege, Impredecible y Rencoroso, con las reglas de §5.3. El Impredecible tiene exactamente 3 y 3 para que su patrón «sin regla» sea reconocible en 6 rondas.
4. **Orden y asignación al azar por partida** (con semilla): el mismo personaje puede tener estilos distintos entre partidas.
5. **Nombres provisionales de los compañeros:** Alba, Bruno, Clara, Darío y Eva (texto del equipo de programación; ilustración del diseñador).
6. **El número de ronda es visible** («Ronda M de 6»): prioriza la claridad sobre el realismo del dilema con final incierto.
7. **Sin límite de tiempo** por ronda y sin registro de tiempos de respuesta.
8. **Historial del encuentro visible** durante la partida (solo decisiones de ambos por ronda, sin puntos acumulados), necesario para poder descubrir el estilo.
9. **Estilo supuesto** al final de cada encuentro, obligatorio con opción `No estoy seguro`; alimenta la métrica «Estilos descubiertos».
10. **Respuesta ante una traición** medida solo en la ronda inmediatamente siguiente y con el mismo compañero; las traiciones de la ronda 6 no tienen respuesta observable.
11. **Puntaje grupal** = estudiante + todos los compañeros; se muestra junto a su máximo (180).
12. **Umbrales de cooperación** de los mensajes: alto ≥ 67 %, medio 34–66 %, bajo ≤ 33 %.
13. **Atajos de teclado** `C` (Cooperar) y `P` (Protegerme).
14. **Sin ajustes propios de TrustLoop** ni modos de dificultad.
