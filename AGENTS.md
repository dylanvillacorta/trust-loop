# AGENTS.md

Reglas base para todo agente (humano o IA) que trabaja en este repositorio. Este archivo define **quién hace qué**
y **cuándo se detiene un agente**.

- **Parte I (secciones 1 a 4):** núcleo portable, idéntico entre repositorios.
- **Parte II (secciones 5 en adelante):** adapta y concreta para este repositorio. No repite la especificación del
  producto: remite a ella.

Jerarquía ante conflicto: **AGENTS.md > skills > docs**. Dentro de `AGENTS.md`, la Parte II puede agregar puertas o
concretar valores, pero no relajar la Parte I. Ningún otro archivo redefine la Política de autonomía (§2).

---

# Parte I — Núcleo portable

## 1. Roles

| Rol | Dónde trabaja | Hace | No hace |
| :--- | :--- | :--- | :--- |
| **Coordinador** | Workspace raíz (`develop` o rama principal) | Planifica, escribe prompts, lanza y monitorea worktrees, revisa diffs, integra a `develop` (o abre PRs según §3), prepara releases. | Escribir código, docs o configs en la raíz. |
| **Agente de worktree** | Su worktree de Orca (`feature/*`, `fix/*`, `release/*`) | Implementa, prueba, corrige y commitea en su propia rama. | Tocar `develop`, `main` u otros worktrees. |

**Regla Cero:** la raíz solo admite lectura, `merge` y `push` de `develop`, y servicios locales de desarrollo (ver
Parte II). Todo cambio de archivos —código, docs o skills—, incluida la preparación del release, ocurre en un worktree (§4).

---

## 2. Política de autonomía (SSOT)

Por defecto **todo es autónomo**. Esta política aplica salvo que la Parte II agregue puertas adicionales. Un agente se
detiene a esperar confirmación del usuario **solo** en estas puertas:

| # | Puerta (detenerse y esperar un sí explícito) | Por qué |
| :--- | :--- | :--- |
| G1 | **Puerta de producción (publicar):** cualquier escritura en `main` (merge del PR, commit, `push`), crear o subir un tag, publicar un GitHub Release, publicar paquetes o imágenes de contenedores (GitHub Packages, GHCR u otro registro) y ejecutar scripts que escriben en producción (ver Parte II). | Publicar entrega el software a sus usuarios o a producción. Hay proyectos que despliegan desde `main` y otros desde el tag, el release o el paquete: ninguno se publica solo. |
| G2 | **Puerta de cuota:** usar la cuota propia del agente coordinador cuando se agotaron los modelos delegados (§4). | La cuota del coordinador es el último recurso. |
| G3 | **Dependencias:** subir una dependencia de versión mayor o agregar una dependencia nueva en cualquier manifiesto de dependencias (ver Parte II). | Cambia la superficie de mantenimiento a largo plazo. |

**Todo lo demás procede sin preguntar**, incluido:
- Commits, merges y `push` en ramas de trabajo (`feature/*`, `fix/*`, `release/*`) y en `develop`.
- `fetch` y `pull`.
- Crear ramas de trabajo.
- Abrir, actualizar y comentar PRs (incluido el PR hacia `main`).
- Esperar checks de CI.
- **Preparar un release completo** (§3): rama `release/vX.Y.Z`, versión, build interna de artefactos, notas y PR hacia
  `main` en verde. Preparar no es publicar.
- Crear y eliminar worktrees y ramas locales ya integradas.
- Resolver conflictos en la rama de trabajo.
- Levantar, recrear o vaciar los servicios o stack local de desarrollo (ver Parte II); migraciones locales; dependencias en versión menor o parche.

### Cómo se actúa
- **No preguntar "¿continúo?" ni "¿procedo?".** Anunciar en una línea lo que se va a hacer y hacerlo.
- **Ambigüedad de negocio:** elegir la opción más conservadora y coherente con la documentación del proyecto (ver Parte II),
  continuar y registrarla en la sección *Supuestos* del reporte. El usuario la revisa después; no bloquea el trabajo.
