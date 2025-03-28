Let's say everything is fine with inlining of lenses in place. now it's time to think about other optics classes. 

The current scheme, which support only lenses looks like that:

```swift
public struct Magical<T> { }

public struct A {
    public A(
	    public let x: B,
	    public let y: String
    ) { }
}

sealed interface __A_optics_impl {
    public func __x_impl_view(source: A): B
    public func __x_impl_update(source: A, focus: B): A
    public func __y_impl_view(source: A): String
    public func __y_impl_update(source: A, focus: String): A
    public func __x_optics(_: Magical<A>): Magical<B>
    public func __x_optics(_: Magical<A>): Magical<String>
}

extend Magical<A> <: __A_optics_impl { ... }

// Initial magic
func magic<T>(_: T): Magical<T> { ... }
```

We need to somehow guarantee type safety and composition overloading with this mechanism.

**Hypothesis**

> Due to deforested nature of current implementation, we cannot imitate the hierarchy using classes (probably). Thus, we're stuck with functional, typeclass-like approach. And, just like in haskell, have to imitate hierarchy using typeclasses instead of an OO approach with classes.
> 
> We can hypothetically avoid code duplication if change the scheme to look even more type class, and emulate typeclass constraints somehow.
> 
> But all of that is for later
# Idea 1: Struct Multi-Magic

Let's split our `Magical<T>` into multiple magical structs:

- `public struct Lenses<T>`
- `public struct Prisms<T>`
- `public struct Optionals<T>`

The problem is that we do not have any hierarchy, it becomes especially complicated with deforested pairs of optics which we are forced to use for something to inline, but yet have to make one. Let's imitate it.

We define function `composeForward/composeBackward` with overloading for each possible combination of optics:

```
composeBackward<S, A, B>(_: Lenses<S>, _: Lenses<A>, _: Magical<B>) {
	{ l1view: (S) -> A, l2view: (A) -> B, l1update: (S, A) -> S, l2update: (A, B) -> A =>
		{ source: S, focus: B => l1update(source, l2update(l1view(source), focus))}
	}
}

composeForward<S, A, B>(_: Lenses<S>, _: Lenses<A>, _: Magical<B>) {
	{ l1view: (S) -> A, l2view: (A) -> B, l1update: (S, A) -> S, l2update: (A, B) -> A =>
		{ source: S, focus: B => l2view(l1view(source)) }
	}
}
```

**Positives**
- It's just 5-8 times slower than conventional approach, potentially, with inlining, it's even on par, of faster
**Negatives**
- We're re inventing hierarchy in a language which already has hierarchy via classses
- $n^2$ overloads, where $n$ - number of elements in hierarchy (e. g. $n = 5$)
## Idea ???: Almost true typeclasses

- [ ] #TODO Finish section on. **How to imitate typeclasses if you're so eager to have them**