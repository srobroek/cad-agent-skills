# Install

Install exactly one variant into each product project. Do not install both `full/` and `lean/` into the same project: their skill and role
names are identical.

## Claude Code

Add the public GitHub marketplace at the scope you install at. For a project install, run both commands from the project root:

| Scope   | Command                                                                   |
| ------- | ------------------------------------------------------------------------- |
| project | `claude plugin marketplace add srobroek/cad-agent-skills --scope project` |
| user    | `claude plugin marketplace add srobroek/cad-agent-skills --scope user`    |

The project form declares the marketplace in the project's `.claude/settings.json`, beside the plugin it enables. Claude Code 2.1.318
still records it in the user's `known_marketplaces.json`, so `claude plugin marketplace list` shows it in other projects too, where the
plugin stays disabled.

Run the one command for your variant and scope:

| Variant | Scope   | Command                                                                        |
| ------- | ------- | ------------------------------------------------------------------------------ |
| full    | project | `claude plugin install cad-agent-skills@cad-agent-skills --scope project`      |
| full    | user    | `claude plugin install cad-agent-skills@cad-agent-skills --scope user`         |
| lean    | project | `claude plugin install cad-agent-skills-lean@cad-agent-skills --scope project` |
| lean    | user    | `claude plugin install cad-agent-skills-lean@cad-agent-skills --scope user`    |

Restart Claude Code or run `/reload-plugins`. Confirm the selected skills appear under the plugin's namespace, and invoke an agent from the
picker, e.g. `@agent-cad-agent-skills:cad-part-designer`.

## OMP

Add the public GitHub marketplace once:

```sh
omp plugin marketplace add srobroek/cad-agent-skills
```

Run the one command for your variant and scope:

| Variant | Scope   | Command                                                                     |
| ------- | ------- | --------------------------------------------------------------------------- |
| full    | project | `omp plugin install --scope project cad-agent-skills@cad-agent-skills`      |
| full    | user    | `omp plugin install --scope user cad-agent-skills@cad-agent-skills`         |
| lean    | project | `omp plugin install --scope project cad-agent-skills-lean@cad-agent-skills` |
| lean    | user    | `omp plugin install --scope user cad-agent-skills-lean@cad-agent-skills`    |

Restart after CLI install; after marketplace mutation in an interactive session, run `/reload-plugins`. Confirm the five agent roles and
skills are discovered.

## Copy installs

Codex, OpenCode and Kiro install by copying files from a checkout of this repository. Set `CAD_AGENT_SKILLS_SRC` to the checkout's absolute
root, run the host's block from the target project root, and edit its first lines before you run it: `variant=full` or `variant=lean`, and
the destination lines for your scope from the host's table. Each block runs in a subshell, so a failure never closes your shell. It checks
every target first and copies nothing when one exists, including a dangling symlink, then names that path. It copies the variant's
`LICENSE` and `NOTICE` into the installed `cad-flow/` skill directory.

## Codex CLI

| Scope   | Destination lines                                                 |
| ------- | ----------------------------------------------------------------- |
| project | `skills=.agents/skills` and `agents=.codex/agents`                |
| user    | `skills="$HOME/.agents/skills"` and `agents="$HOME/.codex/agents"` |

```sh
(
  set -eu
  variant=full
  skills=.agents/skills
  agents=.codex/agents
  src="${CAD_AGENT_SKILLS_SRC:?set CAD_AGENT_SKILLS_SRC to the repository root}/$variant"
  test -f "$src/LICENSE" && test -f "$src/NOTICE" || { printf 'no variant at %s\n' "$src" >&2; exit 1; }
  for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
    test ! -e "$skills/$name" && test ! -L "$skills/$name" || { printf 'exists: %s\n' "$skills/$name" >&2; exit 1; }
  done
  for source in "$src"/harness/codex/agents/*.toml; do
    target="$agents/${source##*/}"
    test ! -e "$target" && test ! -L "$target" || { printf 'exists: %s\n' "$target" >&2; exit 1; }
  done
  mkdir -p "$skills" "$agents"
  for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
    cp -R "$src/skills/$name" "$skills/$name"
  done
  cp "$src/LICENSE" "$src/NOTICE" "$skills/cad-flow/"
  for source in "$src"/harness/codex/agents/*.toml; do
    cp "$source" "$agents/${source##*/}"
  done
)
```

