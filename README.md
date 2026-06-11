# Power BI Project Lifecycle · v0.3.3

[![Version](https://img.shields.io/badge/version-0.3.3-201d15.svg)](#roadmap) [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE) [![Agent Skill](https://img.shields.io/badge/agent-skill-d97757.svg)](./SKILL.md) [![Power BI](https://img.shields.io/badge/Power%20BI-lifecycle-F2C811.svg)](#)


End-to-end skill for the disk lifecycle of a Power BI delivery project. Takes a project from "empty folder" to "version-controlled, env-promotable" without requiring the operator to know git, the PBIP file format, or the MAX_PATH 256 quirks of Power BI Desktop.

## What it does (six modes)

| Mode | Status | What it does |
|---|---|---|
| 1. Bootstrap folders | ✅ Available | Creates the standard client + project folder structure (with auto-junction at `C:\PBI-Clients\<slug>\` if path budget requires). Strictly additive — never deletes, moves, renames, or overwrites. |
| 2. Scaffold PBIP | ✅ Available | Generates a working `.pbip` from scratch in PBIR-Legacy with Auto Date/Time disabled (`__PBI_TimeIntelligenceEnabled = 0`), client theme auto-applied, and short internal folder names to fit MAX_PATH 256. |
| 3. Init Git + GitHub | ✅ Available | One repo per client. `gh` CLI install + auth flow (with the often-missed `gh auth setup-git` step). Always `--private`. Project-scoped commit pattern with `[ProjectName]` prefixes. |
| 4. Refresh theme | ✅ Available | Propagates a Global Theme update across all projects, respecting per-project overrides declared in `theme.md`. All-or-nothing validation, mandatory backup, `--dry-run` preview. |
| 5. Promote env + lifecycle report | 🟡 Design locked, queued v0.5 + v0.5.1 | Promote = copy `02_Build/`, rewrite catalog/host/path/auth from target's `source.json`, generate **diff-findings report** (editorial deck, same family as pbi-model-doc) comparing source vs target before commit. On Test → Prod: auto-trigger **deployment lifecycle report** (timeline, commits, audit history, sign-off) into `03_Docs/client_handoff/docs/`. |
| 6. Help me read changes (no git knowledge) | ✅ Available | Educational walkthrough of VS Code Source Control panel — read diffs, stage, commit, push without typing a single git command. |

## Folder structure (Phase 1)

```
<Clients root>/                                ← operator provides; must exist
  <Client>/                                    ← created if missing
    Global Info/                               ← shared client info
    Global Theme/                              ← shared client visual identity
      logo/
      brand-guideline/
      pbi-theme/                               ← output of pbi-theme companion skill
        theme/                                 ← .json + canvas SVG
        design-system/                         ← .pdf + .html documentation
        design-tokens/                         ← .css custom properties
        html-visual-style/                     ← DAX HTML examples
    Power BI Repository/                       ← env hub
      01. Dev/<Project>/
        01_Context/                            ← scope, source.json, theme.md, wireframe mapping
        02_Build/                              ← where .pbip lives (Phase 2)
        03_Docs/
          internal/{audit,docs}/
          client_handoff/{audit,docs}/
      02. Test/<Project>/                      ← replicated empty until promote
      03. Prod/<Project>/                      ← replicated empty until promote
```

## The locked recipes (Phase 2)

Every aspect of PBIP generation was calibrated empirically. The skill's Phase 2 generates files satisfying ALL of these constraints:

| Aspect | Rule |
|---|---|
| Format | PBIR-Legacy (NOT PBIR-modular) |
| Encoding | UTF-8 **without BOM** for every file |
| `model.tmdl` | Includes `annotation __PBI_TimeIntelligenceEnabled = 0` (mandatory — eliminates MAX_PATH root cause) |
| Theme `name` field | ≤ 16 chars |
| Enum values | CamelCase (`Left`, `Top`, `Bottom`, etc.) — never lowercase |
| Internal folder names | Short `Report/` and `SemanticModel/` |
| MAX_PATH | 256 chars total — junction at `C:\PBI-Clients\<slug>\` if root > 80 chars |
| Base theme | `BaseThemes/CY26SU04.json` always copied |
| `compatibilityLevel` | 1600 |
| `definition.pbism` | `version: "4.2"` |
| `definition.pbir` | `version: "4.0"` |
| `report.json` | At root of `Report/`, NOT in `definition/` |
| GUIDs | Fresh `logicalId` per `.platform`, fresh `lineageTag` per expression |

## Pre-flight (Phase 3)

The three commands the agent runs before touching git:

```powershell
# 1. Install gh CLI (one-time per machine)
winget install --id GitHub.cli --accept-source-agreements --accept-package-agreements -e

# 2. Authenticate
gh auth login
# Answer: GitHub.com / HTTPS / Yes auth git / web browser

# 3. Connect git's credential helper to gh's token (THE STEP EVERYONE MISSES)
gh auth setup-git
```

Without step 3, `git push` fails with "Invalid username or token. Password authentication is not supported for Git operations" even after a successful `gh auth login`. This skill always runs `gh auth setup-git` automatically after login is confirmed.

## Installation

### Claude Code

```bash
git clone https://github.com/gusbavia/pbi-lifecycle.git
cp -r pbi-lifecycle ~/.claude/skills/          # global, every project
# or into a single project: cp -r pbi-lifecycle your-project/.claude/skills/
```

Then just ask in plain language (the skill triggers from your phrasing), or invoke it explicitly with `@pbi-lifecycle`.

### Claude.ai

Download this repository as a ZIP, then **Settings > Capabilities > Skills > Upload skill**.

## How to use

Once registered (linked into `~/.claude/skills/pbi-lifecycle/`), invoke conversationally in any session:

> Setup my new Power BI project.

> Quero iniciar a estrutura de um novo projeto Power BI.

> Init git for this client.

> Help me read changes in VS Code.

The skill starts with a numbered triage prompt. Type 1–8 to pick a mode, or let it infer from your phrasing.

## Hard safety rules

The skill is conservative by design. Real client deliverables live on synced drives where a single accidental delete, move, or rename destroys weeks of work.

- Phase 1 is **strictly additive** — never deletes, moves, renames, or overwrites.
- Phase 2 **never overwrites** an existing PBIP — stops if `02_Build/` is non-empty.
- Phase 3 **always creates private repos** — `--private` flag mandatory, no opt-out.
- Every Phase has pre-flight checks that are mandatory.
- Every mutation requires explicit operator y/N confirmation.
- Companion skills (`pbi-theme`, `pbi-model-audit`, `pbi-theme-audit`, `pbi-model-doc`) are mentioned only as one-line pointers in handoffs — never auto-invoked.

See `SKILL.md` for the full forbidden-operations list and failure-mode recovery table.

## Why these guardrails (every one was paid for)

Each guardrail traces back to a specific failure observed during 2026-05-04 calibration:

- **MAX_PATH 256** → real save failure with `DateTableTemplate_<GUID>.tmdl`
- **`gh auth setup-git` missing** → real "Invalid username or token" error after successful login
- **Auto Date/Time on by default** → real cause of MAX_PATH failure
- **BOM in TMDL files** → silent Power BI parser failure
- **Lowercase enums in theme JSON** → silent theme application failure
- **Theme `name` field > 16 chars** → MAX_PATH overflow on internal hash filename
- **PBIR-modular** → theme not auto-applying

Each was a multi-hour debug. Each is now a one-line pre-flight check.

## Roadmap

| Version | Status |
|---|---|
| **v0.3.3** (current) | Phases 1, 2, 3, 4, 6 + companion-skill alignment contract. Phase 5 design-locked. |
| v0.5.0 | Phase 5 (env promote) implementation after real-PBIP calibration |
| v0.6.0 | Auto-scoped `commit` mode + repo-wide `lint` mode |

## Companion-skill alignment contract (v0.3.3)

Once a client is bootstrapped with this skill, the four companion skills auto-detect they're inside a lifecycle-managed structure and route their outputs to the right subfolder. Independent use of each companion skill remains supported — the contract enriches without making the lifecycle a hard dependency.

| Skill | Auto-routes to | Falls back to |
|---|---|---|
| `pbi-theme` (mode 2) | `<Client>/Global Theme/pbi-theme/{theme,design-system,design-tokens,html-visual-style}/` | Operator-chosen folder |
| `pbi-model-audit` (≥ 0.2.1) | `<Project>/03_Docs/internal/audit/{YYYY-MM-DD}/` | `_outputs/audit/` |
| `pbi-model-doc` (≥ 0.2.2) | `<Project>/03_Docs/internal/docs/{YYYY-MM-DD}/` | `_outputs/doc/` |
| `pbi-theme-audit` (≥ 0.1.1) | `<Project>/03_Docs/internal/audit/{YYYY-MM-DD}/` (shared with model-audit) | `_outputs/theme-audit/` |

Internal (`03_Docs/internal/`) is gitignored by default — work-in-progress accumulates locally. Operator manually promotes approved artifacts into `client_handoff/` with explicit commits, creating a traceable record of what was shared.

Each version is additive. Lower-numbered phases stay backward-compatible forever.

## Author

Gus Bavia · [linkedin.com/in/gusbavia](https://linkedin.com/in/gusbavia)

## License

MIT — see `LICENSE`.
