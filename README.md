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

## Install with npx

Requires Node.js 18 or later. No global package installation is required.

### All skills, all supported agents

Install into the current project:

```bash
npx skills add SukonDev/modern-skills-cpp --all
```

Install globally for all supported agents:

```bash
npx skills add SukonDev/modern-skills-cpp --all -g
```

`--all` selects every skill and every agent supported by the CLI and skips
interactive prompts.

If the system does not allow symlinks, install independent copies instead:

```bash
npx skills add SukonDev/modern-skills-cpp --all --copy
```

### Select one skill

Install only `modern-cpp-style` for every supported agent:

```bash
npx skills add SukonDev/modern-skills-cpp \
  --skill modern-cpp-style --agent '*' -y
```

Replace the skill name with `cpp-static-facade` when only the facade workflow
is needed.

### Inspect or choose interactively

```bash
# List the skills without installing them
npx skills add SukonDev/modern-skills-cpp --list

# Choose skills, agents, scope, and install method interactively
npx skills add SukonDev/modern-skills-cpp

# Update installed skills later
npx skills update
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
