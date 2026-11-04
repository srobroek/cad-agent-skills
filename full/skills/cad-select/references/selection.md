# Use case and route selection

Owner: cad-select. Field and path authority belongs to cad-record. Read `cad-record/references/contract-index.md` and
`cad-record/references/record.md`; use the active brief's absolute skill/schema paths. Do not copy their rules into this reference.

## intake-use-case

### Establish intended use

Ask only questions that resolve this product's actual intended function and constraints. Cover:

- Product purpose; users; installation, operation, environment and service access.
- Load direction and magnitude, impact, vibration, cyclic and sustained duration; motion, speed, duty, friction, wear and required life.
- Temperature, heat sources, moisture, chemicals, UV, corrosion; accuracy, repeatability, finish and dimensional stability.
- Failure consequences, supervision, standards and safety constraints.
- Quantity, budget, lead time, repair, maintenance, available tools/operators and outsourcing.
- Whether appearance, fit, a limited supervised demonstration, a representative working prototype, production or safety-critical service is
  intended.

### Ask and finish intake

1. Ask every open template question in one message, grouped by domain; skip none. Each asked question carries exactly one question id, and
   each id covers one template question. Ask focused questions, not a generic questionnaire dump.
2. After the reply, record every answered fact as KNOWN with a `USER` source
   ([cad-record Facts](../../cad-record/references/record.md#facts)); record "I don't know", a refusal and each question the reply
   leaves out as a typed UNKNOWN with owner and question ([Scope of each unknown](#scope-of-each-unknown)); that question is done.
   Validate each changed file once ([cad-record record-bootstrap](../../cad-record/references/record.md#record-bootstrap), step 4).
3. Ask at most one follow-up batch, only for an answer that contradicts the record or is ambiguous; each follow-up gets a new question id.
   Record its reply as in step 2.
4. Go to `freeze-inputs`. Never ask an answered or recorded question again. The owner's name is recorded as the owner states it, and
   every decision from the owner's statements names that `actor.name`.

Resolving material-, route- and safety-driving questions before a final recommendation (below) governs INTENT, never the GROUND exit.

Before any material comparison, inspect the relevant existing product record and ask what is missing for those needs. Do not infer use from
CAD geometry or from the phrase “prototype”.

Capture **requirements**, **preferences**, **assumptions** and **open questions** separately in `hw/requirements.yaml`. Do not turn an
assumption into a requirement or use an unanswered question as evidence. Resolve questions whose answers can change a material, route,
architecture, safety conclusion or mandatory criterion before making a final recommendation; otherwise label the recommendation CONDITIONAL
and bound the unresolved choice ([Recommend](#recommend)).

Record the user-selected service class independently of rigor. Service class expresses use/maturity; rigor (`maker|pro|regulated`) expresses
documentation depth and never reduces required evidence. A request for a “prototype” is a question about service class, not a selection of
rigor. Do not infer either from CAD or example geometry. Record the class as a QUESTION decision whose statement is the owner's answer
naming it ([cad-flow gates — Decisions outside gates](../../cad-flow/references/gates.md#decisions-outside-gates)); `service.class` cites
that DR in this attempt.

### Scope of each unknown

Set `consequential` on each UNKNOWN intake fact once the service class and `release_scope` are recorded; until then every UNKNOWN is
consequential. Apart from the service class itself, GROUND records every unknown and none of them blocks it; the GROUND gate packet lists
each consequential one as a limit ([cad-flow gates — present-gate](../../cad-flow/references/gates.md#present-gate)). Each blocks only where
it falls due:

| The UNKNOWN feeds                                                                                                                                                                                                                                                                               | Record                                                                               | Blocks                                                                                                                                                                                                                                                                                                                  |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| a criterion, case or choice of a requirement kind the service class admits ([cad-verify check-plan — Evidence by service class](../../cad-verify/references/check-plan.md#evidence-by-service-class)) that `release_scope.exclusions` does not exclude, or any SAFETY or REGULATORY requirement | `consequential: true`, owner, question                                               | the first obligation that needs it: INTENT keeps every candidate it touches CONDITIONAL ([assess-selection](#assess-selection)); SPECIFY `requirements.completeness` is UNKNOWN `REQUIRED_UNKNOWN` while a criterion needs it; a check whose case needs it is UNKNOWN `REQUIRED_UNKNOWN`; else it stays a release limit |
| only an excluded requirement kind or a later scope                                                                                                                                                                                                                                              | `consequential: false`, owner, question, the exclusion as `reason`                   | nothing: an owned open question                                                                                                                                                                                                                                                                                         |
| an in-scope fact a later stage establishes: a capability envelope, material profile or supplier fact ([select-candidates](#select-candidates)), or a product behaviour such as creep, temperature rise or wear rate                                                                             | `consequential: true`, owner, question; `reason` names the stage that establishes it | that stage's obligation stays UNKNOWN until its check establishes the fact                                                                                                                                                                                                                                              |

Set each question's `material_to_choice` and `safety_relevant` by the same table.

## select-candidates

### Form whole candidate routes

Develop credible combinations before ranking. For each candidate, state its architecture and every relevant part's sourcing, exact material
profile or explicit UNKNOWN, and ordered route. Include secondary operations, finishing, joining, inspection, maintenance and replacement.
One product may mix made and purchased components, materials and operations.

Cover each physical part and disconnected component, then each functional contacting pair: relative motion/load, friction/wear,
temperature/environment, corrosion or galvanic contact, dimensional fit, lubrication, sealing, electrical or optical requirements where
relevant. State how the architecture changes load paths, interfaces, assembly, tools, service and retention. A material-only or
shaping-process-only comparison is incomplete.

Include credible additive, resin, subtractive, cutting/forming/joining, casting/moulding/extrusion, purchased and hybrid routes as relevant;
do not default to available equipment or to metal. If stated service exceeds credible evidence for a printed polymer, compare metal,
purchased hardware, a different route or revised architecture and identify the governing requirement. A polymer is appropriate when evidence
and complete checks support the actual requirements.

Do not invent machine, operator, supplier, material approval, work envelope, tool, calibration, tolerance, cost, availability, lead time or
life. Capability templates are field-and-guidance prompts, not capabilities. An unknown route capability remains UNKNOWN until the real
local or outsourced provider and relevant evidence are established.

For a required transparent/optical part, select either a purchased transparent component or identified transparent stock with traceable
grade, thickness, optical acceptance criterion, supplier and suitable approved process. Transparent filament/resin, supplier colour labels
and a visually clear print do not establish optical transmission; never route a transparency requirement through FDM.

## assess-selection

### Eliminate on mandatory constraints first

For every candidate and mandatory requirement, record a sourced criterion, method and one outcome:

- `PASS`: cited evidence and method show that this complete candidate route meets the mandatory criterion.
- `FAIL`: cited evidence and method show that the route cannot meet it; eliminate this candidate.
- `UNKNOWN`: evidence, capability, method, source conditions or material-driving use-case data are unresolved. Keep the candidate
  CONDITIONAL; do not call it feasible or recommend it ([Recommend](#recommend)).

Classify the whole candidate using the selection schema: FEASIBLE has no mandatory FAIL or UNKNOWN; CONDITIONAL has at least one UNKNOWN and
no FAIL; EXCLUDED has a FAIL. An UNKNOWN is not a pass. A score, preference, low price, availability or aesthetic cannot restore an EXCLUDED
or CONDITIONAL candidate to feasible status. Do not convert missing cost or data to zero.

If hard constraints leave one credible candidate, document the forced choice and its evidence/limits; do not manufacture a second option.
Otherwise compare at least two credible complete candidate routes for consequential decisions.

### Compare the feasible routes

Use `hw/manufacturing/selection.yaml` and its candidate shape from `selection.schema.json`. Compare with a concise table using these columns
(add project-specific explanation in prose, not novel record fields):

| Candidate / complete route | Material family and exact grade; form/state | Use suitability; strength, stiffness, fatigue, creep, heat and wear | Accuracy, finish and dimensional stability | Equipment, secondary operations and inspection | Architecture, design changes and assembly interfaces | Cost and lead time at stated quantity | Maintenance, repair and replacement | Evidence confidence and open validation | Pros, cons and accepted tradeoffs |
| -------------------------- | ------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------ | ---------------------------------------------- | ---------------------------------------------------- | ------------------------------------- | ----------------------------------- | --------------------------------------- | --------------------------------- |

Keep hard outcomes distinct from preferences. For each candidate record economics with source-backed facts or explicit UNKNOWNs, limits,
pros/cons/tradeoffs and a reversal condition. Never use an opaque composite score to conceal a mandatory outcome.

### Recommend

The candidates' classes decide the recommendation:

| Candidates | `recommendation` |
| --- | --- |
| at least one FEASIBLE | `status` CONDITIONAL or FINAL ([Ask and finish intake](#ask-and-finish-intake)); a `primary` route and, when useful and separately justified, a low-cost `prototype`, an `end_use` and a `fallback` route, each a FEASIBLE candidate ranked against recorded preferences such as cost, convenience, mass or appearance |
| none FEASIBLE | `status: NONE` with no `primary`, `prototype`, `end_use`, `fallback` or `decision`; `limits` names each blocking UNKNOWN with its owner and question as a gate limit ([cad-flow gates — present-gate](../../cad-flow/references/gates.md#present-gate) step 2) |

A CONDITIONAL candidate is presented labelled with its UNKNOWNs; it is never scored, ranked or named as a route. Where the release scope
needs a route, a NONE recommendation blocks INTENT: `selection.feasibility@product` is UNKNOWN `REQUIRED_UNKNOWN`, the
[stop procedure](../../cad-flow/references/stages.md#stop-procedure) reports the blocking UNKNOWNs and their owners as what unblocks it, and
they are resolved per [Ask and finish intake](#ask-and-finish-intake) before any route is recommended.
[cad-record validation — Semantic check](../../cad-record/references/validation.md#semantic-check) item 3 checks that every route field
names a FEASIBLE candidate and that a FINAL recommendation's decision binds this selection.

Explain the governing requirements, evidence basis, uncertainties and why the primary is preferred. Name what changes the recommendation,
such as greater measured load or temperature, longer duty/life, tighter fit, different provider capability or production quantity. State
what a prototype can and cannot validate; a printed mock-up does not establish final strength, life or thermal performance.

### Evidence and approval boundary

For every property used to compare or calculate, identify the exact profile/property, source and reported statistic, grade/form/process
state, test standard, temperature, conditioning, orientation and uncertainty to the extent the source establishes them. Keep unstated
conditions UNKNOWN. Identify whether evidence is measured, supplier typical, standard minimum, estimated, calculated, assumed or a justified
allowable. Density is not strength. Do not transfer bulk, cast or wrought properties to printed parts, another grade, form, orientation,
moisture state or service condition.

A starter profile is a sourced comparison aid, not lot identification, process approval, physical qualification or an allowable. Confirm
procurement identity and all relevant conditions. Typical supplier data and standard minima remain `typical_not_allowable`; which
calculations need an allowable, and which may use a typical value with its uncertainty, is
[cad-verify mass-loads — load.hand](../../cad-verify/references/mass-loads.md#loadhand) step 4.

Store the assessed options and recommendation in the cad-record `selection` root. Its `assessment` lists `inputs`, every `hw/` file the
assessment read with its sha256 (never `hw/manufacturing/selection.yaml` or a decision written from this assessment), `input_sha256`, the
set digest over `inputs` ([cad-record record — Inventory digest](../../cad-record/references/record.md#inventory-digest)), `method` and
`date`. The lead checks the root, input inventory and evidence; the INTENT gate packet presents the options, and its `choose` reply
approves the selection
([cad-flow gates — Decisions outside gates](../../cad-flow/references/gates.md#decisions-outside-gates)); no separate SELECTION approval is
asked. Selection work does not itself alter a part, requirement, interface, material library or design; until the gate is APPROVED, report
WAITING_FOR_USER and make no design change.
