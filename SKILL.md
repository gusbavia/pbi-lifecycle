---
name: pbi-lifecycle
description: End-to-end Power BI delivery project lifecycle — folder bootstrap, PBIP file scaffold, Git tracking with private GitHub repo, theme refresh propagation, and environment promotion (Dev → Test → Prod). One skill that takes a Power BI delivery project from "empty disk" to "deployable, version-controlled, env-promotable" without requiring the operator to know git, the PBIP file format, or the MAX_PATH 256 quirks of Power BI Desktop. Phase 1 (folder bootstrap, additive-only) is generic and runs against any client. Phase 2 (PBIP scaffold from scratch in PBIR-Legacy with Auto Date/Time disabled by default and theme auto-applied) generates a working .pbip file that opens cleanly. Phase 3 (Git + GitHub CLI integration with always-private repos and project-scoped commits) gives version control without the operator typing a single git command. Phases 4 and 5 (theme refresh, env promotion) ship as design-locked roadmap until real-PBIP calibration completes. Use this skill whenever the operator asks anything resembling starting, setting up, scaffolding, creating, initializing, organizing, bootstrapping, version-controlling, committing, promoting, or refreshing the theme of a Power BI project, repository, or workspace on disk — even when the request is short and casual. Triggers (case-insensitive, partial-match) include "setup new power bi project", "set up my new power bi project", "set up new pbi project", "setup pbi project", "new power bi project", "start a power bi project", "create new power bi project", "create new power bi project folder", "scaffold power bi project", "scaffold pbi project", "initialize power bi project", "initialize power bi repository", "bootstrap power bi project", "organize power bi folders", "new pbip", "scaffold pbip", "create pbip from scratch", "init git for power bi", "git init power bi project", "create github repo for power bi", "version control power bi", "commit power bi changes", "push power bi", "refresh theme across projects", "rebrand all reports", "promote dev to test", "promote test to prod", "promote power bi env", and the Portuguese variants "criar pasta de projeto power bi", "criar novo projeto power bi", "iniciar novo projeto", "estruturar pastas do cliente", "configurar projeto power bi", "novo projeto pbi", "scaffold pbip do zero", "iniciar git para power bi", "criar repositorio github do projeto", "atualizar tema em todos projetos", "rebranding completo dos relatorios", "promover dev para teste", "promover teste para producao". When in doubt about whether a Power-BI-folder-or-lifecycle request matches, prefer to invoke this skill rather than improvising. Do NOT use this skill to generate the visual theme JSON itself (that is the `pbi-theme` companion skill), to run model audits (`pbi-model-audit`), to generate model documentation (`pbi-model-doc`), or to audit the visual layer (`pbi-theme-audit`). Those companion skills write into the folders this skill creates.
license: MIT
metadata:
  author: Gus Bavia
  version: 0.3.3
  category: business-intelligence
  tags: [power-bi, lifecycle, folder-structure, project-bootstrap, pbip, scaffolding, sharepoint, onedrive, git, github, version-control, theme-refresh, env-promotion]
  companions: [pbi-project-hub, pbi-theme, pbi-model-audit, pbi-model-doc, pbi-theme-audit]
  support: https://linkedin.com/in/gusbavia
---

# Power BI Project Lifecycle · v0.3.3

End-to-end skill for the disk lifecycle of a Power BI delivery project. From an empty folder to a version-controlled, env-promotable project, with safety rails that protect non-git users from destructive mistakes.

## Why this skill exists

Power BI delivery has six recurring pain points the operator should not have to relearn each time:

1. The Power BI Desktop **MAX_PATH 256-char limit** — strict, undocumented at file level, instantly broken when Auto Date/Time auto-creates `DateTableTemplate_<GUID>.tmdl` files inside long client paths.
2. The **PBIR-Legacy vs PBIR-Modular** confusion — modular looks cleaner but Power BI Desktop converts to legacy on first save and themes don't auto-apply when generated as modular.
3. **`gh auth setup-git` is a separate step from `gh auth login`** — without it, `git push` fails after a successful login with the misleading "Invalid username or token" error.
4. **Auto Date/Time** is enabled by default in Power BI but considered an anti-pattern by every senior practitioner. The skill ships PBIPs with it disabled by default.
5. Power BI devs are often **not git users**. The skill's audience does not learn git just to use this. The skill explains how to inspect changes via the VS Code Source Control panel without typing a single git command.
6. **Theme propagation across projects** is manual today (copy JSON into each PBIP). The skill (Phase 4) automates this with per-project override semantics so a client rebrand is one command.

This skill captures every workaround once, locks it as a pre-flight check or runtime guard, and lets the operator focus on building reports.

## Generic positioning · MANDATORY for the agent running this skill

This skill ships to any Power BI practitioner. The agent running it must enforce strict neutrality. Violating any rule below is a defect.

1. **Do not infer or use the operator's personal context.** Never read auto-memory, prior conversation history, or session context to invent defaults, examples, or questions. The skill's behavior must be identical for the skill author, an external consultant, or a complete stranger.
2. **Do not reference any specific organization, client, product, project, or person** in any prompt, message, default value, example, or option. If unsure whether a name is generic, omit it.
3. **Do not ask any questions beyond the inputs defined in each Phase's Inputs section.** No project-type wizards, no scaffolding presets, no organization-specific naming taxonomies.
4. **Do not propose any folder, file, or convention beyond what is defined in the Hierarchy section.** The structure is fixed and exhaustive for this version.
5. **All placeholders in user-facing examples must be neutral** — `Client A`, `Project Name`, `My Power BI Project`. No real client names from any context.
6. **The skill is conversational but not creative.** Do not "be helpful" by adding steps, options, or branching logic. The defined inputs and the fixed structure are the entire surface area.

If the operator volunteers a specific client name like "Acme Corp", that is fine — it becomes the value of `clientName` for that run. The agent must not propose, suggest, or remember any specific name on its own.

## Forbidden pre-flight behaviors

Before, during, and after this skill runs, the agent must not perform any of the following:

1. **Do not list, scan, or inspect the operator's filesystem before asking the Clients root question.** No `pwd`, `ls`, `Get-ChildItem`, `dir`, or any equivalent on the current working directory or any directory the operator has not explicitly named. The skill's first interactive action is the **mode triage** prompt; the second is the relevant input question for the chosen mode.
2. **Do not propose candidate locations** based on what already exists on the operator's machine. Every path comes from the operator's typed answer.
3. **Do not auto-wire companion skills** (`pbi-model-doc`, `pbi-model-audit`, `pbi-theme-audit`, etc.) during any phase. Mention them in closing handoffs only as one-line pointers.
4. **Do not skip a Phase to "save time".** Phases run in declared order with explicit operator confirmation between them.

The agent's first message after the skill is invoked must be the **mode triage** prompt described in the next section, exactly. No greeting beyond that, no preamble, no context-gathering, no scanning of cwd.

## Mode triage · the first message the skill always sends

The skill is the lifecycle skill — multiple operations are available. The agent must always start by asking the operator which mode they want.

**First message — verbatim, in the operator's language (PT-BR or EN, match the input):**

```
What do you want to do?

  1. Bootstrap a new Power BI project folder structure
     (creates Client + Global Info + Global Theme + Power BI Repository
      with 01.Dev/02.Test/03.Prod env hubs and per-project subfolders;
      auto-creates a junction at C:\PBI-Clients\<slug>\ if path budget
      requires it)
     — AVAILABLE

  2. Scaffold a PBIP file inside an existing project folder
     (generates working .pbip in PBIR-Legacy from scratch with
      Auto Date/Time disabled, theme auto-applied if Global Theme
      has one, and short internal folder names to fit MAX_PATH 256)
     — AVAILABLE

  3. Initialize Git tracking for the client and push to a private
     GitHub repo (one repo per client, gh CLI flow with the three
     mandatory steps install / auth login / auth setup-git)
     — AVAILABLE

  4. Refresh the visual theme across all projects of a client
     (reads Global Theme, applies per-project theme.md overrides,
      writes new theme.json into each PBIP's RegisteredResources/,
      always backs up first, supports --dry-run preview)
     — AVAILABLE

  5. Promote a project to the next environment OR close out a project
     (Dev → Test or Test → Prod: copies 02_Build/, rewrites only
      catalog / http host / path / auth from the target env's source.json,
      AND generates a diff-findings report (same editorial style as
      pbi-model-doc) comparing source vs target so operator approves
      consistency before the merge.
      On Test → Prod and on close-project: ALSO generates a unified
      deployment lifecycle report covering the entire project history —
      timeline, commits, audit scores, promotion records, sign-off — for
      client handoff into 03_Docs/client_handoff/docs/.)
     — DESIGN LOCKED, IMPLEMENTATION QUEUED

  6. Help me read changes in VS Code (no git knowledge required)
     — AVAILABLE (educational walkthrough, no filesystem changes)

  7. Validate, audit, or document an existing project
     — Use companion skills: pbi-model-audit, pbi-theme-audit, pbi-model-doc

  8. Something else
     — Not in scope for this skill

Type 1–8.
```

**Routing rules:**

- Modes `1`, `2`, `3`, `4`, `6` → proceed to the corresponding Phase section below.
- Mode `5` → reply: *"That phase has the design locked but is not yet executable in this skill version. The agent could perform individual steps manually based on the design notes in this skill, but the automation is queued for v0.5 after real-PBIP calibration. Want a manual walkthrough or stop here?"* If the operator wants the manual walkthrough, present the algorithm from the corresponding phase section but ask for explicit confirmation at every mutation. Do not improvise beyond the documented design.
- Mode `7` → reply: *"Use the companion skill: `pbi-model-audit` to audit, `pbi-theme-audit` for the visual layer, `pbi-model-doc` to generate documentation. This skill (`pbi-lifecycle`) only handles folder lifecycle and version control."* Then stop.
- Mode `8` or off-list → reply: *"That's outside the scope of `pbi-lifecycle`. This skill only handles the disk lifecycle and version control of Power BI projects."* Then stop.
- If the operator's initial invocation already implies a specific mode (e.g. *"setup my new power bi project"* → mode 1, *"create the pbip"* → mode 2, *"init git for this project"* → mode 3), the agent may infer that mode and proceed directly, skipping the triage prompt.

The triage list is fixed and exhaustive for this version. The agent must not add, remove, or reword the eight options.

---

# Phase 1 · Folder Bootstrap (AVAILABLE)

Phase 1 is **strictly additive**. It only creates folders and net-new placeholder files. It never deletes, moves, renames, copies, or modifies anything that already exists.

## What Phase 1 does

