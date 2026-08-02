# Global fullstack hooks and Sol Advisor tooling

## Objective

Provide one machine-wide Codex lifecycle policy for the user's consistent fullstack
projects, remove duplicate repository hook copies from `ragvise-ai` and `code-atlas`,
standardize Python work on `uv`, Ruff, and `ty`, and make the local Sol Advisor fork
enforce the same repository-first tooling contract.

## Scope

The change covers four surfaces:

1. Global Codex hook registration in both active user configurations.
2. Canonical hook scripts under `/home/lhajoosten/.codex/hooks/fullstack`.
3. Repository cleanup and the existing four `code-atlas` review findings.
4. A durable local Sol Advisor fork under `/home/lhajoosten/projects/sol-advisor`.

The repositories remain explicitly trusted individually. The whole `~/github` or
`~/projects` parent directory must not become trusted.

## Global hook architecture

Both `/home/lhajoosten/.codex/config.toml` and
`/mnt/c/Users/lhajo/.codex/config.toml` register the same absolute WSL hook commands.
Only one config is active for a given Codex surface, so this does not duplicate hook
execution.

The canonical scripts are:

- `session_context.py`: detects repository root, branch, dirty state, manifests,
  Taskfile tasks, and installed stack components. It reports only capabilities that
  actually exist.
- `shell_guard.py`: globally blocks Git-AI, mypy, basedpyright, Poetry, and broad
  destructive Git, filesystem, Docker, database, and Kubernetes commands. Read-only
  searches remain allowed.
- `quality_gate.py`: records touched paths after edits and selects lightweight,
  change-aware Taskfile checks when a turn stops.

Global scripts activate fullstack checks only when a repository contains relevant
signals such as a Taskfile, `pyproject.toml`, `uv.lock`, `package.json`,
`pnpm-workspace.yaml`, Compose files, or Kubernetes/Helm directories. Non-fullstack
repositories receive the safety guard but no invented toolchain checks.

## Lifecycle behavior

### SessionStart

SessionStart establishes a per-session baseline and emits concise context containing
only detected tools and existing Taskfile tasks. It must not claim that Import Linter,
Playwright, Storybook, Hey API, Docker, or Kubernetes exists unless the repository
shows evidence for it.

### PreToolUse

PreToolUse matches shell commands. It denies the prohibited tools and destructive
patterns before execution. It may add a Taskfile-first reminder for direct validation
commands but must not block harmless read-only inspection.

### PostToolUse

PostToolUse matches `apply_patch`, `Edit`, and `Write`. It records affected paths only.
It must not run linting, type checking, tests, builds, or generated-client commands.

### Stop

Stop compares the working tree with the SessionStart baseline and chooses only existing
lightweight Taskfile tasks relevant to changed paths. Supported lanes are backend lint,
backend format check, `ty`, Import Linter, frontend Oxfmt, frontend Oxlint, TypeScript,
Compose validation, and optional Kubernetes client dry-run.

Stop must never automatically run full pytest, Vitest, Playwright, Storybook, `task
check`, or `task ci`. When a suitable lightweight Taskfile task is absent, the hook
returns model-visible context rather than inventing a heavy raw fallback.

The scripts construct a controlled subprocess PATH containing known user tool
locations, including `/home/lhajoosten/.local/bin`, uv locations, and the active FNM
Node installation. Checks still run through Taskfile whenever a task exists.

## Repository migration

After the global hooks pass smoke tests, remove the duplicated hook scripts and
hook-only registration from both:

- `/home/lhajoosten/github/ragvise-ai/.codex`
- `/home/lhajoosten/projects/code-atlas/.codex`

Preserve unrelated project-specific Codex configuration. Clean hook-only README text
and `.gitignore` exceptions only when they are no longer needed.

In `code-atlas`, resolve the four accepted findings:

