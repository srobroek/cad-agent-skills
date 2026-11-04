# Live smoke

Owner: release and demonstration docs (U7). This page is the manual live smoke for every harness and both plugin
variants. It proves installation, discovery, resource loading and the first two stages on a fresh project. It
proves no CAD, DFM, analysis or physical result; static CI proves none of these either. Install commands are in
[install.md](install.md); stage behaviour is the `cad-flow` skill's.

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

## Procedure

Run the steps in order for one cell. Write every captured output to `$SMOKE_LOG/<harness>-<variant>/`, a directory
outside the project.

1. Fresh project: `SMOKE_PROJECT=$(mktemp -d)`, then `git -C "$SMOKE_PROJECT" init`. Confirm `hw/` and
   `.cad-work/` are absent. Never reuse a project between cells.
2. Install the variant at project scope with the commands in [install.md](install.md) for this harness. For Claude
   Code and OMP, the marketplace route is the release route; a run with `--plugin-dir` is recorded as a local run.
3. Start the harness in `SMOKE_PROJECT` with the start command below. One session, one lead.
4. Discovery: list skills and agents with the host's own listing (below) → exactly the eight skills `cad-flow`,
   `cad-record`, `cad-select`, `cad-design`, `cad-model`, `cad-verify`, `cad-review`, `cad-release` and the five roles
   `cad-concept-designer`, `cad-part-designer`, `cad-checker`, `cad-mech-dfm-reviewer`, `cad-sourcing-researcher`,
   each once. A duplicate or a missing name is `FAIL`.
5. Send the phone-stand first prompt. At preflight the lead resolves every skill, schema and role reference to an
   absolute path and reports the plugin name and version from the `cad-flow` frontmatter; it must equal the variant
   installed and the tag under test.
6. GROUND: answer each question with its scripted answer. The lead bootstraps `hw/` from the template, writes the
   GROUND verdict and presents the gate packet with the exact approval form; send `approve GROUND <vN>`.
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

The command forms below come from the cad-agent v0.1.1 live runs and the design's research; confirm each against
the installed CLI's `--help` and record any difference as a smoke finding.

| Harness | Interactive start | Headless start and resume | Prompt prefix | Discovery |
|---|---|---|---|---|
| Claude Code | `claude` in `SMOKE_PROJECT` | `claude -p --max-turns 200 "<prompt>"`; `claude --resume <session-id> -p "<reply>"` | `/cad-agent-skills:cad-flow` or `/cad-agent-skills-lean:cad-flow` | `/skills`; the agent picker shows the namespaced roles |
| Codex CLI | `codex -C "$SMOKE_PROJECT"` | `codex exec -C "$SMOKE_PROJECT" "<prompt>"`; `codex exec resume <session-id> "<reply>"` | `$cad-flow` | `/skills`; the five custom agents from `.codex/agents/` |
| OpenCode | `opencode` | `opencode run --format json "<prompt>"`; `opencode run --session <session-id> "<reply>"` | `/cad-flow` | the `skill` tool listing; the five `@` agents |
| Kiro CLI V3 | `kiro-cli chat --v3` | `kiro-cli chat --v3 --no-interactive "<prompt>"`; `kiro-cli chat --v3 --resume-id <session-id> "<reply>"` | name the skill in prose: `Use the cad-flow skill.` | `/agent list` and the skill resources of each profile |
| OMP | `omp --config "$MODEL_ROLES" --config "$MODELS"` | the same, with the harness's headless flag | `/cad-flow` | plugin listing, skill listing and agent listing; each `skill://` reference resolves |

- Claude Code: the plugin's own permission settings are not applied by the host; the parent session's permission mode
  applies to every role. Record that mode.
- Codex CLI: custom agents inherit the parent sandbox and approval mode. Run the smoke in the default
  `workspace-write` sandbox; a step the sandbox blocks is a finding, never a reason to switch to full access.
- OpenCode: do not pass `--auto`; it approves permissions broadly and hides gate behaviour.
- Kiro CLI V3: grant only exact tool ids with `--trust-tools`; a run that needs `--trust-all-tools` is a finding.
  Parent permissions must allow what a role profile allows.
- OMP: `omp config get task.isolation.enabled --json` prints `false` before the run; a local run adds
  `--plugin-dir "$CAD_AGENT_SKILLS_SRC/<variant>"`.

## Evidence to capture

| Item | Source | PASS when |
|---|---|---|
| tag, commit, variant, plugin name and version | prerequisites; preflight report | the reported plugin and version equal the variant and tag installed |
| CLI version, model roles | `--version`; the written role list | recorded before the first prompt |
| discovered skills and roles | step 4 listing, verbatim | eight skills and five roles, each once |
| resolved resources | `.cad-work/lead/preflight.json` | every path absolute and existing; no path inside `SMOKE_PROJECT` for an installed file |
| lead claim and session | `.cad-work/lead/active/claim.yaml` during the run, the session copy after | one ACTIVE claim for this session, released at finish |
| GROUND and INTENT verdicts | `hw/versions/<vN>/stage-verdict.json` and their sha256 | each validates; PASS before its gate is presented |
| gate decisions | `hw/decisions/DR-###.yaml` and the saved statements | `kind: GATE`, actor name starting `SYNTHETIC`, the quoted reply equals the scripted text |
| native dispatch | the harness transcript and `.cad-work/lead/units.yaml` | each unit's role, native task id, brief sha256 and state; the task text is the role, brief path and role reference only |
| deliveries | `.cad-work/<unit>/delivery.json` | validates against `worker-delivery`; boundary digests equal; accepted bytes merged verbatim |
| writes | the transcript's tool calls | no write to an installed skill, schema or role file and none outside `hw/` and `.cad-work/` |
| schema results | step 8 output | every command exits 0 |
| obligations | step 9 comparison | no missing or extra family; no `must_not` |
| transcript | the harness session export or JSONL | kept unedited |

A cell is `PASS` only when every row is PASS. Report each cell with its result, the failing rows and the absolute path
of its evidence directory.

## What the smoke does not show

Static checks and this smoke cover install, discovery and the first two stages. Geometry builds, drawings, BOM,
analysis, review, freeze and release run in the demonstrations in [demo.md](demo.md). Synthetic approvals stand for no
product owner. A permission profile prevents accidental tool calls only; it is no operating-system boundary.
