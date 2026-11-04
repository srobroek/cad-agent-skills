# Live smoke

This page documents the manual live smoke for every harness and both plugin variants. It proves installation, discovery,
resource loading and the first two stages in a fresh project. It proves no CAD, DFM, analysis or physical result; static
CI proves none of these either. Install commands are in [install.md](install.md); stage behaviour is the `cad-flow`
skill's.

## Matrix

Run each cell once per release candidate, in its own fresh project. One cell installs exactly one variant.

| Harness | `full` (`cad-agent-skills`) | `lean` (`cad-agent-skills-lean`) |
|---|---|---|
| Claude Code | required | required |
| Codex CLI | required | required |
| OpenCode | required | required |
| Kiro CLI V3 | required | required |
| OMP | required | required |

A cell's result is `PASS`, `FAIL` or `BLOCKED` (a prerequisite such as a login, a CLI or a model role was not
available). `BLOCKED` is reported with its cause, never counted as `PASS`.

## Prerequisites

- `CAD_AGENT_SKILLS_SRC`: an absolute path to a clean checkout of the release tag under test. Record
  `git -C "$CAD_AGENT_SKILLS_SRC" describe --tags --exact-match` and `git -C "$CAD_AGENT_SKILLS_SRC" rev-parse HEAD`;
  `git -C "$CAD_AGENT_SKILLS_SRC" status --porcelain` prints nothing.
- The harness CLI, logged in; record its `--version` output.
- `uv` on `PATH`; `uvx --from check-jsonschema==0.39.0 check-jsonschema --version` succeeds.
- The model roles the harness uses (model id and reasoning effort per role), written down before the run. OMP keeps
  them in the files passed with `--config`.
- The demo brief: [examples/phone-stand/brief.md](../examples/phone-stand/brief.md). The smoke uses its first prompt,
  scripted answers and gate replies.
- An isolated host environment: the cell runs without the operator's user-level instructions, rules, hooks, plugins,
  extra skills and MCP servers, with the host settings below ("Environment isolation"); whatever stays active is
  recorded as a deviation.

## Procedure

Run the steps in order for one cell. Start the harness and any `tmux` session with a scrubbed environment, for example:

```sh
env -i HOME="$HOME" PATH="$PATH" TERM="$TERM" <needed credential and region variables> <command>
```

Include only variables required for that host. Record the exact environment variable names (never secret values).
Capture every output to `$SMOKE_LOG/<harness>-<variant>/`, outside the project.

For OMP, create `OMP_HOME=$(mktemp -d)` and these directories:

```sh
mkdir -p "$OMP_HOME/home" "$OMP_HOME/.omp" "$OMP_HOME/.config" "$OMP_HOME/home/.local/share" \
  "$OMP_HOME/home/.cache" "$OMP_HOME/home/.local/state"
```

Set `HOME="$OMP_HOME/home"`, `PI_CONFIG_DIR=.omp`, `OMP_PROFILE=cad-smoke`, `XDG_CONFIG_HOME="$OMP_HOME/.config"`,
and XDG data/cache/state roots under `$OMP_HOME/home`. Export these variables to each OMP process, retaining only the
provider credentials the host requires. Project `.omp` discovery remains active, so use only the fresh project/plugin.

1. Fresh project: `SMOKE_PROJECT=$(mktemp -d)`, then `git -C "$SMOKE_PROJECT" init`. Confirm `hw/` and
   `.cad-work/` are absent. Never reuse a project between cells.
2. Install the variant at project scope with the commands in [install.md](install.md) for this harness. For Claude
   Code and OMP, the marketplace route is the release route; a run with `--plugin-dir` is recorded as a local run.
3. Start the harness in `SMOKE_PROJECT` with the start command below. One session, one lead: every reply resumes the
   same session id, and a second session for the project (a fork, or a reply sent to an older id) fails the cell.
   Never set an operator-side deadline shorter than the harness turn limit. OMP can exceed an hour per turn; `--max-time 2h`
   provides a recoverable ceiling for this smoke. If credentials fail mid-turn, resume once with a neutral, recorded line;
   never resend scripted answers. For OMP, `--restore <session-file>` continues the interrupted turn and its subagents under
   the same session id. For OpenCode, `run --session` cannot resume without a message: send one neutral, recorded line (for
   example, `Continue from the last recorded state.`) and record this deviation, or restart in a fresh project and mark the
   original cell `BLOCKED`.
