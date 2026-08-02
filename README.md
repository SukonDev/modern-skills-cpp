# Modern C++ Skills

Focused Agent Skills for writing and reviewing C++ without loading unrelated
guidance into the context window. They use the shared Agent Skills format and
can be installed for every agent supported by the Skills CLI.

## Available skills

| Skill | Use it for |
| --- | --- |
| [`modern-cpp-style`](skills/modern-cpp-style/) | Naming, formatting, comments, file organization, and behavior-safe style reviews |
| [`cpp-static-facade`](skills/cpp-static-facade/) | Assessing and converting true singleton/manager classes to a `Class::Method()` facade |

The Static Facade rules are intentionally separate. A routine formatting task
therefore does not load architecture-specific instructions.

## Install globally with npx

Requires Node.js 22.20 or later. No global npm package installation is
required. Use `-g` so the skills attach to the agent account and remain
available across projects. Use `--copy` to place real files in the resolved
agent skills directory instead of symlinks.

### Claude Code

```bash
npx skills add SukonDev/modern-skills-cpp \
  -g --skill '*' --agent claude-code -y --copy
```

Installed paths:

```text
~/.claude/skills/modern-cpp-style/
~/.claude/skills/cpp-static-facade/
```

### Other agents

Replace the agent ID with the target agent:

```bash
npx skills add SukonDev/modern-skills-cpp \
  -g --skill '*' --agent codex -y --copy
```

Common IDs include `codex`, `cursor`, `opencode`, `gemini-cli`,
`github-copilot`, and `roo`.

Skills CLI currently treats agents configured with the shared
`.agents/skills/` project directory—including Codex, Cursor, OpenCode, Gemini
CLI, and GitHub Copilot—as universal agents. Their global skills are stored
under `~/.agents/skills/`. Agents with a dedicated global directory, such as
Claude Code, use that directory directly.

> Do not use `--all` for a normal installation. It means all skills **and all
> supported agents**, so it intentionally creates many agent folders.

### Select one skill

```bash
npx skills add SukonDev/modern-skills-cpp \
  -g --skill modern-cpp-style --agent claude-code -y --copy
```

Replace the skill name with `cpp-static-facade` when only the facade workflow
is needed.

### Auto-detect or project-only installation

Let Skills CLI detect the installed agent and choose global scope:

```bash
npx skills add SukonDev/modern-skills-cpp -g --copy
```

Omit `-g` only when a project-local installation is intentionally wanted.

```bash
# List the skills without installing them
npx skills add SukonDev/modern-skills-cpp --list

# Update globally installed skills later
npx skills update -g
```

## Agent compatibility

The skills contain standard `SKILL.md` frontmatter and do not require
agent-specific tools. Skills CLI maps them to the correct directories for
supported agents such as Codex, Claude Code, Cursor, OpenCode, Gemini CLI,
GitHub Copilot, Cline, Roo Code, Amp, and others. The optional
`agents/openai.yaml` files add OpenAI UI metadata and can be ignored by other
agents.

## Repository layout

```text
skills/
  modern-cpp-style/
    SKILL.md
    agents/openai.yaml
    references/
  cpp-static-facade/
    SKILL.md
    agents/openai.yaml
    references/
```

Each folder is an independent installable skill. `SKILL.md` contains only the
core workflow and routes to focused references when extra detail is needed.

## License

MIT. See [LICENSE](LICENSE).
