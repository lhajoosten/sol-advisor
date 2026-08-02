# Global Fullstack Hooks and Tooling Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Install one global, stack-detecting Codex hookset, remove duplicate hooks from `ragvise-ai` and `code-atlas`, migrate `code-atlas` to ty-only Python checks, and update a durable local Sol Advisor fork with the same tooling contract.

**Architecture:** The active Desktop and WSL Codex configs register identical absolute WSL commands under the global hook layer. Four focused Python modules implement environment discovery, session context, command policy, and lightweight Taskfile-first quality selection. Application repositories retain only their tooling manifests and tasks; Sol Advisor becomes the versioned source for orchestration and agent tooling guidance, while machine hooks remain user-owned under `~/.codex/hooks/fullstack`.

**Tech Stack:** Codex inline TOML hooks, Python 3.13 standard library, Taskfile, uv, Ruff, ty, Import Linter, pnpm, Oxfmt, Oxlint, TypeScript, Docker Compose, Kubernetes client validation, Codex plugin manifests and custom-agent TOML.

## Global Constraints

- Back up every existing global or repository file before overwriting or deleting it.
- Preserve all user changes in dirty worktrees and stage or commit only explicitly owned paths.
- Keep `approval_policy = "never"` and `sandbox_mode = "workspace-write"`; do not enable `danger-full-access`.
- Trust only explicit repositories; never trust all of `~/github` or `~/projects`.
- Python development is exactly Python 3.13 with uv, Ruff, and ty only; never configure or invoke mypy, basedpyright, Poetry, Python 3.12, or Python 3.14 for project work. Preserve Ubuntu's packaged Python 3.12 only as its internal OS runtime.
- Use existing Taskfile tasks for checks; do not duplicate a raw check when a suitable task exists.
- PostToolUse records changes only and runs no validation.
- Stop may run lightweight lint, format-check, typecheck, import-boundary, Compose, or Kubernetes dry-run tasks only.
- Stop must never run full pytest, Vitest, Playwright, Storybook, `task check`, or `task ci` automatically.
- Generated Hey API clients are never edited manually; API access stays through existing generated clients and hooks.
- Keep FastAPI routers thin and SQLAlchemy asynchronous; no synchronous database calls.
- Do not commit `ragvise-ai` or `code-atlas` changes automatically because both worktrees contain concurrent user work.
- Commit Sol Advisor source changes in small `type(scope): description` commits and never push them unless separately requested.

---

## File Map

### Global machine files

- Create `/home/lhajoosten/.codex/hooks/fullstack/hooklib.py`: repository discovery, manifest detection, Taskfile catalog, subprocess environment, session state, and changed-path utilities.
- Create `/home/lhajoosten/.codex/hooks/fullstack/session_context.py`: SessionStart adapter and concise detected context.
- Create `/home/lhajoosten/.codex/hooks/fullstack/shell_guard.py`: PreToolUse command policy.
- Create `/home/lhajoosten/.codex/hooks/fullstack/quality_gate.py`: PostToolUse state recording and Stop task selection/execution.
- Create `/home/lhajoosten/.codex/hooks/fullstack/tests/test_fullstack_hooks.py`: unit and event-contract tests.
- Modify `/home/lhajoosten/.codex/config.toml`: WSL global hook registration and local Sol Advisor marketplace.
- Modify `/mnt/c/Users/lhajo/.codex/config.toml`: Desktop global hook registration, explicit `code-atlas` trust, and local Sol Advisor marketplace/plugin enablement.

### Code Atlas

- Modify `/home/lhajoosten/projects/code-atlas/pyproject.toml`: replace mypy with ty.
- Modify `/home/lhajoosten/projects/code-atlas/Taskfile.yml`: expose lightweight backend lint, format, and ty tasks.
- Modify `/home/lhajoosten/projects/code-atlas/.gitignore`: remove mypy cache patterns.
- Modify `/home/lhajoosten/projects/code-atlas/uv.lock`: resolve the ty-only dependency graph through uv.
- Delete `/home/lhajoosten/projects/code-atlas/.codex/hooks/*.py`: remove duplicated global hooks after successful global smoke tests.

