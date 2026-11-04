# Use case and route selection

Owner: cad-select. Field and path authority belongs to cad-record. Read `cad-record/references/contract-index.md` and `cad-record/references/record.md`; use the active brief's absolute skill/schema paths. Do not copy their rules into this reference.

## intake-use-case
### Establish intended use

Ask only questions that resolve this product's actual intended function and constraints. Cover:

- Product purpose; users; installation, operation, environment and service access.
- Load direction and magnitude, impact, vibration, cyclic and sustained duration; motion, speed, duty, friction, wear and required life.
- Temperature, heat sources, moisture, chemicals, UV, corrosion; accuracy, repeatability, finish and dimensional stability.

- Failure consequences, supervision, standards and safety constraints.
- Quantity, budget, lead time, repair, maintenance, available tools/operators and outsourcing.
- Whether appearance, fit, a limited supervised demonstration, a representative working prototype, production or safety-critical service is intended.
Ask focused questions and record answers, refusals and unresolved items, not a generic questionnaire dump. Resolve what changes requirements, architecture, route feasibility or safety; keep other unknowns explicit and bounded.

Before any material comparison, inspect the relevant existing product record and ask what is missing for those needs. Do not infer use from CAD geometry or from the phrase “prototype”.

Capture **requirements**, **preferences**, **assumptions** and **open questions** separately in `hw/requirements.yaml`. Do not turn an assumption into a requirement or use an unanswered question as evidence. Resolve questions whose answers can change a material, route, architecture, safety conclusion or mandatory criterion before making a final recommendation; otherwise label the recommendation CONDITIONAL and bound the unresolved choice.

Record the user-selected service class independently of rigor. Service class expresses use/maturity; rigor (`maker|pro|regulated`) expresses documentation depth and never reduces required evidence. A request for a “prototype” is a question about service class, not a selection of rigor. Do not infer either from CAD or example geometry.

## select-candidates
### Form whole candidate routes

Develop credible combinations before ranking. For each candidate, state its architecture and every relevant part's sourcing, exact material profile or explicit UNKNOWN, and ordered route. Include secondary operations, finishing, joining, inspection, maintenance and replacement. One product may mix made and purchased components, materials and operations.

Cover each physical part and disconnected component, then each functional contacting pair: relative motion/load, friction/wear, temperature/environment, corrosion or galvanic contact, dimensional fit, lubrication, sealing, electrical or optical requirements where relevant. State how the architecture changes load paths, interfaces, assembly, tools, service and retention. A material-only or shaping-process-only comparison is incomplete.

Include credible additive, resin, subtractive, cutting/forming/joining, casting/moulding/extrusion, purchased and hybrid routes as relevant; do not default to available equipment or to metal. If stated service exceeds credible evidence for a printed polymer, compare metal, purchased hardware, a different route or revised architecture and identify the governing requirement. A polymer is appropriate when evidence and complete checks support the actual requirements.

Do not invent machine, operator, supplier, material approval, work envelope, tool, calibration, tolerance, cost, availability, lead time or life. Capability templates are field-and-guidance prompts, not capabilities. An unknown route capability remains UNKNOWN until the real local or outsourced provider and relevant evidence are established.

For a required transparent/optical part, select either a purchased transparent component or identified transparent stock with traceable grade, thickness, optical acceptance criterion, supplier and suitable approved process. Transparent filament/resin, supplier colour labels and a visually clear print do not establish optical transmission; never route a transparency requirement through FDM.

## assess-selection
### Eliminate on mandatory constraints first

For every candidate and mandatory requirement, record a sourced criterion, method and one outcome:

- `PASS`: cited evidence and method show that this complete candidate route meets the mandatory criterion.
- `FAIL`: cited evidence and method show that the route cannot meet it; eliminate this candidate.
- `UNKNOWN`: evidence, capability, method, source conditions or material-driving use-case data are unresolved. Keep the candidate CONDITIONAL; do not call it feasible or recommend it as meeting the requirement.

Classify the whole candidate using the selection prose root (`cad-record/references/prose-contract.md#selection`): FEASIBLE has no mandatory FAIL or UNKNOWN; CONDITIONAL has at least one UNKNOWN and no FAIL; EXCLUDED has a FAIL. An UNKNOWN is not a pass. A score, preference, low price, availability or aesthetic cannot restore an EXCLUDED or CONDITIONAL candidate to feasible status. Do not assign preference scores to non-FEASIBLE candidates. Do not convert missing cost or data to zero.

If hard constraints leave one credible candidate, document the forced choice and its evidence/limits; do not manufacture a second option. Otherwise compare at least two credible complete candidate routes for consequential decisions.

### Compare the feasible routes

Use `hw/manufacturing/selection.yaml` and its candidate shape from the cad-record `selection` prose table. Compare with a concise table using these columns (add project-specific explanation in prose, not novel record fields):

| Candidate / complete route | Material family and exact grade; form/state | Use suitability; strength, stiffness, fatigue, creep, heat and wear | Accuracy, finish and dimensional stability | Equipment, secondary operations and inspection | Architecture, design changes and assembly interfaces | Cost and lead time at stated quantity | Maintenance, repair and replacement | Evidence confidence and open validation | Pros, cons and accepted tradeoffs |
|---|---|---|---|---|---|---|---|---|---|

Keep hard outcomes distinct from preferences; rank only FEASIBLE candidates against recorded preferences such as cost, convenience, mass or appearance. Explain the governing requirements, evidence basis, uncertainties and why the primary is preferred. For each candidate record economics with source-backed facts or explicit UNKNOWNs, limits, pros/cons/tradeoffs and a reversal condition. Never use an opaque composite score to conceal a mandatory outcome.

Give a primary route and, when useful and separately justified, a low-cost prototype route, end-use route and fallback. Name what changes the recommendation, such as greater measured load or temperature, longer duty/life, tighter fit, different provider capability or production quantity. State what a prototype can and cannot validate; a printed mock-up does not establish final strength, life or thermal performance.

### Evidence and approval boundary

For every property used to compare or calculate, identify the exact profile/property, source and reported statistic, grade/form/process state, test standard, temperature, conditioning, orientation and uncertainty to the extent the source establishes them. Keep unstated conditions UNKNOWN. Identify whether evidence is measured, supplier typical, standard minimum, estimated, calculated, assumed or a justified allowable. Density is not strength. Do not transfer bulk, cast or wrought properties to printed parts, another grade, form, orientation, moisture state or service condition.

A starter profile is a sourced comparison aid, not lot identification, process approval, physical qualification or an allowable. Confirm procurement identity and all relevant conditions. Typical supplier data and standard minima remain `typical_not_allowable`; structural calculations require an independently justified allowable with competent authority, conditions and derivation.

Store the assessed options and recommendation in the cad-record `selection` prose root. For a consequential choice, prepare the exact immutable proposal and a `SELECTION` decision with `PROPOSAL` binding. The lead checks the selection table, input inventory and evidence, then presents the proposal to the authorized human. A final selection requires an explicit user/authority approval recorded as a new immutable decision; selection work does not itself alter a part, requirement, interface, material library or design. Until approval, report WAITING FOR HUMAN DECISION and make no design change.