- **Bloqueo real** (credencial faltante, servicio externo caído): reportarlo y seguir con lo que no depende de él.
- **Al llegar a una puerta:** preparar todo lo previo, presentar el mensaje de puerta (§3 o §4) y esperar un sí explícito.
  Una aprobación cubre solo la acción presentada.

### Prohibido

Nunca, ni con aprobación implícita:
- Force push (`git push --force` / `--force-with-lease`) o reescritura de historial publicado (`git commit --amend`,
  `rebase`, `reset --hard`, `filter-branch`, `filter-repo`) en cualquier rama; se corrige con commits nuevos.
- Borrar `develop` o `main`, o ejecutar `git push --delete` sobre cualquier rama remota (las ramas en `origin` las
  gestiona el usuario).
- Modificar `user.name` o `user.email` en la configuración de Git.
- Agregar atribución de IA a commits o PRs.

---

## 3. Git y GitHub

### Commits
- Conventional Commits en español: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, con scope opcional
  (p. ej. `fix(api): ...`). Merges a `develop`: `merge: <tarea>`. Release: `release: vX.Y.Z (<resumen>)`.
- Firma solo humana: no tocar `user.name`/`user.email`; sin `Co-Authored-By`, `Signed-off-by`, "Generated with" ni
  menciones a modelos o herramientas, tampoco en descripciones de PR.
- Toda escritura de Git delegada o en la raíz usa `git -C <ruta>` y verifica antes `git -C <ruta> rev-parse --show-toplevel`
  y `git -C <ruta> branch --show-current`; si no coinciden con lo esperado, aborta. Un `cd` fallido puede publicar una
  rama protegida por error.

### Flujo con `develop` (GitFlow)

```
feature/* ─┐
fix/*     ─┴─> develop ──> release/vX.Y.Z ──PR──> main (tag vX.Y.Z + release)
                    ▲                                        │
                    └──────────────── back-merge ────────────┘
```

- Todo nace de `origin/develop` y vuelve a `develop`.
- `main` solo recibe el PR `release/vX.Y.Z → main`.
- Sin hotfixes directos a `main`: una urgencia es un `fix/*` que pasa por `develop` y sale en un release.
- Si se usan worktrees de Orca, el orquestador puede anteponer un prefijo de usuario a la rama física; usar siempre el
  nombre que devuelve `result.worktree.branch` al crear el worktree.

#### Integrar una rama de trabajo (sin puerta)

Requisito: verificación del componente tocado en verde (ver Parte II) y `git status -s` limpio.

```powershell
git -C <raíz> fetch origin
# ¿La rama ya contiene develop actual?
git -C <raíz> merge-base --is-ancestor origin/develop <rama>
```

- **Sí (exit 0):** lo que se probó es exactamente lo que se integra.
  `git -C <raíz> merge --no-ff <rama> -m "merge: <tarea>"` y `git -C <raíz> push origin develop`. Sin atribución de IA en
  commits ni mensajes.
- **No:** el resultado del merge no está probado. Se devuelve a la rama de trabajo (`git merge origin/develop` allí,
  resolviendo conflictos ahí y no en la raíz), se re-verifica y se repite.

Después, si la rama vino de un worktree: tras confirmar `git -C <raíz> merge-base --is-ancestor <rama> develop`, eliminar
el worktree y la rama local (sin puerta, §4). Nunca borrar la rama remota.

#### Preparar el release (sin puerta)

**Cuándo:** al completarse todos los ítems del backlog planificado (ver Parte II) o cuando el usuario lo ordena. No se
pide confirmación para prepararlo: el objetivo es que el usuario encuentre el producto terminado, listo para probar.

1. `develop` local igual a `origin/develop` y sin trabajo pendiente de integrar para esta versión.
2. Versión: semver sobre el último tag (`git describe --tags --abbrev=0`): `fix` → parche, `feat` → menor (o `v0.1.0`
   si no hay tags).
3. Crear el worktree `release/vX.Y.Z` desde `origin/develop` (§4). En él: subir la versión en el manifiesto, pasar la
   verificación completa, construir los artefactos (ver Parte II) y redactar las notas del release en español.
4. Hacer `push` de la rama, abrir o actualizar el PR y esperar sus checks:
   ```powershell
   gh pr create --base main --head <rama-release> --title "release: vX.Y.Z (<resumen>)" --body "<notas>"
   gh pr checks <n> --watch
   ```
