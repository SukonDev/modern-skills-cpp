# Static Facade Eligibility

## Contents

- Required evidence
- Common candidates
- Disqualifiers
- Owned objects and nested proxies
- Uncertain cases

## Required evidence

Confirm all of the following from code or explicit architecture constraints:

1. Exactly one instance exists by design for its entire relevant lifetime.
2. Creating a second concurrent instance would be invalid, not merely unusual.
3. Callers do not need identity to choose which instance receives a call.
4. The object has one authoritative definition across binary modules.
5. Static access will not destroy a required dependency-injection or test seam.

Search constructors, factories, containers, globals, ownership fields, tests,
plugins, and DLL/shared-library boundaries. Do not infer singularity from one
visible call site.

## Common candidates

These may qualify after evidence is found: a process-wide logger,
configuration manager, application/session object, resource manager, thread
pool, or a single hardware device manager.

The category name is not proof. For example, multiple application sessions or
multiple logger instances can be valid in some systems.

## Disqualifiers

Reject classes representing entities, records, documents, sockets, database
connections, UI widgets, workers, vehicles, NPCs, or other one-of-many values.
A singleton connection pool may qualify; each connection it returns does not.

Also reject a class when tests or runtime modes legitimately construct
multiple isolated instances, even if production usually creates one.

## Owned objects and nested proxies

An owned object qualifies for a nested forwarding proxy only when the owner
has exactly one authoritative object of that kind. "Currently active" among
several objects does not qualify.

Keep facade chains to one nested level:

```cpp
CGame::CPlayer::GetCamera();
```

Do not create deeper chains such as
`CGame::CPlayer::CCamera::GetZoom()`. Add a direct forwarding method to the
root or first nested proxy instead.

## Uncertain cases

If the evidence is incomplete, keep instance access and explain what must be
verified. Safety takes precedence over achieving a facade-shaped call site.
