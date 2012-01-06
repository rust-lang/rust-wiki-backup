## Structural object types

* We currently have structural object types.

* There's no current way to make them recursive. For example, you can't write the type of an object that has a clone method that returns the same type again.

* Just as with [[Bikeshed disjoint union types]], introducing recursive structural types makes for a much more complicated type system.

## Nominal object types

* We could consider adding interface types, which would be nominal object types that could be recursive, and we could allow objects to be annotated as implementing specific interfaces.

* As a convenience, we could also implicitly bind every object constructor to an interface type of the same name, where instances are automatically implementations of that type.

## Hybrid approach

* The above two approaches could work together, with structural types not being recursive and nominal types being recursive.

### Pros

* Not as complex as equirecursive types.

* Still provides the expressivity of recursive types.

* Still provides the flexibility of structural types, just not in cases where the type needs to refer to itself.

### Cons

* May not be expressive enough, since nominal interfaces must be pre-declared.

* Two different kinds of object types is confusing for users (which kind do I need this time? what's the difference?)

## Self-types

* Based on Kim Bruce's work.

* Every object type gets an implicitly bound @self@ type that is a recursive binding to that type.

* Object types remain structural.

* Type equality must take into account @self@, but doesn't have to worry about mutually recursive types.

### Pros

* All the benefits of structural types.

* Much of the expressiveness of recursive object types.

* Likely easier to work with, e.g. to compare for equality, though I'm not 100% positive.

* Only one kind of object type.

* Pretty close to the way things are now.

### Cons

* Not as expressive as full recursive types (may not matter).

* Still probably harder to work with than non-recursive structural types; types become graphs again.
