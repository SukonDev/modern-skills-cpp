# Static Facade Implementation

## Contents

- Public wrappers
- Instance storage
- Nested forwarding proxies
- Behavior preservation
- Test and module seams

## Public wrappers

Declare static wrapper methods in the header and place non-trivial forwarding
implementations in the `.cpp` file.

```cpp
class CLogger
{
public:
	static void Write(std::string_view message);
};
```

```cpp
void CLogger::Write(std::string_view message)
{
	Instance().DoWrite(message);
}
```

Mirror the original parameter lists, overloads, default arguments, return
types, exception behavior, and relevant qualifiers. Change call sites only
from instance selection to scope access:

```cpp
// Before
pLogger->Write("startup complete");

// After
CLogger::Write("startup complete");
```

The wrapper forwards and returns the result. Do not add logging, validation,
fallbacks, or unrelated state changes.

## Instance storage

When introducing storage, prefer a function-local static so initialization is
lazy and thread-safe since C++11:

```cpp
CLogger& CLogger::Instance()
{
	static CLogger instance;
	return instance;
}
```

Do not replace existing lifetime semantics casually. If the original object
had explicit startup, shutdown, destruction order, or nullable states, model
those semantics rather than assuming a Meyer's singleton is equivalent.

Avoid namespace-scope instances that can create cross-translation-unit static
initialization-order problems.

## Nested forwarding proxies

A nested class is a thin proxy, not a duplicate definition of the owned type.
It holds no data and exposes only static forwarders to the real object.

```cpp
class CGame
{
public:
	class CPlayer
	{
	public:
		static CCamera& GetCamera();
	};
};
```

Keep the real `CPlayer` class and its instance methods intact. Put forwarding
logic in the `.cpp` file where possible to avoid new include cycles.

## Behavior preservation

Preserve every existing null or validity guard. A call such as
`if (pGame) pGame->Run();` cannot become an unconditional `CGame::Run()` unless
the wrapper reproduces exactly what the guard protected and when.

Preserve evaluation and side-effect order. Do not collapse overloads into a
new defaulted signature or change arguments as part of the conversion.

## Test and module seams

Static APIs are harder to replace in tests. If tests need substitution, keep a
single internal pointer-to-interface or equivalent override seam behind the
facade.

For DLLs or shared libraries, define and export one authoritative instance
from one module. Header-local storage can accidentally create one instance per
module.