Restart if skills or agents are absent; confirm with `/skills` and custom-agent discovery. Codex custom agents inherit the parent session's
sandbox and approvals, and an agent file cannot narrow them, so every role keeps the shell and its write boundary is an instruction. The
reviewer's Codex rule is in [agent-roles](../full/skills/cad-flow/references/agent-roles.md#roles).

## OpenCode

| Scope   | Destination lines                                                                         |
| ------- | ----------------------------------------------------------------------------------------- |
| project | `skills=.opencode/skills` and `agents=.opencode/agents`                                   |
| user    | `skills="$HOME/.config/opencode/skills"` and `agents="$HOME/.config/opencode/agents"` |

```sh
(
  set -eu
  variant=full
  skills=.opencode/skills
  agents=.opencode/agents
  src="${CAD_AGENT_SKILLS_SRC:?set CAD_AGENT_SKILLS_SRC to the repository root}/$variant"
  test -f "$src/LICENSE" && test -f "$src/NOTICE" || { printf 'no variant at %s\n' "$src" >&2; exit 1; }
  for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
    test ! -e "$skills/$name" && test ! -L "$skills/$name" || { printf 'exists: %s\n' "$skills/$name" >&2; exit 1; }
  done
  for source in "$src"/harness/opencode/agents/*.md; do
    target="$agents/${source##*/}"
    test ! -e "$target" && test ! -L "$target" || { printf 'exists: %s\n' "$target" >&2; exit 1; }
  done
  mkdir -p "$skills" "$agents"
  for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
    cp -R "$src/skills/$name" "$skills/$name"
  done
  cp "$src/LICENSE" "$src/NOTICE" "$skills/cad-flow/"
  for source in "$src"/harness/opencode/agents/*.md; do
    cp "$source" "$agents/${source##*/}"
  done
)
```

Restart if files are not discovered. Each role profile carries its own `edit` and `bash` rules, which take precedence over the project
policy; they admit only the absolute-path commands the role contract runs.

## Kiro CLI V3

| Scope   | Destination lines                                                                       |
| ------- | --------------------------------------------------------------------------------------- |
| project | `scope=project`, `skills=.kiro/skills` and `agents=.kiro/agents`                        |
| user    | `scope=user`, `skills="$HOME/.kiro/skills"` and `agents="$HOME/.kiro/agents"`           |

```sh
(
  set -eu
  variant=full
  scope=project
  skills=.kiro/skills
  agents=.kiro/agents
  src="${CAD_AGENT_SKILLS_SRC:?set CAD_AGENT_SKILLS_SRC to the repository root}/$variant"
  test -f "$src/LICENSE" && test -f "$src/NOTICE" || { printf 'no variant at %s\n' "$src" >&2; exit 1; }
  for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
    test ! -e "$skills/$name" && test ! -L "$skills/$name" || { printf 'exists: %s\n' "$skills/$name" >&2; exit 1; }
  done
  for source in "$src"/harness/kiro/"$scope"/agents/*.json; do
    target="$agents/${source##*/}"
    test ! -e "$target" && test ! -L "$target" || { printf 'exists: %s\n' "$target" >&2; exit 1; }
  done
  mkdir -p "$skills" "$agents"
  for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
    cp -R "$src/skills/$name" "$skills/$name"
  done
  cp "$src/LICENSE" "$src/NOTICE" "$skills/cad-flow/"
  for source in "$src"/harness/kiro/"$scope"/agents/*.json; do
    cp "$source" "$agents/${source##*/}"
  done
)
```

Trust the workspace for project profiles. Kiro checks each worker tool call against both the role profile and the parent session's policy:
a call the parent policy does not allow prompts for approval, and a headless run denies it. Restart and verify skill and custom-agent
discovery.

## Variant and provenance

Do not copy a plugin manifest for Codex, OpenCode or Kiro. Installed `cad-flow/SKILL.md` frontmatter supplies plugin name/version. If a
discovered installed skill, role or schema differs, report a defect/open question; never patch installed files.

Each host's per-agent model override mechanism is summarized in [smoke.md](smoke.md#observed-smoke-findings), with links
to host documentation. This documents configuration only; it does not recommend a model or effort tier. Round-4 A/B
results do not support changing per-role defaults; see [limitations](limitations.md#model-routing-evidence).