5. Si la verificación, un check o la build fallan, se corrige en la misma rama `release/*` y se reconstruye. No se llega
   a la Puerta de producción (G1) con nada en rojo.
6. Los artefactos quedan construidos en el worktree de release y **no se suben a ningún sitio**: ni tag, ni GitHub
   Release, ni registro de paquetes o contenedores. Eso es publicar (G1).

#### Puerta de producción (G1) — mensaje y ejecución

Con todo en verde, el agente se detiene y presenta **un solo mensaje**:

```
Release vX.Y.Z → producción          PR #<n> · checks: verde
Commits:     git log --oneline origin/main..<rama-release>   (lista)
Cambios:     git diff --stat origin/main...<rama-release>     (resumen)
Artefactos:  <rutas locales para probar el producto terminado>
Notas:       <notas del release>
Supuestos pendientes de revisión: <lista o "ninguno">
¿Apruebas el merge a main, el tag vX.Y.Z y la publicación del release?
```

El usuario puede probar los artefactos con calma antes de responder. La aprobación cubre exactamente ese PR, ese tag y
esos artefactos. Si la rama de release cambia después, se reconstruye y se vuelve a presentar.

Tras el sí:

```powershell
gh pr merge <n> --merge --subject "release: vX.Y.Z (<resumen>)"
git -C <raíz> fetch origin --tags
git -C <raíz> tag -a vX.Y.Z origin/main -m "Release vX.Y.Z: <resumen>"
git -C <raíz> push origin vX.Y.Z
gh release create vX.Y.Z <artefactos> --title "<Producto> vX.Y.Z" --notes "<notas>"
# Back-merge obligatorio (sin puerta), para que develop no quede detrás de main:
git -C <raíz> merge --no-ff origin/main -m "chore: sincronizar release vX.Y.Z hacia develop"
git -C <raíz> push origin develop
```

Paquetes o imágenes de contenedores (si el proyecto los usa, ver Parte II) se publican en este mismo paso y con la misma
aprobación, nunca antes. Después se limpia el worktree de release (§4).

### Flujo sin `develop` (repos que solo usan `main`)

Flujo directo por PR hacia `main`:

1. **Rama de trabajo (sin puerta):** `git checkout -b <tipo>/<tarea> origin/main`.
2. **Commits y push (sin puerta):** commitear según las reglas de commits de esta sección, verificar y hacer `push` de la
   rama (`git push -u origin <rama>`).
3. **PR y checks (sin puerta):**
   ```powershell
   gh pr create --base main --head <rama> --title "<título>" --body "<descripción>"
   gh pr checks <n> --watch
   ```
   Si un check falla, corregir en la misma rama y hacer `push`.
4. **Puerta de producción (G1):** el merge del PR es la única puerta. Con checks en verde, presentar **un solo mensaje**:

```
Integrar <rama> → producción         PR #<n> · checks: verde
Commits:     git log --oneline origin/main..<rama>   (lista)
Cambios:     git diff --stat origin/main...<rama>     (resumen)
Supuestos pendientes de revisión: <lista o "ninguno">
¿Apruebas el merge a main?
```

La aprobación cubre exactamente ese PR. Tras el sí: `gh pr merge <n> --merge`. Tag, release y paquetes siguen la misma
regla que en el flujo con `develop`: solo con aprobación.

---

## 4. Orca (delegación en worktrees)

Cómo el Coordinador convierte una tarea en código integrado sin escribirlo él: delega en **Antigravity** (`agy`)
dentro de worktrees de Orca.

### Ciclo de una tarea

```
planear → crear worktree (implementación) → esperar → QA en el mismo worktree → leer reporte
        → integrar a develop → limpiar
```

Un worktree por tarea independiente. Implementación y QA comparten worktree: nunca se separan en ramas distintas.

### Modelos

| Fase | `--model` | `--effort` |
| :--- | :--- | :--- |
| Scaffolding, maquetado, tipos, docs, commits | `gemini-4-hydrogen-low` | `low` |
| Lógica de negocio, transacciones, refactor | `gemini-4-hydrogen-medium` | `medium` |
| QA: tests, casos borde, corrección in situ | `gemini-4-hydrogen-high` | `high` |
| Planificación delegada (si aplica) | `gemini-4-hydrogen-high` | `high` |
| Planificación o diagnóstico complejo | `gemini-4-hydrogen-xhigh` | `xhigh` |

