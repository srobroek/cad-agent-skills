# Limitations

This distribution supplies instructions, schemas, templates, profiles, examples
and host agent definitions. It does not itself build or test a product.

## What is not validated here

- Repository CI, examples, demos and live smoke produce or test no physical
  parts. Geometry and simulation do not prove safety, lifetime, fit after
  manufacture, material-lot properties or service performance. Required
  physical evidence is a release gate; see [engineering principles](engineering.md).
- Drawings are model-generated for review. They are not producer-ready,
  universally complete or claimed to conform to ISO, ASME or another standard.
  Unsupported dimensions, views, exports or font checks block the drawing
  stage.
- This repository ships no semantic validator. The
  [cad-record validation reference](../full/skills/cad-record/references/validation.md)
  lists semantic invariants that JSON Schema cannot express. A model-written
  program performs those checks in the product project; an independent reviewer
  inspects it. CI does not include or execute that program.
- The install guard is instruction-only. No hook audits installed files or
  prevents host changes. Report mismatches; do not patch installed content.
- Static schemas cannot prove cross-file references, uniqueness, hash bytes,
  obligation coverage, approval authenticity, freshness, immutability or CAD
  method correctness.

## Lite build limits

- A lite package claims only what its checks observed: valid closed solids, the owner's CAD dimensions and
  features, a produced and shown render, and STEP import in the recorded FreeCAD version. It states
  `LITE prototype: geometry only; no material, fit, strength or manufacturing claims`. A lite model is no evidence
  that a part can be made, fits, holds a load or is safe.
- Static non-interference and requested clearances are CAD measurements at one named static pose, not a fit, motion
  or tolerance qualification.
- The package delivers AP214 STEP, not a native parametric FreeCAD document. Its FreeCAD evidence covers the
  recorded version, not every FreeCAD version.
- Without FreeCAD the readback is UNKNOWN `NOT_RUN` and nothing is sealed; another importer does not substitute.
- Promotion to a full build is always available as a CHANGE that reopens GROUND, but it guarantees no qualification
  without redesign. Lite geometry enters the full build as concept input only.

