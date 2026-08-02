---
name: modern-cpp-style
description: Use this skill whenever writing, editing, refactoring, or reviewing C++ code in a class-based project — games, servers, CLI tools, libraries, embedded firmware, anything. Applies to any request to "modernize", "clean up", "make modern style", "fix formatting", or write new classes/methods. Enforces the Static Facade calling convention (CLogger::Write() instead of pLogger->Write()) but ONLY for true singleton/manager classes, a strict low-comment policy, and clean formatting. Never changes program behavior — style and structure only.
---

# Modern C++ Style — Static Facade Skill

A general-purpose style skill. It works on any C++ project — a game, a server, a
CLI tool, a library, embedded firmware — and does not depend on any particular
folder layout. It only defines *how code is written* once you're looking at a
class and its call sites. Applying this skill must never change runtime
behavior, output, or logic. Style and structure only.

## 1. Before You Touch Anything

- Only apply this skill on the user's own codebase, or when explicitly asked
  for this style. If the code belongs to a third party / an established
  open-source project with its own conventions, don't apply this style
  automatically — point out the mismatch and ask first.
- Look at the project's existing conventions (indentation, brace style, class
  prefix, if any) before editing. If the project already has a consistent
  convention that differs from the defaults in this skill, match the project's
  existing convention instead of overriding it. The defaults here are for
  projects with no strong convention yet, or brand-new code.

## 2. Scope: Which Classes Qualify (read this before §3)

This is the single most important gate in the whole skill. The Static Facade
convention (§3) collapses instance access into `::`. That only makes sense for
a class that has **exactly one instance, by design, for the life of the
program (or of its owning scope)** — a true singleton/manager. It is *wrong*
for any class that can have more than one instance, even if at some particular
moment only one happens to exist.

Before applying §3 to a class, ask:

1. Will exactly one instance of this class exist for its whole lifetime, by
   design — not just by coincidence right now?
2. Could the program legitimately create a second one at runtime (a second
   window, a second open document, a second network connection, a second
   worker thread each with its own, a second player/entity)? If yes → does
   not qualify.
3. If this is a nested/owned object (see §3), is it the *one and only* thing
   of its kind the owner ever holds — not one of several, and not "whichever
   one is currently active" among several coexisting ones? If it's one-of-many
   → does not qualify, even for the nested-proxy form.

**Typically qualifies** (true singletons, in any kind of C++ project): a
logger, a configuration/settings manager, the application/session object
itself, a thread pool, a single resource manager, a device driver in firmware,
the local player/camera in a single-player game.

**Typically does not qualify** (ordinary multi-instance classes): any
entity/record type that can have many instances — vehicles, NPCs, documents,
open sockets, database connections, UI widgets, list/array elements — even
single-player games usually have several vehicles/peds in the world at once,
so those stay as ordinary instance access (`pVehicle->GetSpeed()`), not `::`.
A database connection *pool* may be a singleton; an individual open
*connection* it hands out is not.

If a class doesn't clearly qualify, leave it as ordinary instance-based code.
Applying `::` to a multi-instance class is a functional bug, not a style
choice — there would be no way to say *which* instance you meant.

## 3. The Static Facade Calling Convention

For classes that qualify under §2: callers never chain raw pointers (`->`)
through the instance. The class exposes itself through its name using `::`, as
if the whole call were static — even though it forwards to a real instance
internally.

**Rule:** replace `instancePointer->Method()` with `CClassName::Method()`.
**Rule:** when a qualifying singleton owns another qualifying singleton,
expose the owned one as a nested class, and chain `::` instead of `->`.

### Before → After

```cpp
// old style
pGame->GetArm();

// modern style
CGame::GetArm();
```

```cpp
// old style
pGame->GetArm(pSetArm->Config());

// modern style
CGame::GetArm(CSetArm::Config());
```

```cpp
// old style
pGame->pPlayerPed->GetCamera();

// modern style
CGame::CPlayerPed::GetCamera();
```