### Fallback por cuota y Puerta de cuota (G2)

Tras cada fase, leer la terminal. Si aparece `Individual quota reached`, relanzar **la misma fase, mismo prompt, mismo
worktree** con el siguiente modelo:

1. Gemini 4 Hydrogen según la tabla de modelos.
2. Dentro de Antigravity: `claude-sonnet-5-5-low` o `claude-opus-5-5-low` (planificación: también `-medium`).
3. Subagentes propios del Coordinador → **Puerta de cuota (G2)** (§2): detenerse y pedir aprobación explícita al
   usuario antes de usarlos. Volver a Antigravity cuando se renueve la cuota.

### El prompt

Todo prompt de implementación incluye:

1. **Objetivo** en una frase y el *porqué* de negocio.
2. **Fronteras:** archivos o carpetas que puede tocar. Nada fuera de ellas.
3. **Criterios de aceptación** verificables.
4. **Verificación** a correr (los comandos de verificación del componente tocado, ver Parte II) y **commit** en su rama
   con Conventional Commits y `git -C` (§3).
5. **Autonomía:** "No preguntes; ante una ambigüedad elige lo más conservador según la documentación del proyecto y
   anótalo en Supuestos."
6. **Reporte final** con este formato exacto:
   ```
   Estado: listo | bloqueado (motivo)
   Cambios: <archivos y qué se hizo>
   Verificación: <comando> → <resultado>
   Supuestos: <decisiones tomadas sin consultar, o ninguno>
   Pendientes: <lo que quedó fuera, o ninguno>
   ```

El prompt de QA agrega: revisar el diff contra los criterios, cubrir casos borde, corregir en el mismo worktree y
re-verificar. QA commitea sus correcciones; no integra ni hace push a `develop`.

Formato del texto (se pasa a `agy` por PowerShell 5.1): **sin comillas dobles, backticks ni emoji**; usar «» o nada.
Un prompt largo se escribe en un archivo y se envía desde Bash con `--prompt "$(cat archivo)"`.

### Comandos y monitoreo

```powershell
# Crear worktree con el agente de implementación
orca worktree create --name feature/<tarea> --base-branch origin/develop --agent antigravity `
  --model <modelo> --effort <nivel> --prompt "<prompt>" --no-parent --activate --json
# Guardar result.agentTerminalHandle, result.worktree.branch y la ruta del worktree. Selector: path:<ruta> (name:<rama> no resuelve).
# Orca puede anteponer un prefijo de usuario a la rama física: usar siempre result.worktree.branch.

# Fase siguiente (QA o fallback): nuevo agente en el mismo worktree
orca agent launch --agent antigravity --worktree path:<ruta> --model <modelo> --effort <nivel> --prompt "<prompt>" --focus --json

# Seguimiento en la misma TUI
orca terminal send --terminal <h> --text "<texto>" --enter --json   # sin --enter no se envía