4. Discovery: list skills and agents with the listing below → exactly the eight skills `cad-flow`,
   `cad-record`, `cad-select`, `cad-design`, `cad-model`, `cad-verify`, `cad-review`, `cad-release` and the five roles
   `cad-concept-designer`, `cad-part-designer`, `cad-checker`, `cad-mech-dfm-reviewer`, `cad-sourcing-researcher`,
   each once. A duplicate or a missing name is `FAIL`.
5. Send the phone-stand first prompt. At preflight the lead resolves every skill, schema and role reference to an
   absolute path and reports the plugin name and version from the `cad-flow` frontmatter; it must equal the variant
   installed and the tag under test.
6. GROUND: answer the lead's questions with the brief's scripted answers, batched as the brief says; a question the
   sheet does not cover gets the brief's open-question line, never a value the lead suggests. The lead bootstraps
   `hw/` from the template, writes the GROUND verdict and presents the gate packet with the exact approval form; send
   `approve GROUND <vN>`.
7. INTENT: the lead dispatches at least one worker unit (sourcing for the printer and spools) through the host's
   native subagent call, logs its native task id, accepts or rejects the delivery, writes the INTENT verdict and
   presents its packet. Send the scripted INTENT reply. Stop the session once the INTENT decision is recorded.
8. Validate: for every JSON and YAML file under `hw/` that cad-record `layout.json` maps to a schema, run
   `uvx --from check-jsonschema==0.39.0 check-jsonschema --schemafile <schema> <file>` with the schema from
   `$CAD_AGENT_SKILLS_SRC/<variant>/skills/cad-record/schemas/` → every command exits 0.
9. Compare the GROUND and INTENT obligations in `.cad-work/stage-<vN>/obligations.yaml` and in each verdict with
   [examples/phone-stand/expected-obligations.yaml](../examples/phone-stand/expected-obligations.yaml): no missing and
   no extra family, every `must_show` present, no `must_not` seen.
10. Finish: confirm the lead released its own claim (`.cad-work/lead/active/claim.yaml` absent, the session copy
    present). Keep the project; delete nothing.

## Start and listing commands

Confirm each command against the installed CLI's `--help` and record any difference as a smoke finding. The
discovery forms were verified on 2026-11-04 with Claude Code 2.1.316, Codex CLI 0.166.0, OpenCode 2.2.5, Kiro CLI
3.0.0 and OMP 18.13.2.

| Harness | Interactive start | Headless start and resume | Prompt prefix |
|---|---|---|---|
| Claude Code | `claude` in `SMOKE_PROJECT` | `claude -p --max-turns 200 "<prompt>"`; `claude --resume <session-id> -p "<reply>"` | `/cad-agent-skills:cad-flow` or `/cad-agent-skills-lean:cad-flow` |
| Codex CLI | `codex -C "$SMOKE_PROJECT"` | `codex exec -C "$SMOKE_PROJECT" "<prompt>"`; from `SMOKE_PROJECT` (`exec resume` has no `-C`): `codex exec resume <thread-id> "<reply>"` | `$cad-flow` |
| OpenCode | start `opencode serve --port <p>` in the isolated environment, with fixed `--model <provider/model#variant>` | `opencode run --standalone --format json --model <provider/model#variant> "<prompt>"`; replies use `opencode run --standalone --model <provider/model#variant> --session <session-id> "<reply>"` | `/cad-flow` |
| Kiro CLI V3 | `kiro-cli chat --v3` in a `tmux` session, each tool call approved by hand | not used: 3.0.0 headless needs `--trust-all-tools` | name the skill in prose: `Use the cad-flow skill.` |
| OMP | `env -i OMP_HOME="$OMP_HOME" HOME="$OMP_HOME/home" PI_CONFIG_DIR=.omp OMP_PROFILE=cad-smoke XDG_CONFIG_HOME="$OMP_HOME/.config" XDG_DATA_HOME="$OMP_HOME/home/.local/share" XDG_CACHE_HOME="$OMP_HOME/home/.cache" XDG_STATE_HOME="$OMP_HOME/home/.local/state" PATH="$PATH" TERM="$TERM" <needed provider credentials> omp --profile cad-smoke --config "$OMP_HOME/home/.omp/profiles/cad-smoke/agent/config.yml" --config "$OMP_HOME/home/smoke-config.yml" --no-rules --no-context-files --no-extensions --skills 'cad-*' --max-time 2h` | Same environment/profile/configs with `omp -p --max-time 2h "<prompt>"`; recover with `omp --restore <session-file> --max-time 2h` | `/cad-flow` |

