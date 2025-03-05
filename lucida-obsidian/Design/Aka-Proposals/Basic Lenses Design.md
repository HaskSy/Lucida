## Use-case examples

```swift
@Derive[ToString]
struct A {
	A(
		public var x: String,
		public var y: String
	) {}
}

main() {
	let a = A("Hello", "World")
	let lambda = { => 
		var b = a
		b.x = "Bye"
		println(b)
	}
}
```

A concise way to mutate mutable value in immutable struct might be like that. Both object and value are present
```swift
main() {
	let a = A("Hello", "World")
	let lambda = { => 
		let b = @Optics(a&.x = "Bye")
		println(b)
	}
}
```
If we want to have just lens to some field, then:
```swift
main() {
	let a = A("Hello", "World")
	let xLens: Lens<A, String> = @Optics(A&.x) // Optical object which didn't capture object of type
	let lambda = { =>
		let b = xLens.update(a, "Bye")
		println(b) // "Bye, World"
	}
}
```
If we want to attach some computation and make `view`, then
```swift
main() {
	let a = A("Hello", "World")
	let xCapitalized: View<A, String> = @Optics(A&.x &> capitalize)
	let lambda = { =>
		let b = xCapitalized.view(a)
		println(b) // HELLO
	}
}
```

If we want to attach some computation onto the input, then we can make
```swift
main() {
	let a = A("Hello", "World")
	let xCapitalized: Update<A, String> = @Optics(A&.x &> capitalize)
	let lambda = { =>
		let b = xCapitalized.view(a)
		println(b) // HELLO
	}
}
```