# Bootstrap Task: Fill Project Development Guidelines

**You (the AI) are running this task. The developer does not read this file.**

The developer just ran `trellis init` on this project for the first time.
`.trellis/` now exists with empty spec scaffolding, and this bootstrap task
exists under `.trellis/tasks/`. When they want to work on it, they should start
this task from a session that provides Trellis session identity.

**Your job**: help them populate `.trellis/spec/` with the team's real
coding conventions. Every future AI session — this project's
`trellis-implement` and `trellis-check` sub-agents — auto-loads spec files
listed in per-task jsonl manifests. Empty spec = sub-agents write generic
code. Real spec = sub-agents match the team's actual patterns.

Don't dump instructions. Open with a short greeting, figure out if the repo
has any existing convention docs (CLAUDE.md, .cursorrules, etc.), and drive
the rest conversationally.

---

## Status (update the checkboxes as you complete each item)

- [ ] Fill guidelines for napcat-adapter
- [ ] Fill guidelines for napcat-common
- [ ] Fill guidelines for napcat-core
- [ ] Fill guidelines for napcat-database
- [ ] Fill guidelines for napcat-develop
- [ ] Fill guidelines for napcat-dpapi
- [ ] Fill guidelines for napcat-framework
- [ ] Fill guidelines for napcat-image-size
- [ ] Fill guidelines for napcat-napi-loader
- [ ] Fill guidelines for napcat-native
- [ ] Fill guidelines for napcat-onebot
- [ ] Fill guidelines for napcat-plugin-builtin
- [ ] Fill guidelines for napcat-protobuf
- [ ] Fill guidelines for napcat-protocol
- [ ] Fill guidelines for napcat-pty
- [ ] Fill guidelines for napcat-qrcode
- [ ] Fill guidelines for napcat-rpc
- [ ] Fill guidelines for napcat-schema
- [ ] Fill guidelines for napcat-shell
- [ ] Fill guidelines for napcat-shell-loader
- [x] Fill guidelines for napcat-test
- [ ] Fill guidelines for napcat-types
- [ ] Fill guidelines for napcat-universal
- [ ] Fill guidelines for napcat-vite
- [x] Fill guidelines for napcat-webui-backend
- [x] Fill guidelines for napcat-webui-frontend
- [ ] Add code examples

---

## Spec files to populate

### Package: napcat-adapter (`spec/napcat-adapter/`)

- Frontend guidelines: `.trellis/spec/napcat-adapter/frontend/`

### Package: napcat-common (`spec/napcat-common/`)

- Frontend guidelines: `.trellis/spec/napcat-common/frontend/`

### Package: napcat-core (`spec/napcat-core/`)

- Frontend guidelines: `.trellis/spec/napcat-core/frontend/`

### Package: napcat-database (`spec/napcat-database/`)

- Backend guidelines: `.trellis/spec/napcat-database/backend/`

- Frontend guidelines: `.trellis/spec/napcat-database/frontend/`

### Package: napcat-develop (`spec/napcat-develop/`)

- Frontend guidelines: `.trellis/spec/napcat-develop/frontend/`

### Package: napcat-dpapi (`spec/napcat-dpapi/`)

- Frontend guidelines: `.trellis/spec/napcat-dpapi/frontend/`

### Package: napcat-framework (`spec/napcat-framework/`)

- Frontend guidelines: `.trellis/spec/napcat-framework/frontend/`

### Package: napcat-image-size (`spec/napcat-image-size/`)

- Backend guidelines: `.trellis/spec/napcat-image-size/backend/`

- Frontend guidelines: `.trellis/spec/napcat-image-size/frontend/`

### Package: napcat-napi-loader (`spec/napcat-napi-loader/`)

- Backend guidelines: `.trellis/spec/napcat-napi-loader/backend/`

- Frontend guidelines: `.trellis/spec/napcat-napi-loader/frontend/`

### Package: napcat-native (`spec/napcat-native/`)

- Backend guidelines: `.trellis/spec/napcat-native/backend/`

- Frontend guidelines: `.trellis/spec/napcat-native/frontend/`

### Package: napcat-onebot (`spec/napcat-onebot/`)

- Backend guidelines: `.trellis/spec/napcat-onebot/backend/`

- Frontend guidelines: `.trellis/spec/napcat-onebot/frontend/`

### Package: napcat-plugin-builtin (`spec/napcat-plugin-builtin/`)

- Frontend guidelines: `.trellis/spec/napcat-plugin-builtin/frontend/`

### Package: napcat-protobuf (`spec/napcat-protobuf/`)

- Frontend guidelines: `.trellis/spec/napcat-protobuf/frontend/`

### Package: napcat-protocol (`spec/napcat-protocol/`)

- Backend guidelines: `.trellis/spec/napcat-protocol/backend/`

- Frontend guidelines: `.trellis/spec/napcat-protocol/frontend/`

### Package: napcat-pty (`spec/napcat-pty/`)

- Frontend guidelines: `.trellis/spec/napcat-pty/frontend/`

