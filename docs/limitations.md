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

## Host and tool limits

Native agent permissions and prompts constrain accidental tool use, not
adversarial behavior or indirect filesystem effects. Host parent policy applies
and may differ by host or installation. Codex custom-agent profiles cannot set
a narrower writable filesystem path. Kiro, OpenCode and Claude policy layers do
not provide OS isolation. An OS read-only mount or sandbox can provide stronger
confinement, but this repository installs none. Host discovery, permissions,
model routing and resource loads remain unproved until live smoke runs per host.

### Observed host limitations

- Kiro headless cannot approve denied tools without `--trust-all-tools`.
- A concurrent OMP process can delete the shared plugin cache during a run.
- `claude plugin details` reports zero agents even though the session `init` lists all five.

`uv lock --script` needs package-index/network access to resolve and lock
dependencies. A lock does not imply offline availability. Runtime CAD
dependencies are installed in the product project, not shipped here.

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

Decision 1 defines lean's 11 direct schema roots. The other 17 contracts are
prose tables in
[`lean/skills/cad-record/references/prose-contract.md`](../lean/skills/cad-record/references/prose-contract.md).
Model-written `record.semantic` item 17 checks these tables, and no validator
ships here. Decision 7 requires stage, gate, procedure and check-family parity.
The repository publishes no A/B result.
