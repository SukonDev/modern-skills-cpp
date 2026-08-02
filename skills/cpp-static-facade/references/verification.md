# Static Facade Verification

Before completion, verify:

- Every converted class is single-instance by design, with evidence.
- No multi-instance or active-one-of-many object uses the facade.
- Wrapper signatures mirror every required overload and return type.
- Existing null, readiness, and validity guards preserve their semantics.
- Wrappers only forward; they add no behavior.
- Initialization, shutdown, and destruction timing remain correct.
- A function-local static is used only when it matches the original lifetime.
- Nested proxies hold no data and do not redefine the real class.
- No facade chain is deeper than `Root::Nested::Method()`.
- Tests retain a substitution seam when the project requires one.
- DLL/shared-library builds resolve to one authoritative instance.
- Focused builds and tests pass.
- Each call-site conversion is shown in a focused before/after diff.

If any item cannot be proved, report the uncertainty and leave that portion
instance-based.
