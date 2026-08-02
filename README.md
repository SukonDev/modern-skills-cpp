# README — Modern C++ Style Skill

This explains `skills.md` in plain language, section by section, with concrete
examples of when to apply each rule and when not to. `skills.md` itself is the
compact version an agent reads and follows; this file is the human-readable
walkthrough for understanding *why* each rule exists.

One sentence summary: this skill changes **how C++ code looks**, never **what
it does**. If applying it ever changes behavior, that's a bug in the
application of the skill, not an acceptable outcome.

---

## 1. Before You Touch Anything

**What it says:** only restyle code you're actually authoring or that the user
asked you to restyle. Match the project's existing conventions if it already
has strong ones.

**Why:** an agent that "helpfully" reformats every file it touches will wreck
diffs in shared repos and annoy maintainers who never asked for a style
change.

**✅ Use it:**
> "Here's my engine's `CCamera.cpp`, modernize it." — the user's own code,
> explicit ask.

**❌ Don't use it:**
> "Can you explain what this function from `zlib` does?" — you're reading
> someone else's library to answer a question, not authoring/restyling it.
> Explain the code as-is; don't rewrite it in this style.

---

## 2. Scope: Which Classes Qualify

**What it says:** `::` access only makes sense for a class with **exactly one
instance, by design.** Before converting anything to the facade style, check
whether a second instance could ever legitimately exist.

**Why this is the most important section:** `Class::Method()` carries no way
to say *which* instance you mean. That's fine for "the one logger" — there is
only ever one. It's meaningless for "the vehicle" when the game world has
thirty vehicles in it at once.

**✅ Use it — genuinely singular things:**
```cpp
CLogger::Write("startup complete");     // one logger, always
CConfig::GetValue("timeout");           // one config, always
CGame::CPlayerPed::GetCamera();         // the one local player, in a single-player game
```

**❌ Don't use it — things that come in multiples:**
```cpp
// WRONG — which vehicle? CVehicle has many instances in the world.
CVehicle::GetSpeed();

// RIGHT — keep ordinary instance access
pVehicle->GetSpeed();
```
```cpp
// WRONG — a connection pool is a singleton, but an individual
// connection it hands out is not.
CConnection::Send(data);

// RIGHT
pConnection->Send(data);
```

**The trap to watch for:** a class that *currently* has only one instance
running isn't automatically a singleton. Ask "could the program ever
legitimately spin up a second one?" — a second window, a second open
document, a second worker thread. If yes, leave it as instance-based code
even if today's build only ever creates one.

---

## 3. The Static Facade Calling Convention

**What it says:** for classes that passed the §2 check, replace
`pObj->Method()` with `CClassName::Method()`. If a singleton owns another
singleton, expose it as a nested class instead of a pointer member.

**Why:** it removes pointer-chasing from call sites, makes the call
self-documenting (the class name *is* the entry point), and stops the "is
this pointer valid right now?" question from leaking into every caller.

**Basic example:**
```cpp
// old
pGame->GetArm();
// modern
CGame::GetArm();
```

**Nested example (owner + one owned singleton):**
```cpp
// old
pGame->pPlayerPed->GetCamera();
// modern
CGame::CPlayerPed::GetCamera();
```

**Depth limit example — where to stop:**
```cpp
// fine — one nested class
CGame::CPlayerPed::GetCamera();

// NOT fine — a second level of nesting. Add a direct method instead:
// CGame::CPlayerPed::CCamera::GetZoom();   ❌
CGame::CPlayerPed::GetCameraZoom();          // ✅ flattened
```

**Guard preservation — a subtle way this rule gets broken:**
```cpp
// old — the null check IS part of the behavior
if (pGame) pGame->GetArm();

// WRONG modernization — silently assumes pGame always exists now
CGame::GetArm();

// RIGHT — the guard moves inside the wrapper, behavior is identical
void CGame::GetArm()
{
    if (!s_bInitialized) return;
    Instance().DoGetArm();
}
```
This is the single easiest way to accidentally introduce a real bug while
"just" doing a style pass — always ask what the original guard was protecting
against before deleting it.

**Signature mirror — another subtle break:**
```cpp
// old, two overloads
pGame->GetArm();
pGame->GetArm(pSetArm->Config());

// WRONG — collapsed to one signature, callers of the other overload break
CGame::GetArm(SArmConfig const& config = {});

// RIGHT — mirror both overloads exactly
CGame::GetArm();
CGame::GetArm(CSetArm::Config());
```