| Harness | Skills | Roles |
|---|---|---|
| Claude Code | `claude -p --output-format stream-json --verbose --max-turns 1 "Reply OK."`: `skills` array of first `system` `init` event, names prefixed `<plugin>:` | `agents` array of same event |
| Codex CLI | `codex debug prompt-input` in `SMOKE_PROJECT`: each `- cad-<name>:` skill entry | each native `spawn_agent` call in the transcript, or fallback |
| OpenCode | fallback (v2.2.5 has no skill listing command; `opencode agent list` does not exist) | run `opencode debug agents --standalone`, save JSON, retry until it parses, then use `jq -r '.[].name' <file>` |
| Kiro CLI V3 | interactive `/context show`; headless: fallback | `kiro-cli agent list` in `SMOKE_PROJECT`: rows marked `Workspace` (it also prints global profiles) |
| OMP | `omp read skill://<name>/SKILL.md` per skill: header shows resolved path | fallback over `<installPath>/agents/*.md`, with `installPath` from `omp plugin list --json` |

Fallback, where the host has no listing: enumerate the installed files the install wrote (skills
`<skills dir>/<name>/SKILL.md`; roles the role files), read each frontmatter `name` (Codex TOML `name`, Kiro JSON
`name`; OpenCode roles use file names) and record the command and its output → each expected name once.

### Environment isolation

| Harness | Settings for the cell |
|---|---|
| Claude Code | `--setting-sources project,local --strict-mcp-config` |
| Codex CLI | Create a fresh `CODEX_HOME` with a minimal `config.toml` that sets the model, provider credentials, region and project trust for `SMOKE_PROJECT`; use `--ignore-rules` on `codex exec` and `codex exec resume`. The toolbox wrapper rewrites the config: pre-create an empty `.builder-mcp-setup-complete` to prevent the builder-mcp server; it still adds `approvals_reviewer` and telemetry, which must be recorded as deviations. |
| OpenCode | Set `OPENCODE_CONFIG_DIR` to an empty directory and `OPENCODE_DISABLE_CLAUDE_CODE=1`. Use `--standalone` on each `opencode run` and agent listing; that starts a private server and avoids the background service. Pass provider credentials and region in environment variables and fix the model with `--model <provider/model#variant>`. |
| Kiro CLI V3 | none found: record the user-level agents, steering and MCP servers that load |
| OMP | Temporary HOME, OMP root and XDG roots; named profile `cad-smoke`; profile config disables memory, advisor, autolearn and TTSR; no unrelated project MCP config; flags disable rules, context and extensions; `--skills 'cad-*' --max-time 2h`. |

With `PI_CONFIG_DIR=.omp` and `OMP_PROFILE=cad-smoke`, the profile config is
`$OMP_HOME/home/.omp/profiles/cad-smoke/agent/config.yml`. Create it with:

```yaml
memory:
  backend: off
advisor:
  enabled: false
autolearn:
  enabled: false
ttsr:
  enabled: false
```

Create `$OMP_HOME/home/smoke-config.yml`:

```yaml
mcp:
  enableProjectConfig: false
```

Pass both files with `--config`. The temporary HOME and XDG roots isolate user-level OMP settings, plugins, sessions and
caches. The project-scoped plugin remains discoverable in the fresh project. Flags disable OMP rules, context files and
ambient extensions. Record any advisor, memory, TTSR, MCP server or bash interceptor that remains active. Do not install
or mutate plugins during the cell. These controls are discovery isolation, not an OS boundary.

