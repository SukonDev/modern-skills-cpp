---
name: cpp-static-facade
description: Assess, implement, or review C++ Static Facades that replace access through a true singleton or manager pointer with Class::Method() wrappers. Use when a request mentions static facade style, singleton facade conversion, CLogger::Write(), replacing singleton pointer chains, nested forwarding proxies, or reviewing whether a class is safe to expose through static methods. Preserve runtime behavior and reject multi-instance classes.
---

# C++ Static Facade

Expose a true single-instance object through static forwarding methods without
changing the behavior of the underlying implementation.

## Non-negotiable rule

Apply this pattern only when exactly one instance exists by design for the
relevant lifetime. A class that merely happens to have one instance today does
not qualify. Converting a multi-instance class is a functional bug.

## Workflow

1. Inspect construction, ownership, call sites, tests, and module boundaries.
2. Read [references/eligibility.md](references/eligibility.md) and record why
   each candidate qualifies or does not qualify before editing.
3. Leave ambiguous and multi-instance classes using ordinary instance access.
4. For an approved conversion, read
   [references/implementation.md](references/implementation.md) and implement
   only the forwarding surface the request needs.
5. Preserve overloads, arguments, return types, guards, side-effect order,
   initialization timing, and failure behavior.
6. Run the project's focused build and tests when available.
7. Read [references/verification.md](references/verification.md) before
   completion. Show each changed call pattern as a focused before/after diff.

## Review mode

When asked only to review, do not edit. Report eligibility mistakes and
behavior risks first, followed by maintainability observations.

## Related skill

Use `modern-cpp-style` alongside this skill only when the request also includes
general naming, formatting, comment, or class-organization work.
