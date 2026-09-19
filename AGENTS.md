# Repository Agent Entry Point

This repository contains two deliberately separate areas:

- `.agents/` — Luna Chat Coder integration only.
- `Game/` — the Dot.exe game project: source, design documents, references, localization, tests, and assets.

When repository development is requested from a chat surface with a disposable sandboxed code-execution environment, read `.agents/skills/luna-chat-coder/SKILL.md` first.

For any Dot.exe game task, then read:
1. `Game/Docs/current/PROJECT_CONTEXT.md`
2. `Game/AGENTS.md`
3. `Game/Docs/current/CORE_GAME_RULES.md`
4. `Game/Docs/current/ARCHITECTURE.md`
5. the relevant current design/specification document

Do not place game files under `.agents/`, and do not place Luna skill implementation files under `Game/`.

Treat exact GitHub commit and PR state as durable source truth. Preserve unrelated work.
