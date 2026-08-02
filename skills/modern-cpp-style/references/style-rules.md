# C++ Style Rules

## Contents

- Behavior boundary
- Naming
- Comments
- Formatting and control flow
- Headers and source files
- Class organization

## Behavior boundary

- Keep existing runtime behavior, outputs, branches, and side-effect order.
- Keep existing names and public signatures unless the request authorizes a
  rename or API change.
- Preserve license headers and short `TODO` or `FIXME` markers.
- Avoid broad drive-by formatting outside the requested scope.

## Naming

- Match the project's established naming convention.
- With no established convention, use `PascalCase` for class and method names
  and prefer verb-first methods such as `GetCamera`, `SetConfig`, and `Update`.
- Use prefixes such as `C`, `S`, `T`, or `I` only when the project already uses
  them consistently.
- Prefer names that make explanatory comments unnecessary.

## Comments

Default to no comment when a clearer name or smaller function can explain the
code. Keep or add comments only for:

- non-obvious algorithms or math;
- workarounds that would otherwise look like bugs;
- license and copyright notices;
- concise `TODO` or `FIXME` markers.

Delete comments that merely narrate the next statement, but never remove a
comment whose rationale is not represented in the code.

## Formatting and control flow

- Match the surrounding indentation and brace style. For a new project with
  no convention, use Allman braces and tabs.
- Put one blank line between logically distinct blocks and none inside a tight
  group of related statements.
- Prefer early `return` or `continue` to deeply nested conditionals when the
  transformation is behavior-identical.
- Keep functions focused. Extract unrelated responsibilities only when doing
  so preserves ordering, lifetime, and error behavior.
- Align related declarations only when it improves scanning without creating
  noisy future diffs.

## Headers and source files

- Keep declarations in headers and non-trivial implementations in source
  files unless the project intentionally uses a header-only design.
- Forward-declare related types where it is valid and useful; include complete
  definitions where the language requires them.
- Do not move code across translation units if that could change linkage,
  initialization order, template visibility, or build boundaries.

## Class organization

- Follow the repository's access-section order. With no convention, place
  constructors and destructors first, followed by public, protected, and
  private members.
- Group members by responsibility rather than interleaving unrelated state.
- Do not change ownership, object lifetime, virtual dispatch, or
  const-qualification as part of a style-only pass.