1. Asks the operator for the absolute local path of the **Clients root** (must already exist).
2. Asks the **client name**. If the client folder is missing, creates it together with the shared client-level subtrees: `Global Info/`, `Global Theme/`, and `Power BI Repository/`.
3. Asks the **project name**. Creates the project folder under `Power BI Repository/01. Dev/<Project>/` with the per-project subfolders (`01_Context/`, `02_Build/`, `03_Docs/internal|client_handoff/audit|docs/`). Replicates the empty project shell into `02. Test/` and `03. Prod/` for future promotion.
4. Adds `.gitkeep` files in empty env folders and `Global Info/` so the structure persists in version control.
5. **Computes the path budget.** If the absolute path of the client root exceeds 80 characters, **automatically creates a NTFS junction** at `C:\PBI-Clients\<client-slug>\` pointing to the OneDrive/long path and tells the operator to use the junction path going forward. If ≤ 80 chars, no junction is created — the operator works directly at the original path.
6. Optionally accepts a `--theme <path>` flag (or interactive prompt) to seed the client's theme.json into `Global Theme/pbi-theme/theme/` from an existing source.

## Hierarchy created

The skill creates this exact structure. Names in `<…>` are operator-provided. All other names are fixed defaults — the agent must not rename them.

```
<Clients root>/                                   ← operator provides; must exist
  <Client>/                                       ← created if missing
    Global Info/                                  ← shared client info; .gitkeep placeholder
    Global Theme/                                 ← shared client visual identity
      logo/                                       ← operator manually places client logos
      brand-guideline/                            ← operator manually places brand guidelines
      pbi-theme/                                  ← output of pbi-theme companion skill
        theme/                                    ← .json (importable) + canvas SVG
        design-system/                            ← .pdf + .html documentation
        design-tokens/                            ← .css custom properties
        html-visual-style/                        ← DAX HTML visual examples
    Power BI Repository/                          ← env hub
      01. Dev/                                    ← Dev env (uses "01. " prefix with period and space)
        .gitkeep                                  ← preserves env folder when projects absent
        <Project>/                                ← created per project; the work happens here
          01_Context/                             ← project briefing (lowercase snake_case)
            scope.md                              ← scope template (placeholder)
            source.json                           ← per-env connection schema (env=dev)
            theme.md                              ← optional per-project theme override notes
            wireframe_to_data_mapping.md          ← placeholder for wireframe XLSX
          02_Build/                               ← where .pbip lives (Phase 2 target; empty after Phase 1)
          03_Docs/
            internal/                             ← internal-only outputs (gitignored by default)
              audit/                              ← auto-target for pbi-model-audit + pbi-theme-audit when invoked inside lifecycle
              docs/                               ← auto-target for pbi-model-doc when invoked inside lifecycle
            client_handoff/                       ← shareable with client (Git-tracked)
              audit/                              ← operator manually promotes approved audits here
              docs/                               ← operator manually promotes approved docs / manuals here
      02. Test/                                   ← Test env (replicated empty)
        .gitkeep
        <Project>/                                ← same internal subtree, empty until promoted
      03. Prod/                                   ← Prod env (replicated empty)
        .gitkeep
        <Project>/
```

## Critical naming rules (every one was paid for during calibration)

| Aspect | Rule | Why |
|---|---|---|
| Env folder prefix | `01. Dev/` (with period AND space) | Visual ordering + readability; do not use `01_Dev/` |
| Per-env folders | `01. Dev`, `02. Test`, `03. Prod` (numbered in this exact order) | Promotion path is left-to-right |
| Per-project subfolders | `01_Context`, `02_Build`, `03_Docs` (underscore, no space) | Distinguishes from env-level prefix |
| Files inside `01_Context/` | lowercase snake_case (`scope.md`, `source.json`, `theme.md`, `wireframe_to_data_mapping.md`) | Filesystem-friendly + cross-platform |
| Empty folders | Get a `.gitkeep` so they persist in git | Git does not track empty dirs |
| `Global Info/` and `Global Theme/` | Created at client level, shared across all projects | Theme + reference docs are client-wide |

## Pre-flight: path budget detection

Before creating folders, the agent computes the **absolute path length of the proposed client root**:

```
$clientPath = "$clientsRoot\$clientName"
$pathLength = $clientPath.Length
```

Decision tree:

| Path length | Action |
|---|---|
| ≤ 80 chars | No junction. Skill proceeds directly at the original path. Note in handoff: "Path budget is comfortable; no junction created." |
| 81–110 chars | Junction recommended. Prompt: *"Your client root is `$pathLength` chars long. Power BI Desktop's MAX_PATH limit is 256 chars total, and complex projects can grow paths quickly. Recommend creating a NTFS junction at `C:\PBI-Clients\<slug>\`. Create junction? (y/N)"* — default Y. |
| > 110 chars | Junction MANDATORY. Prompt: *"Your client root is `$pathLength` chars long. Without a junction at `C:\PBI-Clients\<slug>\`, Power BI Desktop will fail to save certain files. The junction is mandatory for this path."* — proceed to create. |

Junction creation (no admin required):

```powershell
$slug = $clientName.ToLower() -replace '[^a-z0-9]+','-' -replace '^-+|-+$',''
$junctionRoot = "C:\PBI-Clients"
if (-not (Test-Path $junctionRoot)) {
  New-Item -ItemType Directory -Path $junctionRoot | Out-Null
}
$junctionPath = "$junctionRoot\$slug"
if (Test-Path $junctionPath) {
  # If junction exists pointing elsewhere, abort with clear error.
  # Otherwise (target matches), reuse silently.
}
New-Item -ItemType Junction -Path $junctionPath -Target $clientPath
```

After junction creation, **the agent informs the operator explicitly**:

> Junction created at `C:\PBI-Clients\<slug>\` → `<original path>`. Always open `.pbip` files via the junction path. The OneDrive/SharePoint folder is the sync source of truth; the junction is your working entry point. Both stay in sync automatically.

## Cross-cutting · Companion skill auto-routing contract

Once a client is bootstrapped with this skill, the four companion skills auto-detect they're inside a lifecycle-managed structure and route their outputs to the right subfolder. This is the contract every skill in the IF Power BI Quality Suite respects:

| Companion skill | Detects lifecycle when invoked at | Auto-routes output to | Standalone fallback |
|---|---|---|---|
| `pbi-theme` (mode 2) | Any path under `<Client>/` | `<Client>/Global Theme/pbi-theme/{theme,design-system,design-tokens,html-visual-style}/` | Operator-chosen folder (mode 1) |
| `pbi-model-audit` (v0.2.1+) | A `.pbip` inside `<Client>/Power BI Repository/<env>/<Project>/02_Build/` | `<Project>/03_Docs/internal/audit/{YYYY-MM-DD}/` | `{project_root}/_outputs/audit/{YYYY-MM-DD}/` |
| `pbi-model-doc` (v0.2.2+) | Same | `<Project>/03_Docs/internal/docs/{YYYY-MM-DD}/` | `{project_root}/_outputs/doc/{YYYY-MM-DD}/` |
| `pbi-theme-audit` (v0.1.1+) | Same | `<Project>/03_Docs/internal/audit/{YYYY-MM-DD}/` (shared with model-audit, prefixed by tool) | `{project_root}/_outputs/theme-audit/{YYYY-MM-DD}/` |

### Detection algorithm (each companion skill runs this independently)

```
Walk up from the .pbip path. If the chain matches:
  02_Build/<Project>/<env>/Power BI Repository/<client>/