### Package: napcat-qrcode (`spec/napcat-qrcode/`)

- Frontend guidelines: `.trellis/spec/napcat-qrcode/frontend/`

### Package: napcat-rpc (`spec/napcat-rpc/`)

- Backend guidelines: `.trellis/spec/napcat-rpc/backend/`

- Frontend guidelines: `.trellis/spec/napcat-rpc/frontend/`

### Package: napcat-schema (`spec/napcat-schema/`)

- Frontend guidelines: `.trellis/spec/napcat-schema/frontend/`

### Package: napcat-shell (`spec/napcat-shell/`)

- Frontend guidelines: `.trellis/spec/napcat-shell/frontend/`

### Package: napcat-shell-loader (`spec/napcat-shell-loader/`)

- Backend guidelines: `.trellis/spec/napcat-shell-loader/backend/`

- Frontend guidelines: `.trellis/spec/napcat-shell-loader/frontend/`

### Package: napcat-test (`spec/napcat-test/`)

- Frontend guidelines: `.trellis/spec/napcat-test/frontend/`

### Package: napcat-types (`spec/napcat-types/`)

- Frontend guidelines: `.trellis/spec/napcat-types/frontend/`

### Package: napcat-universal (`spec/napcat-universal/`)

- Frontend guidelines: `.trellis/spec/napcat-universal/frontend/`

### Package: napcat-vite (`spec/napcat-vite/`)

- Frontend guidelines: `.trellis/spec/napcat-vite/frontend/`

### Package: napcat-webui-backend (`spec/napcat-webui-backend/`)

- Backend guidelines: `.trellis/spec/napcat-webui-backend/backend/`

- Frontend guidelines: `.trellis/spec/napcat-webui-backend/frontend/`

### Package: napcat-webui-frontend (`spec/napcat-webui-frontend/`)

- Frontend guidelines: `.trellis/spec/napcat-webui-frontend/frontend/`


### Thinking guides (already populated)

`.trellis/spec/guides/` contains general thinking guides pre-filled with
best practices. Customize only if something clearly doesn't fit this project.

---

## How to fill the spec

### Step 1: Import from existing convention files first (preferred)

Search the repo for existing convention docs. If any exist, read them and
extract the relevant rules into the matching `.trellis/spec/` files —
usually much faster than documenting from scratch.

| File / Directory | Tool |
|------|------|
| `CLAUDE.md` / `CLAUDE.local.md` | Claude Code |
| `AGENTS.md` | Codex / Claude Code / agent-compatible tools |
| `.cursorrules` | Cursor |
| `.cursor/rules/*.mdc` | Cursor (rules directory) |
| `.windsurfrules` | Windsurf |
| `.clinerules` | Cline |
| `.roomodes` | Roo Code |
| `.github/copilot-instructions.md` | GitHub Copilot |
| `.vscode/settings.json` → `github.copilot.chat.codeGeneration.instructions` | VS Code Copilot |
| `CONVENTIONS.md` / `.aider.conf.yml` | aider |
| `CONTRIBUTING.md` | General project conventions |
| `.editorconfig` | Editor formatting rules |

### Step 2: Analyze the codebase for anything not covered by existing docs

Scan real code to discover patterns. Before writing each spec file:
- Find 2-3 real examples of each pattern in the codebase.
- Reference real file paths (not hypothetical ones).
- Document anti-patterns the team clearly avoids.

### Step 3: Document reality, not ideals

**Critical**: write what the code *actually does*, not what it should do.
Sub-agents match the spec, so aspirational patterns that don't exist in the
codebase will cause sub-agents to write code that looks out of place.

If the team has known tech debt, document the current state — improvement
is a separate conversation, not a bootstrap concern.

---

## Quick explainer of the runtime (share when they ask "why do we need spec at all")

- Every AI coding task spawns two sub-agents: `trellis-implement` (writes
  code) and `trellis-check` (verifies quality).
- Each task has `implement.jsonl` / `check.jsonl` manifests listing which
  spec files to load.
- The platform hook auto-injects those spec files + the task's `prd.md`
  into every sub-agent prompt, so the sub-agent codes/reviews per team
  conventions without anyone pasting them manually.
- Source of truth: `.trellis/spec/`. That's why filling it well now pays
  off forever.

---

## Completion

When the developer confirms the checklist items above are done with real
examples (not placeholders), guide them to run:

```bash
python3 ./.trellis/scripts/task.py finish
python3 ./.trellis/scripts/task.py archive 00-bootstrap-guidelines
```

After archive, every new developer who joins this project will get a
`00-join-<slug>` onboarding task instead of this bootstrap task.

---

## Suggested opening line

"Welcome to Trellis! Your init just set me up to help you fill the project
spec — a one-time setup so every future AI session follows the team's
conventions instead of writing generic code. Before we start, do you have
any existing convention docs (CLAUDE.md, .cursorrules, CONTRIBUTING.md,
etc.) I can pull from, or should I scan the codebase from scratch?"
