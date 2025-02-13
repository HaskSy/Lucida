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
- [ ] #DESIGNQ Range as argument of optics construction
- [ ] #DESIGNQ Extension method or something special for tuples to check in compile time correctness of taken argument?
### Range Type

- [ ] #DESIGNQ How should optics work with Range types?
	> I'd rather not to touch this, right now, because it's not really that important
### Slice Type

- [ ] #DESIGNQ How should optics work with Slice types?
	> I'd rather not to touch this, right now, because it's not really that important

### Function Type
- [ ] #DESIGNQ How should optics work with Function types?
	> A suggestions which might occur is whether to interpret optics as functions themselves. An answer to which will be dependent very much on encoding and public API. Whether that would be necessary. But right now let's assume that optics is capable of being a functional object and first class
- [ ] #TODO Come up with some example
- [ ] #DESIGNQ Automatic optics deconstruction into functional type

### Enum Type
- [ ] #DESIGNQ How should optics work with Enum type (exaustive)?
- [ ] #DESIGNQ Pattern synonyms, custom patterns, prisms as patterns
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

```swift
enum E {
	| mkE(Bool)
	| mkT(String)
}
// Let's say we have an instance of enum
let x = mkE(false)
// some computation
func comp(input: String): Int64;
// And we want to perform some set of operations if this is `mkT`
let result = x &. mkT &> comp
// Being similar to:
let result = if (let mkT(s) -> x) { Some(comp(s)) } else { None }
// Maybe It's reasoanble to consider following syntax?

let result = E&.mkT(x) &> comp
```
It's a weird prism, but prism nonetheless: 
$$\text{Prism}\,s\,t\,a\,b \cong \text{Prism}\quad\texttt{E}\quad\texttt{Option<Int64>}\quad\texttt{String}\quad \texttt{Int64} $$
- [ ] #QUESTION Is it a polymorphic prism or an affine traversal? I'm yet to figure how to interpret one? Also, is this the same?

$$\text{AffineTraverse}\,s\,t\,a\,b \cong \text{Prism}\quad\texttt{Option<s>}\quad\texttt{Option<t>}\quad a \quad b$$
But that was an example of polymorphic prism, Is we want to see a monomorphic example, then our options are quite limited:
- Either change value stored in enum and rebuild it
- Or create same enum, but with different label
- [ ] #TODO Find use cases for changing the enum value
- [ ] #TODO Find use-cases for creating label from same enum via another label

- [ ] #DESIGNQ What's the need for prisms if there's `prop` and `match (this)`? Them being first-class?
- [ ] #DESIGNQ Think about how it should work with associated types and whether they even matter
- [ ] #QUESTION There's a mention of **Implicit convert functions** in the spec. What's that?
### Option Type

As it is mentioned separately, I'll also talk about it separately. There's one particular feature of `Option<T>`, that I consider being quite interesting for prisms. A postfix `?` operator, which allows to make a function calls and member accessing over `Option` type. Which is basically an **Affine Traverse**.

An postfix `?` is basically a prism with the following signature:
$$\text{Prism}\quad\texttt{Option<a>}\quad\texttt{Option<b>}\quad a \quad b$$
It would be nice to generalize it for any enum in some way. There's a constraint that enum should be parametrized by single argument for it to work
```swift
let x = mkT("hallu")

Prism.of(x, mkT).trim().somethingElse() // Option<String>
```

If operation doesn't leave the domain of enum (we simply call a different constructor for the same enum), then type can be inferred to:

```swift
let x = mkT("hallu")

Prism.of(x, mkT).map(it -> mkE(it.size())) // E
```
- [ ] #DESIGNQ How should postfix `?`  be generalized for any enum?
### Array Type (Array\<T\>)

Let's start with some examples. For example, the lens `at`

$$ \text{at(Int64)} \cong \text{AffineTraverse}\quad\texttt{Array<a>}\quad\texttt{Array<a>}\quad \texttt{a} \quad \texttt{a} $$
Does nothing, simply look up the value if present

- [ ] #DESIGNQ Think of **Spread syntax** `*`

### VArray Type

Doesn't feel like there's any critical difference with `Array<T>`, it might be different in details, but from the interface and UX standpoint it better be almost the same as Array

- [ ] #DESIGNQ How should optics work with **VArray**

### Struct Type

Now that's a big one.

- [ ] #DESIGNQ Think how struct and field visibility would affect everything. For example. Let's say we have first class object `Lens`, and then we serialize it. After that we change the visibility of an object and deserialize it. Depending on the implementation, either nothing will happened or vulnerability will occur

> When a value is defined as **value type**, so the struct type copies the value when performing operations such as assignment or function passing
- [ ] #DESIGNQ Name resolution
## Summary
```tasks
filename includes Spec Walkthrough (first iteration)
```