### Ragvise AI

- Delete `/home/lhajoosten/github/ragvise-ai/.codex/config.toml` only if it contains hook registration and no unrelated project setting.
- Delete `/home/lhajoosten/github/ragvise-ai/.codex/hooks/*.py` after global smoke tests.
- Modify `/home/lhajoosten/github/ragvise-ai/.codex/README.md` and `.gitignore` only where text or exceptions refer solely to the removed local hookset.

### Sol Advisor fork

- Modify `plugins/sol-advisor/.codex-plugin/plugin.json`: bump the local fork version from `0.2.0` to `0.3.0`.
- Modify all three files under `plugins/sol-advisor/agents/`: add the shared repo-first fullstack tooling contract appropriate to implementer or reviewer role.
- Modify `plugins/sol-advisor/skills/orchestration/SKILL.md`: require repository discovery, Taskfile-first verification, ty-only Python, generated-client protection, and targeted/broad gate separation.
- Modify `plugins/sol-advisor/skills/orchestration/references/role-contracts.md`: add exact tooling and verification evidence requirements to implementation and review packets.
- Modify `README.md`: document the local fork workflow and tooling contract.
- Modify `plugins/sol-advisor/scripts/verify.sh` only where expected template content or version fixtures require it.

---

### Task 1: Freeze state and create recoverable backups

**Files:**
- Read: both global configs and all owned repository/plugin paths listed above.
- Create: the directory stored in `backup_root`, initialized with `backup_root="/home/lhajoosten/.codex/backups/$(date -u +%Y%m%dT%H%M%SZ)-global-fullstack-hooks"`.

**Interfaces:**
- Consumes: current global configs, repo statuses, hook sources, plugin fork.
- Produces: immutable inventory text and byte-for-byte backup copies used by every later task.

- [ ] **Step 1: Capture status and hashes without changing any worktree**

Run:

```bash
git -C /home/lhajoosten/projects/code-atlas status --short
git -C /home/lhajoosten/github/ragvise-ai status --short
git -C /home/lhajoosten/projects/sol-advisor status --short
sha256sum /home/lhajoosten/.codex/config.toml /mnt/c/Users/lhajo/.codex/config.toml
```

Expected: statuses are recorded; no file changes occur. Treat all pre-existing paths as user-owned until an exact before/after diff proves otherwise.

- [ ] **Step 2: Create one explicit backup tree**

Copy with preserved modes:

```bash
install -D -m 0644 /home/lhajoosten/.codex/config.toml "$BACKUP/wsl/config.toml"
install -D -m 0644 /mnt/c/Users/lhajo/.codex/config.toml "$BACKUP/desktop/config.toml"
cp -a /home/lhajoosten/projects/code-atlas/.codex "$BACKUP/code-atlas/.codex"
cp -a /home/lhajoosten/github/ragvise-ai/.codex "$BACKUP/ragvise-ai/.codex"
cp -a /home/lhajoosten/projects/code-atlas/pyproject.toml /home/lhajoosten/projects/code-atlas/Taskfile.yml /home/lhajoosten/projects/code-atlas/.gitignore /home/lhajoosten/projects/code-atlas/uv.lock "$BACKUP/code-atlas/"
cp -a /home/lhajoosten/projects/sol-advisor/plugins "$BACKUP/sol-advisor/"
```

Expected: every intended overwrite/delete target has a recoverable copy. Resolve `$BACKUP` to one explicit validated path before any copy; never use an unresolved variable for deletion.

- [ ] **Step 3: Verify the backup inventory**

Run:

```bash
find "$BACKUP" -type f -print | sort
```

Expected: both global configs, both hooksets, all four Code Atlas tooling files, and Sol Advisor plugin sources appear.

---

### Task 2: Build and test the global hook library

