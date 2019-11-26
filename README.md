# proposal-readonly-collections

## `snapshot`,`diverge`,`readOnlyView` methods for all collections

by Mark S. Miller (@erights) and Peter Hoddie (@phoddie)

## Status

Presented to TC39 (Javascript standards committee), achieving stage 1.

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

Instances of these objects add three new methods: `diverge`, `snapshot`, and `readOnlyView`.

- `diverge` - Performs a shallow clone of the collection into  a new instance with the same prototype. If the original collection changes, the diverged copy does not see those changes.
- `snapshot` - Returns a new instance of the object that that refers to the same collection, however this instance excludes any methods that would modify the collection. If the original collection changes, the snapshot sees those changes.
- `readOnlyView` - Performs a shallow clone of the collection and returns a new instance of the object that excludes any methods that would modify the collection.

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

On a suitable implementation such as [Moddable's XS](https://github.com/Moddable-OpenSource/moddable), the "\*Fixed" versions of these collections can often be placed in ROM. This is especially valuable for data collections such as `ArrayBuffer`. The XS engine is currently able store instances of nearly all built-in objects in ROM. This proposal excludes methods that modify the instance from the object prototype. In XS, methods which would modify the collection contents throw a `TypeError` when applied to instances in ROM. The details are explained in the [XS Linker Warnings](https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/xs/XS%20linker%20warnings.md#exceptions) document.

This proposal does not provide a way to determine if a an object represents a read-only view on a mutable collection (e.g. a snapshot) or a read-only collection (e.g. a read only view). `Object.freeze` has `Object.isFrozen` to determine if an object has been frozen. Something analogous here seems desirable. 

## Shim

[Readonly Collections Shim](https://github.com/Agoric/readonly-collections-shim) is a start on a shim of this proposal.

# Related

This proposal is based on experience with E's [collection classes](http://erights.org/elang/collect/tables.html).
