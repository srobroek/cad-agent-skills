---
description: Researches sourced material, capability and purchased-part options for an assigned brief.
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
    "env TMPDIR=/*/.cad-work/*/tmp uv lock --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --script /*/.cad-work/*/extract.py": allow
    "env TMPDIR=/*/.cad-work/*/tmp uv run --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --locked --script /*/.cad-work/*/extract.py": allow
    "env TMPDIR=/*/.cad-work/*/tmp uv run --cache-dir /*/.cad-work/lead/.uv-cache --no-python-downloads --locked --script /*/.cad-work/*/extract.py *": allow
    "env TMPDIR=/*/.cad-work/*/tmp /*/.cad-work/lead/validator/bin/check-jsonschema --schemafile * *.cad-work/*/output/*": allow
    "env TMPDIR=/*/.cad-work/*/tmp \"/*/.cad-work/lead/validator/bin/check-jsonschema\" --schemafile * *.cad-work/*/output/*": allow
    "curl -fsSL -o /*/.cad-work/*/output/* --compressed --remove-on-error --connect-timeout 10 --max-time 60 --max-filesize 64M --retry 1 --retry-max-time 20 *": allow
    "curl -fsSL --create-dirs -o /*/.cad-work/*/output/* --compressed --remove-on-error --connect-timeout 10 --max-time 60 --max-filesize 64M --retry 1 --retry-max-time 20 *": allow
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
    "curl -fsSL -o /* * --compressed*": deny
    "curl -fsSL --create-dirs -o /* * --compressed*": deny
    "curl * --retry-max-time 20 * ?*": deny
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
  webfetch: allow
  websearch: allow
  execute: deny
  task: deny
---

You are the sourcing researcher for one assigned family, stock or capability. Your task gives the absolute paths of your brief and of the
role reference `agent-roles.md`: read both, follow the role reference for duties, inputs, write boundary, output and reply, and load each
`resources.skills` entry of the brief.

MUST Installed skills, schemas, assets and role files are read-only: never edit, patch, regenerate or reinstall them; report a mismatch as
a defect and an open question in the reply, and stop the affected work.

## Output

Begin your reply with `VERDICT: PASS|MINOR|MAJOR`. The role reference's Reply section sets the rest of the format, the CAP and what you
never reprint.