The same pattern applies outside games, for any qualifying singleton:

```cpp
// old style
pLogger->Write("startup complete");
pApp->pConfig->GetValue("timeout");

// modern style
CLogger::Write("startup complete");
CApplication::CConfig::GetValue("timeout");
```

### Why this requires static methods, not operator overloading

`::` is the compile-time scope-resolution operator — it can't be intercepted
or overloaded the way `->` can. The only way `CClassName::Method()` can work
at all is if `Method` really is a `static` member function that forwards to
the real instance internally. Don't try to fake this with an overloaded
`operator->` or similar; write real static wrapper methods.

### Depth limit

A chain is `Root::Nested::Method()`. Keep it to **at most one nested class**
between the root and the final method — `Root::Method()` and
`Root::Nested::Method()` are both fine. If you find yourself needing
`Root::Nested::NestedAgain::Method()`, stop: that third level always hurts
readability. Add a direct method on `Root` (or on `Nested`) instead of
nesting further.

### Nested class = thin forwarding proxy, not a redefinition

`CGame::CPlayerPed` is a **forwarding proxy**, not a second definition of a
class that already exists elsewhere. If `CPlayerPed` already exists as its
own top-level class, do not redeclare its members inside `CGame`. The proxy
holds **no data members** — only static methods that forward to the real
object:

```cpp
class CGame
{
public:
    class CPlayerPed
    {
    public:
        static CCamera& GetCamera() { return Instance().m_pPlayerPed->GetCamera(); }
    };
};
```

The real `CPlayerPed` class keeps its normal instance methods untouched. Only
the proxy is new, and it must qualify under §2 in its own right (see rule 3
in the scope checklist above).

### Signature mirror

A wrapper's parameter list, overloads, default arguments, and
const-qualification must exactly match the original instance method. A call
site should need no change other than swapping `->` for `::` — never a
change in arguments, order, or return type.

### The wrapper does exactly one thing

The wrapper's body forwards the call and returns whatever the underlying call
returns — nothing else. No added logging, validation, or side effects that
weren't already there. If the class needs new behavior, that's a separate
change from applying this skill.

### Preserve existing guards

If the original call site had a null/validity check
(`if (pGame) pGame->GetArm();`), that guard's behavior must be preserved —
either inside the wrapper (matching the original semantics with an assert or
early-return) or left at the call site if the check did more than test for
null. Don't silently assume validity that the old code didn't assume; that
would be a behavior change, which this skill forbids.

### Testing seam (use if the project has, or will have, unit tests)

Static facades make swapping in a fake/mock for tests harder, because the
public entry point is no longer an object you can inject. Mitigate this by
keeping exactly one seam inside `Instance()` — e.g., an internal
pointer-to-interface that a test build can redirect — so the public `::` API
stays static while the underlying implementation stays swappable in tests.
This is optional, but call it out to the user if their project has (or will
likely need) a test suite.

## 4. Naming Convention

- Class names: `PascalCase`. A single-letter type prefix (commonly `C` for
  ordinary classes, `S`/`T` for plain structs, `I` for interfaces) is optional
  — use it if the target project already uses it, skip it if the project
  doesn't.
- Method names: `PascalCase`, verb-first (`GetCamera`, `SetConfig`, `Update`).
- Keep the facade name and the underlying instance name aligned, e.g. a global
  `TheLogger` instance is wrapped by `class CLogger { static ... };`.

## 5. Header / Source Placement

- Static wrapper methods are *declared* in the `.h` file only (signature, no
  body), except a genuinely trivial one-line forward, which may be inline.
- The forwarding logic (dereferencing the singleton) is *implemented* in the
  `.cpp` file, never inline in the header, to keep headers small as the
  facade layer grows.
- The private singleton accessor (`Instance()`) stays entirely inside the
  `.cpp` file — never exposed in the header.
- Forward-declare owned/related types in the header where possible; include
  their full headers only in the `.cpp`, to avoid circular includes between
  an owner and what it owns.
