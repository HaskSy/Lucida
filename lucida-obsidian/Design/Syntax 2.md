> Like there was any prequel to that...

Alright, the syntax. With current set of ideas i've concluded on something like that:

## Option 1 - Constraint based

**Lens** - `.*`
**Prism** - ???
**Option** - `?.*`
**Iso** - `.convert<Type>()` 
> Iso has some issues, which can't be easily fixed with current implementation...
  `Optics<T, StructCoercibleToString>.convert<Array<Byte>>` is impossible, you have to perform each step:
  `Optics<T, StructCoercibleToString>.convert<String>().convert<Array<Byte>>()`. That's basically a different feature, so, ignoring it

**Setter** - ???

In that case user defined optics (whatever is it's kind), should specify somehow where it belongs and it will simply work. Even some overloading over optics will work (but why)

## Option 2 - Implementation based

**Derived Lenses/Prisms** - `.Identifier
> F\*ck constructor overloading.
> I can, of course come up with something like:
```swift
 @Lucida(x.y.Cons(_,.))
```
> But ugh...

**User defined optics** - `.CallExpr(args...)`
**Derived Iso** - `convert<Type>()` (We will just create a special method)
**First Class** - `@Ref(identifier).@Ref(identifier)`.
```swift
let custom = @Lucida(@Type(StructCoercibleToString).convert<String>().convert<Array<Byte>>()) // produces first class object

@Lucida(x.@Ref(custom) = Byte(42))
```