**Files:**
- Create: `/home/lhajoosten/.codex/hooks/fullstack/hooklib.py`
- Create: `/home/lhajoosten/.codex/hooks/fullstack/tests/test_fullstack_hooks.py`

**Interfaces:**
- Produces:
  - `repository_root(cwd: Path) -> Path | None`
  - `build_subprocess_env(base: Mapping[str, str] | None = None) -> dict[str, str]`
  - `detect_stack(root: Path) -> frozenset[str]`
  - `task_catalog(root: Path, env: Mapping[str, str]) -> frozenset[str]`
  - `session_state_path(root: Path, session_id: str) -> Path`
  - `git_changed_paths(root: Path) -> tuple[Path, ...]`
  - `select_lightweight_tasks(root: Path, changed: Sequence[Path], tasks: AbstractSet[str]) -> tuple[str, ...]`
- Consumed by all three event adapters in Task 3.

- [ ] **Step 1: Write failing environment and detection tests**

Add tests that construct temporary Git-like directory trees and assert:

```python
def test_build_subprocess_env_adds_user_tool_paths(monkeypatch):
    env = build_subprocess_env({"PATH": "/usr/bin"})
    assert env["PATH"].split(":")[0] == "/home/lhajoosten/.local/bin"
    assert "/usr/bin" in env["PATH"].split(":")


def test_detect_stack_reports_only_present_manifests(tmp_path):
    (tmp_path / "pyproject.toml").write_text("[project]\nname='x'\nversion='0'\n")
    assert detect_stack(tmp_path) == frozenset({"python"})


def test_select_backend_tasks_never_selects_tests(tmp_path):
    changed = (tmp_path / "apps/api/main.py",)
    tasks = frozenset({
        "lint:backend",
        "format:check:backend",
        "typecheck:backend",
        "test:backend",
        "check",
        "ci",
    })
    assert select_lightweight_tasks(tmp_path, changed, tasks) == (
        "lint:backend",
        "format:check:backend",
        "typecheck:backend",
    )
```

- [ ] **Step 2: Run the tests and confirm the new module is missing**

Run:

```bash
/usr/bin/python3.13 -m unittest discover -s /home/lhajoosten/.codex/hooks/fullstack/tests -v
```

Expected: FAIL because `hooklib` and its public functions do not exist.

- [ ] **Step 3: Implement the minimal library**

Implement the exact interfaces above using only the Python standard library. Requirements:

- `repository_root` calls `git rev-parse --show-toplevel` with a five-second timeout and returns `None` outside Git.
- `build_subprocess_env` deduplicates PATH entries while prepending existing user binary locations and the resolved FNM default alias `~/.local/share/fnm/aliases/default/bin`.
- `task_catalog` calls `task --list --json`, supports `Taskfile.yml` and `Taskfile.yaml`, returns an empty set on missing Task or invalid JSON, and never throws into a lifecycle event.
- `detect_stack` is manifest-driven and contains no assumed tools.
- `select_lightweight_tasks` uses ordered aliases for the two repositories, includes only tasks present in the catalog, and has an explicit forbidden-name filter for `test`, `check`, `ci`, `playwright`, `storybook`, and build tasks.
- Session state lives below `/tmp/codex-fullstack-hooks/{sha256_root}/{safe_session_id}.json` with a strict alphanumeric/dash/underscore session id.

- [ ] **Step 4: Run library tests**

Run the unittest command from Step 2.

Expected: PASS, including an assertion that no selected task name contains a forbidden broad-suite token.

---

### Task 3: Implement and test the three global event adapters

**Files:**
- Create: `/home/lhajoosten/.codex/hooks/fullstack/session_context.py`
- Create: `/home/lhajoosten/.codex/hooks/fullstack/shell_guard.py`
- Create: `/home/lhajoosten/.codex/hooks/fullstack/quality_gate.py`
- Modify: `/home/lhajoosten/.codex/hooks/fullstack/tests/test_fullstack_hooks.py`

