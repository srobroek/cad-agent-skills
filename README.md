# CAD Agent Skills

CAD Agent Skills is an instruction-only tool for mechanical product modeling
and architecture. Skills own workflow and domain rules. Harness-native agents
perform bounded specialist work. Schemas and declarative examples define
records and expected outcomes. The model creates product CAD and semantic
checks in the product project's work area. This repository ships no executable
product code, CAD engine, installer, generator or enforcement hook.

## Plugin

[`full/`](full/) (`cad-agent-skills`) is the one plugin root: 8 skills, 5 roles and 28 cad-record schema roots. Install it in each
product project. An earlier reduced `lean` variant was removed; see the [changelog](CHANGELOG.md).

## Lite or full build

At the first intake the lead asks whether you want a full or lite build and recommends lite for initial
prototyping. The plugin offers both builds.

| Build | Stages | Delivers | Claims |
| --- | --- | --- | --- |
| lite | GROUND, LAYOUT and RELEASE; owner approval at GROUND and RELEASE | an `APPEARANCE_CONCEPT` 3D model: part and assembly STEP, render and the evidence that the STEP opens in the recorded FreeCAD version | `LITE prototype: geometry only; no material, fit, strength or manufacturing claims` |
| full | all 13 stages and their gates | material, sourcing, manufacturing and the validation its approved use requires | what its approved release scope and evidence support, within the release ceiling |

Lite needs FreeCAD's console on the host; without it the build stops before RELEASE
([install](docs/install.md#freecad-for-lite-builds)). A lite project can be promoted to a full build later
through an approved change that reopens GROUND. See [architecture](docs/architecture.md#lite-build).

## Supported harnesses

The plugin has native definitions for Claude Code, Codex CLI, OpenCode,
Kiro CLI V3 and OMP. Claude Code and OMP use root marketplace catalogs. Codex,
OpenCode and Kiro use native profiles. The latest live-smoke round covered
Claude Code, OpenCode and Kiro CLI V3 only; the Codex CLI and OMP definitions
stay supported but were not verified in it. See the live checks in
[smoke.md](docs/smoke.md) for host discovery, permission enforcement and model
behavior. Repository content alone does not establish runtime results.

- [Install the plugin](docs/install.md)
- [Run the demonstrations and A/B protocol](docs/demo.md)
- [Run the live harness smoke matrix](docs/smoke.md)

## Release ceiling and status

A product release for manufacture or service requires its physical tests and
authority evidence. Valid demonstration ends include a concept-definition
package, a stop with its named blocker, or an approved non-safety trial package
where the release policy permits it. Geometry, drawings, simulation, typical
material data and synthetic approvals are not physical validation.

The plugin contains eight owner skills and five canonical OMP roles with
native profiles. It adds 28 record schemas, each with fixtures.
The repository also holds authoring schemas, regression cases, three declarative
examples (two full builds and one lite build) and static CI. Source documents describe model-run
semantic checks. This repository ships no semantic validator. Static CI and a
live smoke do not prove design safety, manufacturability, certification or
physical validation. See [limitations](docs/limitations.md) and
[architecture](docs/architecture.md).

## License and attribution

The repository uses the Apache-2.0 license. See [LICENSE](LICENSE) and
[NOTICE](NOTICE) for attribution and source boundaries. Each self-contained
plugin root carries its own copies of these files.
