# Changelog

## Unreleased

- Add eight full-variant skills, five canonical roles and native profiles for Claude Code, Codex CLI, OpenCode, Kiro CLI V3 and OMP.
- Add full record and authoring schemas, fixtures, regression cases and phone-stand and lamp-arm declarative examples.
- Add the lean variant: the same skills, roles, stages, gates and check families with ten record schemas and shared common definitions;
  the other contracts stay prose.
- Document architecture, engineering evidence rules, limitations, source provenance, installation, demonstrations and live smoke guidance.
- Expand pinned GitHub and GitLab CI for full and lean schemas, fixtures, templates, layout parity, host profiles, examples, regressions,
  version provenance, Markdown, frontmatter and local links.
- Round-4 fix wave, in both variants:
  - Workers no longer need network for uv: the lead warms one shared cache, `.cad-work/lead/.uv-cache`, in preflight with one network
    escalation, and every lead and unit uv command runs `--offline` against it (the sourcing `extract.py` stays online). A program locks
    once and then only runs `--locked`. Preflight records the paths `command -v uv curl` prints.
  - Each unit writes temporary files under `.cad-work/<unit>/tmp/`, never `/tmp`, and every program and validator command runs with
    `TMPDIR` set there; the layout adds `scratch.unit_tmp` and `scratch.lead_tmp` and drops `scratch.unit_uv_cache`. Briefs gain
    required `limits` (wall seconds and model turns), and a unit delivers after its first fully valid run. The lead stops a stalled or
    lost unit, including one lost to an HTTP 429, sends it again once, then records UNKNOWN `NOT_RUN` or `TIMEOUT` (a stalled
    reviewer's lead review result included).
  - Checkers read lead-owned facts from brief snapshots (`input/lead/attempts.txt`, `input/installed/<skill>/<path>`) instead of the
    live tree, and each brief's input inventory follows from the checks it assigns. Checker and reviewer briefs must list
    `expected.obligations`.
  - `delivery.integrity` applies when the attempt dispatches at least one worker unit; dispatched work that merges nothing is UNKNOWN,
    never a vacuous PASS. Accept writes ACCEPTED or REJECTED and the merged triples before the next dispatch.
  - A typed-UNKNOWN sourcing delivery is valid, and a BLOCKED stage still freezes its attempt and writes its verdict. Sourcing
    deliveries name literal targets, write only inside the boundary and count every fetch attempt against the budget. The lead checks
    that each seed URL resolves before dispatch. Codex units are spawned with `fork_turns: "none"`, and the dispatch log keeps the task
    text.
  - A gate reply that ends `then end the session` records the decision, advances nothing and releases the claim in the same turn.
  - The lead validates every record file in scratch before publishing. A defective earlier snapshot is judged only by its own attempt
    and bound by hash (`RECORDED` basis), and is never repaired in place.
  - Check evidence cites only published logs: command `stdout` and `stderr` name `stdout.txt` and `stderr.txt` beside the result, and the
    schema rejects scratch paths. One rule now fixes each check bundle manifest's writer, moment and exact file list. The schema-validation
    reference describes the real check-jsonschema 0.39.0 JSON report keys.
  - GROUND writes requirements, preferences and assumptions whole in the first record batch, volunteered facts included. Owner
    equipment becomes a `make_capability_profile` criterion, and owner-stated quantities become criteria or assumptions. Each unknown
    subject gets its own UNKNOWN and question. Release-scope exclusions and answered fields never become questions. SPECIFY writes no
    requirement, and a requirement gap is a change that reopens GROUND.
  - Material identity settles at INTENT. Consumable form, such as filament diameter, is checked by `capability.route` at PARTS and
    never keeps a candidate CONDITIONAL. Capability approvals take an optional `diameter`. Material profiles carry qualitative ratings
    and quoted out-of-set units, and melt-flow units `g_10min` and `cm3_10min` are added. A MOTION case's joints resolve from LAYOUT on.
  - OpenCode and Kiro role profiles allow the `TMPDIR`-prefixed, offline uv forms against the shared cache and the `TMPDIR`-prefixed
    validator and YAML loader. The per-unit cache forms are removed.
  - Examples and regressions follow the new rules: per-subject questions in lamp-arm, new phone-stand obligations, the consumable-form
    limit on printed candidates, and `OTHER` among the phone-stand release-scope exclusions.
  - Smoke, limitation and installation docs cover the round-4 host findings and per-agent model override keys. Model defaults are
    unchanged.
