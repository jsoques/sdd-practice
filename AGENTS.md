# AGENTS.md

Repo de práctica de spec-driven development (SDD). No hay código de aplicación: no existen build, test, lint ni typecheck que correr.

## Elige el workflow primero

- El usuario escribe `/speckit.*` → Spec Kit. Dice `opsx ...` o `openspec ...` → OpenSpec. Dice `/specsmd-*` → specsmd/AI-DLC.
- Si no nombra ninguno → pregunta cuál usar antes de empezar cualquier ciclo SDD.

## Stack (siempre, salvo que se pida lo contrario en la petición)

- **Go + SQLite3**, templates HTML renderizados en servidor con **HTMX**.
- Otro stack solo si el usuario lo pide explícitamente en esa misma petición.

## Idioma

- Toda interacción con el usuario, la documentación y los specs se escriben en **español dominicano**.
- En inglés quedan solo los términos técnicos: identifiers, keywords, comandos CLI, nombres de archivos, mensajes de commit.

## Spec Kit (`.specify/`, `.agents/commands/speckit.*.md`)

- Ciclo: `/speckit.specify` → `clarify` → `plan` → `tasks` → `implement` (más `analyze`, `checklist`, `converge`).
- Los archivos de comando son prompts markdown; se invocan como slash commands, no se ejecutan como scripts.
- Estado del feature: los scripts resuelven el directorio desde `SPECIFY_FEATURE_DIRECTORY` o `.specify/feature.json`. Una sesión fresca no tiene ninguno (`feature.json` es machine-local y está en `.specify/.gitignore`) → `check-prerequisites.sh` falla con `Feature directory not found` hasta que corra `/speckit.specify`. El estado no se crea a mano; lo crea specify.
- `.specify/scripts/bash/*` y `.specify/templates/*` los maneja el Specify CLI (SHA-256 en `.specify/integrations/speckit.manifest.json`); las reglas custom van en `.specify/memory/constitution.md`, no ahí.
- `.specify/memory/constitution.md` define las reglas del proyecto (stack Go+SQLite3+HTMX, español dominicano); el Constitution Check de `speckit.plan` la usa como referencia.
- No existe `.specify/extensions.yml` → los pre-checks de hooks de los commands se saltan (los prompts mandan saltarlos en silencio).

## OpenSpec (`openspec/`, `.agents/skills/openspec-*`)

- CLI `openspec` v2.0.0 en PATH; raíz inicializada (`openspec list --json` devuelve `root`; `specs/` y `changes/` vacíos).
- **Límite de planificación**: `propose` / `new change` crean artifacts de planificación y se detienen; `apply` arranca solo con una petición nueva y explícita. La misma respuesta que crea artifacts no toca código de proyecto.
- `openspec/config.yaml` tiene `context` con las restricciones del proyecto (stack, idioma); `rules` sigue vacío → aplican los defaults por artifact.

## specsmd / AI-DLC (`.specsmd/`, `.opencode/command/specsmd-*.md`)

- Flujo AI-DLC (AWS): fases secuenciales Inception → Construction → Operations, orquestadas por `/specsmd-master-agent`. Comandos en `.opencode/command/`, personas en `.specsmd/aidlc/agents/`, skills en `.specsmd/aidlc/skills/` (manifest `.specsmd/manifest.yaml`, flow `aidlc`).
- Sesión fresca sin `memory-bank/`: el master-agent se detiene y redirige a la skill `project-init` hasta inicializar; no intentes saltártela.
- Los command files apuntan a `.specsmd/skills/<fase>/*.md`, pero la ruta real es `.specsmd/aidlc/skills/<fase>/*.md` → confía en la tabla de skills del persona (`aidlc/agents/*.md`), que sí tiene los paths correctos.