**The wrapper does one thing only:**
```cpp
// WRONG — sneaks in new behavior during a "style-only" pass
static void GetArm()
{
    LogCall("GetArm");          // new logging — not part of the original
    Instance().DoGetArm();
}

// RIGHT
static void GetArm()
{
    Instance().DoGetArm();
}
```

---

## 4. Naming Convention

**What it says:** `PascalCase` classes/methods, optional `C` prefix — but only
if the project already uses it.

**✅ Use the default:** a brand-new project with no existing convention →
`class CLogger`, `Write()`, `GetValue()`.

**❌ Don't force the default:** the project already uses `Logger` (no
prefix) and `write()` (camelCase) everywhere → keep matching that, don't
introduce `CLogger`/`Write` halfway through the codebase.

---

## 5. Header / Source Placement

**What it says:** declare the static wrapper in the header, implement the
forwarding logic in the `.cpp`, keep `Instance()` as a function-local static,
never a namespace-scope static.

**Why the function-local static matters:**
```cpp
// WRONG — namespace-scope static; initialization order across
// translation units is unspecified, this can crash on startup
// depending on link order (the "static init order fiasco").
static CGame g_instance;

// RIGHT — initialized lazily and thread-safely on first use
CGame& CGame::Instance()
{
    static CGame instance;
    return instance;
}
```

**✅ Use it:** any project, single- or multi-threaded — this pattern is safe
either way and the C++11 standard guarantees the thread safety.

**❌ Don't skip it just because the project is single-threaded today:**
projects grow threads over time; there's no cost to doing it right from the
start.

---

## 6. Comment Policy

**What it says:** default to zero comments; only keep the four allowed
categories.

**✅ Keep this comment (workaround):**
```cpp
// workaround: driver X returns 1 frame late on this GPU family
```

**✅ Keep this comment (non-obvious math):**
```cpp
// solves for t where the parabola crosses y=0 (quadratic formula)
```

**❌ Delete this comment (restates the code):**
```cpp
// increment counter
counter++;
```

**❌ Delete this comment (name already says it):**
```cpp
// get the camera
CGame::CPlayerPed::GetCamera();
```

---

## 7. Formatting

**What it says:** match the file's existing indent/brace style; if there is
none, default to Allman braces + tabs. Group related declarations. Prefer
early returns over nested `if`s.

**✅ Example — early return over nesting:**
```cpp
// before
void Update()
{
    if (IsValid())
    {
        if (IsActive())
        {
            DoWork();
        }
    }
}

// after
void Update()
{
    if (!IsValid()) return;
    if (!IsActive()) return;
    DoWork();
}
```

---

## 8. What This Skill Must Never Do

This is the cheat sheet of red flags — if you catch yourself about to do any
of these while "modernizing" a file, stop:

- Applying `::` to a class from §2's "does not qualify" list
- Changing a return value, branch, or order of side effects
- Renaming anything beyond what §3 requires at call sites
- Deleting a license header, `TODO`, or `FIXME`
- Redefining a real class's members inside a nested proxy
- Adding new logic (logging, validation) inside a wrapper
- Dropping a null/validity guard that existed before

---

## 9. Trigger Phrases → Actions

| User says | Agent applies |
|---|---|
| "modernize this" | §§2–7, checking scope (§2) first |
| "clean this up" / "fix formatting" | §7 only |
| "convert to static/facade style" | §§2–3 only |
| "add a new class/method" | write it in modern style from the start |
| "review this" | point out violations of §§2–8, change nothing |

---

## 10. Checklist Before Calling It Done

Treat this as the final gate before handing a "modernized" file back — every
box should be checkable with a straight face, not assumed:

- [ ] Every faceted class is confirmed single-instance-by-design (§2)
- [ ] No stray `pX->` chains remain where `CX::` was intended
- [ ] Wrapper signatures exactly mirror the originals
- [ ] Original null/validity guards are preserved
- [ ] `Instance()` is a function-local static
- [ ] No comment just restates the code
- [ ] Formatting matches the surrounding file
- [ ] Nested proxies hold no data and redefine nothing
- [ ] No chain nested more than one class deep
- [ ] Behavior is provably unchanged
