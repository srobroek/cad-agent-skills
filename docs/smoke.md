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

For OMP, run every OMP command of the cell, including install, listing and cleanup, through one wrapper with a temporary
HOME and the named profile `cad-smoke`:

```sh
OMP_HOME=$(mktemp -d) && mkdir "$OMP_HOME/home"
OMP_PATH=<directories of the real omp, uv and git binaries>:/usr/bin:/bin
omp_smoke() {
  env -i HOME="$OMP_HOME/home" PATH="$OMP_PATH" TERM="$TERM" OMP_PROFILE=cad-smoke OMP_SKIP_SETUP=1 \
    <needed provider credential variables> omp "$@"
}
```

`OMP_PATH` names real binaries, never version-manager shims: a mise shim reads the user's home configuration and fails
under the temporary HOME (for mise, use `dirname "$(mise which omp)"`). Settings, credentials and what still loads are
under "Environment isolation" below. Project `.omp/` discovery remains active, so use only the fresh project and plugin.

1. Fresh project: `SMOKE_PROJECT=$(mktemp -d)`, then `git -C "$SMOKE_PROJECT" init`. Confirm `hw/` and
   `.cad-work/` are absent. Never reuse a project between cells.
2. Install the variant at project scope with the commands in [install.md](install.md) for this harness. For Claude
   Code and OMP, the marketplace route is the release route; a run with `--plugin-dir` is recorded as a local run.
