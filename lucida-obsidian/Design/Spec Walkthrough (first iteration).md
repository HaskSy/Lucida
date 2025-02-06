It's about time to come up with something. I'll try to go through each section of language specification and try to imagine the most perfect and beautiful optics,. which comes to my mind. This version will be very invasive and won't try to be  very diligent of other features. This document is mostly to collect use cases

## 1. Lexical Structure

Let's introduce 2 operators:

- `&.` - Optic access operator
- `&>` - `flatMap` or optics composition operator (not sure yet)

Their use cases are yet to be decided, maybe they won't be present in the language at all, but they will be used as a "language" when we will be talking about optics in some context

## 2. Types

### Tuples

- [ ] #DESIGNQ How should optics work with tuples?
	> That's probably the simplest of them all. If optics should work with tuples, then there's only some options, that come to mind:
	>
	> - Prism: not an option here 
	> - Lens: A magical lens literal, named `nth(UInt64)` which would work something like that:
```swift
	let second: Lens<(...), String> = (Int64, String, Object).nth(2)
```
>	  -  Lens: An operator `&.` which would behave similarly to member referencing `Type&.optic` **(primary choice)**
```swift
	let second: Lens<(...), String> = (Int64, String, Object)&.[2]
```
>	  - Lens: A scope, in which member accessing behaves differently: (`do`?)
```swift
	let second: Lens<(...), String> = optics { (Int64, String, Object)[2] }
```
>	  - Traverse: Can theoretically be combined with `Range`, but there's lots of catches:
>		  - All types should be the same over the field we work with via ranges. Which is impossible to check at compile time (Runtime typing errors is not what we want), unless it's a literal
>		  - An example of syntax (Range Types):
>			  - `(...)&.[1, 2]`
>			  - `(...)&.[1..3]`
>			  - `(...)&.[1,7..10,15]`
>		  - An example of syntax (Slice Types):
>			  - `(...)&.[1, 2]`
>			  - `(...)&.[1:3]`
>			  - `(...)&.[1,7:10,15]`
>			  - `(...)&.[:]` - `Why?...`

- [ ] #DESIGNQ Think about subtyping here and how lack of variance blocks us here (Full stop?)
- [ ] #DESIGNQ Should this actually be a traverse? Or it might be a lens from `tuple` to `tuple`. We know the size of tuple in compile time anyway
### Range Type

- [ ] #DESIGNQ How should optics work with Range types?
	> I'd rather not to touch this, right now, because it's not really that important
### Slice Type

- [ ] #DESIGNQ How should optics work with Range types?
	> I'd rather not to touch this, right now, because it's not really that important

### Function Type
- [ ] #DESIGNQ How should optics work with Function types?
	> A suggestions which might occur is whether to interpret optics as functions themselves. An answer to which will be dependent very much on encoding and public API. Whether that would be necessary. But right now let's assume that optics is capable of being a functional object and first class
- [ ] #TODO Come up with some examples

### Enum Type
- [ ] #DESIGNQ How should optics work with Enum type (exaustive)?
	> let's say we have the following enum `E` from the spec:
```swift
enum E {
	| mkE
	| mkE(Bool)
	| mkT(String)
}
```
> There is no easy way to reference overloaded constructor of enum:
```swift
let e: E = mkE
let f: (Bool) -> E = mkE
```
The use case that comes up to mind here is chaining operations over `Option<T>` (aka `Maybe Monad`)
```swift
let x: ?X = X()
let res = x.flatMap(op1).flatMap(op2).flatMap(op3)
```
For shortness, let's make it the following:
```swift
let res = x &> op1 &> op2 &> op3
```
- [ ] #PROBLEM Variance in return type? 
	Sometimes it's `map     :: Option<T> -> (T -> R) -> Option<R>`
	Sometimes it's `flatMap :: Option<T> -> (T -> Option<R>) -> Option<R>`

Now talking about why such syntactic sugar would be needed:

> For simplicity we will consider the case without overloaded constructors:
- [ ] #DESIGNQ think about enum constructor overloading

Let's say we access

- [ ] #TODO На работе пришлось делать работу... Продолжить в пятницу