1. Replace the mypy development dependency with `ty`.
2. Remove `[tool.mypy]` and add the minimal appropriate `[tool.ty]` configuration.
3. Change the backend Taskfile typecheck from mypy to `uv run ty check` and expose
   lightweight tasks that the global gate can select.
4. Remove mypy cache patterns and keep the complete pytest suite explicit rather than
   automatic.

Update or create the uv lock only through uv. Do not introduce Poetry or
basedpyright. Preserve concurrent user changes and do not rewrite unrelated files.

## Sol Advisor local fork

Maintain `/home/lhajoosten/projects/sol-advisor` as the durable local source. Configure
the personal Sol Advisor marketplace to use that checkout instead of editing an
ephemeral installed cache. Keep the upstream GitHub repository available as an
`upstream` remote; local changes are committed only to the local fork unless the user
later requests publication.

Update the plugin version and these sources consistently:

- Luna and Terra custom-agent developer instructions.
- The orchestration skill.
- Shared role contracts and verification packets.
- README installation/update guidance.
- Plugin verification fixtures when exact template hashes or expected content change.

The tooling contract requires agents to:

- read repository instructions, manifests, and Taskfile tasks before editing;
- use Taskfile commands instead of duplicating existing raw checks;
- use Python 3.13 exactly for development, with uv, Ruff, and ty only;
- reject mypy, basedpyright, and Poetry;
- respect FastAPI router-service-repository boundaries and async SQLAlchemy;
- support PostgreSQL and pgvector without synchronous database calls;
- use pnpm workspaces, strict TypeScript, React, Oxfmt, and Oxlint;
- use Vitest/happy-dom, Playwright, and Storybook only when present and appropriate;
- use existing Hey API/Axios generated clients and never hand-edit generated output;
- support Docker and optional Kubernetes through existing repository tasks;
- run targeted checks during implementation and reserve broad suites for explicit
  completion or CI gates.

The Sol reviewer must verify the actual diff and the primary session's concrete
Taskfile evidence. It must flag use of prohibited Python tooling or bypassing generated
client and architecture conventions.

Ubuntu's packaged Python 3.12 remains installed only as the distribution runtime for
system utilities. User-facing `python` and `python3`, project pins, hook commands, and
developer tooling resolve to Python 3.13. Python 3.14 is not part of the WSL toolchain.

## Safety and compatibility

Create timestamped backups before modifying global configuration or existing files.
Install global hooks before deleting repository copies. Preserve executable modes.
Do not broaden sandbox mode or trust parent directories. `approval_policy = "never"`
remains separate from sandbox permissions.

Non-managed hook definitions require one-time review through `/hooks`; changed hashes
require review again. Do not bypass hook trust persistently.

## Verification

The implementation is acceptable when all of the following hold:

- Both global TOML configurations parse and Codex reports approval policy `Never`.
- All hook scripts parse, are executable, and emit valid event-specific JSON.
- SessionStart behaves correctly in `ragvise-ai`, `code-atlas`, and a non-fullstack
  directory.
- The shell guard allows safe inspection and denies all named prohibited cases.
- PostToolUse performs no validation command.
- Stop selects only relevant lightweight tasks and never selects a full test suite.
- `uv` and pnpm resolution works from the hook environment.
- No duplicate repository hook definition remains.
- Active `code-atlas` configuration contains ty and no mypy, basedpyright, or Poetry.
- Existing targeted Taskfile lint, format, imports, and typecheck tasks are run where
  available; failures are reported without blanket suppression.
- The Sol Advisor plugin validator, skill validator, verifier, shell syntax checks,
  JSON/TOML validation, and agent-template exactness checks pass.
- The local marketplace points at the durable fork and a fresh Codex task exposes the
  updated plugin and custom-agent templates after installation.

## Out of scope

- Fixing every pre-existing ty diagnostic in either application repository.
- Publishing or pushing the Sol Advisor fork to a remote GitHub account.
- Enabling `danger-full-access`.
- Automatically running broad test or CI suites after ordinary tool calls.
- Refactoring application code unrelated to the tooling migration.
