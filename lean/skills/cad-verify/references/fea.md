# FEA

Owner: cad-verify. Method for `load.fea`. FEA cases are the [cad-record check-plan prose root](../../cad-record/references/prose-contract.md#check-plan) (solver, loads, boundary
conditions, material, mesh, convergence, `units: mm-N-MPa`, limits).

## load.fea

Per FEA case:

1. The solver is installed, found by preflight at the case's exact `solver.version`, and approved by the user; its
   own documentation (for CalculiX 2.22 or 2.23, the ccx manual) is the method source. A mesher such as Gmsh is not a
   solver. No approved solver → UNKNOWN `UNSUPPORTED_METHOD`; an alternative method is a change to the case, never a
   substitute.
2. Geometry is the version's STEP in the case state; a pose comes from the hash-checked `motion.sweep` artifact,
   never re-posed by the analysis.
3. The checker writes its own mesh and deck in scratch; every input is in mm, N and MPa.
4. Material values follow references/mass-loads.md `load.hand` step 4.
5. Loads, boundary conditions and contacts exactly as the case states; a changed one is a change to the case.
6. Convergence: at least the case's `levels` refinements; the case's `quantity` changes by less than its `tolerance`
   between the last two. Repeat noise of one mesh (a 0.2–0.4 % spread from mesher optimisation) is no convergence.
7. Equilibrium: reaction sums equal the applied loads within a stated tolerance.
8. Name the evaluation region; exclude a singular point (re-entrant corner, point load) only with that reason.
9. Read the solver version from its own output and record it with the deck, mesh statistics, log, results and
   convergence table as artifacts.
10. Linear static analysis supports static stress and displacement criteria only: no fatigue, creep, wear,
    buckling, life or physical claim, and a yield safety factor is no certification margin. The facet is
    SIMULATION_SUPPORTED.
