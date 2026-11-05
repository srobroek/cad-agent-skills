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
2. After the reply, write its whole record in one batch, before any follow-up, and validate each changed file once
   ([cad-record record-bootstrap](../../cad-record/references/record.md#record-bootstrap), step 4):
   - Every fact the reply states, including a fact no question asked, as KNOWN with a `USER` source
     ([cad-record Facts](../../cad-record/references/record.md#facts)), and every intake field settled or left open by
     [Settle each intake field](#settle-each-intake-field).
   - Every requirement, preference and assumption those facts state
     ([Write criteria a stage can evidence](#write-criteria-a-stage-can-evidence)).
   - Each field that table leaves open, as a typed UNKNOWN with owner and a new question id
     ([Scope of each unknown](#scope-of-each-unknown)); the template question the reply answered is RESOLVED.
3. Ask at most one follow-up batch, only for an answer that contradicts the record or is ambiguous; each follow-up gets a new question id.
   Record its reply as in step 2. An answer in words without a number, such as "for hours", is its value, never ambiguous. A topic
   `release_scope.exclusions` excludes that no intake field holds, such as a protection, access or fit claim the request or a reply names,
   gets no question and no UNKNOWN: the exclusion is its whole record. An exclusion never reopens an intake field: step 2 settles or leaves
   open a field on an excluded topic like any other.
4. Go to `freeze-inputs`. Never ask an answered or recorded question again. The owner's name is recorded as the owner states it, and
   every decision from the owner's statements names that `actor.name`.

Before any material comparison, inspect the relevant existing product record and ask what is missing for those needs. Do not infer use from
CAD geometry or from the phrase “prototype”.

Capture **requirements**, **preferences**, **assumptions** and **open questions** separately in `hw/requirements.yaml`. GROUND writes the
file whole (step 2) and no later stage writes a byte of it: a requirement, criterion, preference, assumption or question to add, change or
resolve after the GROUND gate, an owner's later answer included, follows
[cad-flow change impact — Procedure](../../cad-flow/references/change-impact.md#procedure). A question a later stage settles from a
retained source stays OPEN there; the retained fact answers it in its own file. Do not turn an assumption into a requirement or use an
unanswered question as evidence. At INTENT, a question still open whose answer can change a material, route, architecture, safety
conclusion or mandatory criterion keeps each candidate whose hard outcome it feeds CONDITIONAL ([assess-selection](#assess-selection)); at
GROUND it blocks nothing ([Scope of each unknown](#scope-of-each-unknown)).

Record the user-selected service class independently of rigor. Service class expresses use/maturity; rigor (`maker|pro|regulated`) expresses
documentation depth and never reduces required evidence. A request for a “prototype” is a question about service class, not a selection of
rigor. Do not infer either from CAD or example geometry. Record the class as a QUESTION decision whose statement is the owner's answer
naming it ([cad-flow gates — Decisions outside gates](../../cad-flow/references/gates.md#decisions-outside-gates)); `service.class` cites
that DR in this attempt.

#### Settle each intake field

Read each answer against every intake field its template question names, in its own words and in the context of the whole reply, and
record the field by the first row that matches. A statement on the shared subject of several fields, such as upkeep for maintenance and
repair, answers each of them. A field an answer settles is never an UNKNOWN or a question, also when `release_scope.exclusions` excludes
its topic.

| The answer, for this field | Record |
| --- | --- |
| the owner does not know or will not say, or names the field as present without its value, such as a load of unstated force | typed UNKNOWN ([Scope of each unknown](#scope-of-each-unknown)) |
| it asks nothing of the product, in any words: "none", "not needed", "not applicable" or "no requirement"; the conditions it depends on are absent, as dry air without chemicals is for corrosion; a motion the user makes by hand when wanted, which has no speed requirement; the owner's own repair stated as the whole upkeep, which leaves no other maintenance | settled: KNOWN with the answer's words as value and the answer as `USER` source, or NOT_APPLICABLE with `reason` quoting the answer and its question id and `reviewed_by` the owner who gave it ([cad-record Facts](../../cad-record/references/record.md#facts)); a field that holds a quantity takes NOT_APPLICABLE |
| its value in any words: a number, range, description or frequency | KNOWN with the answer as `USER` source. A quantity it states only in words, such as "for hours", stays in those words in a text field or the item's `note`, and an optional quantity field stays absent; never a question for a number |
| nothing about the field | typed UNKNOWN ([Scope of each unknown](#scope-of-each-unknown)) |

### Write criteria a stage can evidence

Write each constraint the owner states that a facet the service class admits can evidence
([cad-verify check-plan — Evidence by service class](../../cad-verify/references/check-plan.md#evidence-by-service-class)) as a whole
requirement by [cad-verify check-plan — specify-requirements](../../cad-verify/references/check-plan.md#specify-requirements) steps 1-6,
each criterion a property such a facet evidences:

- A finish or appearance requirement, when the class admits no PHYSICALLY_VALIDATED facet: a finish condition of the process plan (base
  facet PROCESS_PLANNED), never an achieved surface quality. Word it as planned and give it a plan-property criterion such as
  `support_on_visible_faces EQUALS false`.
- The owner's equipment, materials on hand and outsourcing: one MANUFACTURING requirement whose criteria are route properties in this form.
  `make_capability_profile EQUALS <profile id>` (`ONE_OF` for several machines or providers) names the capability profile GROUND writes in
  `hw/manufacturing/capabilities.yaml` from the owner's statement, each subject the owner leaves unstated a typed UNKNOWN
  ([Scope of each unknown](#scope-of-each-unknown)). `make_material ONE_OF [<material profile ids>]` names the owner's materials on hand
  by identity. Its facets are PROCESS_PLANNED, which `capability.route` evidences, and GEOMETRICALLY_MANUFACTURABLE, which the process
  obligations of each part made on that profile evidence
  ([cad-verify process rules](../../cad-verify/references/process-checks.md#rules-for-every-process-family) step 4); `due` PARTS.
- A material criterion settles at INTENT by identity: a candidate whose material profile is one `make_material` names passes it. Whether the
  machine runs that material in the consumable form the owner holds, such as a filament diameter or a stock size, is
  [cad-verify process-checks — capability.route](../../cad-verify/references/process-checks.md#capabilityroute) step 2 at PARTS. Until
  then the consumable form, like a process setting such as supports or drying, is no intake field and no subject of its own: no UNKNOWN or
  question records it at GROUND or INTENT, and it is never an INTENT outcome, a reason to keep a candidate CONDITIONAL or an answer a gate
  packet invites. Each candidate states it as a limit due at PARTS.
- Every quantity the owner states is recorded typed, never dropped: in the intake fact its template question fills, where one does
  ([Settle each intake field](#settle-each-intake-field)). A quantity that sizes or bounds the product, such as a dimension of an item it
  must hold, receive, fit within or clear, a footprint or a travel, is also a criterion of the requirement it constrains, each stated
  dimension its own criterion, and never an assumption instead. Such a dimension on the model has the CAD_VALID facet in every class, so
  where `release_scope.exclusions` excludes fit it stays a form dimension whose `conditions` say that no fit is claimed. A quantity that
  bounds a load the product carries, such as the heaviest item it holds, is a criterion where a facet the class admits evidences that load;
  otherwise, like a value the owner gives as an estimate or does not commit to, it is an assumption with `bounds`, `owner` and
  `validation`.
- Quantity, budget and lead time: `product.yaml` intake facts, never a requirement, because no facet evidences them; each candidate's
  `economics` states its figures against them ([Compare the feasible routes](#compare-the-feasible-routes)).

### Scope of each unknown

This table types each field [Settle each intake field](#settle-each-intake-field) leaves open and each fact sourcing or a check
establishes; it never reopens a field an answer settles. Set `consequential` on each UNKNOWN intake fact once the service class and
`release_scope` are recorded; until then every UNKNOWN is consequential. Apart from the service class itself, GROUND records every unknown
and none of them blocks it; the GROUND gate packet lists each consequential one as a limit
([cad-flow gates — present-gate](../../cad-flow/references/gates.md#present-gate)). Each blocks only where it falls due:

| The UNKNOWN is | Record | Blocks |
| --- | --- | --- |
| an owner's answer that a criterion, case or choice needs, for a requirement kind the service class admits ([cad-verify check-plan — Evidence by service class](../../cad-verify/references/check-plan.md#evidence-by-service-class)) that `release_scope.exclusions` does not exclude, or for any SAFETY or REGULATORY requirement | `consequential: true`, owner, question | the first obligation that needs it: each INTENT hard outcome it feeds is UNKNOWN and keeps its candidate CONDITIONAL ([assess-selection](#assess-selection)); SPECIFY `requirements.completeness` is UNKNOWN `REQUIRED_UNKNOWN` while a criterion needs it; a check whose case needs it is UNKNOWN `REQUIRED_UNKNOWN`; else it stays a release limit |
| an answer only an excluded requirement kind or a later scope needs | `consequential: false`, owner, question, the exclusion as `reason` | nothing: an owned open question |
| a fact sourcing or a check establishes, not an owner's answer: a capability envelope, material profile or supplier fact ([select-candidates](#select-candidates)), or a product behaviour such as creep, temperature rise or wear rate | `consequential: true`, owner, question; `reason` names the stage that establishes it | only the first obligation that uses it as evidence: INTENT only through a criterion INTENT settles ([assess-selection](#assess-selection)), whose hard outcome stays UNKNOWN; a later criterion that needs it is PLANNED at INTENT, and its own stage's obligation stays UNKNOWN until a check establishes the fact; the owner is asked only after sourcing returns it UNKNOWN |

An UNKNOWN takes the first row when the owner can answer it and the third when sourcing or a check establishes it, never both. INTENT is
blocked only by an UNKNOWN hard outcome: a criterion whose value the owner has not given, or a criterion INTENT settles whose evidence is
missing. A fact that only a later criterion or case uses, such as a printer envelope or creep data, blocks nothing at INTENT: it is a limit
there and falls due at the stage whose obligation uses it.

Record each fact sourcing establishes (third row) as its own typed UNKNOWN with its own question, even when one source may settle several;
never fold two such subjects into one UNKNOWN or question. A capability profile the owner names has four such subjects, each its own
UNKNOWN when the owner leaves it unstated: machine identity, operator, envelope and material approvals. Its consumable form and process
settings are none of them ([Write criteria a stage can evidence](#write-criteria-a-stage-can-evidence)).

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
local or outsourced provider and relevant evidence are established. List each supplier, material and capability fact a candidate needs as a
fact to source; the cad-sourcing-researcher units establish it first. The owner is asked only for an owner-held fact, such as an approval
or the identity of an owned machine, or for a fact a unit returned UNKNOWN.

For a required transparent/optical part, select either a purchased transparent component or identified transparent stock with traceable
grade, thickness, optical acceptance criterion, supplier and suitable approved process. Transparent filament/resin, supplier colour labels
and a visually clear print do not establish optical transmission; never route a transparency requirement through FDM.

## assess-selection

### Eliminate on mandatory constraints first

For every candidate, record one outcome per criterion of each mandatory requirement, with the criterion and a method. INTENT assesses a
criterion only when INTENT holds its evidence:

| The criterion constrains | Outcome at INTENT |
| --- | --- |
| the candidate's own route: sourcing, route steps, capability profile or provider, material profile, purchase identity, or a property a retained source states for one of them | `PASS`, `FAIL` or `UNKNOWN` |
| what only a later stage evidences: model geometry, a part's process plan (orientation, support contact or slicing), an analysis or a made article | `PLANNED` with `due` the first stage whose facet evidences it ([cad-verify check-plan — specify-requirements](../../cad-verify/references/check-plan.md#specify-requirements) step 5); `FAIL` when INTENT evidence already shows that the route cannot meet it |

- `PASS`: cited evidence and method show that this complete candidate route meets the criterion.
- `FAIL`: cited evidence and method show that the route cannot meet it; eliminate this candidate.
- `UNKNOWN`: the evidence, capability, method or source conditions of an INTENT criterion are unresolved, or the criterion itself is UNKNOWN
  ([Scope of each unknown](#scope-of-each-unknown)). `question` names the OPEN question in `hw/requirements.yaml` whose answer settles it,
  the one GROUND wrote for that fact. INTENT writes no question ([Ask and finish intake](#ask-and-finish-intake)): an UNKNOWN that no OPEN
  question covers is a change to the GROUND record
  ([cad-flow change impact — Procedure](../../cad-flow/references/change-impact.md#procedure)). Keep the candidate CONDITIONAL; do not
  call it feasible or recommend it ([Recommend](#recommend)).
- `PLANNED`: a KNOWN later-stage criterion, with `due` and a `reason` naming the evidence that stage produces. PLANNED is never a pass: the
  requirement's obligation at `due` settles it, and a FAIL or UNKNOWN there blocks that stage
  ([cad-flow stages — Stop procedure](../../cad-flow/references/stages.md#stop-procedure)). Under a class without PHYSICALLY_VALIDATED, a
  finish or appearance criterion is the planned condition its PROCESS_PLANNED facet evidences, never an achieved surface
  ([Write criteria a stage can evidence](#write-criteria-a-stage-can-evidence)). A criterion that no facet the service class admits
  evidences has no `due` stage: its outcome is UNKNOWN.

A route that needs a material, machine, tool or provider the owner neither has nor has approved acquiring is never FEASIBLE: its outcome on
the owner's equipment requirement is FAIL where the owner's statements exclude it, else UNKNOWN with the owner's question. A material the
owner names by its grade is one the owner has; its consumable form falls due at PARTS
([Write criteria a stage can evidence](#write-criteria-a-stage-can-evidence)).

Classify the whole candidate using the selection prose root (`cad-record/references/prose-contract.md#selection`): FEASIBLE has no FAIL or
UNKNOWN outcome and may hold PLANNED ones; CONDITIONAL has at least one UNKNOWN and no FAIL; EXCLUDED has a FAIL. Neither UNKNOWN nor
PLANNED is a pass. A score, preference, low price, availability or aesthetic cannot restore an EXCLUDED or CONDITIONAL candidate to feasible
status. Do not convert missing cost or data to zero.

If hard constraints leave one credible candidate, document the forced choice and its evidence/limits; do not manufacture a second option.
Otherwise compare at least two credible complete candidate routes for consequential decisions.

### Compare the feasible routes

Use `hw/manufacturing/selection.yaml` and its candidate shape from the cad-record `selection` prose table. Compare with a concise table
using these columns (add project-specific explanation in prose, not novel record fields):

| Candidate / complete route | Material family and exact grade; form/state | Use suitability; strength, stiffness, fatigue, creep, heat and wear | Accuracy, finish and dimensional stability | Equipment, secondary operations and inspection | Architecture, design changes and assembly interfaces | Cost and lead time at stated quantity | Maintenance, repair and replacement | Evidence confidence and open validation | Pros, cons and accepted tradeoffs |
| -------------------------- | ------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------ | ---------------------------------------------- | ---------------------------------------------------- | ------------------------------------- | ----------------------------------- | --------------------------------------- | --------------------------------- |

Keep hard outcomes distinct from preferences. For each candidate record economics with source-backed facts or explicit UNKNOWNs, stated
against the `product.yaml` quantity, budget and lead time; a cost or time that slicing or geometry decides stays UNKNOWN, and an UNKNOWN or
exceeded figure is a candidate limit. Record limits, pros/cons/tradeoffs and a reversal condition. Never use an opaque composite score to
conceal a mandatory outcome.

### Recommend

The candidates' classes decide the recommendation:

| Candidates | `recommendation` |
| --- | --- |
| at least one FEASIBLE | `status` CONDITIONAL, since FINAL needs `decision`, the APPROVED INTENT gate DR, which INTENT stage work never has ([cad-record validation — Semantic check](../../cad-record/references/validation.md#semantic-check) item 3); a `primary` route and, when useful and separately justified, a low-cost `prototype`, an `end_use` and a `fallback` route, each a FEASIBLE candidate ranked against recorded preferences such as cost, convenience, mass or appearance; `limits` names each PLANNED outcome of a named route with its `due` stage as a gate limit ([cad-flow gates — present-gate](../../cad-flow/references/gates.md#present-gate) step 2) |
| none FEASIBLE | `status: NONE` with no `primary`, `prototype`, `end_use`, `fallback` or `decision`; `limits` names each blocking UNKNOWN by its `question` id and owner as a gate limit ([cad-flow gates — present-gate](../../cad-flow/references/gates.md#present-gate) step 2) |

A CONDITIONAL candidate is presented labelled with its UNKNOWNs; it is never scored, ranked or named as a route. Where the release scope
needs a route, a NONE recommendation blocks INTENT: `selection.feasibility@product` is UNKNOWN `REQUIRED_UNKNOWN`. Record the NONE
selection and finish the attempt to its BLOCKED verdict, as the [stop procedure](../../cad-flow/references/stages.md#stop-procedure)
requires; its report asks each blocking question by its id, in one message, as [Ask and finish intake](#ask-and-finish-intake) step 1
asks. The reply, "I don't know" included, changes the GROUND record: record it per step 2 through
[cad-flow change impact — Procedure](../../cad-flow/references/change-impact.md#procedure); it resolves those questions. The INTENT
attempt after that reopen follows even when it can only record NONE again; its report names the decision that unblocks each remaining
UNKNOWN, such as a requirement change or an approved acquisition, and never asks a recorded question again.
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

Store the assessed options and recommendation in the cad-record `selection` prose root. Its `assessment` lists `inputs`, every `hw/` file
the assessment read with its sha256 (never `hw/manufacturing/selection.yaml` or a decision written from this assessment), `input_sha256`,
the set digest over `inputs` ([cad-record record — Inventory digest](../../cad-record/references/record.md#inventory-digest)), `method` and
`date`. The lead checks the selection table, input inventory and evidence; the INTENT gate packet presents the options, and its `choose`
reply approves the selection
([cad-flow gates — Decisions outside gates](../../cad-flow/references/gates.md#decisions-outside-gates)); no separate SELECTION approval is
asked. Selection work does not itself alter a part, requirement, interface, material library or design; until the gate is APPROVED, report
WAITING_FOR_USER and make no design change.
