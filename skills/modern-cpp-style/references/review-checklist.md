# Style Review Checklist

Use this as the final gate for a review or multi-file cleanup.

- The edited scope matches the user's request.
- Project-local conventions take precedence over fallback defaults.
- Public APIs and observable behavior are unchanged unless explicitly in
  scope.
- Control flow, return values, side-effect order, ownership, and lifetime are
  preserved.
- Comments explain rationale rather than restating code.
- License headers, `TODO`, and `FIXME` markers remain intact.
- Formatting is internally consistent and does not create unrelated diff
  noise.
- Header/source placement remains valid for linkage, templates, and builds.
- Review-only requests produce findings rather than unrequested edits.
- Material edits are summarized with a focused diff or before/after example.