**Interfaces:**
- SessionStart reads one JSON object and returns `hookSpecificOutput.additionalContext`.
- PreToolUse reads `tool_input.command` and returns either `{}` or the supported deny shape.
- PostToolUse records touched paths and returns `{}`.
- Stop runs each selected task as `task {task_name}` and returns `{}` on success or `decision: block` with bounded evidence on failure.

- [ ] **Step 1: Add failing event-contract tests**

Use `subprocess.run` against each executable script. Include these assertions:

```python
assert session["hookSpecificOutput"]["hookEventName"] == "SessionStart"
assert "mypy" not in session["hookSpecificOutput"]["additionalContext"].lower()
assert denied["hookSpecificOutput"]["permissionDecision"] == "deny"
assert allowed == {}
assert post_tool == {}
assert stop_hook_active == {}
assert "test:backend" not in recorded_task_invocations
```

Test shell commands for Git-AI, mypy, basedpyright, Poetry, `git reset --hard`, force clean/push, broad `rm -rf`, Docker prune, database drop, and Kubernetes bulk delete. Also test read-only `rg -n 'mypy' .` is allowed.

- [ ] **Step 2: Confirm event tests fail before adapters exist**

Run the unittest suite.

Expected: FAIL for missing event scripts.

- [ ] **Step 3: Implement SessionStart and PreToolUse**

SessionStart must cap dirty-status context at 2,500 characters and list only detected stack/tool/task names. PreToolUse must emit exactly:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "git reset --hard is blocked because it can destroy uncommitted work."
  }
}
```

Use precise token-aware regular expressions so a read-only search mentioning a prohibited tool is not denied.

- [ ] **Step 4: Implement PostToolUse and Stop**

PostToolUse parses `tool_input.command` for patch paths, stores a union with current Git changes, and does not spawn subprocess checks. Stop honors `stop_hook_active`, selects tasks through `hooklib`, runs each task serially with a 300-second timeout, and bounds each failure output to its final 8,000 characters.

When no matching lightweight task exists, Stop returns model-visible context naming the changed lane and missing task instead of blocking.

- [ ] **Step 5: Run event tests and permission checks**

Run:

```bash
/usr/bin/python3.13 -m unittest discover -s /home/lhajoosten/.codex/hooks/fullstack/tests -v
find /home/lhajoosten/.codex/hooks/fullstack -maxdepth 1 -type f -name '*.py' -exec chmod 0755 {} +
find /home/lhajoosten/.codex/hooks/fullstack -maxdepth 1 -type f -name '*.py' -printf '%m %f\n'
```

Expected: all tests pass and every event script reports mode `755`.

---

### Task 4: Register global hooks in Desktop and WSL configs

**Files:**
- Modify: `/home/lhajoosten/.codex/config.toml`
- Modify: `/mnt/c/Users/lhajo/.codex/config.toml`

**Interfaces:**
- Consumes the absolute scripts from Task 3.
- Produces one SessionStart, one Bash PreToolUse, one edit PostToolUse, and one Stop registration per active global config.

- [ ] **Step 1: Add identical inline registrations to staged config copies**

Use these definitions in both configs:

```toml
[[hooks.SessionStart]]
matcher = "^(startup|resume|clear|compact)$"

[[hooks.SessionStart.hooks]]
type = "command"
command = '/usr/bin/python3.13 /home/lhajoosten/.codex/hooks/fullstack/session_context.py'
timeout = 10
additionalContextLimit = 3000

[[hooks.PreToolUse]]
matcher = "^Bash$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = '/usr/bin/python3.13 /home/lhajoosten/.codex/hooks/fullstack/shell_guard.py'
timeout = 5

[[hooks.PostToolUse]]
matcher = "^(apply_patch|Edit|Write)$"

[[hooks.PostToolUse.hooks]]
type = "command"
command = '/usr/bin/python3.13 /home/lhajoosten/.codex/hooks/fullstack/quality_gate.py post-tool'
timeout = 5

[[hooks.Stop]]