- Codex CLI: custom agents inherit the parent sandbox and approval mode. Use
  the fresh `CODEX_HOME` and the `--ignore-rules` route above; a blocked step is
  a finding, never a reason to grant broader access.
- Codex CLI: sandbox escalation handling (including the `auto_review` guardian) is a recorded deviation. Record each
  escalation and guardian decision; never treat approval as a workaround for the documented sandbox boundary.
- Kiro CLI V3: grant only exact V3 tool ids, never `--trust-all-tools`. In 3.0.0, `--no-interactive` denied
  `execute_bash` even when the exact id was trusted, so the headless route needs `--trust-all-tools` and is not used;
  run the cell interactively in `tmux` and approve each call. The ids: `read_file`, `list_directory`, `file_search`,
  `grep_search`, `code`, `introspect` (read); `fs_write`, `fs_append`, `str_replace`, `delete_file` (write);
  `execute_bash` (shell); `web_fetch`, `remote_web_search` (web); `disclose_context` (skill activation),
  `invoke_sub_agent` and `subagent/<agent>`. Parent permissions must allow what a role profile allows. A machine
  agent that rewrites the installed `.kiro/agents/cad-*.json` (observed: an AIM `creds-agent` injector adding its
  tool and MCP server) is a recorded deviation: the installed role files no longer equal the release. Kiro's
  workspace-trust prompt writes `~/.kiro/workspace-roots/`; record this user-level side effect and remove the entry
  after the cell.
- OpenCode: use `--standalone` on all run and agent-list commands to isolate from an existing background server. Check
  with `opencode debug config --standalone`; `~/.config/opencode` must not appear. If `run --session` fails mid-turn,
  save the failure, send one neutral recorded line and note the deviation, or restart the cell with a fresh project.

## Evidence to capture

| Item | Source | PASS when |
|---|---|---|
| tag, commit, variant, plugin name and version | prerequisites; preflight report | the reported plugin and version equal the variant and tag installed |
| CLI version, model roles | `--version`; the written role list | recorded before the first prompt |
| discovered skills and roles | step 4 listing, verbatim | eight skills and five roles, each once |
| resolved resources | `.cad-work/lead/preflight.json` | every path absolute and existing; no installed file under `hw/` or `.cad-work/` |
| lead claim and session | `.cad-work/lead/active/claim.yaml` during the run, the session copy after | one ACTIVE claim whose `session` is the host's native id (or `UNAVAILABLE-<stamp>` with the limit reported), released at finish |
| GROUND and INTENT verdicts | `hw/versions/<vN>/stage-verdict.json` and their sha256 | each validates; PASS before its gate is presented |
| gate decisions | `hw/decisions/DR-###.yaml` and the saved statements | `kind: GATE`, actor name starting `SYNTHETIC`, the quoted reply equals the scripted text |
| native dispatch | the harness transcript and `.cad-work/lead/units.yaml` | each unit's role, native task id, brief sha256 and state; the task text is the role, brief path and role reference only |
| deliveries | `.cad-work/<unit>/delivery.json` | validates against `worker-delivery`; boundary digests equal; accepted bytes merged verbatim |
| writes | the transcript's tool calls | no write to an installed file and no record/evidence write outside `hw/` and `.cad-work/`; uv/package caches, bytecode, session stores and other named host caches may be elsewhere and are recorded |
| schema results | step 8 output | every command exits 0 |
| obligations | step 9 comparison | no missing or extra family; no `must_not` |
| transcript | the harness session export or JSONL | kept unedited |

A cell is `PASS` only when every row is PASS. Report each cell with its result, the failing rows and the absolute path
of its evidence directory.

## What the smoke does not show

Static checks and this smoke cover install, discovery and the first two stages. Geometry builds, drawings, BOM,
analysis, review, freeze and release run in the demonstrations in [demo.md](demo.md). Synthetic approvals stand for no
product owner. A permission profile prevents accidental tool calls only; it is no operating-system boundary.