- Implement `Instance()` as a function-local `static` (a "Meyer's singleton"):
  ```cpp
  CGame& CGame::Instance()
  {
      static CGame instance;
      return instance;
  }
  ```
  The C++11 standard guarantees this initializes exactly once and is
  thread-safe, and it sidesteps the classic static-initialization-order
  problem that namespace-scope static objects have across translation units.
  Do not use a namespace-scope `static CGame g_instance;` for this reason.
- If the project builds across multiple binary modules (DLLs/shared
  libraries), make sure the singleton is defined/exported from exactly one
  module — otherwise each module can end up with its own separate instance
  and silently divergent state.

## 6. Comment Policy

Do not write a comment if the code can be made to say the same thing through a
better name. Default to zero comments.

Comments are only allowed for:
- A non-obvious algorithm or math step that isn't self-evident from naming
- A workaround/hack that would look like a bug otherwise (`// workaround: ...`)
- An existing license/copyright header — never remove these
- `TODO` / `FIXME` markers, kept short

Never write a comment that restates the line beneath it (`// increment counter`
above `counter++;` is always deleted, not written).

## 7. Formatting

- Match whatever indentation (tabs vs. spaces) and brace style the surrounding
  file already uses. Do not mix styles within one file. For brand-new files
  with no existing convention, default to Allman style (opening brace on its
  own line) and tabs for indent.
- One blank line between logically distinct blocks; no blank line inside a
  tight block of related statements.
- Group related declarations together (all members of one subsystem adjacent,
  not interleaved with unrelated ones).
- Order within a class: constructor/destructor first, then public methods,
  then protected, then private.
- Prefer early `return`/`continue` over deep nested `if` chains.
- Keep functions short and single-purpose; split a function rather than let it
  grow multiple unrelated responsibilities.
- Line up related declarations vertically when it helps scanning (e.g. a
  block of member variable declarations), but don't force alignment where it
  hurts future diffs.

## 8. What This Skill Must Never Do

- Never apply the Static Facade convention (§3) to a class that doesn't
  clearly qualify under §2 — that's a functional bug, not a style choice.
- Never change what a function returns, which branch executes, or the order
  of side effects. A refactor under this skill must be behavior-identical.
- Never rename a class, method, or file the user didn't ask you to rename,
  beyond applying the `::` convention to call sites.
- Never delete a license header, `TODO`, or `FIXME` while cleaning up
  comments.
- Never redefine a class's real members inside a nested proxy (§3) — the
  proxy only forwards.
- Never add logic to a wrapper beyond forwarding the call.
- Never drop a null/validity guard that existed at the original call site.

## 9. Trigger Phrases → Actions

When the user says any of the following, apply this skill to the file(s) in
question and report back the before/after diff, not a rewritten explanation
of the logic:

- "modernize this" / "make this modern style" → apply §§2–7, in order (check
  scope in §2 before touching anything)
- "clean this up" / "fix formatting" → apply §7 only
- "convert to static/facade style" → apply §§2–3 only
- "add a new class/method" → write it already in modern style from the start
  (check §2 first; static facade only if it qualifies; §6–7 regardless)
- "review this" → point out any violations of §§2–8 without changing code

## 10. Checklist Before Calling It Done

- [ ] Every class the facade was applied to is confirmed single-instance-by-
      design (§2) — not just currently-the-only-one
- [ ] No stray `pX->` chains remain at call sites that should be `CX::`
- [ ] Wrapper signatures exactly match the original instance methods
      (overloads, default args, const-qualification)
- [ ] Any null/validity guard from the original call site is preserved
- [ ] `Instance()` uses a function-local static, not a namespace-scope static
- [ ] No comment restates what the code already says
- [ ] Indent/brace style matches the rest of the file
- [ ] Nested proxies forward to real objects; they hold no data and redefine
      nothing
- [ ] No chain deeper than one nested class
- [ ] Behavior is provably unchanged — same inputs produce the same outputs

Always show the before/after for any call-site change so the user can verify
no logic moved.
