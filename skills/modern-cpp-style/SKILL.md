---
name: modern-cpp-style
description: Write, clean up, format, or review C++ code with behavior-safe naming, comments, layout, headers, and class organization. Use for requests to modernize C++, clean it up, fix formatting, add a C++ class or method, or review C++ style. Do not use this skill alone for singleton-to-static-facade conversions; use cpp-static-facade for that architecture-specific workflow.
---

# Modern C++ Style

Improve how C++ code is expressed without silently changing what existing code
does.

## Workflow

1. Confirm the code belongs to the user or the user explicitly requested a
   style pass. Do not restyle third-party code merely because it is being read.
2. Inspect nearby files and repository instructions before editing. Prefer a
   consistent project convention over this skill's fallback defaults.
3. Select the requested mode:
   - For new code, apply the style rules from the start.
   - For cleanup or formatting, change presentation and local structure only.
   - For review, report violations without editing unless the user also asks
     for fixes.
4. Read [references/style-rules.md](references/style-rules.md) before writing
   or changing code. Use only the sections relevant to the request.
5. Preserve public APIs, control flow, return values, side-effect order,
   ownership, and lifetime unless the user separately authorizes such changes.
6. Read [references/review-checklist.md](references/review-checklist.md) before
   completing a style review or a multi-file cleanup.
7. Show a focused diff or concise before/after examples for material changes.

## Scope boundary

Do not introduce a Static Facade merely because a class currently has one
instance. When the request explicitly involves singleton eligibility,
`Class::Method()` wrappers, or replacing singleton pointer chains, use the
`cpp-static-facade` skill.
