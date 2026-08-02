# Modern C++ Skills

Focused Codex skills for writing and reviewing C++ without loading unrelated
guidance into the context window.

## Available skills

| Skill | Use it for |
| --- | --- |
| [`modern-cpp-style`](skills/modern-cpp-style/) | Naming, formatting, comments, file organization, and behavior-safe style reviews |
| [`cpp-static-facade`](skills/cpp-static-facade/) | Assessing and converting true singleton/manager classes to a `Class::Method()` facade |

The Static Facade rules are intentionally separate. A routine formatting task
therefore does not load architecture-specific instructions.

## Install with Codex

Ask Codex to use its built-in `skill-installer`:

> Use skill-installer to install `skills/modern-cpp-style` and
> `skills/cpp-static-facade` from `SukonDev/modern-skills-cpp`.

Install only the folder you need, or install both. Skills installed by
`skill-installer` are available on the next turn.

Equivalent installer arguments are:

```text
--repo SukonDev/modern-skills-cpp \
--path skills/modern-cpp-style skills/cpp-static-facade
```

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
