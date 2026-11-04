# Source map

This map records where selected responsibilities from the public cad-agent
v0.1.1 project and manufacturing-intent source work live in this instruction-
only redesign. The source work included released v0.1.1 material and a separate
manufacturing-intent extension; the extension is not represented as part of the
v0.1.1 release. This is a map of new owners, not byte-for-byte migration or
relicensing.

| Source responsibility | New home | Boundary |
| --- | --- | --- |
| Fifteen cad-agent skills and manufacturing-intent workflow | Eight owner skills under [`full/skills/`](../full/skills/): [`cad-flow`](../full/skills/cad-flow/), [`cad-record`](../full/skills/cad-record/), [`cad-select`](../full/skills/cad-select/), [`cad-design`](../full/skills/cad-design/), [`cad-model`](../full/skills/cad-model/), [`cad-verify`](../full/skills/cad-verify/), [`cad-review`](../full/skills/cad-review/) and [`cad-release`](../full/skills/cad-release/). | Responsibilities were regrouped by ownership; legacy helpers and runtime are not bundled. |
| Record fields, path layout, schemas, templates and contract fixtures | `full/skills/cad-record/` (`layout.json`, `schemas/`, references and assets). | New record contract; not cad-agent v1. Legacy records are explicitly imported as data. |
| Material facts and manufacturing-intent evidence | `full/skills/cad-select/references/materials.md`, `full/skills/cad-verify/references/process-checks.md`, cad-record material schemas and sourced profiles in `full/skills/cad-record/assets/manufacturing/material-profiles/`. | Profile source fields retain supplier/standard attribution, conditions and uncertainty; values are not automatically allowables or qualifications. |
| CAD kernel contract, frames, deterministic inputs and export/readback | `full/skills/cad-model/references/`. | Runtime build programs are authored per product project. No old runner, cache, helper library or validator is copied. |
| Motion, fits, process checks, loads, analysis and physical evidence | `full/skills/cad-verify/references/`; [`cad-record validation`](../full/skills/cad-record/references/validation.md) describes semantic checks. | Model-written semantic checks are not shipped; geometry and simulation are not physical proof. |
| Mechanical and design-for-manufacture challenge | `full/skills/cad-review/references/`. | One reviewer role and severity rubric; the lead owns changes and decisions. |
| Drawings, BOM, freeze and release package | `full/skills/cad-release/references/` and `assets/`. | Reviewed model-generated drawings; no producer-ready or standards-conformance claim. |
| Legacy lead/worker responsibilities | `full/skills/cad-flow/references/` and `agent-roles.md`; five OMP profiles in `full/agents/` and native profiles in `full/harness/`. | Ordinary harness lead runs flow; it is not a sixth agent. Native permissions are not an OS boundary. |
| Regression and demonstration cases | [`examples/regressions/`](../examples/regressions/), [`examples/phone-stand/`](../examples/phone-stand/) and [`examples/lamp-arm/`](../examples/lamp-arm/). | Declarative data, not runnable end-to-end CAD or proof of runtime behavior. |
| Repository installation, architecture, evidence, limitations and static validation | [`docs/install.md`](install.md), this doc, [`engineering.md`](engineering.md), [`limitations.md`](limitations.md) and CI. | CI checks static structure and schema shape; live smoke is separate. |

The source repositories are attributed in [NOTICE](../NOTICE). Source-specific
terms remain attached to cited material; this map makes no relicensing claim.
