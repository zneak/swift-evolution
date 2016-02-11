# Treat uniform tuples as collections

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-name.md)
* Author(s): [Swift Developer](https://github.com/swiftdev)
* Status: **Awaiting review**
* Review manager: TBD

## Introduction

This proposal aims at adding collection operations to uniform tuples: tuples in
which every element has the same type, and no element has a label.

Swift-evolution thread: [Proposal: Contiguous Variables (A.K.A. Fixed Sized Array Type)](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009520.html)

## Motivation

Fixed-size arrays in C structures are imported as tuples. This means that on the
Swift side, developers lose the ability to use even the most basic collection
operations on them, like subscripts. When collection operations are needed, it
is usually necessary to transform the tuple into a full-fledged Swift `Array`
using unsafe pointers.

## Proposed solution

This proposal suggests adding `CollectionType` conformance to uniform tuples.

## Detailed design

Any tuple of more than one element, in which every element has the same type,
is eligible for automatic `CollectionType` conformance. The implementation has
to be generated by the compiler. `CollectionType` members are generated as such:

* `Generator`: opaque. Possibly based off `UnsafeBufferPointer`s, possibly the
	same for all uniform tuples (to a generic parameter).
* `Index`: `Int`.
* `SubSequence`: `Array`, since tuples are value types and cannot be shared
	without being copied wholesale anyway, and the dynamic nature of
	`SubSequence` means that it usually is impossible to simply return a smaller
	tuple (whose size has to be known at compile-time).
* `count`: the number of elements in the tuple.
* `first`: `tuple.0`
* `isEmpty`: `false`
* `subscript(_: Self.Index)`: single element at given index. Bounds-checked.
* `subscript(_: Range<Self.Index>)`: sub-sequence `Array`. Bounds-checked.
* `startIndex`: 0.
* `endIndex`: `count`.

It is worth noting that uniform tuples do not lose static field access. This
avoids needlessly breaking existing code and allows developers to bypass
bounds-checking for indices known to be safe.

## Impact on existing code

No impact on existing code; the feature is purely additive.

## Alternatives considered

A simpler syntax to declare uniform tuples from Swift, like `(Int x 4)`, was
found to be contentious. This proposal forks off the original for incremental
implementation of the feature.

The original proposal only called for a subscript on uniform tuples. Adding full
`CollectionType` support seemed simple enough and useful enough to suggest it.

It was suggested that uniform tuples should lose their static field access
syntax to ensure that there is Just One Way to access tuple elements. However,
this could have surprising side effects on existing code and wouldn't be
possible with generic code.