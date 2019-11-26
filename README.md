# proposal-readonly-collections

## `snapshot`,`diverge`,`readOnlyView` methods for all collections

by Mark S. Miller (@erights) and Peter Hoddie (@phoddie)
<br/>
updated November 25, 2019

## Status

Presented to TC39 (JavaScript standards committee), achieving stage 1.

[<img src="readonly-miniplayer.png" alt="Presentation to TC39" width="40%">](https://www.youtube.com/watch?v=N-X_4Xe9lEw&list=PLzDw4TTug5O0ywHrOz4VevVTYr6Kj_KtW)

[Slides](https://github.com/tc39/agendas/blob/master/2019/10.readonly-collections-as-recorded.pdf)

## Proposal

Currently JavaScript collections are unconditionally mutable. However, many use cases could benefit from separating the ability to query a collection from the ability to mutate it. Indeed, we have seen repeated calls for such support. To minimize cognitive load of new API, we propose the addition of only three new methods to all collections, and the addition of new collection classes whose APIs are only a subset of the existing collection abstractions.

We propose these apply to all EcmaScript enumerable collections:

- `Map`
- `Set`
- `TypedArray` (all typed arrays, including `Uint8Array`, `Int8Array`, `Uint64Array`, etc.)

We propose additionally applying these to the following EcmaScript non-enumerable collections:

- `DataView`
- `ArrayBuffer`
- `WeakMap`
- `WeakSet`

We illustrate the general case using `Map` as a concrete example. For brevity, we refer to class-like constructors, such as `Map`, as if they are classes.

Instances of these objects add three new methods: `diverge`, `readOnlyView`, and `snapshot`.

- `diverge()` - Performs a shallow clone of the collection into a new instance with a prototype that contains mutating methods. Changes to the original collection are not visible through the diverged copy.
- `readOnlyView()` - Returns an object that that refers to the same collection, however this instance excludes mutating methods. Changes to the original collection are visible to the read-only view. When applied to a read-only collection (created by `readOnlyView` or `snapshot`), returns itself.
- `snapshot()` - Performs a shallow clone of the collection into a new instance of the object that excludes mutating methods. When applied to a snapshot collection (created by `snapshot()`), returns itself.

Where currently we have one `Map` class that represents the mutable variant, we would add a `FixedMap` and a `ReadOnlyMap`. We add these classes as static members of the `Map` class to avoid polluting the global name space. All three of these classes would support the `snapshot`, `diverge`, and `readOnlyView` methods proposed here. In addition, `FixedMap` and `ReadOnlyMap` would support the query-only methods of `Map` but not any of the methods that would mutate a map. They have precisely the same API but different behavioral contracts.

> **Note**: the proposed static member constructors are not valid to invoke in this proposal. Perhaps they should not be visible in the public API?

Following the Liskov substitutability principle, we consider a type to be a behavioral contract. Type B is a behavioral subtype of type A when instances of B obey A's specification of the behavior of A instances. By these criteria `Map`, `FixedMap`, and `ReadOnlyMap` are behavioral subtypes of a hypothetical `AbstractMap`, which again has exactly the API of `FixedMap` and `ReadOnlyMap`, but a weaker behavioral type that `Map` also obeys. In addition, `FixedMap` is a behavioral subtype of `ReadOnlyMap` At this time, we do not propose actually creating this abstract supertype `AbstractMap`, but rather treat it as a specification fiction.

The types in the API definitions below are stated according to these behavioral subtyping relationships.

## APIs

```js
class AbstractMap {

        // A FixedMap, not necessarily fresh, whose state is this map's current
        // state.
        snapshot() :FixedMap;

        // A fresh Map whose initial state is this map's current state.
        diverge() :Map;

        // A ReadOnlyMap, not necessarily fresh, whose state is a read only
        // view of this map's current state.
        readOnlyView() :ReadOnlyMap;

        // query-only methods of Map
        ...
}

class Map obeys AbstractMap {

        // methods of AbstractMap
        ...

        snapshot() :FixedMap;  // necessarily fresh

        readOnlyView() :ReadOnlyMap();  // necessarily fresh

        // also the mutating methods of Map
        ...
}

// An instance of a ReadOnlyMap provides only the ability to query, not update.
class ReadOnlyMap obeys AbstractMap {

        readOnlyView() :ReadOnly;  // returns itself
}

// The contents of this map cannot change.
class FixedMap obeys ReadOnlyMap {

        snapshot() :FixedMap;  // returns itself

        readOnlyView() :ReadOnly;  // returns itself
}

```

On a suitable implementation such as [Moddable's XS](https://github.com/Moddable-OpenSource/moddable), the "\*Fixed" versions of these collections can be stored in ROM. This is especially valuable for data collections such as `ArrayBuffer`. The XS engine is currently able store instances of nearly all built-in objects in ROM. In this proposal, the instances stored in ROM would have prototypes that exclude mutating methods from the prototype. By contrast, in XS collection instances stored in ROM retain all original methods, however mutating methods throw a `TypeError` when called. The details are explained in the [XS Linker Warnings](https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/xs/XS%20linker%20warnings.md#exceptions) document.

This proposal does not provide a way to determine if an object represents a read-only view on a mutable collection (e.g. a readOnlyView) or an immutable collection (e.g. a snapshot). `Object.freeze` has `Object.isFrozen` to know if an object has been frozen. Something analogous here may be desirable. 

## Shim

[Readonly Collections Shim](https://github.com/Agoric/readonly-collections-shim) is a start on a shim of this proposal.

# Related

This proposal is based on experience with E's [collection classes](http://erights.org/elang/collect/tables.html).