[[hooks.Stop.hooks]]
type = "command"
command = '/usr/bin/python3.13 /home/lhajoosten/.codex/hooks/fullstack/quality_gate.py stop'
timeout = 300
```

- [ ] **Step 2: Parse staged copies before installation**

Run `tomllib.loads` for both copies.

Expected: both parse and expose exactly one handler for each event.

- [ ] **Step 3: Install the copies and validate Codex configuration**

Install with mode `0644`, then run:

```bash
codex doctor --summary
```

Expected: config loads, hooks are enabled, and sandbox reports approval `Never`. A non-interactive `TERM=dumb` note is acceptable; a config parse error is not.

---

### Task 5: Resolve the four Code Atlas findings with ty-only Taskfile tasks

**Files:**
- Modify: `/home/lhajoosten/projects/code-atlas/pyproject.toml`
- Modify: `/home/lhajoosten/projects/code-atlas/Taskfile.yml`
- Modify: `/home/lhajoosten/projects/code-atlas/.gitignore`
- Modify: `/home/lhajoosten/projects/code-atlas/uv.lock`

**Interfaces:**
- Produces Taskfile tasks `lint:backend`, `format:check:backend`, `typecheck:backend`, and aggregate `check:backend`.
- Global Stop selection in Task 2 consumes those exact names.

- [ ] **Step 1: Assert the pre-migration state contains mypy**

Run:

```bash
rg -n 'mypy|basedpyright|poetry' pyproject.toml Taskfile.yml .gitignore uv.lock
```

Expected: mypy appears in the dependency, tool section, Taskfile, ignore file, and lock; basedpyright and Poetry do not appear.

- [ ] **Step 2: Patch `pyproject.toml`**

Set the development group to include:

```toml
"ruff>=0.11.0",
"ty>=0.0.65",
```

Remove `[tool.mypy]` completely and add:

```toml
[tool.ty.analysis]
respect-type-ignore-comments = true
```

- [ ] **Step 3: Split lightweight Taskfile tasks**

Define:

```yaml
  lint:backend:
    desc: Check backend linting with Ruff
    cmds:
      - uv run ruff check packages apps

  format:check:backend:
    desc: Check backend formatting with Ruff
    cmds:
      - uv run ruff format --check packages apps

  typecheck:backend:
    desc: Type-check backend code with ty
    cmds:
      - uv run ty check packages/backend/src apps/api/src apps/worker/src

  check:backend:
    desc: Run backend linting, formatting, and type checks
    deps:
      - lint:backend
      - format:check:backend
      - typecheck:backend
