| Finding | Fixed/refuted | Files | Evidence |
|---|---|---|---|
| #1 variant licensing | Fixed | full/LICENSE, lean/LICENSE, full/NOTICE, lean/NOTICE, manifests, docs/install.md (FixHosts), CI | License files copied from root; NOTICE copies refreshed; manifests declare Apache-2.0; CI compares each plugin-root file to root. FixHosts installs variant-specific files with cad-flow. Focused `cmp -s` equality check passed. |
| #2 safety severity | Fixed | examples/lamp-arm/hw/decisions/DR-001.yaml | L-01 quote is corrected. Lamp-arm L-14 still says severity unknown; remove `severity: MINOR` from product.yaml and record fact UNKNOWN in schema shape. |
| #3 imported file count | Fixed | examples/lamp-arm/expected-obligations.yaml | Mapping contains 32 files; expected count is 32. |
| #5 L-01 quote mismatch | Fixed | examples/lamp-arm/hw/decisions/DR-001.yaml | Quote matches scripted L-01 at `brief.md:44`. |
| #6 imported REQ-005 source | Fixed | examples/lamp-arm/hw/requirements.yaml | Mounting size criterion cites imported v0.1.1 REQ-005 via POLICY, DR-001. Full requirements schema passed earlier. |
| #7/#8 NOTICE | Fixed | NOTICE, full/NOTICE, lean/NOTICE | Both profiles/READMEs are cited; AgentCAD 0.7.0 boolean volume method attributed; copies refreshed. |
| #9 engineering authority wording | Fixed | docs/engineering.md | Links to release ceiling; only named approver may authorize allowed scope. |
| #10 CI git failure masking | Fixed, mirrored | workflows/ci.yml, .gitlab-ci.yml | Captures `git ls-files` before grep. |
| #11 inventory check | Fixed, mirrored | workflows/ci.yml, .gitlab-ci.yml | Broadened extensions and executable mode scan. |
| #12 ignore node_modules | Fixed | .gitignore | Added `node_modules/`. |
| #13 GitLab image digest | Fixed | .gitlab-ci.yml | Uses observed sha256:593bd06efe90efa80dc4eee3948be7c0fde4134606dd40d8dd8dbcade98e669c. |
| #14 public labels | Fixed | README.md, AGENTS.md, docs, example briefs | Searches found no Decision N/U labels. |
| #15 fixture errors | Fixed, mirrored | workflows/ci.yml, .gitlab-ci.yml | Contract and authoring fixture invalid cases require their expected-errors substring. |
| Smoke round 2 (OpenCode, Codex, Kiro) | Fixed | docs/smoke.md | Persistent OpenCode server, env/model, same-server debug-agent listing; Codex CODEX_HOME/trust/wrapper; scrubbed env; Kiro workspace-roots side effect documented. |
| Smoke concept scope | Fixed | phone stand product record and docs/demo.md | SAFETY/REGULATORY added to APPEARANCE_CONCEPT exclusions. |
| Batch 1 release contracts | Fixed | expected-obligations.yaml examples; docs/demo.md | RELEASE expects verification.index; CRITIQUE no longer expects conditional delivery.integrity; procedure summary updated. |
| Batch 2 statement contracts | Partly fixed | example decision records; regression decision records + new `statements/DR-*.txt` | Added statement paths and transcript hashes to the three example product DRs. Regression decision records got statement paths, hashes and statement files. Validate those four records against FixFlow schemas when available. |
| Batch 2 out-of-scope waiver | Scenario fixed, fixture pending | regressions/scenarios/waiver-out-of-scope.md | Explains `write-verdict` step 3 maps only in-scope obligations before writing; successor semantic item 10 checks published verdict. Invalid waiver expected error still old (`until_change` required); revise after schema validation. |
| Batch 2 remaining migration | Pending | regression selections/verdicts/waivers/check results | Selection inputs + hashes, PASS obligations objects, structured `until_change`, check-result cleanup, invalid expected substring require applying FixFlow schemas then running validators. Use exact sources/hashes; see FixFlow message/schema branch. |
| Verification already run | Passed earlier; batch 2 not yet revalidated | branch worktree | Debian `cad-skills-ci` passed original validation body pre-batch-2 changes. Focused markdown and original schema checks also passed pre-batch-2. Re-run selected schemas against FixFlow worktree before handoff. |
