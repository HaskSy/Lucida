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

# Idea: Multi-Magic

- [ ] #TODO Multi-magic optics hierarchy