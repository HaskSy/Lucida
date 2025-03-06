## Use-case examples

### Lenses
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
Let's introduce a function `capitalize`
```swift
public func capitalize(a: String): String
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

If we want to attach some computation onto the input, then we can make it. But, why?
```swift
main() {
	let a = A("Hello", "World")
	let xCapitalized: Update<A, String> = @Optics(A&.x <& capitalize)
	let lambda = { =>
		let b = xCapitalized.update(a, "Bye")
		println(b) // "BYE, World"
	}
}
```

If user provides lens instead of function, then the composition `&>` would stay in the same realm of lenses

```swift
main() {
	let a = A("Hello", "World")
	let AxAsCharArr: Lens<A, Char[]> = @Optics(A&.x &> stringCharArrLens)
	let lambda = { =>
		let b = AxAsCharArr.update(a, ['B', 'y', 'e'])
		println(b) // "BYE, World"
	}
}
```

```
View<...>
Update<...>
Lens<...>
```

But that's all `ok` and fine, while we talk about what about `structs`. With classes though...

The first question is how to perform cloning of that object:

1. Builtin deep copy + value reassignment
	- Most of the state will be separate, but some things won't be saved even by that (e.g. connections)
	- #QUESTION Who in the right mind will use optics for effectful stuff?
2. Builtin shallow copy + deep value reassignment
	- `class` inside `class` inside `class`...
3. Primary constructor
	- Object already went through some shit..., is it right to create brand new one?
4. `interface Cloneable` + `@Derive[Cloneable]` + value reassignment
	- Basically forcing user to solve our problems in case of classes.
	- #QUESTION it's not really an interface, because `struct` and `class` is not part of the type. What is it?
	- It's almost user-defined lens for each field of class
	- #DESIGNQ Look closely into user-defined lenses for non-public stuff
5. `extend`

Yeah. It's worth reminding again that we're planning to automatically derive optics only for public stuff. Because tracking where you can and cannot use optics is basically `ownership`


### Prisms

```swift
@Derive[ToString]
enum E {
	| E1
	| E2
	| E3(String, String)
}

main() {
	let e = E3("Hello", "World")
	let lambda = { => 
		if (let E3(_, x) <- e) {
			println("Bye, ${x}")
		}
	}
}
```

Make some computation on match
```swift
main() {
	let e = E3("Hello", "World")
	let lambda = { => 
		let matcher3: Prism<E, (String, String)> = @Optics(E&.E3)
		matcher3.match(e) // Some((String, String))
	}
}
```

Construct something on build. Useless for enums itself
```swift
main() {
	let e = E3("Hello", "World")
	let lambda = { => 
		let matcher3: Prism<E, (String, String)> = @Optics(E&.E3)
		let eCopy = matcher3.build(("Bye", "World")) // E3("Bye", "World")
	}
}
```
Another useless usecase
```swift
main() {
	let e = E3("Hello", "World")
	let lambda = { => 
		let eCopy = @Optics(e&.E3 = ("Bye, "World"))
	}
}
```
Usage prisms with coalescing `??`
```swift
main() {
	let e = E3("Hello", "World")
	let lambda = { => 
		@Optics(e&.E3 ?? println("That's not E3"))
	}
}
```
Usage prisms with coalescing `??`
```swift
main() {
	let e = E2
	let lambda = { => 
		@Optics(e&.E3 ?? println("That's not E3"))
	}
}
```
Usage prisms with pattern matching. Useless for enums, but for user defined.
```swift
main() {
	let e = E3("Hello, World")
	let lambda = { => 
		if (let Some((_, x)) <- @Optics(e&.E3)) {
			println("Bye, ${x}")
		}
	}
}

main() {
	let e = E2
	let lambda = { => 
		match {
		    case @Optics(e&.E3) => ...
		}
	}
}
```
- [ ] #DESIGNQ Think of destructuring syntax. This is ugly

Composition with function using `&>`
```swift
main() {
	let e =
	let lambda = { => 
		let preview: Match<E, String> = @Optics(E&.E3 &> at(2) &> capitalize
		// Alternatively? I don't like it
		let preview: View<E, Option<String>> = @Optics(E&.E3 &> at(2) &> capitalize
	}
}
```

Composition with function using `<&`. Feels even more useless
```swift
main() {
	let lambda = { => 
		let builder: Build<E, String> = @Optics(E&.E3 &> at(1) <& capitalize)
	}
}
```

Composition of lens and prism (Serializable Example)

```swift
main() {
	let dm: DataModel;
	let deserialziePrism: Prism<DataModel, A> = ...
	let lambda = { => 
		let setXCapitalized: Update<DataModel, String> = @Optics(deserializePrism &> A&.x <& capitalzied)
		let newDm = setXCapitalized.update(dm, "Bye");
	}
}
```