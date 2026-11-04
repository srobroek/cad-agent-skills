# Engineering principles

## Service class and rigor

A product's **service class** states its intended use and sets the required
evidence. It describes maturity and use, from appearance concepts and fit/assembly
through functional demo, working prototype, production and safety-critical use.
**Rigor** (`maker`, `pro`, `regulated`) sets documentation depth. It does not
lower service-class evidence requirements. A request for a “prototype” prompts
service-class intake, not a rigor choice. Production and safety-critical scope
needs a competent authority and supplied criteria, allowables and evidence.
`UNKNOWN` remains unknown; it cannot satisfy a PASS condition.

## Evidence classes

Results name method, scope, criterion, evidence, assumptions and uncertainty.
The five facets are distinct: `CAD_VALID`, `GEOMETRICALLY_MANUFACTURABLE`,
`PROCESS_PLANNED`, `SIMULATION_SUPPORTED` and `PHYSICALLY_VALIDATED`. A result
in one facet cannot stand in for another. A drawing, preview, nominal pose,
typical datasheet value or successful model execution does not verify another
facet.

`PASS`, `FAIL`, `UNKNOWN`, `NOT_APPLICABLE`, `WAIVED` and `PLANNED` are distinct
outcomes. Incomplete, stale or unrun required evidence is `UNKNOWN` or blocked,
never PASS. `NOT_APPLICABLE` needs a reason and review. A waiver keeps missing
evidence visible and qualified. It leaves the underlying result unchanged.

## Physical evidence and the release ceiling

Geometry checks and simulations give bounded evidence about modeled inputs and
methods, not service performance. They cannot establish material lot, process
variation, manufactured fit, actual load, wear, fatigue, creep, environment or
workmanship. If a service class or requirement needs a physical test, perform
it on the relevant physical item and retain the evidence. Without that test,
the result remains UNKNOWN or PLANNED and release is withheld. A competent
authority may permit a narrowly scoped, supervised, expiring non-safety trial
under release policy. Production and safety-critical scope cannot waive
required physical or authority evidence. A demo may end in a concept-definition
package or a named stop. It cannot imply a functional pass without evidence.

## Manufacturing intent

A material/process route names the part, component and operation. Record sourced
grade, product form, condition, orientation, setup, support, finishing,
inspection and capability as relevant. Profiles provide sourced starting
points; they do not qualify purchased lots or processes. Supplier-typical values
and standard minima are not design allowables. Keep unknown conditions unknown.
A transparency requirement uses purchased transparent material or identified
transparent stock with traceable purchase criteria. FDM filament or a
transparent-looking print is not qualified optical material. Unsupported or
unplanned manufacturing intent remains a blocker, not an inferred capability.

## Transparency rule

Records and evidence show scope, source, method, limits, assumptions,
uncertainty, unknowns, waivers and decision authority. State which requirements
and facets were assessed and what was not assessed. State what evidence could
change an outcome. Separate sourced facts and quotations from estimates and
assumptions. Do not claim repeatability beyond fixed inputs and methods.
Generated code, geometry and cross-host CAD bytes need not be identical. Prompts
and host permissions are not an operating-system security boundary.
