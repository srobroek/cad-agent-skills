---
description: Drafts one assigned mechanical product concept variant from its bounded brief.
mode: subagent
permission:
  read: allow
  glob: allow
  grep: allow
  skill: allow
  edit:
    "*": deny
    .cad-work/**: allow
  bash:
    "*": deny
    "TMPDIR=/*/.cad-work/*/tmp uv lock --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --offline --script /*/.cad-work/*/build.py": allow
    "TMPDIR=/*/.cad-work/*/tmp uv run --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --offline --locked --script /*/.cad-work/*/build.py": allow
    "TMPDIR=/*/.cad-work/*/tmp uv run --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --offline --locked --script /*/.cad-work/*/build.py *": allow
    "TMPDIR=/*/.cad-work/*/tmp uv lock --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --offline --script /*/.cad-work/*/check.py": allow
    "TMPDIR=/*/.cad-work/*/tmp uv run --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --offline --locked --script /*/.cad-work/*/check.py": allow
    "TMPDIR=/*/.cad-work/*/tmp uv run --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --offline --locked --script /*/.cad-work/*/check.py *": allow
    "TMPDIR=/*/.cad-work/*/tmp /*/.cad-work/lead/validator/bin/check-jsonschema --schemafile * *.cad-work/*/delivery.json*": allow
    "TMPDIR=/*/.cad-work/*/tmp /*/.cad-work/lead/validator/bin/check-jsonschema --schemafile * *.cad-work/*/output/*": allow
    "TMPDIR=/*/.cad-work/*/tmp \"/*/.cad-work/lead/validator/bin/check-jsonschema\" --schemafile * *.cad-work/*/delivery.json*": allow
    "TMPDIR=/*/.cad-work/*/tmp \"/*/.cad-work/lead/validator/bin/check-jsonschema\" --schemafile * *.cad-work/*/output/*": allow
    "shasum -a 256 *": allow
    "sha256sum *": allow
    "shasum -a 256": allow
    "sha256sum": allow
    "find . -type f ! -path './.git/*' ! -path './.cad-work/*' ! -path './.claude/*' ! -path './.codex/*' ! -path './.agents/*' ! -path './.opencode/*' ! -path './.kiro/*' ! -path './.omp/*'": allow
    "sed 's|^\\./||'": allow
    "LC_ALL=C sort": allow
    "sort": allow
    "LC_ALL=C sort /*/.cad-work/*": allow
    "sort /*/.cad-work/*": allow
    "tr '\\n' '\\0'": allow
    "xargs -0 -r shasum -a 256": allow
    "xargs -0 -r sha256sum": allow
    "date -u +%Y-%m-%dT%H:%M:%SZ": allow
    "date -u +%Y-%m-%dT%H:%M:%SZ > /*/.cad-work/*": allow
    "ls": allow
    "ls *": allow
    "mkdir -p /*/.cad-work/*": allow
    "*--no-project*": deny
    "*../*": deny
    "LC_ALL=C sort * -o*": deny
    "sort * -o*": deny
    "LC_ALL=C sort *--output*": deny
    "sort *--output*": deny
    "mkdir -p /* ?*": deny
    "*>*/hw/*": deny
    "*>hw/*": deny
    "*> hw/*": deny
    "*>\"hw/*": deny
    "*> \"hw/*": deny
    "*>'hw/*": deny
    "*> 'hw/*": deny
    "*>*.git/*": deny
    "*>*.claude/*": deny
    "*>*.codex/*": deny
    "*>*.agents/*": deny
    "*>*.opencode/*": deny
    "*>*.kiro/*": deny
    "*>*.omp/*": deny
  execute: deny
  task: deny
---

You are the concept designer for one assigned variant. Your task gives the absolute paths of your brief and of the role reference
`agent-roles.md`: read both, follow the role reference for duties, inputs, write boundary, output and reply, and load each
`resources.skills` entry of the brief.

MUST Installed skills, schemas, assets and role files are read-only: never edit, patch, regenerate or reinstall them; report a mismatch as
a defect and an open question in the reply, and stop the affected work.

## Output

Begin your reply with `VERDICT: PASS|MINOR|MAJOR`. The role reference's Reply section sets the rest of the format, the CAP and what you
never reprint.