3. Start the harness in `SMOKE_PROJECT` with the start command below. One session, one lead: every reply resumes the
   same session id, and a second session for the project (a fork, or a reply sent to an older id) fails the cell.
   Never set an operator-side deadline shorter than the harness turn limit. OMP can exceed an hour per turn; `--max-time 2h`
   provides a recoverable ceiling for this smoke. If credentials fail mid-turn, resume once with a neutral, recorded line;
   never resend scripted answers. For OMP, interactive `--restore <session-id>` continues the interrupted turn and its
   subagents under the same session id; it refuses `-p`. For OpenCode, `run --session` cannot resume without a message:
   send one neutral, recorded line (for example, `Continue from the last recorded state.`) and record this deviation, or
   restart in a fresh project and mark the original cell `BLOCKED`. A turn must not end while a dispatched unit still
   runs ([delegation](../full/skills/cad-flow/references/delegation.md#dispatch-units) step 5); on headless Claude Code
   this also needs the background-task setting under "Environment isolation".
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
3.0.0 and OMP 18.13.2; the Claude Code isolation, auth and background-task forms again with 2.1.318, and the OMP forms
in a scratch project. `claude_smoke`, `omp_smoke` and `omp_cell` are the wrappers under "Environment isolation".

| Harness | Interactive start | Headless start and resume | Prompt prefix |
|---|---|---|---|
| Claude Code | `claude_smoke` in `SMOKE_PROJECT` | `claude_smoke -p --max-turns 200 "<prompt>"`; `claude_smoke --resume <session-id> -p --max-turns 200 "<reply>"` | `/cad-agent-skills:cad-flow` or `/cad-agent-skills-lean:cad-flow` |
| Codex CLI | `codex -C "$SMOKE_PROJECT"` | `codex exec -C "$SMOKE_PROJECT" "<prompt>"`; from `SMOKE_PROJECT` (`exec resume` has no `-C`): `codex exec resume <thread-id> "<reply>"` | `$cad-flow` |
| OpenCode | start `opencode serve --port <p>` in the isolated environment, with fixed `--model <provider/model#variant>` | `opencode run --standalone --format json --model <provider/model#variant> "<prompt>"`; replies use `opencode run --standalone --model <provider/model#variant> --session <session-id> "<reply>"` | `/cad-flow` |
| Kiro CLI V3 | `kiro-cli chat --v3` in a `tmux` session, each tool call approved by hand | not used: 3.0.0 headless needs `--trust-all-tools` | name the skill in prose: `Use the cad-flow skill.` |
| OMP | `omp_cell` in `SMOKE_PROJECT` | `omp_cell -p "<prompt>"`; `omp_cell --resume <session-id> -p "<reply>"`; recover interactively with `omp_cell --restore <session-id>` | `/cad-flow` |

| Harness | Skills | Roles |
|---|---|---|
| Claude Code | `claude_smoke -p --no-session-persistence --output-format stream-json --verbose --max-turns 1 "Reply OK."`: `skills` array of first `system` `init` event, names prefixed `<plugin>:` | `agents` array of same event |
| Codex CLI | `codex debug prompt-input` in `SMOKE_PROJECT`: each `- cad-<name>:` skill entry | each native `spawn_agent` call in the transcript, or fallback |
| OpenCode | fallback (v2.2.5 has no skill listing command; `opencode agent list` does not exist) | run `opencode debug agents --standalone`, save JSON, retry until it parses, then use `jq -r '.[].name' <file>` |
| Kiro CLI V3 | interactive `/context show`; headless: fallback | `kiro-cli agent list` in `SMOKE_PROJECT`: rows marked `Workspace` (it also prints global profiles) |
| OMP | `omp_smoke read skill://<name>/SKILL.md` per skill: header shows resolved path | fallback over `<installPath>/agents/*.md`, with `installPath` from `omp_smoke plugin list --json` (`.marketplace[].entries[].installPath`) |

Fallback, where the host has no listing: enumerate the installed files the install wrote (skills
`<skills dir>/<name>/SKILL.md`; roles the role files), read each frontmatter `name` (Codex TOML `name`, Kiro JSON
`name`; OpenCode roles use file names) and record the command and its output → each expected name once.

### Environment isolation

| Harness | Settings for the cell |
|---|---|
| Claude Code | Start, resume and list through `claude_smoke` below: `--setting-sources project,local --strict-mcp-config`, an auth-only `--settings` file, and `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS=0`. `claude plugin` install and cleanup commands run without the wrapper. |
| Codex CLI | Create a fresh `CODEX_HOME` with a minimal `config.toml` that sets the model, provider credentials, region and project trust for `SMOKE_PROJECT`; use `--ignore-rules` on `codex exec` and `codex exec resume`. The toolbox wrapper rewrites the config: pre-create an empty `.builder-mcp-setup-complete` to prevent the builder-mcp server; it still adds `approvals_reviewer` and telemetry, which must be recorded as deviations. |
| OpenCode | Set `OPENCODE_CONFIG_DIR` to an empty directory and `OPENCODE_DISABLE_CLAUDE_CODE=1`. Use `--standalone` on each `opencode run` and agent listing; that starts a private server and avoids the background service. Pass provider credentials and region in environment variables and fix the model with `--model <provider/model#variant>`. |
| Kiro CLI V3 | none found: record the user-level agents, steering and MCP servers that load |
| OMP | `omp_smoke` from the procedure: temporary HOME, named profile `cad-smoke`, `OMP_SKIP_SETUP=1`; the profile settings below turn off memory, advisor, autolearn, TTSR and project MCP config; `omp_cell` adds the model-role overlay and `--no-rules --no-context-files --no-extensions --skills 'cad-*' --max-time 2h`. |

Claude Code wrapper, with `CLAUDE_AUTH` the absolute path of the auth-only settings file:

```sh
claude_smoke() {
  env -i HOME="$HOME" PATH="$PATH" TERM="$TERM" CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS=0 <needed credential variables> \
    claude --setting-sources project,local --strict-mcp-config --settings "$CLAUDE_AUTH" "$@"
}
```

- `--setting-sources project,local` drops the user settings, and with them provider auth kept in
  `~/.claude/settings.json`: round 2 failed with AWS authentication errors (403) on Bedrock. `CLAUDE_AUTH` holds only
  the provider credential key and the `env` keys auth needs, copied from the user settings (round 2 on Bedrock:
  `awsCredentialExport`, and `env` with `CLAUDE_CODE_USE_BEDROCK`, `AWS_REGION`, `ANTHROPIC_DEFAULT_HAIKU_MODEL` and
  `CLAUDE_CODE_ENABLE_AUTO_MODE`). Keep the file outside the project and the evidence directory, mode `0600`; record
  its key names, never its values. Built-in plugins still load (2.1.318: `cc-plugin-agents-md`, `cc-plugin-privacy`,
  `cc-plugin-plugin-authoring`), and the toolbox wrapper still merges its telemetry environment into `--settings`;
  record both as deviations.
- Headless `-p` stops background tasks 10 minutes after the last turn ends ("Background tasks still running 10m after
  the last turn … stopping them"). `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS=0` removes that ceiling; it is not in
  `claude --help`. It does not excuse the lead: a turn that ends while a dispatched unit still runs breaks
  [delegation](../full/skills/cad-flow/references/delegation.md#dispatch-units) step 5 and fails the native dispatch row.
- `--max-turns` is hidden from `claude --help` (2.1.316 and 2.1.318) but parsed: a non-number fails with
  `option '--max-turns <turns>' argument … is invalid`.
- Claude Code has no per-role filesystem isolation; see [limitations](limitations.md#observed-host-limitations).

OMP: the temporary HOME hides the user's `~/.omp` (settings, stored logins, plugins, extensions, rules, memory,
advisor roster, sessions) and the other hosts' user roots (`~/.claude`, `~/.codex` and so on) that OMP can read.
`OMP_PROFILE=cad-smoke` keeps all OMP state under `$OMP_HOME/home/.omp/profiles/cad-smoke/`: `omp_smoke config path`
prints its `agent` directory, sessions go to `agent/sessions/`, logs to `logs/` and plugin installs to `plugins/`.
`OMP_SKIP_SETUP=1` skips the first-run provider sign-in screen of the interactive start. Before the install, set the
profile settings, then read them back from the project, where project settings also apply:

```sh
omp_smoke config set memory.backend off
omp_smoke config set advisor.enabled false
omp_smoke config set autolearn.enabled false
omp_smoke config set ttsr.enabled false
omp_smoke config set mcp.enableProjectConfig false
(cd "$SMOKE_PROJECT" && for k in memory.backend advisor.enabled autolearn.enabled ttsr.enabled \
  mcp.enableProjectConfig task.isolation.enabled; do printf '%s=' "$k"; omp_smoke config get "$k"; done)
```

Each value read back is `off` or `false`. A fresh profile defaults to `ttsr.enabled: true` and
`mcp.enableProjectConfig: true`; the other settings default to off and are set to make the record explicit.
`config set` refuses an unknown key (`Unknown setting`). Start the lead through `omp_cell`, with `OMP_ROLES` the
model-role overlay (`modelRoles` and `enabledModels`):

```sh
omp_cell() {
  omp_smoke --config "$OMP_ROLES" --no-rules --no-context-files --no-extensions --skills 'cad-*' --max-time 2h "$@"
}
```

Stored OMP logins stay in the user's profile and are not visible, so pass provider credentials as environment
variables. For Bedrock, pass `AWS_PROFILE` and an absolute `AWS_CONFIG_FILE` (and `AWS_SHARED_CREDENTIALS_FILE` for
static keys), because `~/.aws` now resolves under the temporary HOME. A `credential_process` that itself reads the
user's home fails there (observed: `toolbox: Unable to run codex`). Point `AWS_CONFIG_FILE` at a copy of the profile,
outside the project and the evidence directory, whose line is
`credential_process = /usr/bin/env HOME=<real home> <original command>`.

In a scratch run of these wrappers (OMP 18.13.2), no advisor, memory, TTSR, MCP server, user plugin, extension or bash
interceptor loaded. The tools were `read`, `bash`, `edit`, `eval`, `glob`, `grep`, `task`, `wait`, `todo`,
`web_search` and `write`. Still active, and recorded as deviations: startup probes of local model servers (Ollama,
LM Studio, llama.cpp on localhost), the online model-catalog refresh and the update notice; never update OMP during the
cell. Record anything else that loads. Do not install or mutate plugins during the cell. These controls are discovery
isolation, not an OS boundary.

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
| resolved resources | `.cad-work/lead/preflight.json`; the `.cad-work/<unit>/input/` copies | every path absolute and existing; no write to an installed file; a copy of an installed file under `.cad-work/<unit>/input/` is allowed only when byte-identical to it (equal sha256) |
| lead claim and session | `.cad-work/lead/active/claim.yaml` during the run, the session copy after | one ACTIVE claim whose `session` is the host's native id (or `UNAVAILABLE-<stamp>` with the limit reported), released at finish |
| GROUND and INTENT verdicts | `hw/versions/<vN>/stage-verdict.json` and their sha256 | each validates; PASS before its gate is presented |
| gate decisions | `hw/decisions/DR-###.yaml` and the saved statements | `kind: GATE`, actor name starting `SYNTHETIC`, the quoted reply equals the scripted text |
| native dispatch | the harness transcript and `.cad-work/lead/units.yaml` | each unit's role, native task id, brief sha256 and state; the task text is the role, brief path and role reference only; no turn ends while a dispatched unit still runs |
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
