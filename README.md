# CAD Agent Skills

CAD Agent Skills is an instruction-only tool for mechanical product modeling
and architecture. Skills own workflow and domain rules. Harness-native agents
perform bounded specialist work. Schemas and declarative examples define
records and expected outcomes. The model creates product CAD and semantic
checks in the product project's work area. This repository ships no executable
product code, CAD engine, installer, generator or enforcement hook.

## Choose a variant

| Variant | Record contract | Use it when |
| --- | --- | --- |
| [`full/`](full/) (`cad-agent-skills`) | Full-parity design: 8 skills, 5 roles and 28 cad-record schema roots. | Use full as the complete baseline. |
| [`lean/`](lean/) (`cad-agent-skills-lean`) | Same 8 skills and 5 roles with 11 schema roots (`common` plus 10 domain roots); remaining details are prose. Stages, gates, procedures and all 54 check-family names match full. | Use lean when its reduced contract and model-performed semantic checks fit the use. Compare it with full using the controlled A/B protocol in [demo.md](docs/demo.md). |

Install one variant in each product project. Skill and role names are
identical. The A/B protocol fixes the brief, harness, model roles, environment
and project setup. It compares stage reach, gate outcomes, defects, record
validity, cost, time and determinism. Full is the baseline and wins a tie. A
result covers only tested harnesses. Both variants ship in this repository, and
the repository publishes no A/B result.

## Supported harnesses

Both variants have native definitions for Claude Code, Codex CLI, OpenCode,
Kiro CLI V3 and OMP. Claude Code and OMP use root marketplace catalogs. Codex,
OpenCode and Kiro use native profiles. See the live checks in
[smoke.md](docs/smoke.md) for host discovery, permission enforcement and model
behavior. Repository content alone does not establish runtime results.

- [Install one variant](docs/install.md)
- [Run the demonstrations and full-versus-lean A/B protocol](docs/demo.md)
- [Run the live harness smoke matrix](docs/smoke.md)

## Release ceiling and status

A product release for manufacture or service requires its physical tests and
authority evidence. Valid demonstration ends include a concept-definition
package, a stop with its named blocker, or an approved non-safety trial package
where the release policy permits it. Geometry, drawings, simulation, typical
material data and synthetic approvals are not physical validation.

Each variant contains eight owner skills and five canonical OMP roles with
native profiles. Full adds 28 record schemas and lean 11, each with fixtures.
The repository also holds authoring schemas, regression cases, two declarative
examples and static CI. Source documents describe model-run
semantic checks. This repository ships no semantic validator. Static CI and a
live smoke do not prove design safety, manufacturability, certification or
physical validation. See [limitations](docs/limitations.md) and
[architecture](docs/architecture.md).

## License and attribution

The repository uses the Apache-2.0 license. See [LICENSE](LICENSE) and
[NOTICE](NOTICE) for attribution and source boundaries. Each self-contained
plugin root carries its own copies of these files.