```

Keep `test:backend` explicit and independent; do not make it a dependency of any lightweight task.

- [ ] **Step 4: Remove mypy ignore entries and update the lock**

Use uv from its absolute installed path with a writable cache directory if required:

```bash
UV_CACHE_DIR=/tmp/code-atlas-uv-cache /home/lhajoosten/.local/bin/uv lock
```

Expected: mypy and mypy-only transitive packages leave the lock; ty enters it.

- [ ] **Step 5: Run the smallest repository checks**

Run through Taskfile:

```bash
task lint:backend
task format:check:backend
task typecheck:backend
```

Expected: each task invokes the intended tool. Existing ty diagnostics may fail the typecheck and must be reported exactly; do not suppress or replace them with casts. Do not run `task test:backend` automatically.

- [ ] **Step 6: Verify the prohibited-tool inventory**

Run:

```bash
rg -n -i 'mypy|basedpyright|poetry' pyproject.toml Taskfile.yml .gitignore uv.lock || true
git diff --check
```

Expected: no active occurrence and no whitespace errors. Do not commit because the worktree contains staged concurrent user work.

---

### Task 6: Update the Sol Advisor fork to version 0.3.0

**Files:**
- Modify: plugin manifest, three agent TOMLs, orchestration skill, role contracts, README, and verifier as mapped above.

**Interfaces:**
- Produces byte-exact 0.3.0 agent templates consumed by `install-agents.sh`.
- Produces orchestration prompts that require Taskfile evidence and the shared tooling contract.

- [ ] **Step 1: Read plugin and skill authoring instructions before editing**

Invoke the available `plugin-creator`, `superpowers:writing-skills`, and `skill-creator` instructions. Record their required validators and manifest constraints in the execution log.

- [ ] **Step 2: Add failing verifier assertions**

Extend `verify.sh` fixtures/assertions so verification fails unless:

- plugin version is `0.3.0`;
- Luna and Terra templates contain `Taskfile`, `uv`, `Ruff`, and `ty`;
- no template contains mypy, basedpyright, or Poetry as an allowed tool;
- reviewer instructions require checking actual Taskfile evidence and prohibited-tool absence;
- orchestration and role contracts contain the targeted-versus-broad gate distinction.

Run:

```bash
sh plugins/sol-advisor/scripts/verify.sh
```

Expected: FAIL on the old 0.2.0 content.

- [ ] **Step 3: Update manifest and agent profiles**

Bump the manifest to `0.3.0`. Add the same compact tooling contract to Luna and Terra, including repository instructions first, Taskfile-first, ty-only Python, generated-client protection, architecture boundaries, targeted checks, and preservation of concurrent edits. Add the corresponding read-only review checks to Sol without granting implementation behavior.

- [ ] **Step 4: Update orchestration and role contracts**

Require every five-part implementation contract to list:

- discovered repository instruction files;
- selected existing Taskfile commands;
- why each command is targeted or broad;
- confirmation that no prohibited Python tool or hand-edited generated client is used;
- exact verification output rather than a worker completion claim.

The final Sol packet must reject missing Taskfile evidence when a Taskfile exists.

- [ ] **Step 5: Update README and local-fork remote metadata**

Document `~/projects/sol-advisor` as the local source. Configure remotes as:

```bash
git remote remove origin
git remote add upstream https://github.com/DannyMac180/sol-advisor.git
```

Expected: there is no push remote; `upstream` is fetch-only by convention and no GitHub publication occurs.

- [ ] **Step 6: Validate and commit the plugin update**

Run the repository verifier, plugin validator, skill validator, shell syntax, JSON/TOML parsing, and `git diff --check`. Expected: all pass.

Commit only plugin-owned files:

```bash
git add README.md plugins/sol-advisor
git commit -m "feat(ci): align Sol Advisor with fullstack tooling"
```

---

### Task 7: Switch both Codex homes to the durable local Sol Advisor marketplace

**Files:**
- Modify both global configs through the Codex plugin CLI.
- Update installed plugin caches and custom-agent templates for both Codex homes.

**Interfaces:**
- Consumes local fork version 0.3.0.
- Produces enabled `sol-advisor@sol-advisor` for Desktop and WSL with exact companion templates.

- [ ] **Step 1: Remove the old WSL marketplace and add the local path**

Run with explicit home:

```bash
CODEX_HOME=/home/lhajoosten/.codex codex plugin marketplace remove sol-advisor
CODEX_HOME=/home/lhajoosten/.codex codex plugin marketplace add /home/lhajoosten/projects/sol-advisor
CODEX_HOME=/home/lhajoosten/.codex codex plugin add sol-advisor@sol-advisor
```

Expected: source type becomes local and resolves to the durable fork.

- [ ] **Step 2: Add and install the local marketplace for Desktop**

Run with explicit Desktop home:

```bash
CODEX_HOME=/mnt/c/Users/lhajo/.codex codex plugin marketplace add /home/lhajoosten/projects/sol-advisor
CODEX_HOME=/mnt/c/Users/lhajo/.codex codex plugin add sol-advisor@sol-advisor
```

Expected: Desktop lists the local marketplace and enabled plugin.

- [ ] **Step 3: Install exact companion templates into both agent directories**

Run `install-agents.sh --target-dir` for `/home/lhajoosten/.codex/agents` and `/mnt/c/Users/lhajo/.codex/agents`. If an existing differing role file is found, back it up and inspect the diff before deliberately replacing it; never force through the installer.

- [ ] **Step 4: Verify plugin and templates**

Run `codex plugin list`, marketplace list, and `install-agents.sh --check` for both target directories.

Expected: plugin version 0.3.0 and byte-exact templates in both homes. A new Codex task is required before new custom-agent definitions are visible.

---

### Task 8: Remove duplicated repository hooksets

**Files:**
- Delete Code Atlas and Ragvise AI hook files listed in the file map.
- Conditionally remove hook-only configs/docs/ignore exceptions.

**Interfaces:**
- Consumes passing global event smoke tests and loaded global registration.
- Produces zero repository-local duplicate handler definitions.

- [ ] **Step 1: Smoke-test global scripts from both repositories first**

Feed real SessionStart payloads with each repository cwd, safe and denied Bash payloads, PostToolUse payloads, and `stop_hook_active: true` Stop payloads.

Expected: valid JSON, correct stack/task detection, no heavy task invocation, and correct deny decisions.

- [ ] **Step 2: Inspect every repository `.codex` file immediately before deletion**

If `ragvise-ai/.codex/config.toml` or any README contains unrelated project settings, preserve those parts and remove only hook definitions. Otherwise remove the hook-only file completely.

- [ ] **Step 3: Delete exact backed-up hook paths**

Delete only the three named scripts in each repository. Do not recursively remove `.codex` directories. Use explicit paths and verify the backup exists first.

- [ ] **Step 4: Confirm no duplicate source remains**

Run:

```bash
find /home/lhajoosten/projects/code-atlas/.codex /home/lhajoosten/github/ragvise-ai/.codex -maxdepth 3 -type f -print
rg -n '^\[\[hooks\.|^\[hooks\]' /home/lhajoosten/projects/code-atlas/.codex /home/lhajoosten/github/ragvise-ai/.codex 2>/dev/null || true
```

Expected: no repository-local lifecycle registration or duplicated global script remains.

---

### Task 9: Final cross-surface verification

**Files:**
- Read all modified global, repository, and plugin files.
- Do not create new application artifacts except tool caches under `/tmp`.

**Interfaces:**
- Produces the final evidence report and manual `/hooks`/restart instructions.

- [ ] **Step 1: Validate syntax, modes, and configuration**

Parse both TOML configs and all modified pyproject/TOML files; parse plugin JSON; parse Taskfile YAML; compile hook Python without leaving bytecode in repositories; run `sh -n` on plugin scripts; verify executable modes.

- [ ] **Step 2: Run hook unit and integration smoke tests**

Run the full unittest suite and event payloads in Code Atlas, Ragvise AI, and a temporary non-fullstack Git repository.

Expected: fullstack checks activate only in the first two; the temporary repo receives only safety behavior.

- [ ] **Step 3: Run targeted repository validation**

Code Atlas: lint, format check, and ty through Taskfile. Ragvise AI: existing backend/frontend lint, format-check, typecheck, and Import Linter tasks. Do not run tests, `task check`, or `task ci` as part of the hook migration.

- [ ] **Step 4: Run plugin verification and Codex diagnostics**

Run all Sol Advisor validators and both-home plugin/template checks, followed by `codex doctor --summary` and `codex mcp list`.

Expected: configs and plugin load; CodeGraph remains enabled; approval reports Never. Report transient connectivity or `TERM=dumb` separately from configuration failures.

- [ ] **Step 5: Audit prohibited tools, duplicates, and dirty worktrees**

Search active configs/manifests/tasks/locks for Git-AI installation remnants, mypy, basedpyright, and Poetry. Exclude intentional blocker/guard text, backups, changelog history, and the Sol Advisor prohibition text from the active-tool finding count.

Run `git diff --check`, `git status --short`, and scoped diffs in all three repositories. Confirm no user file was reverted or accidentally staged.

- [ ] **Step 6: Hand off manual trust/restart step**

Report the new global hook source and hash review requirement. Tell the user to open `/hooks`, trust the reviewed global definitions once per active Codex home, and start a fresh task so updated plugin skills and custom-agent profiles are discovered.
