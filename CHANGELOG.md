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
  - OpenCode and Kiro role profiles use `env TMPDIR=<unit-tmp>` for offline uv commands against the shared cache and for validator and
    YAML-loader commands. The per-unit cache forms are removed.
  - Examples and regressions follow the new rules: per-subject questions in lamp-arm, new phone-stand obligations, the consumable-form
    limit on printed candidates, and `OTHER` among the phone-stand release-scope exclusions.
  - Smoke, limitation and installation docs cover the round-4 host findings and per-agent model override keys. Model defaults are
    unchanged.

- Round-5 fix wave, in both variants:
  - The lead derives check rows from authoritative attempt logs, asserts counts before writing results, and applies the stop rule to every
    attempt result before publishing; only assigned worker roles produce their families. A lead-fixable BLOCKED that needs no owner,
    authority or operator action advances to the next attempt in the same turn, while a stop asks only for the needed answer or action.
    Finish writes a BLOCKED verdict for each frozen attempt without one; the claim-release line applies only while the claim is held.
    Preflight uv commands use the shared cache, except `uv --version`.
  - Gate statement files contain exactly the sent bytes, without added quotes or a newline. GROUND records owner-settled intake fields as
    KNOWN or NOT_APPLICABLE and leaves only unanswered fields UNKNOWN; product-sizing quantities become criteria, GROUND alone writes
    requirements, and consumable form and process settings are deferred to PARTS. Equipment requirements carry PROCESS_PLANNED and
    GEOMETRICALLY_MANUFACTURABLE facets; an INTENT recommendation stays CONDITIONAL until its approved gate decision.
  - Worker briefs inventory result artifacts and published logs, complete predecessor evidence chains, and hashes confirmable from
    snapshots or the append-only attempts-log prefix. Seeds are required per sourced subject and stay on the vendor's origin; only
    `extract.py` runs online, and missing or cancelled search leaves facts UNKNOWN. Tools are listed by role, units follow one ordered
    `units` list, and delivery acceptance requires results and verdicts to agree and merges every cited artifact. Worker limits preserve
    unfinished work as UNKNOWN, while the lead bounds waits and stops stalled units. `boundary.read_only` is removed from the worker-brief
    contract; the read set is instruction-only and explicitly includes installed
    files of listed skills while excluding live `hw/` and other `.cad-work/` directories.
  - OpenCode and Kiro shell profiles use `env TMPDIR=<unit-tmp>` for uv and validator commands; Kiro profiles also deny file-tool reads of
    live `hw/`, lead scratch and stage scratch, and checker-only two-argument `cp` supports retained check programs. Examples record the
    owner's 210 g phone as a typed intake load and annotate the hours-long phone load without inventing a duration. Host documentation
    records the pinned cell shell, driver monitor linger, exact gate-statement bytes, required operator response to stops, and model-routing
    evidence without changing model defaults.

- Lite build, in both variants: at the first intake the lead asks for a full or lite build (`Q-000`) and recommends lite for initial
  prototyping. The answer is the sourced fact `product.build_mode`; a record without it stays a full record.
  - A lite build runs GROUND, LAYOUT and RELEASE only, with owner approval at GROUND and RELEASE. Its intake asks no material, process,
    printer, vendor, load or capability question, and skipped stages get no verdict, decision or result.
  - LAYOUT models the requested finished form and exports part and assembly STEP (AP214), GLB and SVG. An independent checker builds,
    reads back, identifies features, renders, inspects the form against the frozen intent and imports every STEP in the installed
    FreeCAD console. Without FreeCAD, `geometry.readback` is UNKNOWN `NOT_RUN` and LAYOUT stops BLOCKED.
  - RELEASE seals a reduced package with CAD_VALID claims only, and every packet and the build guide carry `LITE prototype: geometry
    only; no material, fit, strength or manufacturing claims`. Promotion to full is an approved CHANGE that reopens GROUND and keeps every
    lite byte.
  - Records gain conditional lite shapes for product, part and release and a lite template, with no new schema root; lean carries the
    same contract in its existing roots and prose tables.
  - The authoring `expected-obligations` schema gains `build_mode`: FULL keeps the 13 ordered stages, and LITE requires exactly GROUND,
    LAYOUT and RELEASE, the lite gates, no future obligation, `APPEARANCE_CONCEPT` and the exact disclosure line.
    `examples/lite-phone-stand` adds a lite brief, its obligations and a GROUND reference record. The phone-stand and lamp-arm prompts
    name the full build, and their obligations and reference records carry the full mode; their engineering obligations are unchanged.
  - Docs cover the lite flow, the intake question, its claims, FreeCAD as an optional runtime prerequisite, promotion, and lite smoke
    cells, negatives and unmeasured host time targets for Claude Code, OpenCode and Kiro CLI V3. CI checks that the disclosure line is
    equal in `cad-flow`, the build-guide asset and the authoring schema of both variants.