# Esperar
orca terminal wait --terminal <h> --for tui-idle --timeout-ms <ms>
orca terminal read --terminal <h>
```

Reglas de monitoreo:
- Tras lanzar, `terminal read` una vez para confirmar que el agente arrancó; si `agy` reporta `unexpected argument`,
  el prompt tiene caracteres prohibidos (ver formato del prompt arriba).
- `tui-idle` puede volver antes de tiempo: si las últimas líneas contienen `esc to cancel`, sigue trabajando; volver a
  esperar.
- Un `wait` con una opción mal escrita falla al instante con exit 0: revisar `"ok": false` en la salida.
- Pantalla de un carácter por línea: `orca terminal switch --terminal <h>`.

### Cierre de una tarea

1. Leer el reporte. `bloqueado` o verificación en rojo → nueva fase de QA, no integración.
2. Revisar el diff (`git diff origin/develop...<rama>`): fronteras respetadas y criterios cumplidos.
3. **Integrar a `develop`:** seguir el procedimiento de integración de §3 (guardia `git -C <raíz>`, `fetch`,
   `merge-base`, `merge --no-ff` y `push`; si no es ancestro, devolver al worktree). Si la tarea es un ítem del backlog
   (ver Parte II), la misma rama lo marca como completado.
4. **Limpiar** (sin puerta), tras confirmar `git -C <raíz> merge-base --is-ancestor <rama> develop`:
   ```powershell
   orca terminal close --worktree path:<ruta> --all --json
   orca worktree rm --worktree path:<ruta> --delete-branch --json
   ```
   Solo se borra la rama local; la remota en `origin` se queda.
5. Los *Supuestos* de cada reporte se acumulan y se entregan al usuario en el resumen final o en la siguiente aprobación
   que se pida al usuario.
6. Si con esta tarea se completó el backlog planificado, se prepara el release (§3) sin preguntar.

### Paralelismo

- Máximo 3 worktrees activos.
- Solo en paralelo si sus fronteras de archivos no se cruzan; si comparten archivos (ver ejemplos en Parte II), van en
  secuencia.
- Base de datos u otros servicios en un worktree: siempre con puertos aislados, nunca los del entorno principal. Si una
  suite de pruebas (p. ej. E2E) usa puertos fijos, ejecutar un worktree a la vez.
- No se envían mensajes a un agente cuyo worktree ya fue eliminado.

---

# Parte II — Este repositorio: TrustLoop

## 5. Fuentes de verdad

| Documento | Contiene |
| :--- | :--- |
| [`README.md`](README.md) | Descripción ejecutiva: qué es y para qué sirve. Sin detalles técnicos. |
| [`docs/SPECIFICATION.md`](docs/SPECIFICATION.md) | Todo lo técnico y funcional: stack y dependencias aprobadas, pantallas, reglas del juego, métricas, datos, persistencia, convenciones de código y empaquetado. |
| `docs/BACKLOG.md` | Backlog con un sprint. **Aún no existe:** se crea al planificar el sprint. Cada ítem se marca como completado en la misma rama que lo integra; con todos completados se prepara el release (§3). |
| [`docs/wireframe/wireframe-trustloop.html`](docs/wireframe/wireframe-trustloop.html) | Flujo e interfaz funcional. Manda sobre la especificación hasta que exista el diseño oficial del diseñador. |

La estética (colores, tipografía, iconos, ilustraciones de los compañeros) la define el diseñador externo: ningún agente
la inventa.

## 6. Concreciones de la Parte I

- **G2:** el commit en la raíz que hace el subagente `haiku` del Coordinador (ver [`CLAUDE.md`](CLAUDE.md)) no es puerta.
- **G3:** aplica a `package.json`. Las dependencias que `docs/SPECIFICATION.md` lista como aprobadas no requieren puerta.
- **Gestor de paquetes:** solo `pnpm`; prohibido `npm`/`yarn` y sus lockfiles.
- **Archivos compartidos que obligan a secuenciar worktrees:** `package.json`, `pnpm-lock.yaml`, `src/shared/types.ts`.
- **Sin producción remota:** no hay scripts que escriban en producción ni stack local de servicios. Publicar es solo el
  GitHub Release con los instaladores.

## 7. Verificación obligatoria

| Tipo de tarea | Verificación |
| :--- | :--- |
| Código | `pnpm typecheck` · `pnpm test` · `pnpm build`, y la app abre sin errores en la consola de Electron. |
| Persistencia o flujo de usuario | Además, probar con una carpeta de datos temporal (nunca `%LOCALAPPDATA%\TrustLoop` real): (1) desde cero, (2) con datos previos, (3) reingresando con el mismo perfil. |
| Interfaz | Además, regenerar las capturas con `pnpm docs:screenshots` (perfil temporal) y revisarlas. |
| Solo documentación | Todo enlace relativo apunta a un archivo existente. |

## 8. Release

- Versión en `package.json`. Artefactos con `pnpm dist` en `release/`: instalador por usuario (sin administrador) y
  portable, ambos compartiendo la carpeta de datos.
- Notas del release en español, orientadas al usuario final.
- En la Puerta de producción, `Artefactos:` lista las rutas de ambos `.exe` en el worktree de release, y
  `gh release create` adjunta esos dos archivos.
