---
name: dependency-inversion-principle
description: How to create decoupled software components in a multi-module Gradle/Kotlin project by splitting each component into a stable public API surface and a hidden implementation, so clients depend only on a contract and never on internal details. Use this skill whenever the user wants to structure a feature or component into api/impl modules, expose a public entry point (factory, builder, or DI binding) without leaking the concrete class, enforce module-coupling rules (composition root, no cross-feature :impl dependencies), or apply the Dependency Inversion Principle in a Gradle/Kotlin codebase — even if they don't name the principle explicitly and just describe wanting "decoupled modules", "an api and impl split", or "clients that shouldn't see the implementation".
---

# Creating Decoupled Components

This skill describes how to create a decoupled software component in a project. A
component is split into a public **API surface** and a hidden **implementation**, so
that clients depend only on a stable contract and never on internal details.

## Core idea

A component exposes an API surface to its users. This surface is a contract: it should
not change except for a major version bump. Everything else — the implementation — is
free to change at any time, because clients never see it.

This split is enforced at the module level. There are (at least) two Gradle modules:

- `$component-api` — the public contract. Clients depend on this.
- `$component-impl` — the implementation. Clients do **not** depend on this directly.

## The API module (`$component-api`)

Contains only the contract: the interfaces clients call, plus any data types that are
part of the public surface. Nothing here exposes how the work is actually done.

```kotlin
// module: $component-api

data class SampleData(val userName: String, val age: Int)

interface ComponentApi {
    suspend fun loadData(): SampleData
    suspend fun saveData(data: SampleData)
}
```

Both `ComponentApi` and `SampleData` are part of the public API. Neither can change in a
breaking way without a major version bump (Semantic Versioning).

> If the component lives inside a closed internal project and is never published (e.g.
> not pushed to Maven Central), strict versioning matters less, since there are no
> external consumers to break. The *discipline* of keeping the surface stable still
> pays off, because it keeps internal callers decoupled.

## The implementation module (`$component-impl`)

Depends on `$component-api` and implements the contract. Because clients never depend on
this module, its classes can and should be `internal` (or `private` where it fits), so
nothing leaks accidentally.

```kotlin
// module: $component-impl

internal class ComponentApiImpl : ComponentApi {
    override suspend fun loadData(): SampleData { /* ... */ }
    override suspend fun saveData(data: SampleData) { /* ... */ }
}
```

The implementation class itself is `internal`: a client could not reference
`ComponentApiImpl` even if it wanted to, because it is not visible outside its module.

## Connecting the client to the implementation

The interesting question: an interface cannot do anything on its own, so the client
needs an actual instance — but it must obtain one **without depending on the impl
module**. The component therefore has to provide a public **entry point** that hands out
an instance typed as the interface.

The public part typically exposes a **factory**, but a factory is not the only option —
use whatever construction shape fits the component. What matters is that the entry point
is public, lives in (or is reachable via) the API surface, and returns the interface
type, never the concrete class.

### The public entry point is part of the contract

Whatever construction shape you pick lives in the public surface and is therefore subject
to the same stability rule as the rest of the API: it should not change in a breaking way
between major versions. Further interfaces returned or required by the entry point follow
the **same pattern** — they are declared in the API module, implemented `internal`ly, and
obtained only through the public surface.

### Option A — Factory function in the API module

The API module declares the factory; the impl module provides the body. The client calls
the factory and receives a `ComponentApi`, never knowing the concrete type.

```kotlin
// module: $component-api
fun ComponentApi(/* required collaborators */): ComponentApi = createComponentApi(/* ... */)
```

This is the lightest-weight option, but requires a runtime registry pattern (e.g., a
factory lambda registered by the `:impl` module at startup), because the API module cannot
statically reference the implementation class due to Gradle dependency constraints.

### Option B — Factory object exposed by the impl module

The impl module exposes one public factory; everything else stays `internal`. The client
depends on `$component-api` for the *type* and calls the factory only at the composition
root.

```kotlin
// module: $component-impl
object ComponentFactory {
    fun create(/* required collaborators */): ComponentApi = ComponentApiImpl(/* ... */)
}
```

Here only `ComponentFactory` is visible from outside; `ComponentApiImpl` remains hidden.

### Option C — Builder in the API module (for configuration)

When the component needs configuration — values that are optional, or that the client
*must* supply before a valid instance can exist — expose a **builder** in the API module.
The builder lets the client set the relevant configuration step by step and produces the
interface type at the end. This keeps required wiring explicit and self-documenting while
still hiding the implementation.

```kotlin
// module: $component-api
class ComponentBuilder {
    // optional configuration
    fun withTimeout(ms: Long): ComponentBuilder = apply { /* ... */ }

    // mandatory configuration: build() should fail or be impossible
    // to call without it, depending on how strict you want to be
    fun withStorage(storage: Storage): ComponentBuilder = apply { /* ... */ }

    fun build(): ComponentApi = /* hand off to the internal implementation */
}
```

Use a builder when there are enough configuration knobs — or hard requirements — that a
single factory call would be awkward or easy to misuse. For one or two simple
collaborators, a factory (Option A or B) is usually clearer.

### Option D — Dependency injection (optional refinement)

Dependency injection is **one possible way** to deliver the instance, not a requirement.
With a DI framework like Koin (using Koin Annotations), the implementation module
automatically binds the implementation to the interface via `@Single` or `@Factory`. The
client injects the dependency without knowing the concrete class.

```kotlin
// module: $component-impl
@Single(binds = [ComponentApi::class])
internal class ComponentApiImpl : ComponentApi { /* ... */ }
```

This automates what the other options do by hand, at the cost of adopting the framework.

## What the client sees

Regardless of which entry point is chosen, the client's view is identical and minimal:

```kotlin
// client module — depends ONLY on $component-api
val component: ComponentApi = /* factory call, builder.build(), or DI lookup */
val data = component.loadData()
```

The client knows the interface and the entry point. It knows nothing about
`ComponentApiImpl`, and a change to the implementation module never forces the client to
recompile against a new type.

## Summary of the rules

- The API surface is a contract; treat it as immutable between major versions.
- Put the contract (interfaces + public data types) in `$component-api`.
- Keep `$component-api` pure Kotlin: avoid dependencies on databases, network, or
  platform libraries.
- Put the implementation in `$component-impl`, marked `internal`/`private`.
- Clients depend on `$component-api` only.
- **No feature module may depend directly on another feature's `:impl` module.** Only the
  composition root (typically the `:app` module, which loads the DI modules) is allowed to
  depend on `$component-impl` modules, so that it alone wires the object graph.
- Expose a public **entry point** so clients can obtain an instance typed as the
  interface, without depending on the implementation. A factory is the common shape, but
  a factory object, a builder (for optional or mandatory configuration), or dependency
  injection all satisfy this — choose per component.
- Further interfaces follow the **same pattern**: declared in the API module, implemented
  `internal`ly, obtained through the public surface.
