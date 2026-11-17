# Especificación Técnica y Funcional — TrustLoop

> Única fuente de verdad técnica y funcional de **TrustLoop** (en la propuesta original, *Confía*). El flujo e interfaz funcional están alineados con [wireframe-trustloop.html](wireframe/wireframe-trustloop.html), que prevalece ante cualquier divergencia funcional mientras no exista el diseño visual definitivo. La estética (colores, tipografía, íconos, ilustraciones de los compañeros y grosores) la define el diseñador externo.
>
> Origen de las reglas: la propuesta de los cuatro software educativos (ficha *Confía* y elemento común) y su prototipo de referencia definen la idea esencial: matriz de puntos, estilos base, qué se mide, progresión de dificultad y autorreporte. Lo que esta especificación concreta sin que esas fuentes lo fijen está marcado con **(S)** y se lista en [Supuestos para revisar](#16-supuestos-para-revisar).

---

## 1. Propósito y alcance

**TrustLoop** es una aplicación de escritorio educativa en la que el estudiante universitario juega **partidas** de rondas con **compañeros virtuales**. En cada ronda decide entre **cooperar** o **protegerse**; cada compañero responde según un **estilo** propio que el estudiante debe descubrir. La actividad estimula:

- **La toma de decisiones en situaciones de interacción:** el resultado de cada ronda depende de lo que eligen ambos.
- **La cooperación y la confianza:** cooperar de forma sostenida produce más puntos para ambos cuando el otro también coopera.
- **El ajuste de la conducta:** reconocer cómo juega cada compañero, incluso cuando cambia, y adaptar las propias decisiones.

El estudiante no recibe instrucciones sobre cómo comportarse: juega y, al final, descubre cómo construye o rompe la confianza, cómo responde cuando lo traicionan y si prioriza la ganancia propia o la del grupo. El sistema registra automáticamente las decisiones tomadas, el nivel de cooperación, la respuesta ante una traición, los estilos identificados y el puntaje individual y grupal, y ofrece **retroalimentación al final de cada partida**. Dentro de la partida solo se muestra el resultado de cada ronda.

### Contexto operativo y restricciones de producto

- **Uso:** PC con Windows sin conexión a internet, en aula o laboratorio. Varios estudiantes comparten el mismo equipo mediante perfiles locales aislados.
- **Revisión:** presencial en la propia pantalla junto al docente.
- **Alcance no clínico:** herramienta de entrenamiento y reflexión socioemocional; **no** es una prueba psicométrica estandarizada, no mide la personalidad ni ofrece diagnósticos.
- **Tono e idioma:** el nombre del producto (`TrustLoop`) y la carpeta de datos (`%LOCALAPPDATA%/TrustLoop`) se mantienen siempre en inglés; toda la interfaz, instrucciones y retroalimentación se redactan en español con tono formativo, claro y no punitivo. Ningún texto califica al estudiante (prohibido «egoísta», «ingenuo», «tramposo», «perdedor» o equivalentes); los textos describen decisiones y consecuencias.
- **Sin red ni exportación:** prohibido exportar datos en CSV u otros archivos externos, y prohibido enviar información por red. La propuesta original contemplaba exportar a hoja de cálculo; queda **fuera de alcance** (ver §16).

### Glosario

| Término | Definición |
| :--- | :--- |
| **Compañero** | Personaje virtual ilustrado por el diseñador, con nombre propio, que juega contra el estudiante. |
| **Estilo** | Regla fija con la que un compañero decide en cada ronda (§5.3). El estudiante no la conoce hasta la retroalimentación. |
| **Cooperar** | Decisión que suma puntos para ambos si el otro también coopera, pero deja al estudiante sin puntos si el otro se protege. |
| **Protegerse** | Decisión que asegura un mínimo de puntos y aprovecha la cooperación del otro, pero reduce el puntaje grupal. |
| **Ronda** | Una decisión simultánea del estudiante y del compañero, con su resultado en puntos. |
| **Encuentro** | Serie de rondas consecutivas con el mismo compañero, cerrada con el estilo supuesto. |
| **Bloque** | Grupo de encuentros con la misma dificultad: Bloque 1 · Base y Bloque 2 · Avanzado (§5.1). |
| **Autorreporte** | Dos escalas de 1 a 5 (estado de ánimo y nivel de estrés o frustración) al inicio y al final de cada partida. |
| **Partida** | Ciclo completo: autorreporte inicial + Bloque 1 + Bloque 2 + autorreporte final + retroalimentación. |
| **Traición** | Ronda en la que el estudiante cooperó y el compañero se protegió. Término técnico de las métricas; en la interfaz se acompaña siempre de su descripción. |
| **Respuesta ante una traición** | Lo que decide el estudiante en la ronda siguiente a una traición con el mismo compañero (§5.5): volver a cooperar (perdonar) o protegerse. |

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

TrustLoop **no usa Three.js** ni ninguna escena 3D: toda la parte visual es interfaz e ilustraciones entregadas por el diseñador. Las ilustraciones se empaquetan localmente dentro del bundle, sin red ni CDN. Se reutiliza la base de TheirView: guardado local, motor de audio, instalador y portable, y estructura de pantallas.

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
│   │   ├── partners.ts         # Los 6 compañeros (id, nombre, recurso de ilustración)
│   │   ├── styles.ts           # Los 6 estilos: bloque, nombre y descripción neutra
│   │   ├── payoffs.ts          # Matriz de puntos por ronda
│   │   └── feedbackTexts.ts    # Catálogo de textos de retroalimentación
│   ├── logic/                  # Lógica pura (sin Electron ni DOM): estilos, puntos, métricas, reanudación
│   └── renderer/
│       ├── main.tsx            # Entrypoint React 18
│       ├── App.tsx             # Orquestador de las 10 pantallas
│       ├── assets/             # Ilustraciones de los compañeros y logo entregados por el diseñador
│       ├── audio/              # Motor de efectos sonoros
│       ├── components/         # Componentes reutilizables (p. ej. BackButton, diálogos, escala 1-5)
│       ├── hooks/              # Hooks de estado global sincronizado con IPC
│       ├── screens/            # Las 10 pantallas de la aplicación
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

La aplicación consta de **10 pantallas** (`ScreenId`). Los encuentros transcurren en `GAME`, que alterna cuatro fases (`GamePhase`): presentación del compañero, decisión, resultado de la ronda y estilo supuesto.

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
       │                 PRE (autorreporte inicial)
       │                  │
       └──────────▶  GAME: Bloque 1 (3 compañeros × 6 rondas) ─▶ Bloque 2 (3 compañeros × 5 rondas)
                     por compañero: intro ─▶ [decide ─▶ result] × rondas ─▶ guess
                                  │
                                  ▼
                     POST (autorreporte final) ──▶ FEEDBACK ──▶ MENU
```

- **Sin callejones sin salida:** todas las pantallas secundarias (`GUIDE`, `SETTINGS`, `RECORDS`) incluyen un botón `BackButton` arriba a la izquierda para regresar a `MENU` (o a `SETTINGS` cuando `GUIDE` se abrió desde allí). `PRE` ofrece `Volver al menú` (no crea la partida). `GAME` ofrece `Pausar y salir al menú` en todas sus fases.

| `ScreenId` | Propósito y comportamiento clave |
| :--- | :--- |
| `SPLASH` | Logotipo centrado de `TrustLoop` con fundido breve (respeta `reduceMotion` y `prefers-reduced-motion`). Sin botones; avanza automáticamente a `USER_SELECT` tras ~2.5 s. En paralelo precarga por IPC usuarios, perfil activo, ajustes e historial; ante cualquier fallo usa valores por defecto sin bloquear el arranque. |
| `USER_SELECT` | Lista perfiles locales aislados (nombre y última actividad) con botón eliminar por ícono y `aria-label`. Campo único «Ingresar o crear perfil» (máx. 32 caracteres; botón deshabilitado si está vacío o inválido) cuyo botón alterna entre `Ingresar` (si coincide) y `Crear perfil` (nuevo). Seleccionar con clic, `Enter` o `Espacio` lleva a `MENU`. Eliminar abre diálogo modal (`role="dialog"`, `aria-modal="true"`, `aria-labelledby`, foco atrapado, cierre con `Escape`); si se elimina el perfil activo, ajustes e historial vuelven a valores por defecto. |
| `MENU` | Botón superior izquierdo `Cambiar usuario` con el nombre del perfil activo (lleva a `USER_SELECT`). Marca `TrustLoop` centrada y **una fila centrada de botones cuadrados** con ícono y nombre corto: `Continuar` (solo si existe partida `in_progress`; primero y destacado, con línea secundaria «Compañero N de 6 · Ronda M»), `Jugar`, `Ver avance` y `Ajustes`. `Jugar` con una partida `in_progress` abre el diálogo de confirmación que llama a `discardMatch()`. Incluye el aviso de alcance no clínico. |
| `GUIDE` | Guía de 3 pasos (`Paso N de 3`) con `Anterior`, `Siguiente` / `Empezar`, `Omitir` y `BackButton`: (1) cooperar o protegerse y la tabla de puntos, (2) cada compañero tiene un estilo que se descubre observando; dos bloques, el segundo con menos rondas y estilos más difíciles, (3) antes y después de jugar se responde cómo te sientes; durante la partida solo se ve el resultado de cada ronda y la retroalimentación llega al final. Al finalizar u omitir por primera vez marca `hasSeenGuide = true` y avanza a `PRE`; si se abrió desde `SETTINGS`, regresa a `SETTINGS`. |
| `SETTINGS` | Ajustes por perfil aplicados y guardados al instante (sin botón guardar): **Alto contraste**, **Tamaño de texto** (`normal` / `large`), **Reducir movimiento**, **Sonido** (interruptor) y **Volumen** (deslizador `0..1` con `aria-label`, deshabilitado cuando el sonido está apagado), **Ver guía de nuevo** (abre `GUIDE`) y **Vista previa en vivo** (§7). Botón `Volver al menú`. No hay ajustes propios de TrustLoop. |
| `RECORDS` | Historial del perfil activo: tabla de partidas con fecha, estado (`COMPLETADA` / `EN CURSO`), puntaje individual, puntaje grupal, nivel de cooperación, respuesta ante una traición, estilos identificados y variación de ánimo y estrés inicial → final (`—` en lo que aún no se jugó o no aplica). Seleccionar una fila (clic, `Enter` o `Espacio`) muestra el detalle por compañero (bloque, estilo real, estilo supuesto, cooperación, puntos de cada uno) y por ronda. Una partida `in_progress` muestra solo lo jugado y no revela estilos. Sin exportación de archivos. |
| `PRE` | Autorreporte inicial: dos escalas obligatorias de 1 a 5, `Estado de ánimo` y `Nivel de estrés o frustración`. `Empezar partida` (`Enter`) se habilita al responder ambas y crea la partida (primer checkpoint); `Volver al menú` no crea nada. |
| `GAME` | Partida en dos bloques (§5.1). Fases: `intro` (presenta al compañero N de 6 con su ilustración, el bloque y las rondas; al empezar el Bloque 2 avisa que hay menos rondas y estilos más difíciles; `Empezar encuentro`, `Enter`/`Espacio`), `decide` (dos botones grandes `Cooperar` [`C`] y `Protegerme` [`P`]; sin límite de tiempo), `result` (revela ambas decisiones y los puntos de esa ronda; `Siguiente ronda`, `Enter`/`Espacio`) y `guess` (tras la última ronda: «¿Cómo crees que juega <nombre>?» con los 6 estilos + `No estoy seguro`). Encabezado fijo con bloque, `Compañero N de 6`, `Ronda M de R` y `Pausar y salir al menú`. Panel lateral con el historial de decisiones del encuentro actual y la tabla de puntos. No muestra acumulados, estilos ni valoraciones. |
| `POST` | Autorreporte final al cerrar el sexto encuentro: las mismas dos escalas. `Ver retroalimentación` (`Enter`) se habilita al responder ambas y marca la partida como `completed`. |
| `FEEDBACK` | Retroalimentación de la partida completada: cinco tarjetas (`Puntaje individual`, `Puntaje grupal`, `Nivel de cooperación`, `Confianza con quien coopera`, `Respuesta ante una traición`), estilos identificados, variación de ánimo y estrés, mensajes formativos del catálogo (§5.6), tabla por compañero (bloque, estilo revelado, estilo supuesto, tu cooperación, tus puntos, sus puntos) y detalle por ronda desplegable. Botones `Volver al menú` y `Jugar otra partida`. También se abre desde `RECORDS` para una partida completada. |

---

## 5. Reglas del juego

### 5.1 Estructura de la partida

La propuesta fija varios compañeros con estilos distintos y una dificultad que avanza con «estilos más difíciles de identificar, menos rondas para descubrirlos y, en niveles avanzados, compañeros que cambian de estilo a mitad de la partida». TrustLoop lo concreta así **(S)**:

| Bloque | Compañeros | Rondas por compañero | Estilos (§5.3) |
| :--- | :--- | :--- | :--- |
| **Bloque 1 · Base** | 3 | 6 | Siempre coopera, Recíproco, Oportunista (los tres de la propuesta) |
| **Bloque 2 · Avanzado** | 3 | 5 | Rencoroso, Impredecible, Cambiante |

- **Total:** 6 encuentros y 33 rondas por partida (estimado de 12 a 18 minutos con los autorreportes).
- **Orden y asignación:** al crear la partida se genera una semilla (`seed`). Con ella, la lógica pura baraja el orden de los estilos **dentro de cada bloque** (el Bloque 1 siempre va primero) y asigna los 6 estilos a los 6 personajes al azar, para que el estilo no se memorice por la ilustración.
- **Decisión simultánea:** el estudiante elige sin conocer la decisión del compañero de esa ronda; el compañero decide solo con el historial de rondas anteriores del encuentro (o con su plan fijo).
- **Ronda visible:** se muestra «Ronda M de R» **(S)**.
- **Sin límite de tiempo** por ronda: la decisión es reflexiva **(S)**.

### 5.2 Matriz de puntos por ronda

Valores de la propuesta y su prototipo:

| Estudiante \ Compañero | Coopera | Se protege |
| :--- | :--- | :--- |
| **Coopera** | Estudiante **3** · Compañero **3** | Estudiante **0** · Compañero **5** (traición) |
| **Se protege** | Estudiante **5** · Compañero **0** | Estudiante **1** · Compañero **1** |

- Cumple la estructura clásica del dilema (5 > 3 > 1 > 0 y 2 × 3 > 5 + 0): ninguna opción es siempre la correcta; protegerse tienta en cada ronda aislada, pero la cooperación mutua sostenida da el mayor puntaje grupal.
- La matriz vive en `src/shared/payoffs.ts` y se muestra siempre en `GAME` (panel lateral) y en el paso 1 de `GUIDE`.

### 5.3 Estilos de los compañeros

| `PartnerStyle` | Bloque | Nombre en la interfaz | Regla de decisión en la ronda `r` del encuentro |
| :--- | :--- | :--- | :--- |
| `always_cooperate` | 1 | Siempre coopera | Coopera en todas las rondas. *(Propuesta.)* |
| `reciprocal` | 1 | Recíproco | Coopera en la ronda 1; desde la ronda 2 repite la decisión del estudiante en la ronda `r − 1`. *(Propuesta: «devuelve lo que recibe».)* |
| `opportunist` | 1 | Oportunista | Coopera, salvo en las rondas 2 y 4, en las que se protege, haga lo que haga el estudiante. *(Propuesta: «a veces traiciona»; rondas tomadas del prototipo.)* |
| `grudger` | 2 | Rencoroso | Coopera hasta que el estudiante se protege por primera vez en el encuentro; desde la ronda siguiente se protege en todas las rondas restantes. **(S)** |
| `random` | 2 | Impredecible | Sigue un plan de 5 decisiones generado con la semilla al crear la partida: al menos 2 cooperaciones y 2 protecciones, en orden barajado. No depende de lo que hace el estudiante. **(S)** |
| `switcher` | 2 | Cambiante | Coopera en las rondas 1 y 2 y se protege desde la ronda 3 hasta el final: cambia de estilo a mitad del encuentro. *(Propuesta: «compañeros que cambian de estilo»; regla concreta **(S)**.)* |

- El plan del estilo `random` se guarda en el registro de la partida (`randomPlan`) para que reanudar reproduzca exactamente las mismas decisiones.
- Las descripciones breves de cada estilo que se muestran en `guess` y en `FEEDBACK` (en `src/shared/styles.ts`) son neutras y describen conducta (por ejemplo, Recíproco: «Responde como lo trataste en la ronda anterior»; Cambiante: «Empieza de una forma y a mitad del encuentro cambia»).

### 5.4 Estilo supuesto

- La propuesta mide «si logra identificar el estilo de cada compañero». Al terminar la última ronda de cada encuentro, el estudiante elige qué estilo cree que tenía el compañero (`GuessOption`: uno de los 6 estilos o `unsure`). La elección es obligatoria para avanzar, pero `No estoy seguro` siempre es válida.
- Se ofrecen los 6 estilos en ambos bloques, para no delatar el bloque del compañero **(S)**.
- El estilo real **no** se revela en ese momento: solo en `FEEDBACK`.
- **Estilos identificados:** número de encuentros cuyo estilo supuesto coincide con el real (0 a 6). `unsure` cuenta como no identificado.

### 5.5 Métricas registradas

Todas se calculan en `src/logic/` a partir de las rondas guardadas; nunca se editan a mano.

- **Decisiones tomadas:** cada ronda guarda la decisión del estudiante y la del compañero, y los puntos de cada uno.
- **Puntaje individual:** suma de los puntos del estudiante en las 33 rondas (rango 0–165).
- **Puntaje grupal:** suma de los puntos del estudiante y de todos los compañeros (rango 33–198; 198 si ambos cooperan siempre). Se muestra junto a su máximo posible. *(Propuesta: «puntaje propio y el del grupo».)*
- **Nivel de cooperación:** porcentaje de rondas en que el estudiante cooperó, global y por compañero (`cooperationRate`, `0..1`, mostrado como porcentaje entero).
- **Confianza con quien coopera:** porcentaje de rondas en que el estudiante cooperó con los compañeros `always_cooperate` y `reciprocal` (12 rondas), donde mantener la cooperación es lo que más le conviene al grupo. *(Propuesta: «si sabe mantener la confianza cuando le conviene al grupo»; fórmula **(S)**.)*
- **Respuesta ante una traición** *(propuesta: «si perdona después de una traición»)*:
  - Una **traición** es una ronda `r` en que el estudiante cooperó y el compañero se protegió **(S)**: si ambos se protegen no cuenta como traición (el prototipo contaba cualquier protección del compañero).
  - Para cada traición que no sea la última ronda del encuentro, se observa la decisión del estudiante en la ronda `r + 1` del mismo encuentro: **volvió a cooperar** (perdonó) o **se protegió**.
  - Las traiciones en la última ronda se cuentan como recibidas pero no tienen respuesta observable (no entran en el denominador).
  - Se registran `betrayalsReceived`, `betrayalResponses.cooperated` y `betrayalResponses.protected`. En la interfaz se muestra «Tras una traición volviste a cooperar X de N veces» (N = `cooperated + protected`). Con N = 0 se muestra `—` y el texto «No hubo traiciones con respuesta en esta partida».
- **Estilos identificados:** según §5.4.
- **Autorreporte:** ánimo y estrés o frustración inicial y final (1–5) y su variación. *(Elemento común de la propuesta.)*

### 5.6 Retroalimentación

- **Durante la partida (fase `result`):** solo el resultado de la ronda, con texto que describe y no valora. Por ejemplo: «Cooperaste y Alba se protegió. Tú sumas 0 y Alba suma 5 en esta ronda». Cuatro casos: ambos cooperan, tú cooperas y el otro se protege, tú te proteges y el otro coopera, ambos se protegen. Cada caso se distingue por texto y forma, nunca solo por color. No se muestran totales ni puntaje grupal acumulado (a diferencia del prototipo).
- **Al final de la partida (`FEEDBACK`):** resumen en lenguaje sencillo, pensado para la reflexión y no como calificación *(elemento común)*: las métricas, la tabla por compañero, la variación del autorreporte y **cuatro mensajes formativos** elegidos por reglas deterministas desde `src/shared/feedbackTexts.ts` (sin IA ni texto generado) **(S)**:
  1. **Cooperación** según el nivel global: alto (≥ 67 %), medio (34–66 %) o bajo (≤ 33 %). Describe la consecuencia y propone una observación.
  2. **Confianza con quien coopera**: si fue alta (≥ 67 %), reconoce que sostuvo la cooperación donde sumaba para ambos; si no, invita a mirar cuántos puntos grupales se dejaron de ganar con esos compañeros.
  3. **Respuesta ante una traición**: mayoría de cooperación, mayoría de protección, empate o sin traiciones con respuesta.
  4. **Ajuste al cambio**: con el compañero `switcher`, compara la cooperación del estudiante en las rondas 1–2 con las rondas 4–5; si bajó, el texto reconoce que notó el cambio; si no, invita a revisar ese encuentro.
- Ningún mensaje usa adjetivos sobre la persona; todos hablan de decisiones, consecuencias y qué observar en la próxima partida. La variación de ánimo y estrés se muestra sin interpretarla.

---

## 6. Perfiles

- **Campo de perfil:** acepta un nombre o un código de estudiante. Sin contraseña ni inicio de sesión.
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
- **Checkpoint:** se invoca `saveCheckpoint` (a) al confirmar `PRE` (crea la partida con `pre`), (b) al resolver cada ronda (entrada a la fase `result`), (c) al confirmar el estilo supuesto de cada encuentro y (d) al confirmar `POST` (`status = 'completed'`, `completedAt`). Al retomar se entra en el punto que indica `getResumeState` (§10), nunca a mitad de una fase `result`.

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
  | 'PRE'
  | 'GAME'
  | 'POST'
  | 'FEEDBACK';

export type TextSize = 'normal' | 'large';
export type MatchStatus = 'in_progress' | 'completed';
export type GamePhase = 'intro' | 'decide' | 'result' | 'guess';
export type Choice = 'cooperate' | 'protect';
export type BlockId = 1 | 2;
export type PartnerStyle =
  | 'always_cooperate'
  | 'reciprocal'
  | 'opportunist'
  | 'grudger'
  | 'random'
  | 'switcher';
export type GuessOption = PartnerStyle | 'unsure';

export interface SelfReport {
  mood: number;               // 1..5
  stress: number;             // 1..5 (estrés o frustración)
}

export interface RoundRecord {
  round: number;              // 1..6 en Bloque 1, 1..5 en Bloque 2
  studentChoice: Choice;
  partnerChoice: Choice;
  studentPoints: number;      // 0 | 1 | 3 | 5
  partnerPoints: number;      // 0 | 1 | 3 | 5
  betrayal: boolean;          // estudiante coopera y compañero se protege
}

export interface EncounterRecord {
  index: number;              // 1..6 en el orden de la partida
  block: BlockId;
  roundsTotal: number;        // 6 en Bloque 1, 5 en Bloque 2
  partnerId: string;          // id de src/shared/partners.ts
  style: PartnerStyle;
  randomPlan: Choice[] | null; // solo para 'random'
  guess: GuessOption | null;  // null hasta cerrar el encuentro
  rounds: RoundRecord[];
}

export interface MatchMetrics {
  individualScore: number;
  groupScore: number;
  groupScoreMax: number;      // 6 × rondas jugadas
  cooperationRate: number;    // 0..1 sobre rondas jugadas
  trustWithCooperators: number | null; // 0..1 con always_cooperate y reciprocal
  betrayalsReceived: number;
  betrayalResponses: { cooperated: number; protected: number };
  stylesIdentified: number;   // 0..6, solo encuentros cerrados
}

export interface MatchRecord {
  id: string;
  startedAt: string;
  completedAt: string | null;
  status: MatchStatus;
  seed: number;
  pre: SelfReport;
  post: SelfReport | null;
  encounters: EncounterRecord[]; // siempre 6, creados al iniciar la partida
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
| `saveCheckpoint(match: MatchRecord): Promise<MatchRecord[]>` | Valida el `MatchRecord` (autorreportes en `1..5`, 6 encuentros con estilos distintos y bloque coherente, rondas dentro de `roundsTotal`, decisiones y puntos coherentes con la matriz, `randomPlan` presente solo en `random`, como máximo una partida `in_progress`) y lo persiste atómicamente. |
| `discardMatch(): Promise<MatchRecord[]>` | Descarta la partida `in_progress` del perfil activo cuando el usuario confirma empezar una nueva, conservando las completadas. |
| `getStoragePath(): Promise<string>` | Devuelve la ruta efectiva de `%LOCALAPPDATA%/TrustLoop`. |

---

## 10. Lógica pura (`src/logic/`)

Todas las reglas del juego, métricas, reanudación, destino inicial y validación de perfil se implementan como funciones puras en `src/logic/` (sin dependencias de Electron ni DOM), con pruebas unitarias:

- **`createMatch(seed, pre, now)`:** genera los 6 encuentros (Bloque 1 primero, orden barajado dentro de cada bloque), asigna los estilos a los 6 personajes al azar y genera el `randomPlan` del estilo `random`, todo con un generador pseudoaleatorio determinista a partir de `seed`. Misma semilla → misma partida.
- **`payoff(student, partner)`:** devuelve `{ studentPoints, partnerPoints }` según §5.2.
- **`partnerDecision(style, previousRounds, roundNumber, randomPlan)`:** aplica la regla de §5.3 usando solo las rondas previas del encuentro o el plan fijo.
- **`resolveRound(encounter, studentChoice)`:** obtiene la decisión del compañero, los puntos y `betrayal`, y devuelve el `RoundRecord`.
- **`computeMetrics(match)`:** recalcula `MatchMetrics` (§5.5) a partir de las rondas guardadas.
- **`selectFeedback(match)`:** elige los cuatro mensajes formativos del catálogo según §5.6.
- **Reanudación (`getResumeState(matches)`):** busca la partida `in_progress`. En el primer encuentro no cerrado (`guess === null`): con 0 rondas devuelve fase `'intro'`; con rondas por debajo de `roundsTotal`, fase `'decide'` en la ronda siguiente; con todas las rondas, fase `'guess'`. Si los 6 encuentros están cerrados, devuelve `'POST'`.
- **Primera vez / guía (`getStartDestination(settings)`):** devuelve `'GUIDE'` si `!settings.hasSeenGuide`, o `'PRE'` en caso contrario.
- **Validación de perfil (`validateProfileName(name)`):** reglas de §6.

---

## 11. Convenciones técnicas

- **Tokens CSS:** los colores y variables visuales de la interfaz se consumen exclusivamente mediante tokens CSS (capa de abstracción para soportar `data-contrast="high"`, no para múltiples temas).
- **Frontera estética:** ninguna decisión estética (colores, tipografía, íconos, ilustraciones, grosores) se inventa en el código; proviene del diseñador externo. Mientras no lleguen las ilustraciones, se usa un marcador neutro con el nombre del compañero.
- **Accesibilidad (mouse y teclado):**
  - Todos los controles son operables con teclado. En `decide`, `C` elige Cooperar y `P` elige Protegerme (además de `Tab` + `Enter`); en las escalas, las teclas `1` a `5` eligen el valor; `Espacio` y `Enter` avanzan en presentaciones, resultados y transiciones.
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
3. Lanza la aplicación compilada con una semilla fija, recorre las pantallas y estados relevantes (alto contraste, texto grande, `PRE`, las cuatro fases de `GAME` en ambos bloques, los cuatro casos de resultado, `POST`, `FEEDBACK`, historial y diálogos) y guarda los PNG en `docs/capturas/`.
4. Limpia el directorio temporal al finalizar.

---

## 14. Criterios de aceptación

1. Navegación completa entre las 10 pantallas sin callejones sin salida; toda pantalla secundaria cuenta con `BackButton` hacia `MENU` (o `SETTINGS` si `GUIDE` viene de allí) y `GAME` permite pausar y salir en cualquier fase.
2. Creación, selección y eliminación de perfiles por nombre o código de estudiante con validación de caracteres permitidos (`1..32`, letras con tildes/`ñ`, dígitos, espacios, `-` y `.`) y coincidencia sin distinguir mayúsculas.
3. Aislamiento total entre perfiles: cambiar de perfil carga sus propios ajustes e historial sin contaminar otros perfiles.
4. Alto contraste, tamaño de texto, reducir movimiento y sonido con volumen se aplican al instante, se reflejan en la vista previa de `SETTINGS` y persisten tras reiniciar.
5. Autorreporte inicial y final obligatorios (ánimo y estrés o frustración, 1–5); la variación se muestra en `FEEDBACK` y en `RECORDS`.
6. Una partida tiene Bloque 1 (3 compañeros × 6 rondas) y Bloque 2 (3 compañeros × 5 rondas); cada compañero sigue exactamente la regla de su estilo y los puntos coinciden con la matriz de §5.2 en todas las combinaciones.
7. Durante la partida solo se muestra el resultado de la ronda y el historial de decisiones del encuentro actual: ni acumulados, ni estilos, ni valoraciones. El estilo real se revela solo en `FEEDBACK`.
8. `FEEDBACK` muestra puntaje individual, puntaje grupal (con su máximo), nivel de cooperación, confianza con quien coopera, respuesta ante una traición (o `—` sin traiciones con respuesta), estilos identificados, variación del autorreporte y los cuatro mensajes formativos; ningún texto califica al estudiante.
9. Checkpoint y reanudación: cerrar la aplicación en cualquier fase y volver a abrirla ofrece `Continuar` en `MENU` y retoma en el punto de `getResumeState`, con las mismas decisiones del compañero `random`.
10. Empezar una partida nueva existiendo una `in_progress` exige confirmación en diálogo modal e invoca `discardMatch()`.
11. Suite de tests en Vitest cubriendo lógica pura (matriz, cada estilo, `createMatch` determinista y con el Bloque 1 primero, métricas incluidas traiciones en la última ronda, mensajes, reanudación incluida `POST`, validación de perfiles), aislamiento entre perfiles y escritura atómica con lock por archivo.
12. Verificación funcional con carpeta de datos temporal en los tres escenarios: (1) desde cero, (2) con datos previos y (3) reingreso con el mismo perfil, además de `pnpm typecheck`, `pnpm test`, `pnpm build` y `pnpm docs:screenshots` en verde.

---

## 15. Decisiones tomadas

1. **Alineación con la propuesta original:** matriz 3/3, 0/5, 5/0, 1/1; los tres estilos base (siempre coopera, devuelve lo que recibe, a veces traiciona); qué se mide (cooperación, identificar estilos, perdón tras traición, confianza que conviene al grupo, puntaje propio y grupal); dificultad progresiva con un compañero que cambia de estilo; autorreporte de inicio y fin; resumen final para la reflexión e historial por usuario.
2. **Sin exportación:** se descarta la exportación a hoja de cálculo de la propuesta para cumplir la restricción de producto «sin red ni exportación»; la revisión es en pantalla.
3. **Sin 3D ni zona generada por el programa:** toda la parte visual (pantallas, compañeros ilustrados, logo) la entrega el diseñador; el equipo de programación implementa lógica, contenido, datos y textos.
4. **Una sola pantalla de juego (`GAME`) con fases:** presentación, decisión, resultado y estilo supuesto comparten disposición para que la retroalimentación inmediata aparezca en el mismo lugar donde se decidió. La distribución del prototipo (puntajes acumulados en la cabecera) no se usa.
5. **Retroalimentación solo al final:** dentro de la partida se muestra únicamente el resultado de la ronda; las métricas, los estilos y los mensajes formativos se reservan para `FEEDBACK`.
6. **Partida determinista por semilla:** orden, asignación de estilos y plan del estilo `random` se derivan de `seed`, lo que permite reanudar sin cambios y probar con semillas fijas.
7. **Persistencia por checkpoint:** `saveCheckpoint` se invoca al confirmar `PRE`, al resolver cada ronda, al cerrar cada encuentro y al confirmar `POST`.
8. **Descarte de partida en IPC (`discardMatch`):** iniciar una partida nueva con otra `in_progress` exige confirmación modal; se conserva como máximo una partida en curso por perfil.
9. **Vista previa en `SETTINGS`:** muestra una vista reducida de la fase de decisión que refleja al instante el contraste y el tamaño de texto.

---

## 16. Supuestos para revisar

Decisiones tomadas sin consulta, elegidas por ser las más conservadoras y coherentes pedagógicamente. Lo que la propuesta ya fija (matriz, estilos base, métricas, autorreporte) no figura aquí.

1. **Estructura en dos bloques:** Bloque 1 · Base con 3 compañeros × 6 rondas y Bloque 2 · Avanzado con 3 compañeros × 5 rondas (6 encuentros, 33 rondas). El prototipo usaba 3 compañeros × 5 rondas; la propuesta solo dice «varias rondas» y «menos rondas» en niveles avanzados. Cambiar estos números afecta `roundsTotal`, `groupScoreMax` y el plan del estilo `random`.
2. **Estilos avanzados:** Rencoroso, Impredecible y Cambiante, con las reglas de §5.3. «Siempre se protege», mencionado como ejemplo en el encargo, no se incluye como estilo propio: su conducta aparece en la segunda mitad del Cambiante.
3. **Regla del Oportunista:** se protege en las rondas 2 y 4 (tomado del prototipo) sin depender del estudiante.
4. **Regla del Cambiante:** coopera en rondas 1–2 y se protege desde la 3.
5. **Impredecible:** al menos 2 cooperaciones y 2 protecciones en 5 rondas, para que su patrón «sin regla» sea reconocible.
6. **Orden:** Bloque 1 siempre primero; orden barajado dentro de cada bloque y personajes asignados a estilos al azar (con semilla).
7. **Nombres provisionales de los compañeros:** Alba, Bruno, Clara, Darío, Eva y Fabián (texto del equipo de programación; ilustración del diseñador).
8. **Número de ronda visible** («Ronda M de R»): prioriza la claridad sobre el realismo del dilema con final incierto.
9. **Sin límite de tiempo** por ronda y sin registro de tiempos de respuesta.
10. **Historial del encuentro visible** durante la partida (solo decisiones de ambos por ronda, sin puntos acumulados), necesario para poder identificar el estilo.
11. **Identificar el estilo** se mide con una elección explícita al final de cada encuentro, obligatoria con opción `No estoy seguro`, ofreciendo siempre los 6 estilos.
12. **Traición** = el estudiante coopera y el compañero se protege (el prototipo contaba cualquier protección del compañero); la respuesta se mide solo en la ronda siguiente con el mismo compañero.
13. **Confianza con quien coopera** = cooperación del estudiante con Siempre coopera y Recíproco.
14. **Puntaje grupal** = estudiante + todos los compañeros; se muestra junto a su máximo (198).
15. **Mensajes formativos:** cuatro, con umbrales de 67 % y 33 %.
16. **Autorreporte:** «Estado de ánimo» (1 · Muy bajo … 5 · Muy bueno) y «Nivel de estrés o frustración» (1 · Muy tranquilo … 5 · Muy tenso); sin interpretación de la variación.
17. **Sin exportación** pese a que la propuesta la contemplaba (restricción de producto); el docente revisa en pantalla.
18. **Solo efectos de sonido**, sin música, aunque la base reutilizable de la propuesta incluye música.
19. **Atajos de teclado** `C` (Cooperar), `P` (Protegerme) y `1`–`5` en las escalas.
20. **Sin ajustes propios de TrustLoop** ni modos de dificultad seleccionables.
