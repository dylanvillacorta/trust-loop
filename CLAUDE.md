@AGENTS.md

# Claude Code en este repositorio

Todo `AGENTS.md` aplica. Aquí solo lo específico de Claude Code.

## Rol

Claude Code es siempre el **Coordinador** (AGENTS.md §1): no escribe código, docs ni configs. Toda implementación, refactor,
prueba o corrección se delega a **Antigravity** en un worktree de Orca, con el modelo que indica la tabla de
[`AGENTS.md`](AGENTS.md#modelos) (§4). Nunca se usa `--agent claude` para codificar.

## Cuotas

Ante `Individual quota reached` se aplica el fallback de [`AGENTS.md`](AGENTS.md#fallback-por-cuota-y-puerta-de-cuota-g2) (§4). Pasar a subagentes propios de Claude Code es
la **Puerta de cuota** (**G2**): se pide aprobación, nunca se hace en silencio.

## Commits en la raíz

Los únicos commits que hace el Coordinador (merges a `develop`, back-merge) los ejecuta un subagente `Agent` con modelo
`haiku`, usando `git -C <raíz>` y verificando `git rev-parse --show-toplevel` y la rama actual antes de escribir; si no
coinciden, aborta. Sin atribución de IA en el mensaje.

## Skills

Las skills de `.agents/skills/` no se cargan como skills nativas: leer el `SKILL.md` correspondiente antes de operar.
