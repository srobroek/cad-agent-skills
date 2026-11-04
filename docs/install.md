# Install

Install exactly one variant into each product project. Do not install both `full/` and `lean/` into the same project: their skill and role names are identical.

Set `CAD_AGENT_SKILLS_SRC` to the absolute root of this checked-out `cad-agent-skills` repository before running native copy commands. These examples copy the full variant; for lean, replace `/full/` with `/lean/` in all source paths.

## Claude Code

Add the public GitHub marketplace once:

```sh
claude plugin marketplace add srobroek/cad-agent-skills
```

Choose one variant and one scope:

```sh
claude plugin install cad-agent-skills@cad-agent-skills --scope user
claude plugin install cad-agent-skills@cad-agent-skills --scope project
claude plugin install cad-agent-skills-lean@cad-agent-skills --scope user
claude plugin install cad-agent-skills-lean@cad-agent-skills --scope project
```

Restart Claude Code or run `/reload-plugins`. Confirm the selected skills appear under the plugin's namespace, and invoke an agent from the picker, e.g. `@agent-cad-agent-skills:cad-part-designer`.

## OMP

Add the public GitHub marketplace once:

```sh
omp plugin marketplace add srobroek/cad-agent-skills
```

Choose one variant and one scope:

```sh
omp plugin install --scope user cad-agent-skills@cad-agent-skills
omp plugin install --scope project cad-agent-skills@cad-agent-skills
omp plugin install --scope user cad-agent-skills-lean@cad-agent-skills
omp plugin install --scope project cad-agent-skills-lean@cad-agent-skills
```

Restart after CLI install; after marketplace mutation in an interactive session, run `/reload-plugins`. Confirm the five agent roles and skills are discovered.

## Codex CLI

From the target project root, install the full variant at project scope. Every path guard refuses existing files or directories, including dangling symlinks.

```sh
mkdir -p .agents/skills .codex/agents
for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
  target=".agents/skills/$name"
  test ! -e "$target" && test ! -L "$target" || exit 1
  cp -R "$CAD_AGENT_SKILLS_SRC/full/skills/$name" "$target"
done
for source in "$CAD_AGENT_SKILLS_SRC"/full/harness/codex/agents/*.toml; do
  target=".codex/agents/${source##*/}"
  test ! -e "$target" && test ! -L "$target" || exit 1
  cp "$source" "$target"
done
```

For user scope, replace `.agents/skills` with `$HOME/.agents/skills` and `.codex/agents` with `$HOME/.codex/agents` in mkdir, guards and targets. Restart if skills or agents are absent; confirm with `/skills` and custom-agent discovery. DFM disables its shell tool. All Codex custom agents inherit the parent sandbox/approval; the permission-template explains there is no per-role filesystem allowlist.

## OpenCode

From the target project root, install full at project scope, refusing any pre-existing target:

```sh
mkdir -p .opencode/skills .opencode/agents
for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
  target=".opencode/skills/$name"
  test ! -e "$target" && test ! -L "$target" || exit 1
  cp -R "$CAD_AGENT_SKILLS_SRC/full/skills/$name" "$target"
done
for source in "$CAD_AGENT_SKILLS_SRC"/full/harness/opencode/agents/*.md; do
  target=".opencode/agents/${source##*/}"
  test ! -e "$target" && test ! -L "$target" || exit 1
  cp "$source" "$target"
done
```

For user scope, replace `.opencode/skills` with `$HOME/.config/opencode/skills`, and `.opencode/agents` with `$HOME/.config/opencode/agents` throughout mkdir, guards and target assignments. Restart if files are not discovered. The permission-template is a starting point for edit boundaries; preserve the user's project policy for shell/tools.

## Kiro CLI V3

From the target project root, install full project skill and agent files, refusing existing destinations:

```sh
mkdir -p .kiro/skills .kiro/agents
for name in cad-flow cad-record cad-select cad-design cad-model cad-verify cad-review cad-release; do
  target=".kiro/skills/$name"
  test ! -e "$target" && test ! -L "$target" || exit 1
  cp -R "$CAD_AGENT_SKILLS_SRC/full/skills/$name" "$target"
done
for source in "$CAD_AGENT_SKILLS_SRC"/full/harness/kiro/project/agents/*.json; do
  target=".kiro/agents/${source##*/}"
  test ! -e "$target" && test ! -L "$target" || exit 1
  cp "$source" "$target"
done
```

For user scope, replace project paths `.kiro/skills` and `.kiro/agents` with `$HOME/.kiro/skills` and `$HOME/.kiro/agents`, and `harness/kiro/project/agents` with `harness/kiro/user/agents`; apply every replacement in mkdir, guards, targets and source path. Trust the workspace for project profiles. Kiro permissions are user-scoped: project at `~/.kiro/workspace-roots/<hash>/permissions.yaml`, user at `~/.kiro/settings/permissions.yaml`. Merge the matching YAML example with the existing policy. Profile `file://` resources use the absolute `role_reference` and `worker-brief` paths carried by the brief; skill `skill://` URIs use the standard project/home skill roots. Kiro hardcoded prompts and parent policies still apply; these templates are not OS isolation. Restart and verify skill and custom-agent discovery.

## Variant and provenance

Do not copy a plugin manifest for Codex, OpenCode or Kiro. Installed `cad-flow/SKILL.md` frontmatter supplies plugin name/version. If a discovered installed skill, role or schema differs, report a defect/open question; never patch installed files.