Lite uv cache size varies by build and host. One round-7 full lite cell reported about 5 GB and 130,000 files/inodes
(`smoke-logs-r7/lite-claude-full/EVIDENCE-NOTE-uv-cache.txt`); a separate lean lite cell reported about 1.1 GB and
130,000 inodes (`smoke-logs-r7/lite-kiro-lean/REPORT.md`). A full-build cell's project used about 43,000 additional
inodes without its cache (`smoke-logs-r7/lite-claude-full/REPORT.md`). The round-7 `/tmp` tmpfs had 1,048,576 inodes
and ran out. Keep smoke projects and their caches on a disk-backed filesystem, never `/tmp`; see the
[smoke procedure](smoke.md#procedure) and [install guidance](install.md#freecad-for-lite-builds).

- The lite host time targets are unmeasured goals ([smoke.md](smoke.md#lite-timing-targets)); no completed lite cell has been timed.

## Host and tool limits

Native agent permissions and prompts constrain accidental tool use, not
adversarial behavior or indirect filesystem effects. Host parent policy applies
and may differ by host or installation. Codex custom-agent profiles cannot set
a narrower writable filesystem path. Kiro, OpenCode and Claude policy layers do
not provide OS isolation. An OS read-only mount or sandbox can provide stronger
confinement, but this repository installs none. Host discovery, permissions,
model routing and resource loads remain unproved until live smoke runs per host.
The latest smoke round covered Claude Code, OpenCode and Kiro CLI V3; the Codex CLI and OMP profiles stay in both
plugins but are not verified in it ([smoke matrix](smoke.md#matrix)).

### Observed host limitations

- Kiro headless cannot approve denied tools without `--trust-all-tools`.
- A concurrent OMP process can delete the shared plugin cache during a run.
- `claude plugin details` reports zero agents even though the session `init` lists all five.
- Claude Code has no per-role filesystem isolation: a worker's shell and file tools reach every path the session can.
  In the round-2 smoke a checker wrote and then deleted `/tmp/live.txt` and `/tmp/inv.txt`; only the worker's own
  `boundary.violations` report and the lead's rejection of the delivery caught it.
- Kiro (round 3): a machine-managed agent on the host rewrote all five installed `.kiro/agents/cad-*.json` at the first
  `kiro-cli chat --v3` launch, adding a tool to `tools` and `allowedTools`, an `mcpServers` entry and an `mcp <server>/*`
  allow rule. The role files a run dispatches then differ from the release; the smoke records this as a deviation. The
  injected tool was never called.
- Codex CLI 0.166.0 runs every command through the user's login shell (`/bin/zsh -lc`), so the user's zsh startup files
  apply despite a scrubbed environment, and zsh semantics apply to POSIX forms: a lead loop `while read -r expected path`
  overwrote zsh's `path` array and broke `PATH` (`command not found: sha256sum`).
- Claude Code on Bedrock lists `WebFetch` but no `WebSearch` in the session's tools (2.1.318), so a Claude role that names `WebSearch`,
  such as the sourcing researcher, has no search there and works from URLs it is given, such as a brief's `seed_urls`. In round 5, a
  Claude sourcing unit was run with empty seeds and rejected; the tool listing is not an observed search result. The brief now requires a
  passing vendor-origin seed per subject; without one, facts stay typed UNKNOWN. See [live-smoke evidence](smoke.md#observed-smoke-findings).
- OpenCode 2.2.5 `websearch` returned `Web search cancelled` for every lead and worker call in both round-4 cells;
  the evidence does not establish why it was cancelled. See [live-smoke evidence](smoke.md#observed-smoke-findings).
- Codex CLI 0.166.0 on Bedrock: round-4 probes found `gpt-6.1-terra` and `gpt-6-luna` in `us-east-1`; probes in
  `us-east-2` and `us-west-2` returned 404. Verify a selected model in the provider region before a cell; catalog
  availability does not establish regional availability. See [smoke findings](smoke.md#observed-smoke-findings).
- Kiro CLI V3's approval driver used a 1.0 s polling default; the round-4 driver used `--poll 0.25`. One full cell's
  approval wait share was 6.93% at the default and 5.33% after switching, including a 75.93 s driver stall. These
  observed shares do not verify a target. See [smoke findings](smoke.md#observed-smoke-findings).
- Kiro CLI 3.0.0 matches role-profile shell rules with a parser chosen from `SHELL` (its log records
  `policy.shell.type.resolved`). The fish parser also evaluates each command without its leading `VAR=value`
  assignments, and either denial denies: in the round-5 full cell, whose tmux applied the user's fish `default-shell`,
  every worker `TMPDIR=… uv …` and validator call was denied. The posix parser (bash) keeps assignments but drops
  output redirects before matching, so a profile cannot deny `> hw/…` in a bash session. The role profiles therefore
  prefix programs with `env TMPDIR=…`, which both parsers match as written. A redirect into `hw/` is still caught by
  the lead's boundary digest; one into `.git/` or a host directory lies outside the digest.
- Kiro role profiles deny file-tool reads of `hw/**`, `.cad-work/lead/**` and `.cad-work/stage-v*/**`. A static
  profile cannot name the unit, so another unit's `.cad-work/<unit>/` stays readable, and shell commands such as `ls`
  are not file-tool reads.
- Claude Code 2.1.318 `claude plugin marketplace add --scope project` declares the marketplace in the project settings,
  but still records it at user level and lists it from other projects (where the plugin stays disabled). Two cells
  that register the same marketplace are therefore not isolated from each other.

### Model-routing evidence

Each host's per-agent model selection, with the host documentation or smoke log that supports each part, is in
[smoke.md](smoke.md#observed-smoke-findings). These describe routing only, not which model or effort a role should use.

Round-4 A/B evidence does not support changing per-role defaults. OMP's terra/high and luna/xhigh runs confound model
and effort, and both OMP cells failed obligations. Claude's baseline failed before equivalent INTENT; only checker
acceptance compares (8/10 sonnet versus 7/9 opus, directional). Codex A/B outcomes were blocked at GROUND or sourcing
and do not establish a general tier result. Keep shipped role model lines unchanged; a matched-effort A/B after the fix
wave is needed to decide defaults.

The lead warms one shared uv cache with network in preflight
([Warm the uv cache](../full/skills/cad-flow/references/preflight.md#warm-the-uv-cache)); every later uv command runs
`--offline` against it, except the sourcing researcher's PDF `extract.py`, which locks and runs on the network its
fetches use ([uv commands](../full/skills/cad-flow/references/agent-roles.md#uv-commands)). A package missing from the
warmed cache fails offline and is never fetched by a unit. Runtime CAD dependencies are installed in the product project,
not shipped here.

Retain font files and licenses by their bytes. A font family name alone is not
deterministic input. Drawing text may become outlines with no text layer.
Review the rendered sheet. Block font mismatches rather than silently using a
host font.

## Technical and evidence boundaries

Sampled motion covers declared poses unless the method proves a continuous
bound; unsampled intervals remain unknown. A nominal pose does not establish
full travel or assembly access. Material profiles cite supplier or standards
sources, but do not qualify lots, processes, equipment, operators or allowables.
Some source details are incomplete and appear in profile guidance.

The distribution provides no CAM or slicer toolpaths, equipment operation,
automatic purchasing, certification, universal feature/GD&T recognition, or
unqualified fatigue, creep, life or nonlinear analysis. Unsupported required
methods, data, approvals or capabilities remain UNKNOWN or BLOCKED. Do not infer
claims from concept packages, previews, regression fixtures or synthetic
approvals.

Lean has 11 direct schema roots. The other 17 contracts are prose tables in
[`lean/skills/cad-record/references/prose-contract.md`](../lean/skills/cad-record/references/prose-contract.md).
Model-written `record.semantic` item 17 checks these tables, and no validator
ships here. The variants share stage, gate, procedure and check-family names.
The repository publishes no A/B result. Round-4 evidence does not establish a per-role model or effort recommendation; see [model-routing evidence](#model-routing-evidence).
