# Vocabulary drift check

This distribution and the [cad-agent](https://github.com/srobroek/cad-agent) product project share vocabulary: stage names and order, gate
names, check-family names and agent role names. This page holds a read-only, developer-only check that compares them. It is a command you
paste into a shell, not a tracked script, because the repository ships no executable source and CI rejects tracked `scripts/`, `tools/`,
`hooks/` and source files ([AGENTS.md](../AGENTS.md#safety-and-scope)). Nothing here is part of an installed variant.

Run it by hand from this repository's root, with a local cad-agent checkout as the argument. It needs Python 3.9 or later and the standard
library only. It reads both trees and writes nothing. CI does not run it: it needs a second repository at a chosen ref, and the two
projects are edited independently, so a drift is information for the owners, not a merge gate.

## Sources compared

| Vocabulary     | This repository (`full/`, the owner)                                       | cad-agent checkout                                                              |
| -------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Stages         | `id` order of `stages` in `skills/cad-flow/assets/stages.yaml`, lowercased | `stage` definition in `skills/cad-record/schemas/product.schema.json`                 |
| Gates          | `gate` kinds in `stages.yaml`, and the stages whose gate is not `NONE`     | each kind as a whole word in `skills/cad-flow/`; `<NAME> gate` phrases there    |
| Check families | `check_family` enum in `skills/cad-record/schemas/common.schema.json` (54) | each family as a whole token in `skills/`, `agents/`, `harness/`, `docs/`, README |
| Agent roles    | `agents/*.md` file stems                                                   | `agents/*.md` file stems                                                        |

It also checks that `lean/` agrees with `full/` on stage order, gate kinds, the family list and agent names.

The cad-agent side has no machine-readable policy for gates or families, so those two comparisons are presence checks over its text. A
reported drift means "not found there", not "wrong". The stage comparison does not alias names: `FREEZE` against `design_freeze` is
reported as a difference so that an owner decides.

Exit status: 0 no drift, 1 drift found, 2 a missing path or wrong usage (message on stderr).

## Run

```sh
python3 - /path/to/cad-agent <<'PY'
import json, re, sys
from difflib import unified_diff
from pathlib import Path

def die(msg):
    print(msg, file=sys.stderr)
    return 2

if len(sys.argv) != 2:
    sys.exit(die("usage: python3 - <cad-agent checkout>"))
here, other = Path.cwd(), Path(sys.argv[1])
SK = "skills/cad-flow/assets/stages.yaml"
FAM = "skills/cad-record/schemas/common.schema.json"
PROD = "skills/cad-record/schemas/product.schema.json"
for p in (here / "full" / SK, here / "full" / FAM, other / PROD, other / "agents"):
    if not p.exists():
        sys.exit(die(f"missing {p}: run from the repo root and pass a cad-agent checkout"))

def stages(variant):
    ids, gates, cur = [], {}, None
    for line in (here / variant / SK).read_text().splitlines():
        if line.startswith(("check_order:", "lite_stages:")):
            break
        m = re.match(r"- id: (\w+)", line)
        if m:
            cur = m.group(1)
            ids.append(cur)
        m = re.match(r"  gate: (\w+)", line)
        if m and cur:
            gates[cur] = m.group(1)
    return ids, gates

def families(variant):
    d = json.loads((here / variant / FAM).read_text())
    return d["$defs"]["check_family"]["enum"]

def agents(base):
    return sorted(p.stem for p in (base / "agents").glob("*.md"))

def text(base, subdirs):
    out = []
    for sub in subdirs:
        root = base / sub
        files = [root] if root.is_file() else root.rglob("*") if root.exists() else []
        for f in files:
            if f.is_file() and f.suffix in {".md", ".json", ".py", ".yaml", ".yml", ".toml", ".csv"} and "__pycache__" not in f.parts:
                out.append(f.read_text(errors="replace"))
    return "\n".join(out)

drift = []

def report(kind, items):
    drift.extend(f"{kind}: {i}" for i in items)

ids, gates = stages("full")

# Stages: ordered comparison, no aliasing.
theirs = json.loads((other / PROD).read_text())["$defs"]["stage"]["enum"]
mine = [s.lower() for s in ids]
diff = unified_diff(mine, theirs, "skills(full)", "cad-agent", lineterm="", n=0)
report("stage order", [l for l in diff if not l.startswith(("---", "+++", "@@"))])

# Gates: kinds and gated stages.
flow = text(other, ["skills/cad-flow"])
kinds = sorted(set(gates.values()))
report("gate kind absent from cad-agent cad-flow", [k for k in kinds if not re.search(rf"\b{k}\b", flow)])
theirs_gate = {m.upper() for m in re.findall(r"\b([A-Za-z][A-Za-z_]*) gate\b", flow)}
mine_gated = {s for s, g in gates.items() if g != "NONE"}
report("gated stage without a '<NAME> gate' phrase in cad-agent", sorted(s for s in mine_gated if s not in theirs_gate))

# Check families.
fams = families("full")
corpus = text(other, ["skills", "agents", "harness", "docs", "README.md"])
report("check family absent from cad-agent", [f for f in fams if not re.search(rf"(?<![\w.]){re.escape(f)}(?!\w)", corpus)])

# Agent roles.
a, b = set(agents(here / "full")), set(agents(other))
report("agent only in skills", sorted(a - b))
report("agent only in cad-agent", sorted(b - a))

# lean must agree with full.
lids, lgates = stages("lean")
if lids != ids:
    report("lean stage order differs from full", [f"{ids} vs {lids}"])
if lgates != gates:
    report("lean gates differ from full", [f"{gates} vs {lgates}"])
if families("lean") != fams:
    report("lean check families differ from full", [])
if agents(here / "lean") != agents(here / "full"):
    report("lean agents differ from full", [])

print(f"stages {len(ids)}, gate kinds {kinds}, families {len(fams)}, agents {sorted(a)}")
print("\n".join(drift) if drift else "no drift")
sys.exit(1 if drift else 0)
PY
```

## Changing the check

Keep the sources in the table above in step with their owners ([AGENTS.md](../AGENTS.md#canonical-ownership)). Change the snippet and the
table together. Do not turn the snippet into a tracked file: the inventory step in CI would fail it.
