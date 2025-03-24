Phew. I've finished benchmarking and presented my results to Mikhail and Andrei.

They're... Odd. But from my perspective, that's the good thing, because it allows us to delve into different directions for their improvement. In short, what's our problems:

## Inlining

- Look into the following code:
  ```swift
struct TestStruct<S, A> {
	TestStruct(
		public let getter: (S) -> A,
		public let setter: (S, A) -> S
	) {}
}

public let lensX = TestStruct(
    { source: A => source.x },
    { source: A, focus: B => A(focus, source.y)}
)
```
  Surprisingly, despite of `lensX` being public top level, immutable, it won't inline.
- Even simple and straightforward code doesn't want to be inlined, like:
  ```swift
 public func composeFunc<A, B, C>(f: (A) -> B, g: (B) -> C): (A) -> C {
     { x: A => g(f(x))}
 }

composeFunc(x0, composeFunc(x1, composeFunc(x2, ... )))
 ```
  It turns out to be more efficient to write multiple functions, for a number of operations
  
- The inlining depth is what I do not understand at all
- `@Frozen` is not reliable
- Constructor inlining is an open thing
- There's something odd with lambda inlining:

This thing inlines badly:
```swift
public struct TestStruct {
    public let getter = { source: A => source.x },
    public let setter = { source: A, focus: B => A(focus, source.y)}
}
```
While this inlines expectedly well (while being in the same package)
```swift
public struct TestStruct {
    public func getter(source: A) { source.x },
    public func setter(source: A, focus: B) { A(focus, source.y) }
}
```
An open question is: Is it true, that you can replace first struct with second in all possible scenarios? I believe, yes

To resolve the **inlining** issue there's 3 path to choose from:

- Implement your own inlining for optics (moving everything into compiler)
- Modify current inlining in language to support our use cases (keep library)
- `@ForceInline` attribute to avoid doing anything smart
- Mixture of both

> #NOTE Before proceeding further it's worth noting that currently I have an open question in my mind whether to move optics in compiler as fully functional language feature or too keep it as library and make improvements along the way.
> 
> With that knowledge you should read the further sections
## Optics Hierarchy Implementation

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

We need to somehow guarantee type safety and composition overloading with this mechanism. Look into [[Magical Hierarchy|separate document]] for design.

## First-classness

That's a pain, most definitely a pain. According to benchmarks, the following code is **70 times slower**, when inlining inside of lambdas is perfect:

```swift
struct Lens<S, A> {
	Lens(
		public let getter: (S) -> A,
		public let setter: (S, A) -> S
	) {}
}
```

What's even more sadder, the "faster" implementations won't allow for first classes due to a requirement of knowing, how this optics should be used beforehand.

The situation is even worse for **Van Laarhoven/Profunctor** optics because we have to know which functor to declare

Need to decide what to do. Fight or Submit. See document [[Optics First Classness|here]]

## User-defined optics

This one feels like way more simpler thing to design. Because:

- There's really not much we can do, to limit user from writing bullshit
- Syntax is secondary, will come up with something
- What's more important, is property testing, to actually forbid user's keyboard to type bullshit (Leaving compiler realm, I won't do this shit with macro)