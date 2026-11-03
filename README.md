# CAD Agent Skills

A purely skill- and agent-driven tool for mechanical product architecture and
CAD modelling. It uses written workflows and structured data contracts; it
does not ship an executable application, installer, generator, hook, or CAD
implementation.

## Variants

- **Full** (`full/`): the full-parity variant described by the project design.
- **Lean** (`lean/`): a smaller-contract variant with the same skill and role set.

Both variants have separate Claude Code and OMP manifests. The root Claude Code
and OMP marketplaces list `cad-agent-skills` and `cad-agent-skills-lean`.

## Harnesses

The planned harness targets are Claude Code, OMP, Codex CLI, OpenCode, and Kiro
CLI. This checkout contains Claude Code and OMP catalog and plugin manifests;
it does not yet contain the skills, agents, schemas, or examples.

## Repository status

This checkout contains the Apache-2.0 license, repository metadata, marketplace
catalogs, plugin manifests, CI configuration, and documentation scaffold. It
does not contain the skills, agents, schemas, or project examples. The
architecture document records planned names and organization only.

## Safety boundary

The design treats generated geometry or simulation as distinct from physical
validation. This repository makes no claim that any design is safe,
manufacturable, certified, or physically validated.

## License and attribution

The repository is licensed under Apache-2.0; see [LICENSE](LICENSE) and
[NOTICE](NOTICE) for source attribution.