- Round-6 fix wave, in both variants:
  - Intake records an owner's actual value and design bound as two typed entries, keeps an owned material separate from any machine
    approval, and never asks a consumable's form, colour or lot at GROUND or INTENT.
  - `delivery.integrity` rows name the unit and the `hw/` target with its sha256, never a scratch path, and results cite inputs by origin
    path and sha256.
  - A PASS attempt with only MINOR findings goes to its gate with those findings as limits. A producer delivery whose MINOR findings
    change no recorded fact or citation is accepted with the finding recorded.
  - Reading file contents outside the read set is a defect; a names-only listing is a recorded finding. Workers read and search with the
    host's file tools, use one stated `mkdir -p` form and validate `delivery.json` with the stated form.
  - Delegation states per host what bounds the lead's wait and what follows a returned batch, and the rule that independent checkers
    dispatch in one batch is fixed at the cause of its round-6 miss. Preflight writes the claim with the lock and compares installed tool
    versions with their pins; the publish order is one list; a lead-fixable FAIL is fixed in a new attempt without asking the owner; and
    `delivery.integrity` applicability follows the dispatch plan.
  - Smoke: OpenCode sends single-line messages as separate argv words and multi-line ones on stdin, so statement files hold the exact
    bytes; round-6 OpenCode gate rows are judged PASS with a note. Kiro cells start a pane at least 160 rows tall, and an approval waiting
    off screen is approved from the store's pending record. Codex CLI and OMP stay supported but are marked not verified in round 6.

- Round-7 fixes:
  - `56229d2`: FreeCAD preflight and readback run with an explicit `HOME`; imports of `model.py` disable bytecode so no
    `__pycache__` is written beside a frozen or published model.
  - `f3d25a2` and `2a66898`: RELEASE packets show the proposal revision, while the owner approves with the single form
    `approve RELEASE <vN>`; authoring fixtures use that same reply form.
- Round-8 fixes, in both variants:
  - J1: Reject a delivery when any path-valued field of its result, bundle manifest or cited artifacts is not a
    published `hw/` path; name the field and value.
  - J2: Judge result paths only for results produced or carried forward by the attempt under check. Re-hash earlier
    results against their recorded sha256 without re-judging paths.
  - J2 extension: semantic items judge only the attempt under check. Items 10, 16 and 17 judge the predecessor verdict once;
    an earlier unlisted path is PASS (REPORTED) only when its own verdict recorded `HASH_MISMATCH`.
  - J3: List every required lead-claim field, including `format` and `schema_version`, and validate after writing it.
  - J4: State `material_to_choice` and `safety_relevant` mappings explicitly per question row, consistent with `consequential`.
  - J5: Compose each stage verdict with each check row exactly once.
  - J6: Cite input files in check results by their origin paths and sha256, not `.cad-work/` snapshots.
  - J7: Keep smoke projects on verified disk-backed storage, never tmpfs `/tmp`. Lite uv caches with CAD kernels used about 130,000
    inodes in round-7 cells. Accept the host-resolved FreeCAD console when preflight and checker agree on path and version; record both
    observations and the executable sha256 in external smoke evidence.
  - J8: Lite intake does not re-ask fields stated in the prompt, and records exclusions verbatim.
  - J9: Lite packets report how the render was actually presented, per the
    [host-specific rules](full/skills/cad-flow/references/gates.md#lite-packets): the SVG path and exactly one `Render:` status line.
  - J10: Version snapshots copy only manifest-listed files, one by one; an unlisted path fails its attempt. `accept-results` hashes
    `hw/versions/vN` against its manifest and reports `HASH_MISMATCH` for any unlisted path.
  - `e4f91fd`: FreeCAD readback keys inputs by their published `hw/` paths.
  - Smoke and examples reflect the `56229d2` FreeCAD `HOME` and no-bytecode fixes and the `f3d25a2`/`2a66898`
    RELEASE reply-form and authoring-fixture changes.