where <env> matches "01. Dev" | "02. Test" | "03. Prod"
→ LIFECYCLE_MODE = true
→ project_folder = parent of "02_Build"
→ output goes to $project_folder/03_Docs/internal/{audit|docs}/{YYYY-MM-DD}/
```

If lifecycle is not detected, the companion skill falls back to its standalone behavior. **Independent use of any companion skill remains fully supported** — this contract enriches the experience inside lifecycle without making lifecycle a hard dependency.

### Internal vs client_handoff promotion (manual operator action)

`03_Docs/internal/` is gitignored by the lifecycle's `.gitignore` template. Audit and doc runs accumulate there as work-in-progress. When an audit or doc is **approved for client sharing**, the operator manually copies the artifact from `internal/` to `client_handoff/`. The act of promoting is a deliberate, traceable decision recorded as a git commit:

```
[<Project>] Promote audit report to client_handoff
[<Project>] Promote model doc to client_handoff
```

This separation is intentional: companion skills generate freely into `internal/`, operator curates into `client_handoff/`. No skill ever writes directly into `client_handoff/`.

## Cross-cutting · BYO theme

If the operator already has a `theme.json` (from `pbi-theme` skill, TabularEditor 3, an existing PBIP, or a brand consultant), Phase 1 supports seeding it into `Global Theme/pbi-theme/theme/` so Phase 2 picks it up automatically.

Three input modes:

| Input | Behavior |
|---|---|
| `--theme <path>` flag with existing JSON | Skill validates JSON, copies to `Global Theme/pbi-theme/theme/<filename>.json` |
| Interactive prompt during Phase 1 | *"Do you have an existing theme.json for this client? (path / no / skip)"* — `path`: provide the path. `no` or `skip`: leave empty; Phase 2 uses Power BI default |
| Pre-populated `Global Theme/pbi-theme/theme/*.json` (typical when `pbi-theme` already ran) | Skill auto-detects + uses |

**Validation on import (mandatory before copy):**

1. JSON parse validity — fail fast on broken JSON.
2. **Lowercase enum sweep** — find every `"left"`, `"top"`, `"bottom"`, `"right"`, `"center"`, `"auto"`, `"bottomonly"`, `"fit"` and verify they are CamelCase (`"Left"`, `"Top"`, etc.). Power BI silently fails the theme if any enum is lowercase.
3. **`name` field length** — must be ≤ 16 characters. Power BI uses this name to derive internal filenames during save (with a 17-char hash + `.json` suffix).
4. **Hex color check** — every value matching `/^#[0-9a-fA-F]{6}$/` is valid; warn on `#XXX` shorthand or invalid characters.

If any validation fails, the agent reports the specific error and offers to either fix the JSON in place (with operator confirmation) or stop.

## Hard safety rules

Phase 1 is **strictly additive**. The skill must never affect anything that already exists on disk.

**Forbidden operations** in Phase 1, no exception:

| Operation | Why forbidden |
|---|---|
| `Remove-Item`, `rm`, `del`, `rmdir` | Phase 1 never deletes. |
| `Move-Item`, `mv` | Phase 1 never moves. |
| `Rename-Item`, `ren` | Phase 1 never renames. |
| `Copy-Item`, `cp`, `xcopy`, `robocopy` of operator content | Phase 1 never copies the operator's content (the BYO theme copy is the only exception, and only after explicit operator confirmation of source path). |
| Any write to an existing file (overwrite, append, replace) | Phase 1 only creates new folders and net-new placeholder files. |
| `git init`, `git add`, `git commit`, any Git command | Git belongs to Phase 3. |
| `gh repo create` or any GitHub operation | GitHub belongs to Phase 3. |
| Creating folders or files the operator did not confirm | Every creation requires explicit operator confirmation of the full creation plan. |

**Only allowed write operations:**

1. `New-Item -ItemType Directory -Path "<absolute path>"` — only when `Test-Path "<absolute path>"` returns `$false`.
2. `New-Item -ItemType File -Path "<absolute path>" -Value "<placeholder content>"` — only for the placeholder template files defined in this skill, when `Test-Path` returns `$false`.
3. `New-Item -ItemType Junction -Path "<short path>" -Target "<long path>"` — only after path budget detection prompts the operator.
4. `Copy-Item <theme-source> -Destination "<Global Theme>/pbi-theme/theme/"` — only after BYO theme validation succeeds and the operator confirms the source path.

If the operator asks for any forbidden operation during Phase 1, refuse and explain which Phase handles it.

## Phase 1 execution algorithm

```
STEP 1 · Clients root
  Ask: absolute local path of Clients root (e.g., C:\Clients\ or OneDrive sync path).
  Validate:
    - Test-Path returns $true → continue
    - Reject SharePoint URLs (https://*.sharepoint.com/...)
    - Reject relative paths (.\, ..\)
    - Reject drive-letter-only paths (C:\)
    - If $false → STOP, tell operator the path does not exist locally

STEP 2 · Client
  Ask: client name (string).
  Compute:
    $clientPath = "$clientsRoot\$clientName"
    $pathLength = $clientPath.Length
  
  Path budget decision (see "Pre-flight: path budget detection" above).
  If junction needed → confirm + create.
  
  Build creation plan for client-level folders:
    $clientPath\Global Info\          (+ .gitkeep)
    $clientPath\Global Theme\
    $clientPath\Global Theme\logo\
    $clientPath\Global Theme\brand-guideline\
    $clientPath\Global Theme\pbi-theme\
    $clientPath\Global Theme\pbi-theme\theme\
    $clientPath\Global Theme\pbi-theme\design-system\
    $clientPath\Global Theme\pbi-theme\design-tokens\
    $clientPath\Global Theme\pbi-theme\html-visual-style\
    $clientPath\Power BI Repository\
    $clientPath\Power BI Repository\01. Dev\           (+ .gitkeep)
    $clientPath\Power BI Repository\02. Test\          (+ .gitkeep)
    $clientPath\Power BI Repository\03. Prod\          (+ .gitkeep)
  
  Test-Path each. Skip those that exist. Present plan + ask y/N.
  On y: New-Item Directory + .gitkeep files for missing entries.
  
  BYO theme prompt:
    Ask: "Do you have an existing theme.json? (path / no / skip)"
    On path: validate (JSON, enums, name length, hex) → copy into Global Theme/pbi-theme/theme/
    On no/skip: leave folder empty; Phase 2 will use Power BI default
  
  If $clientPath already fully scaffolded → report "client already set up" and continue to STEP 3.

STEP 3 · Project
  Ask: project name (string).
  Compute:
    $devProjectPath  = "$clientPath\Power BI Repository\01. Dev\$projectName"
    $testProjectPath = "$clientPath\Power BI Repository\02. Test\$projectName"
    $prodProjectPath = "$clientPath\Power BI Repository\03. Prod\$projectName"
  
  If Test-Path $devProjectPath returns $true → STOP. Tell operator the project folder already exists,
    ask them to choose a different name. Phase 1 must never touch an existing project folder.
  
  Build full creation plan for the project subtree (Dev populated, Test+Prod replicated empty):
    Dev:
      $devProjectPath\01_Context\
        scope.md                              (placeholder)
        source.json                           (placeholder, "environment": "dev")
        theme.md                              (placeholder)
        wireframe_to_data_mapping.md          (placeholder)
      $devProjectPath\02_Build\               (empty; Phase 2 target)
      $devProjectPath\03_Docs\internal\audit\
      $devProjectPath\03_Docs\internal\docs\
      $devProjectPath\03_Docs\client_handoff\audit\
      $devProjectPath\03_Docs\client_handoff\docs\
    Test:
      $testProjectPath\01_Context\
        source.json                           (placeholder, "environment": "test")
        + same other files as Dev
      $testProjectPath\02_Build\
      $testProjectPath\03_Docs\... (same subtree)
    Prod:
      $prodProjectPath\01_Context\
        source.json                           (placeholder, "environment": "prod")
        + same other files as Dev
      $prodProjectPath\02_Build\
      $prodProjectPath\03_Docs\... (same subtree)
  
  Present plan + ask y/N.
  On y: New-Item Directory + New-Item File for every entry.

STEP 4 · Closing handoff
  Print:
    - Absolute path of the Dev project folder (use junction path if junction was created)
    - ASCII tree of the per-project subtree
    - Junction info if created
    - One-line note: "Phase 1 complete. To scaffold the .pbip file, run mode 2 (PBIP scaffold).
      To version-control this client, run mode 3 (Git init). For audits and docs, see
      pbi-model-audit / pbi-theme-audit / pbi-model-doc."
```

## Placeholder file contents

### `01_Context/source.json` (per env, env = `"dev"` | `"test"` | `"prod"`)

```json
{
  "environment": "dev",
  "description": "Connection parameters that change between environments. Phase 5 (env promotion) reads this file from the target env and rewrites the corresponding values in expressions.tmdl. Never store secrets here.",
  "connections": [
    {
      "name": "<connection-name>",
      "type": "<Web | SQL | Databricks | OData | File | Other>",
      "url": "<https://server-or-endpoint>",
      "catalog": "<database-or-catalog>",
      "schema": "<schema-or-folder-or-lakehouse-path>",
      "auth": {
        "method": "<OAuth | Anonymous | Key | UsernamePassword>",
        "credentialsHandledBy": "Power BI Desktop / Service (never store secrets in this file)"
      }
    }
  ]
}
```

### `01_Context/scope.md`, `theme.md`, `wireframe_to_data_mapping.md`

Empty single-line placeholders with H1 title only. The operator fills these as the project starts.

### `theme.md` per-project override format (Phase 4 contract)

If a project requires a theme override on top of the Global Theme, the operator edits `01_Context/theme.md` to declare the override:

```yaml
---
inherits: ../../../../Global Theme/pbi-theme/theme/<theme-filename>.json
overrides:
  dataColors[0]: "#7C3AED"
  neutral: "#7C3AED"
  reason: "Client requested purple primary for this project to avoid color collision with another product"
---
```

Phase 4 (theme refresh) reads this and applies overrides on top of the Global Theme during refresh. If no `overrides:` block exists, the project inherits the Global Theme verbatim.

### `.gitkeep`

Empty file. Just exists so git tracks the parent folder.

---

# Phase 2 · PBIP Scaffold from scratch (AVAILABLE)

Phase 2 generates a working `.pbip` file inside `<Project>/02_Build/` from scratch, using the **PBIR-Legacy** format. The result opens cleanly in Power BI Desktop with the client theme auto-applied (if Global Theme has one) and Auto Date/Time disabled by default.

This was calibrated against multiple iterations during 2026-05-04 — the recipe below is exact and exhaustive.

## Pre-flight checks

Before generating any file, the agent verifies all of the following:

1. The client root exists and contains `Global Info/`, `Global Theme/`, `Power BI Repository/01. Dev/<Project>/02_Build/`. If any is missing → instruct operator to run Phase 1 first. Stop.
2. `<Project>/02_Build/` is **empty**. If a `.pbip` or any subfolder already exists there → STOP, ask operator if they want to delete the existing scaffold first (Phase 2 never overwrites).
3. **Path budget check** — if `<Project>/02_Build/` absolute path > 100 chars and no junction exists → warn operator, recommend running Phase 1 with junction or moving the client root.
4. **MAX_PATH headroom calculation:**
   - Base path: `<absolute path to 02_Build>` 
   - Plus deepest expected file: `Report\StaticResources\RegisteredResources\<theme-filename>.json` (~78 chars)
   - Plus deepest TMDL: `SemanticModel\definition\tables\<TableName>.tmdl` (~52 chars + table name)
   - Total must be ≤ 256 chars assuming reasonable table names (≤ 30 chars). If headroom < 30 chars → strongly recommend junction.
5. **BYO theme detection:** check `Global Theme/pbi-theme/theme/*.json`:
   - If exactly one `.json` exists → use it, validate enums + name length before copying.
   - If multiple → ask operator which to use.
   - If none → ask: "No theme found. Generate one now via the `pbi-theme` companion skill, drop a JSON manually into `Global Theme/pbi-theme/theme/`, or scaffold the PBIP without a custom theme (Power BI default)? (theme/skill/skip)"

## LOCKED Phase 2 recipe (every aspect calibrated empirically)

Generate a `.pbip` directory tree with this exact structure:

```
<Project>/02_Build/
  <Project>.pbip                                    ← wrapper, schema pbipProperties/1.0.0
  Report/                                           ← short folder name (NOT {Project}.Report/)
    .platform                                       ← gitIntegration/2.0.0, fresh logicalId GUID
    definition.pbir                                 ← schema definitionProperties/2.0.0, version 4.0
    report.json                                     ← AT ROOT (NOT under definition/), PBIR-Legacy format
    StaticResources/
      RegisteredResources/<theme-filename>.json     ← copy of the master theme from Global Theme
      SharedResources/BaseThemes/CY26SU04.json      ← built-in base theme (must be present)
  SemanticModel/                                    ← short folder name (NOT {Project}.SemanticModel/)
    .platform                                       ← fresh logicalId GUID, displayName=project name
    definition.pbism                                ← schema definitionProperties/1.0.0, version 4.2
    definition/
      database.tmdl                                 ← compatibilityLevel: 1600
      model.tmdl                                    ← MUST include __PBI_TimeIntelligenceEnabled = 0
      expressions.tmdl                              ← env parameters with fresh lineageTag GUIDs
```

### Critical recipe constraints

| Aspect | Rule | Why |
|---|---|---|
| **Format** | PBIR-Legacy (NOT PBIR-modular) | Power BI Desktop converts modular → legacy on save; theme does not auto-apply when generated as modular |
| **Encoding** | UTF-8 **without BOM** for ALL files: `.pbip`, `.platform`, `.pbir`, `.pbism`, `.json`, `.tmdl` | BOM breaks Power BI Desktop parsing silently |
| **Theme `name` field** | ≤ 16 chars | Power BI derives internal filenames (with 17-char hash + `.json` suffix) from this; longer names create paths that exceed 256 chars |
| **Enum values** | Case-sensitive: `Left`, `Top`, `Bottom`, `Right`, `Center`, `Auto`, `BottomOnly`, `Fit` | NEVER lowercase — Power BI silently rejects the theme |
| **Internal folder names** | Use short `Report/` and `SemanticModel/` | Saves ~17 chars of path per nesting level vs `{Project}.Report/` |
| **MAX_PATH** | 256 chars | Power BI Desktop limit, stricter than Windows 260; pre-flight check mandatory |
| **Section name** | 20 hex chars random per project | Generated fresh per scaffold |
| **GUIDs** | Fresh `logicalId` per `.platform`, fresh `lineageTag` per expression | Each scaffold gets unique IDs |
| **Base theme** | Always copy `BaseThemes/CY26SU04.json` (or current Power BI default) into `SharedResources/` | Required even when custom theme is also present |
| **Custom theme** | Copy from `Global Theme/pbi-theme/theme/` into `Report/StaticResources/RegisteredResources/` | Source-of-truth lives in Global Theme |
| **`compatibilityLevel`** | 1600 in `database.tmdl` | NOT 1567 |
| **`model.tmdl` annotation** | `annotation __PBI_TimeIntelligenceEnabled = 0` | **MANDATORY.** Disables Auto Date/Time. Without this, Power BI auto-creates `DateTableTemplate_<GUID>.tmdl` (60+ chars) for every date column when data is loaded — instantly explodes MAX_PATH 256 |
| **`model.tmdl` annotation** | `annotation PBI_ProTooling = ["DevMode"]` | Required for the file to be recognized as Dev Mode |
| **`definition.pbism`** | `version: "4.2"` | Schema version |
| **`definition.pbir`** | `version: "4.0"`, schema `definitionProperties/2.0.0` | Schema version |
| **`report.json`** | At root of `Report/`, NOT in `definition/` | PBIR-Legacy convention |
| **`report.json` config** | Stringified inner JSON with `themeCollection` (`baseTheme` + `customTheme`), `settings` block | The customTheme `name` references the filename in `RegisteredResources/` |
| **`report.json` resourcePackages** | Numeric types: `1` = RegisteredResources, `2` = SharedResources, `201` = CustomTheme, `202` = BaseTheme | Power BI rejects string types |

### Theme-optional behavior (graceful degradation)

Phase 2 must NOT fail when no custom theme exists. Pre-flight check:

```
if Test-Path "<Client>/Global Theme/pbi-theme/theme/*.json":
    → copy theme into PBIP RegisteredResources, write full themeCollection (baseTheme + customTheme)
else:
    → omit customTheme from themeCollection
    → omit RegisteredResources entry from resourcePackages (or leave items array empty)
    → PBIP opens with Power BI default theme (CY26SU04 only)
    → emit warning: "No custom theme found. PBIP opens with Power BI default. Run pbi-theme later, then re-run this skill in mode 4 (refresh-theme) to propagate."
```

This makes `pbi-theme` genuinely optional — clients without it can still generate PBIPs.

## Phase 2 execution algorithm

```
STEP 1 · Pre-flight (see "Pre-flight checks" above; abort on any failure).

STEP 2 · Generate fresh GUIDs and IDs
  $reportLogicalId  = New-Guid
  $smLogicalId      = New-Guid
  $sectionName      = (-join (1..20 | ForEach-Object { '0123456789abcdef'[(Get-Random -Maximum 16)] }))
  $expressionGuids  = @{
    Source_Url    = New-Guid
    Source_Schema = New-Guid
    Source_Type   = New-Guid
    Source_Auth_Method = New-Guid
  }

STEP 3 · Compute target paths
  $buildPath  = "<Client>/Power BI Repository/01. Dev/<Project>/02_Build"
  $reportPath = "$buildPath/Report"
  $smPath     = "$buildPath/SemanticModel"
  $regResPath = "$reportPath/StaticResources/RegisteredResources"
  $sharedResPath = "$reportPath/StaticResources/SharedResources/BaseThemes"

STEP 4 · Determine theme handling (BYO theme detection from pre-flight)
  $themeFilename = (filename of selected JSON or null)
  $themeSourcePath = (path to selected JSON or null)

STEP 5 · Confirm with operator
  Present:
    - Project name
    - Target build path
    - Theme handling (with theme / without)
    - List of files about to be created (~10-12 files)
  Ask: "Generate? (y/N)"

STEP 6 · Generate files (each as UTF-8 WITHOUT BOM)
  
  6.1  Create folders: $reportPath, $smPath, $regResPath, $sharedResPath, "$smPath/definition"
  
  6.2  Write <Project>.pbip:
       {
         "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/pbiProperties/1.0.0/schema.json",
         "version": "1.0",
         "artifacts": [
           { "report": { "path": "Report" } }
         ]
       }
  
  6.3  Write Report/.platform with $reportLogicalId
  6.4  Write Report/definition.pbir
  6.5  Write Report/report.json (with $sectionName, themeCollection, resourcePackages)
  6.6  Copy CY26SU04.json into $sharedResPath (the bytes of this file are deterministic; embed as a resource bundled with the skill)
  6.7  If $themeSourcePath: validate (enums, name length, hex) → copy to $regResPath
  
  6.8  Write SemanticModel/.platform with $smLogicalId
  6.9  Write SemanticModel/definition.pbism (version 4.2)
  6.10 Write database.tmdl (compatibilityLevel 1600)
  6.11 Write model.tmdl with __PBI_TimeIntelligenceEnabled = 0 + PBI_ProTooling = ["DevMode"]
  6.12 Write expressions.tmdl with the 4 source parameters + fresh lineageTags

STEP 7 · Validate output
  - Test-Path on every generated file
  - Test JSON parse on .pbip, .platform, .pbir, .pbism, report.json, theme.json
  - Lowercase enum sweep on theme.json (must find ZERO lowercase enums)
  - File-by-file BOM check (must be ZERO BOMs)
  - Path budget recheck on every generated path

STEP 8 · Closing handoff
  Print:
    - Absolute path of the .pbip
    - "Open with Power BI Desktop. Theme should auto-apply on first open. If you load data
       and Power BI prompts you about Auto Date/Time, decline (this skill disabled it by design).
       To version-control this work, run mode 3 (Git init)."
```

## Phase 2 hard safety rules

1. **Never overwrite an existing PBIP.** If `02_Build/` is non-empty, stop.
2. **Never modify Global Theme.** Phase 2 only READS the theme JSON; copies it into the PBIP's RegisteredResources/.
3. **Never write BOM.** Use explicit `[System.IO.File]::WriteAllText($path, $content, [System.Text.UTF8Encoding]::new($false))` in PowerShell.
4. **Never run Power BI Desktop programmatically** to "validate" the generated PBIP — only file-level validation. Operator opens it manually.

## Phase 2 reference script

The calibrated reference algorithm lives in `Gus bavia brand/_step-pbip-gen-from-scratch.ps1` from the calibration session. Future versions of this skill should bundle this script under `pbi-lifecycle/references/phase2-pbip-scaffold.ps1` so the agent can invoke it directly. For v0.3.0, the agent generates files inline using the recipe above.

---

# Phase 3 · Git Tracking + GitHub repo (AVAILABLE)

Phase 3 initializes git at the **client root** (one repo per client, all projects share the same repo), creates a private GitHub repo via `gh` CLI, makes the snapshot-zero commit, and establishes the project-scoped commit pattern for ongoing work.

This was validated end-to-end on 2026-05-04 against a disposable test project.

## Why one repo per client (locked architecture decision)

| Ganho | Why |
|---|---|
| Scaffold zero pra novo projeto | Cria pasta dentro de `Power BI Repository/01. Dev/` e pronto, já tá versionado |
| Global Theme convive com os projetos | Mudou cor da marca? 1 commit alcança todos projetos do cliente |
| Visão consolidada do cliente | `git log` mostra atividade de todos devs em todos projetos do cliente |

Custo (e mitigação): histórico mistura projetos, mitigado pela convenção `[ProjectName] msg` no commit + `git log -- "<path>"` filtra. Validado.

When `1 repo por projeto` would make sense (NOT the default): if a single project must be shared with an external party without exposing other projects of the same client. Phase 3 v0.3.0 does not implement this — clients with that need stay on the locked default.

## Pre-flight checks (every gotcha calibrated)

The agent verifies in this exact order:

### Pre-flight 1 · Power BI Desktop is closed

```powershell
Get-Process -Name 'PBIDesktop' -ErrorAction SilentlyContinue
```

If a process exists → ask operator to close it. Power BI Desktop holds locks on `.pbip` files; git operations during an open session can corrupt the repo.

### Pre-flight 2 · gh CLI installed

```powershell
$ghPath = "C:\Program Files\GitHub CLI\gh.exe"
if (-not (Test-Path $ghPath)) {
  # Install via winget
  winget install --id GitHub.cli --accept-source-agreements --accept-package-agreements -e
}
```

If install fails → instruct operator to install manually from https://cli.github.com/ and re-run.

### Pre-flight 3 · gh authenticated

```powershell
& $ghPath auth status
```

If exit code ≠ 0 → run interactive login:

```
The skill prints to operator:
"Run this in a fresh PowerShell:
  gh auth login
Answer the prompts:
  - Where: GitHub.com
  - Protocol: HTTPS
  - Authenticate Git: Yes
  - Auth method: Login with a web browser
  - Open the URL printed, paste the device code, authorize.
Tell me when done."

Operator confirms → recheck `gh auth status`.
```

### Pre-flight 4 · gh auth setup-git (THE STEP THAT IS EASY TO MISS)

```powershell
& $ghPath auth setup-git
```

**This is mandatory and separate from `gh auth login`.** Without it, `git push` fails after a successful login with the misleading error "Invalid username or token. Password authentication is not supported for Git operations."

The skill always runs this command after auth login is confirmed. Idempotent — safe to run repeatedly.

### Pre-flight 5 · git installed and configured

```powershell
git --version
git config --global user.name
git config --global user.email
```

If git missing → instruct install from https://git-scm.com/downloads. If `user.name` or `user.email` empty → ask operator and run `git config --global` for both.

If the operator wants per-repo identity different from global (e.g., personal email for personal client repos): Phase 3 offers the option *"Use this email for this repo only? (current global is `<email>`)"* — if yes, sets `git config user.email` (without `--global`) inside the client root after `git init`.

### Pre-flight 6 · Client root is a directory, not yet a repo

```powershell
Test-Path "$clientPath\.git"
```

If `.git` already exists → ask operator: *"This client root is already a git repo. Skip Phase 3 (already initialized) or stop?"* — never re-init.

## Phase 3 execution algorithm

```
STEP 1 · Pre-flight checks (see all 6 above; abort on any failure).

STEP 2 · git init at client root
  cd "$clientPath"
  git init -b main

STEP 3 · Write .gitignore
  Content (locked template — every line was validated):
    # Power BI Desktop runtime / per-machine
    **/.pbi/cache.abf
    **/.pbi/edit.lock
    **/.pbi/recovery.bak
    **/.pbi/localSettings.json
    **/.pbi/editorSettings.json
    **/.pbi/cache/
    
    # Skill outputs (regenerate-able from PBIP)
    **/03_Docs/internal/audit/*
    **/03_Docs/internal/docs/*
    
    # Phase 4 theme refresh backups (local-only, recovery point per run)
    .pbi-lifecycle-backups/
    
    # OS / editor noise
    .DS_Store
    Thumbs.db
    desktop.ini
    .vscode/
    .idea/

STEP 4 · Stage everything + first commit (snapshot zero)
  git add .
  git commit -m '[Setup] Snapshot zero: client folder structure + Global Theme + initial PBIP projects'
  
  Note: the commit message uses the [Setup] prefix for the snapshot-zero commit. All subsequent
  commits use the project-scoped pattern described below.

STEP 5 · Create private GitHub repo
  $slug = $clientName.ToLower() -replace '[^a-z0-9]+','-' -replace '^-+|-+$',''
  
  Confirm with operator:
    "About to create:
       - GitHub repo: <user-or-org>/<slug>
       - Visibility: PRIVATE (no opt-out — this contains client IP)
       - Description: <one-liner from operator>
     Proceed? (y/N)"
  
  On y:
    & gh repo create $slug --private --source=. --push --description "<description>"
  
  CRITICAL: --private flag is mandatory. The skill never creates a public repo for client work.
  No flag, no prompt, no opt-in to public. Hardcoded in the agent's behavior.

STEP 6 · Verify repo state
  & gh repo view "$user/$slug" --json name,visibility,url,defaultBranchRef
  Confirm visibility == "PRIVATE" before declaring success.
  Print the repo URL.

STEP 7 · Closing handoff
  Print:
    - Repo URL (clickable in modern terminals)
    - "Now open VS Code at the client root path. Source Control panel (Ctrl+Shift+G) shows pending changes.
       For each work session, stage only the files of the project you worked on (one project per commit).
       Commit message convention: [ProjectName] short description.
       Special prefixes: [Global Theme] for theme-wide changes, [Global Info] for shared info, [Repo] for repo-wide changes.
       To open VS Code at this path now, type: code '<client path>'"
```

## Project-scoped commit pattern (validated 2026-05-04)

This is the convention the skill enforces and teaches:

```powershell
# After making changes to ONE project's PBIP / files:
cd "<client root>"

# Stage ONLY that project (auto-scoped — important):
git add "Power BI Repository/01. Dev/<Project>/"

# Commit with the prefix:
git commit -m "[<Project>] <short description of change>"

# Push:
git push
```

If multiple projects changed in the same work session → **separate commits per project**. Never bundle.

### Special prefixes

| Prefix | When |
|---|---|
| `[<ProjectName>]` | Default; any change inside `Power BI Repository/01. Dev/<Project>/` (or 02. Test, 03. Prod after promote) |
| `[Global Theme]` | Change inside `Global Theme/` |
| `[Global Info]` | Change inside `Global Info/` |
| `[Repo]` | Repo-level changes (`.gitignore`, structure renames, README) — rare |
| `[Setup]` | The very first snapshot-zero commit ONLY — never used again |

### Future skill mode: `commit` (queued for v0.4)

A `pbi-lifecycle commit` mode will:
- Run `git status` and parse modified paths
- Detect which project(s) changed
- If exactly 1 → propose auto-staged commit with the right prefix
- If > 1 → force separate commits, one per project, with operator confirmation

Until that ships, the operator commits manually using the pattern above (or via VS Code Source Control GUI — see Education section).

## Phase 3 hard safety rules

1. **Never push to a public repo.** `--private` flag mandatory. No CLI flag to override. No prompt option for public.
2. **Never `git push --force`** to main. The skill does not include a force-push command.
3. **Never `git reset --hard` or `git clean -fdx`.** No destructive operations whatsoever.
4. **Never delete branches.** The skill creates only `main` and never deletes anything.
5. **Never store secrets.** `.gitignore` blocks `.env`, `.credentials`, etc. by default; if the operator commits a file matching those patterns, the agent refuses and asks them to remove it from staging.
6. **Always verify private** after `gh repo create` (Step 6) before declaring success. If repo accidentally appears as `PUBLIC` (very unlikely with `--private` flag, but defensive check), STOP and inform operator immediately.

---

# Phase 4 · Theme Refresh (AVAILABLE)

Phase 4 propagates a Global Theme update across all projects of a client, respecting per-project overrides declared in `theme.md`. Surgical-by-default with `--dry-run` preview, automatic backup before every write, and atomic per-run validation (all-or-nothing — if any project's merged theme is invalid, NO files are written).

## Use cases

1. Client requests a brand refresh — change accent color, font, or palette across all reports.
2. Client requests a fix on a specific project (e.g., color collision with another product) — operator edits that project's `theme.md` to declare an override; subsequent global refreshes preserve the override.
3. Operator runs `pbi-theme` companion skill to regenerate the Global Theme JSON; Phase 4 propagates the new file into all PBIPs.

## Inputs the skill collects

- `--theme <path>` (optional) — path to the source theme JSON. Default: auto-detect from `Global Theme/pbi-theme/theme/*.json` (prompt operator if multiple exist).
- `--project <ProjectName>` (optional) — limit refresh to one project. Default: all projects of the client.
- `--env <dev|test|prod|all>` (optional) — limit to one env. Default: `all` (refresh atinge todos ambientes — tema é cosmético, não quebra promoção).
- `--dry-run` (optional flag) — simulate the run, print what would change, write nothing. Default: false.
- `--commit-message <text>` (optional) — short description of the change for commit messages. Default: prompt operator.

## Pre-flight checks (every gotcha calibrated)

The agent verifies in this exact order. **If any check fails, ABORT before any write.**

### Pre-flight 1 · Power BI Desktop closed

```powershell
Get-Process -Name 'PBIDesktop' -ErrorAction SilentlyContinue
```

If a process exists → ask operator to close it. Power BI Desktop holds locks on `.pbip` files and on the theme JSON files inside `RegisteredResources/`. Writing while open corrupts the model session.

### Pre-flight 2 · Client root + git repo

- `<clientPath>` exists
- `<clientPath>\.git\` exists (Phase 3 ran; Phase 4 commits + pushes)
- `git status` is clean OR has only changes the operator explicitly accepts. If staged/uncommitted changes exist that aren't related to this refresh → warn operator: "You have uncommitted changes. Phase 4 commits will mix with them. Stash or commit first? (stash/commit/proceed-anyway/abort)"

### Pre-flight 3 · Source theme exists and validates

- Resolve `--theme` flag or auto-detect from `Global Theme/pbi-theme/theme/*.json`
- Validate the source JSON BEFORE running discovery:
  - JSON parse validity
  - **Lowercase enum sweep** — find any `"left"`, `"top"`, `"bottom"`, `"right"`, `"center"`, `"auto"`, `"bottomonly"`, `"fit"` and abort if found (must be CamelCase)
  - **`name` field length** — must be ≤ 16 characters
  - **Hex color check** — every hex value matches `/^#[0-9a-fA-F]{6}$/`
- If source is invalid → ABORT, report exact error path. Operator fixes source theme first.

### Pre-flight 4 · Override files (theme.md) parse cleanly

For every project in scope (filtered by `--project` and `--env`), if `01_Context/theme.md` exists with a YAML front-matter `overrides:` block, parse it and validate that each override path resolves to a valid JSON path inside the source theme schema. If any override is malformed → ABORT, report which project's `theme.md` is invalid.

## Override format (YAML front-matter in theme.md)

The skill recognizes this strict format. Anything else is treated as commentary (no override):

```yaml
---
inherits: ../../../../Global Theme/pbi-theme/theme/<theme-filename>.json
overrides:
  dataColors[0]: "#7C3AED"
  neutral: "#7C3AED"
  textClasses.title.color: "#1E3A5F"
reason: "Client requested purple primary for this project to avoid color collision with another product"
---

(rest of theme.md is freeform notes for humans)
```

Override path syntax:
- Top-level keys: `neutral`, `background`, `foreground`
- Array indices: `dataColors[0]`, `dataColors[1]`
- Nested objects: `textClasses.title.color`, `visualStyles.card.\*.background.solid.color`
- Quote string values; numbers and booleans pass through

**Override semantics (non-destructive deep merge):**
- The override path MUST exist in the source theme. If not → ABORT (we never invent new keys).
- The override REPLACES the value at that path. It does not merge nested objects unless the operator specifies sub-paths individually.
- Overrides apply ON TOP of the source theme. The base of the merged result is always the latest source theme JSON.

## Phase 4 execution algorithm

```
STEP 1 · Pre-flight (all 4 checks above; abort on any failure).

STEP 2 · Resolve scope
  $sourceTheme = (--theme path or auto-detected from Global Theme)
  $envs = if --env set → [<env>] else → ["01. Dev", "02. Test", "03. Prod"]
  $projectFilter = (--project flag or null)

STEP 3 · Discover candidate PBIPs
  $candidates = @()
  foreach $env in $envs:
    $glob = "$clientPath\Power BI Repository\$env\*\02_Build\Report\StaticResources\RegisteredResources\*.json"
    foreach match:
      $projectName = (parent of 02_Build folder name)
      if $projectFilter and $projectName ≠ $projectFilter → skip
      $contextThemeMd = "$clientPath\Power BI Repository\$env\$projectName\01_Context\theme.md"
      $candidates += @{
        env = $env
        project = $projectName
        targetThemePath = $match
        contextThemeMd = $contextThemeMd
      }

  if $candidates is empty → STOP, "No PBIPs found matching scope. Nothing to refresh."

STEP 4 · For each candidate, compute target merged theme
  $sourceContent = Read $sourceTheme as JSON
  
  foreach $candidate:
    $merged = deep-clone $sourceContent
    
    if Test-Path $candidate.contextThemeMd:
      $overrides = ParseYamlFrontMatter $candidate.contextThemeMd
      if $overrides.overrides exists:
        foreach $path => $value in $overrides.overrides:
          ApplyOverride $merged $path $value
          # ApplyOverride walks $merged using $path notation
          # If $path doesn't resolve in $merged → ABORT (collect all errors, report after loop, then halt)
        $candidate.appliedOverrides = $true
        $candidate.overrideReason = $overrides.reason
    
    # Validate merged result (enum sweep, name length, hex)
    Validate $merged → if invalid: ABORT
    
    $candidate.mergedTheme = $merged

  if any candidate ABORTED in this step:
    Print all collected errors. Stop. No writes happen.

STEP 5 · Diff each merged theme vs current
  foreach $candidate:
    $current = Read $candidate.targetThemePath as JSON
    if DeepEquals($candidate.mergedTheme, $current):
      $candidate.action = "skip"   # already current
    else:
      $candidate.action = "update"

  $toUpdate = $candidates where action == "update"
  $toSkip = $candidates where action == "skip"

  if $toUpdate is empty:
    Print: "All <N> projects in scope are already current. Nothing to do."
    STOP.

STEP 6 · Present diff summary + ask operator
  Print:
    "<count of $toUpdate> projects will be updated:
       [<Project A>] (env: <env>) — verbatim refresh
       [<Project B>] (env: <env>) — override applied (<override reason>)
       ...
     <count of $toSkip> projects already current (skipped):
       [<Project C>], [<Project D>], ..."
  
  if --dry-run flag is set:
    Print: "Dry-run mode: no files written, no commits made."
    STOP.
  
  Ask: "Apply? (y/N)" → default N

STEP 7 · Backup all targets BEFORE first write
  $timestamp = (Get-Date -Format "yyyyMMdd-HHmmss")
  $backupRoot = "$clientPath\.pbi-lifecycle-backups\refresh-theme-$timestamp"
  
  foreach $candidate in $toUpdate:
    $backupTarget = "$backupRoot\$($candidate.env)\$($candidate.project)\theme-pre-refresh.json"
    Ensure parent folder exists
    Copy $candidate.targetThemePath to $backupTarget
  
  Also write $backupRoot\REFRESH-MANIFEST.json with:
    - timestamp
    - source theme path + hash
    - list of all candidates (action + override status)
    - operator who ran the refresh (from git config user.email)

STEP 8 · Apply writes
  foreach $candidate in $toUpdate:
    Write $candidate.mergedTheme to $candidate.targetThemePath
      - UTF-8 WITHOUT BOM
      - Indent: 2 spaces (preserves existing convention)
    Verify write succeeded:
      - Re-read file, parse JSON
      - DeepEquals with intended $candidate.mergedTheme
    if verification fails:
      RESTORE from backup
      Continue with remaining candidates
      Collect failure into report

  if any verification failed:
    Print all failures.
    For successfully written: COMMIT them anyway (partial success preserves the work that worked).
    For failed: backup restored to original state.

STEP 9 · Stage + commit + push
  cd $clientPath
  
  Determine commit strategy:
    $verbatimUpdates = $toUpdate where appliedOverrides == false
    $overrideUpdates = $toUpdate where appliedOverrides == true
  
  if $projectFilter is set:
    # Single project scope → single commit
    git add "Power BI Repository/<env>/<project>/02_Build/Report/StaticResources/RegisteredResources/*.json"
    git commit -m "[<project>] Refresh theme: <commit-message>"
  
  elif $overrideUpdates is empty:
    # All verbatim → single Global Theme commit
    git add (each $candidate.targetThemePath)
    git commit -m "[Global Theme] Refresh propagated: <commit-message>"
  
  else:
    # Mix of verbatim + override → separate commits
    if $verbatimUpdates is non-empty:
      git add (each verbatim $candidate.targetThemePath)
      git commit -m "[Global Theme] Refresh propagated: <commit-message>"
    foreach $candidate in $overrideUpdates:
      git add $candidate.targetThemePath
      git commit -m "[<project>] Refresh theme (override preserved): <commit-message>"
  
  git push

STEP 10 · Closing handoff
  Print:
    - Updated repo URL
    - List of affected projects + which got override-merge vs verbatim
    - Backup folder path: "$backupRoot"
    - "To revert this refresh:
       OPTION A (git): git revert <commit-hashes-listed-above>
       OPTION B (filesystem): copy theme-pre-refresh.json from backup folder back to RegisteredResources/
       Either way, no data reload needed in Power BI Desktop. Theme is read on file open."
```

## Phase 4 hard safety rules

1. **Power BI Desktop must be closed.** Pre-flight enforced.
2. **All-or-nothing validation.** If any project's merged theme is invalid, ZERO writes happen. The skill aborts in Step 4 before reaching Step 7.
3. **Always backup before write.** Step 7 captures every target file into a timestamped backup folder. Operator can restore manually if anything looks wrong post-refresh.
4. **Never modify the source Global Theme during Phase 4.** The skill READS the source theme; it never writes to `Global Theme/pbi-theme/theme/`. To update the source, use the `pbi-theme` companion skill, then run Phase 4.
5. **Override merge is non-destructive.** Overrides only REPLACE existing keys; they never INSERT new keys (would require the source theme to have the path). They never DELETE keys.
6. **Per-write verification.** After each file write, re-read and deep-equals check vs intended content. If verification fails, restore from backup and continue with remaining candidates.
7. **`--dry-run` always available.** Operator can preview every change before committing. Especially recommended for the first refresh of a client to validate override merging.
8. **Backup folder is git-ignored.** The skill writes to `$clientPath\.pbi-lifecycle-backups\` which the .gitignore template already excludes (added in v0.3.3). Operator never sees backups in `git status`.

## Backup folder layout (auto-created)

```
<clientPath>/.pbi-lifecycle-backups/refresh-theme-20260504-183000/
  REFRESH-MANIFEST.json                       ← what was changed, by whom, when
  01. Dev/
    <Project A>/theme-pre-refresh.json
    <Project B>/theme-pre-refresh.json
  02. Test/
    <Project A>/theme-pre-refresh.json
  03. Prod/
    <Project A>/theme-pre-refresh.json
```

The backup is local-only. To preserve refresh history in the repo, rely on git: each refresh produces commits with the project-scoped pattern, so `git log` is the authoritative history.

## Failure modes and recovery

| Failure | Behavior | Recovery |
|---|---|---|
| Source theme JSON is invalid (Pre-flight 3) | Abort before any work. | Operator fixes source theme JSON, re-runs. |
| One project's `theme.md` overrides reference an invalid path | Abort in Step 4 (all-or-nothing). | Operator fixes the offending `theme.md`, re-runs. |
| Operator rejects diff summary (Step 6) | Stop without writes. | No recovery needed; nothing changed. |
| Mid-write file system error | Restore failed file from backup. Continue with remaining. | Manual: copy successful writes from backup if needed. Git history shows partial success. |
| Power BI Desktop opened during refresh | Pre-flight 1 catches it. If race-condition slip-through detected by file lock error during write → restore backup, abort. | Operator closes PBI, re-runs. |
| `git push` fails (auth, network) | Local commits intact. | `git push` later, or troubleshoot `gh auth setup-git`. |

## Why this is shippable now

Phase 4 ships in v0.3.3 with the following calibration coverage:
- **Validated empirically:** the surgical override mechanism (changing 3 colors only in one project's RegisteredResources/, leaving Global Theme + other projects untouched) was tested manually during 2026-05-04 calibration. Result: visually verified in Power BI Desktop on first open.
- **Algorithm safety:** all-or-nothing pre-validation + mandatory backup + per-write verification means worst-case failure leaves the system recoverable. No partial corruption possible.
- **Pending real-multi-project test:** the YAML front-matter parsing in `theme.md` and the override merge logic have not been run in a single transaction across 3+ projects. The first real client refresh is also the validation; the safety rails handle the unknowns.

If the first real run uncovers a bug, recovery is via `git revert` + restore from backup. Operator never loses work.

---

# Phase 5 · Environment Promotion + Lifecycle Reporting (DESIGN LOCKED, IMPLEMENTATION QUEUED for v0.5)

Phase 5 is the most consequential phase: it moves the project across environments and produces the artifacts that govern client trust. It has TWO sub-features that always run together on promotion:

1. **Promote** — copy `02_Build/` from source env to target env + rewrite the 3-4 connection parameters from the target env's `source.json`.
2. **Diff-findings report** — generate an editorial-deck-style report (same visual family as `pbi-model-doc`) comparing source vs target so the operator approves consistency BEFORE the commit lands. Without this, promotion is a black-box copy. With this, every promotion is auditable.

A third sub-mode runs at project close-out:

3. **Deployment lifecycle report** — unified HTML deck covering the entire project journey (Dev → Test → Prod). Auto-triggered on Test → Prod promotion, OR invoked explicitly via `close-project` mode for archival. This is the document that ships to the client at handoff.

## What changes between environments

**Empirically confirmed during 2026-05-04 calibration:** the only fields that change between Dev/Test/Prod are the data source connection parameters. Specifically, the values in `01_Context/source.json` per env, which map to parameters in `expressions.tmdl`:

- `catalog` (database name)
- `url` / `http host` (server URL)
- `path` / `schema` (folder, lakehouse path, or schema name)
- Optionally `auth method` (rare; usually same across envs)

**Everything else (model, measures, visuals, theme, M code structure) is identical across envs.** Any other diff between source and target is a defect or a divergence — and the diff-findings report surfaces it.

## Sub-mode A · Promote with diff-findings

### Algorithm

```
STEP 1 · Inputs
  --project <ProjectName>           required
  --from <dev|test>                 required
  --to <test|prod>                  required (must be next in sequence; never dev→prod)
  --dry-run                         optional flag (preview only, no commit)
  --skip-diff-report                optional flag (NOT recommended; for emergency hotfixes)

STEP 2 · Pre-flight
  - Power BI Desktop CLOSED
  - $sourcePath = "$clientPath/Power BI Repository/<from-env>/<Project>"
  - $targetPath = "$clientPath/Power BI Repository/<to-env>/<Project>"
  - $sourcePath/02_Build/<Project>.pbip exists
  - $targetPath/01_Context/source.json exists with `environment: "<to>"` and all connection params filled
    - If any source.json field is still a placeholder (e.g. <https://server-or-endpoint>) → STOP, ask operator to fill it
  - $targetPath/02_Build/ state:
    - Empty → first promotion, proceed
    - Non-empty → re-promotion, STOP and ask: "Target has prior promotion. Overwrite (re-promote)? (y/N)"

STEP 3 · Read source/target env params
  Parse $sourcePath/01_Context/source.json → $sourceParams
  Parse $targetPath/01_Context/source.json → $targetParams
  Validate parameter count matches and names match. Mismatch → STOP.

STEP 4 · Pre-copy diff capture
  If $targetPath/02_Build/<Project>.pbip exists (re-promotion case):
    $previousTargetSnapshot = capture target's current 02_Build content (in-memory or temp folder)
  Else:
    $previousTargetSnapshot = null (first promotion; nothing to compare on the target side yet)

STEP 5 · Copy source 02_Build → target 02_Build (in temp staging)
  $stagingPath = "$clientPath/.pbi-lifecycle-staging/promote-<from>-to-<to>-<timestamp>"
  Copy-Item -Recurse $sourcePath/02_Build/* $stagingPath/
  
  Note: stage in a temp folder first, run diff + parameter substitution, then move into place ONLY after operator approval.

STEP 6 · Substitute parameters in staged expressions.tmdl
  Open $stagingPath/SemanticModel/definition/expressions.tmdl
  For each parameter in $targetParams:
    Replace value in matching expression
  Validation: every $targetParams entry must have matched a substitution. Unmatched → STOP, restore.

STEP 7 · Generate diff-findings report
  Compare:
    A) Source 02_Build (read-only) ←→ Staged 02_Build (post-substitution) ←→ $previousTargetSnapshot (if exists)
  
  For each layer, run a structured diff:
    
    Layer 1 · Model (TMDL)
      Diff $sourcePath/SemanticModel/definition/* vs $stagingPath/SemanticModel/definition/*
      Output:
        - Tables added / removed / renamed
        - Columns added / removed / renamed / type changed
        - Measures added / removed / DAX modified (with before/after DAX block)
        - Relationships added / removed / cardinality or direction changed
        - Roles (RLS) added / removed / filter expression changed
      Severity classification:
        - Critical: relationship deletions, role deletions, table renames (downstream visuals break)
        - Major: measure DAX changes, column type changes
        - Minor: measure rename, description changes
        - Info: parameter substitutions (the legitimate env-driven changes — flagged as "expected")
    
    Layer 2 · Power Query (M)
      Diff $sourcePath/SemanticModel/definition/expressions.tmdl vs $stagingPath/.../expressions.tmdl
      Output:
        - Expressions added / removed
        - Step-level diff for changed expressions (using shared M parser from pbi-model-doc/audit)
        - Apply sanitizer (credentials/paths redacted) before rendering
      Severity:
        - Critical: source-type change (DirectQuery → Import or vice versa)
        - Major: filter changes, business-rule recodings
        - Minor: comment-only changes
        - Info: parameter substitutions (catalog/url/path values — expected)
    
    Layer 3 · Visuals (report.json + page JSON)
      Diff $sourcePath/Report/* vs $stagingPath/Report/*
      Output:
        - Pages added / removed / renamed / reordered
        - Per page: visuals added / removed / type changed
        - Visual-level binding changes (which fields each visual references)
      Severity:
        - Critical: page deletions, visual binding breakage
        - Major: visual type changes, filter context shifts
        - Minor: visual repositioning, formatting tweaks
    
    Layer 4 · Theme
      Diff $sourcePath/Report/StaticResources/RegisteredResources/<theme>.json vs staged version
      Should normally be IDENTICAL across envs (theme is client-level, not env-level)
      Any difference is a defect → flag as Major
  
  Render report:
    HTML deck (editorial style, same visual family as pbi-model-doc) with 8 sections:
      01. Promotion Header (from→to, project, operator, source/target commit hashes, dates)
      02. Overview (totals: critical/major/minor/info counts per layer)
      03. Model Changes (with severity-grouped findings)
      04. Power Query Changes (sanitized, severity-grouped)
      05. Visual Changes (pages + visuals with thumbnails when feasible)
      06. Theme Changes (should be empty in healthy promotion)
      07. Parameter Substitutions (the legitimate env-driven changes — green/expected)
      08. Approval Block (Approve / Reject + reason field)
    
    Markdown sidecar (for git history + searchability)
    
  Output paths (lifecycle-managed mode):
    HTML  → $targetPath/03_Docs/internal/audit/promote-<from>-to-<to>-<YYYY-MM-DD>.html
    MD    → $targetPath/03_Docs/internal/audit/promote-<from>-to-<to>-<YYYY-MM-DD>.md
    JSON  → $targetPath/03_Docs/internal/audit/promote-<from>-to-<to>-<YYYY-MM-DD>-findings.json (machine-readable)

STEP 8 · Present diff summary to operator
  Print top-level summary:
    "Promotion <from> → <to> for <Project>:
       Critical: <N>   ← MUST be 0 to proceed (or override with --force)
       Major:    <N>
       Minor:    <N>
       Info:     <N> (param substitutions, expected)
     
     Diff-findings report: <html-path>
     
     Open the report and review before approving."
  
  if --dry-run flag:
    Print: "Dry-run: no commit, no copy applied. Staging at: $stagingPath"
    Operator can inspect manually. Cleanup = remove staging folder.
    STOP.
  
  if Critical findings > 0 AND --force not set:
    Print: "Critical findings present. Promotion blocked. Either:
       - Fix the source PBIP and re-run promote
       - Use --force to proceed anyway (logged as override)"
    STOP.
  
  Ask: "Approve promotion? (y/N) [type 'y' to apply, 'n' to abort]"
  If operator types reject reason, save it to the report's Approval Block.

STEP 9 · Apply promotion (operator approved)
  Move staging → target:
    if $previousTargetSnapshot existed (re-promotion):
      Backup current target into .pbi-lifecycle-backups/promote-<from>-to-<to>-<timestamp>/
    Move-Item $stagingPath/* → $targetPath/02_Build/
  
  Per-write verification: re-read 02_Build files, parse, deep-equals vs staging snapshot.

STEP 10 · Stage + commit + push
  git add "Power BI Repository/<to-env>/<Project>/02_Build/"
  git add "Power BI Repository/<to-env>/<Project>/03_Docs/internal/audit/promote-<from>-to-<to>-<YYYY-MM-DD>.*"
  git commit -m "[<Project>] Promote <from> → <to>: <criticalN> critical, <majorN> major, <minorN> minor (approved by <operator>)"
  git push

STEP 11 · If promoting to Prod, trigger Sub-mode C (lifecycle report)
  Auto-invoke close-project sub-mode (Sub-mode C below) if --to == prod.
  Operator can opt-out with --skip-lifecycle-report.

STEP 12 · Closing handoff
  Print:
    - Target path
    - Diff-findings report path
    - Repo URL with the new promotion commit
    - "Open the promoted PBIP in Power BI Desktop, refresh data with the target env credentials,
       and validate against the diff-findings expectations."
```

### Sub-mode A safety rules

1. **Power BI Desktop must be closed.** Pre-flight enforced.
2. **Staging folder, never direct overwrite.** Phase 5 always stages to `.pbi-lifecycle-staging/` first, runs diff, asks operator, then moves on approval.
3. **Critical findings block promotion.** `--force` is the only override and it's logged in the diff-findings report.
4. **Re-promotion always backs up first.** Previous target state captured before overwrite.
5. **All-or-nothing parameter substitution.** If any target.source.json field is unmatched in expressions.tmdl, abort. Operator must reconcile before promotion.
6. **Diff-findings report is mandatory.** `--skip-diff-report` exists for emergency hotfixes only and is logged in the commit message.

## Sub-mode B · Diff-findings report only (no promotion)

Operator can ask for a "what would change?" preview without promoting. Same algorithm as Sub-mode A through Step 7, then STOP with the report rendered for inspection. No staging move, no commit.

Useful for:
- Pre-promotion review (did dev work yesterday introduce drift?)
- Post-promotion verification (does target match expectations after manual hotfix?)
- Doc generation for change requests (HTML report can be attached to a CR ticket)

Invoked via `pbi-lifecycle promote --project <X> --from <env-a> --to <env-b> --dry-run`.

## Sub-mode C · Deployment lifecycle report (close-project)

The lifecycle report is the ARTIFACT that ships to the client at handoff. It tells the entire story of the project in one HTML deck.

Triggered automatically on Test → Prod promotion (operator can opt out), OR explicitly via:

```
pbi-lifecycle close-project --project <ProjectName>
```

### Algorithm

```
STEP 1 · Inputs
  --project <ProjectName>            required
  --output-mode <html|md|both>       optional (default: both)
  --include-audit-history            optional flag (default: true) — embeds prior audit scores
  --include-doc-snapshot             optional flag (default: true) — embeds last pbi-model-doc HTML
  --signoff <name>                   optional (operator's name for sign-off block; default: git config user.name)
  --client-name <name>               optional override (default: extract from path)

STEP 2 · Aggregate project history
  Walk git log filtered by paths matching:
    "Power BI Repository/{01. Dev,02. Test,03. Prod}/<Project>/**"
    "Global Theme/**" (theme changes that affected this project)
  
  Extract per commit:
    - commit hash, date, author, message
    - which env(s) the commit touched
    - prefix tag ([<Project>], [Global Theme], etc.)
  
  Identify milestones:
    - First commit ([Setup] snapshot zero or first [<Project>] commit)
    - Each Dev → Test promotion
    - Each Test → Prod promotion
    - Major theme refreshes affecting the project
    - Last commit before close
  
  Aggregate audit history:
    Glob "Power BI Repository/*/<Project>/03_Docs/internal/audit/*"
    For each audit run, extract: date, score, critical/major counts (from JSON sidecar)
    Plot score trend over time.
  
  Aggregate doc snapshots:
    Glob "Power BI Repository/*/<Project>/03_Docs/internal/docs/*"
    Use the most recent doc.html from prod env (or test if no prod).

STEP 3 · Render lifecycle deck
  HTML editorial deck (same visual family as pbi-model-doc) with sections:
    
    01 · Project Header
       - Client name + logo
       - Project name
       - Dates (first commit → close date)
       - Total commits, total contributors
       - Final environments deployed (Dev/Test/Prod)
    
    02 · Timeline
       - Vertical timeline with milestones (date, type, description)
       - Color-coded by env: Dev (blue), Test (amber), Prod (green)
       - Promotion arrows between envs
       - Major theme refresh markers
    
    03 · Commit Summary
       - Total commits by month (bar chart)
       - By author (if multi-dev)
       - By prefix tag (project / theme / repo)
    
    04 · Major Changes
       - Extracted from commit messages with "feat", "add", "model", "measure", "promote" keywords
       - Grouped by month
       - Each change links to the commit + the diff if available
    
    05 · Promotion Records
       - One block per Dev → Test promotion: date, source/target hashes, finding counts (critical/major/minor/info), approval status, link to diff-findings report
       - One block per Test → Prod promotion: same + sign-off
    
    06 · Audit History
       - Score trend line chart
       - Final audit summary (date, score, top remaining findings if any)
       - Link to last audit report in client_handoff/
    
    07 · Doc Snapshot
       - Embed link / preview to the latest pbi-model-doc HTML for prod env
       - 1-page summary of the model: tables count, measures count, RLS roles count
    
    08 · Final State
       - Snapshot of current Prod model: theme used, measure classification breakdown, key relationships
       - "What lives in Prod today" — the deliverable description
    
    09 · Sign-off
       - Operator signature block (name, date, "Production-ready, handoff to <Client>")
       - Optional: client signature placeholder
       - "Production-Ready" stamp icon
  
  Markdown sidecar with the same content for git-based reading + future search.

STEP 4 · Save to client_handoff
  $reportPath = "$clientPath/Power BI Repository/03. Prod/<Project>/03_Docs/client_handoff/docs/deployment-lifecycle-<Project>-<YYYY-MM-DD>.html"
  $mdPath = same path with .md extension
  
  Write both files. Verify writes.

STEP 5 · Stage + commit + push
  git add "Power BI Repository/03. Prod/<Project>/03_Docs/client_handoff/docs/"
  git commit -m "[<Project>] Generate deployment lifecycle report (close-project handoff)"
  git push

STEP 6 · Closing handoff
  Print:
    - Lifecycle report path (HTML)
    - One-line summary: "Project <X> ready for client handoff. <N> commits over <M> months. Final audit score: <S>/100. Report at <path>."
    - "If client signs off, archive the project — no further automated changes from this skill."
```

### Sub-mode C safety rules

1. **Lifecycle report is read-only on git history.** Never modifies past commits, never rewrites history.
2. **Outputs go to `client_handoff/`.** This is the only skill action that writes to `client_handoff/` automatically (rest of the time, `client_handoff/` is operator-curated). The act of generating the lifecycle report is itself the "handoff promotion".
3. **Never deletes audit or doc artifacts.** The report aggregates by reference; originals stay in `internal/`.
4. **Sign-off block defaults to git author.** Operator can override with `--signoff <name>`.
5. **Idempotent re-runs.** Generating the lifecycle report multiple times produces a fresh-dated report each time; old ones stay as historical record.

## Why this is queued for v0.5+

Phase 5 is the most ambitious phase of this skill. The implementation needs:

- **TMDL diff engine** — parse and compare two TMDL trees, classify changes by severity. Reuses M parser from `pbi-model-doc/audit`.
- **report.json diff engine** — JSON-aware diff that understands page/visual semantics, not just text-level diff.
- **HTML deck rendering** — same editorial template family as `pbi-model-doc`. Probably extracts a shared template module.
- **Git history aggregation** — walk `git log`, filter by path globs, classify commits.
- **Real-PBIP calibration** — must validate against a client PBIP with actual env-aware connections, multiple promotions, and a real handoff.

The diff-findings sub-mode (A) ships first in v0.5.0. The lifecycle report sub-mode (C) ships in v0.5.1 once handoff workflow is validated against a real project close. Sub-mode B (diff preview without promote) is automatic — it's just sub-mode A with `--dry-run`.

For v0.3.3, the operator can perform Phase 5 manually with the algorithm above as a guide. The skill provides the commit pattern and the report structure but does not automate the diff/copy/aggregation. Manual close-out: produce HTML by hand using the structure above; commit to `client_handoff/docs/`.

---

# Cross-cutting · Education layer for non-git users (AVAILABLE)

The skill's audience often includes Power BI developers who have never used git. This skill never asks them to type `git`-anything; it teaches them to use VS Code's Source Control panel, which provides the same functionality through a GUI.

## Mode 6 · "Help me read changes" walkthrough

When the operator picks mode `6`, the agent walks them through reading file changes in VS Code:

```
Walkthrough for non-git users:

1. Make sure VS Code is open AT the client root folder (not parent, not child).
   The path in the title bar should match: <client root path>

   If VS Code is open at the wrong path:
   - Close that window OR
   - File → Open Folder → navigate to <client root> and Open

2. Open the Source Control panel (left sidebar):
   - Keyboard: Ctrl+Shift+G
   - Or click the icon that looks like a branch fork (3rd from top)

3. The Source Control panel shows three sections:
   - "Changes": files modified since the last commit (yellow M decoration)
   - "Staged Changes": files marked for the next commit (green decoration; empty by default)
   - "Untracked": new files git hasn't seen yet (green U decoration)

4. To see WHAT changed in a file:
   - Click the file name in the Source Control panel (NOT in the Explorer)
   - VS Code opens a side-by-side diff:
     - Left = file as it was in the last commit
     - Right = file as it is now
     - Lines with red background = removed
     - Lines with green background = added
     - Gutter ▍ marks indicate change locations

5. To stage a file (mark it for commit):
   - In the Source Control panel, hover over the file
   - Click the + (plus) icon that appears

6. To commit:
   - In the message box at the top of Source Control panel, type:
     [ProjectName] short description of what changed
   - Click the ✓ Commit button (or Ctrl+Enter)

7. To push to GitHub:
   - Click the ... (three dots) menu in Source Control panel header
   - Choose "Push"
   - Or use the sync icon at the bottom-left of VS Code

8. To see the commit history:
   - In Explorer, right-click any file → Open Timeline
   - Shows all commits that touched the file with diffs

9. Recommended extension: GitLens
   - Ctrl+Shift+X (Extensions panel)
   - Search "GitLens"
   - Install
   - Adds inline blame info, file history, comparison views

That's the entire git workflow for daily Power BI work — no terminal commands needed.
```

This walkthrough is read-only. It does not modify any files. The operator can return to mode 6 at any time for a refresher.

## Fallback: terminal commands for advanced users

If the operator prefers terminal:

```powershell
cd "<client root>"
git status                                          # see what changed
git add "Power BI Repository/01. Dev/<Project>/"    # stage one project
git commit -m "[<Project>] short description"       # commit
git push                                            # send to GitHub
git log -- "Power BI Repository/01. Dev/<Project>/" # history of one project
```

---

# Hard safety rules (skill-wide)

The skill is conservative by design because its working surface is real client deliverables on synced drives. A single accidental delete, move, or rename can destroy weeks of work and break shared links.

## Universal rules (apply to every Phase)

1. **Never delete operator content.** No `Remove-Item`, `rm`, `del`, `git rm` (except inside Phase 3 cleanup operations that the operator explicitly confirmed), `rmdir`, `Clear-Content` against operator files.
2. **Never overwrite a file that was not generated by this skill in the same run.** If a file exists, leave it untouched and log "skipped (already exists)".
3. **Never move or rename operator files.** Folders the skill created in the same run can be renamed only with explicit confirmation.
4. **Never push to a public GitHub repo.** Always `--private`.
5. **Never force-push** to any branch.
6. **Never `git reset --hard`** or any destructive git operation.
7. **Never skip pre-flight checks.** Each Phase has pre-flight checks that are mandatory and not skippable.
8. **Never run companion skills automatically.** They are mentioned only as one-line pointers in handoffs.
9. **Always confirm before write.** No mutation happens without explicit operator y/N confirmation of the full plan.
10. **Always validate after write.** Every write is followed by a Test-Path or content check before reporting success.

## Failure modes and recovery

| Failure | Recovery |
|---|---|
| Phase 1 path budget detection: junction creation fails | Stop. Report the error. The operator can run Phase 1 again or skip the junction. The OneDrive folder is unaffected. |
| Phase 2: PBIP generation fails mid-write | Delete the partially-generated `02_Build/` content (only files the skill itself created in this run). Operator re-runs Phase 2. |
| Phase 2: theme validation fails | Stop before any write. Tell operator which validation failed and where in the JSON. |
| Phase 3: `gh auth login` fails | Stop. Operator runs the manual auth flow and re-invokes Phase 3. |
| Phase 3: `gh repo create` fails (name conflict, permission, etc.) | Stop. Local commits still exist; nothing was pushed. Operator can re-run with a different repo name. |
| Phase 3: `git push` fails after successful repo create | Most likely cause: missed `gh auth setup-git`. Run it, retry. |
| Phase 4 (when implemented): theme JSON merge produces invalid result | Stop before any write. Report the invalid keys. |
| Phase 5 (when implemented): expressions.tmdl rewrite mismatches expected param count | Stop before commit. Report which param is missing in target source.json. |

Every failure leaves the system in a state the operator can recover from manually. No partial mutations are committed.

---

# Why these guardrails

Power BI delivery on synced drives (OneDrive, SharePoint) is a high-stakes environment. A single typo in a destructive command can lose:
- Weeks of model work captured in TMDL files that don't have an obvious "undo"
- Client trust if a private repo accidentally goes public
- Sync links if a folder is renamed and OneDrive treats it as deletion + recreation
- Theme inheritance if a Global Theme update overwrites a project's locked override

This skill ships with the assumption that **every operator using it does NOT want to learn the failure modes the hard way**. Each guardrail in this document was paid for by a real failure we hit during 2026-05-04 calibration:

- MAX_PATH 256 → real save failure with `DateTableTemplate_<GUID>.tmdl`
- `gh auth setup-git` missing → real "Invalid username or token" error after successful login
- Auto Date/Time on by default → real cause of MAX_PATH failure
- BOM in TMDL files → silent Power BI parser failure during early calibration
- Lowercase enums in theme JSON → silent theme application failure
- Theme `name` field > 16 chars → MAX_PATH overflow on internal hash filename
- PBIR-modular → theme not auto-applying

Each was a multi-hour debug. Each is now a one-line pre-flight check or runtime guard so the next operator does not pay it again.

---

# Versioning + roadmap

| Version | Status | Phases |
|---|---|---|
| v0.3.0 | superseded | 1, 2, 3 + Mode 6 available. Phases 4, 5 design-locked. |
| v0.3.1 | superseded | + Phase 4 fully executable (theme refresh with override merge). |
| v0.3.2 | superseded | + Companion-skill alignment contract (audit / doc / theme-audit auto-route to `03_Docs/internal/{audit,docs}/`). Companion skill versions: pbi-model-audit ≥ 0.2.1, pbi-model-doc ≥ 0.2.2, pbi-theme-audit ≥ 0.1.1. |
| **v0.3.3** (this) | RELEASED | + Phase 5 design EXPANDED to include diff-findings report (editorial deck on every promote, same family as pbi-model-doc) AND auto-triggered deployment lifecycle report on Test → Prod (close-project artifact for client handoff). Implementation queued for v0.5. |
| v0.5.0 | QUEUED | Phase 5 sub-mode A (promote + diff-findings). Real-PBIP calibration prerequisite. |
| v0.5.1 | QUEUED | Phase 5 sub-mode C (deployment lifecycle report / close-project). |
| v0.6.0 | TBD | `commit` mode (auto-scoped staging). `lint` mode (validation across the whole client repo). |

Each version is additive. Lower-numbered phases stay backward-compatible.

---

# Author + License

**Author:** Gus Bavia · [linkedin.com/in/gusbavia](https://linkedin.com/in/gusbavia)

**License:** MIT (see `LICENSE`)

This skill is generic and shippable to any Power BI practitioner. Calibrated empirically against real Power BI Desktop builds during 2026-05-04. Every guardrail traces back to a specific failure observed during calibration.
