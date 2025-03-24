This summary contains most of things found so far which block us from comfortable implementation

## Extend bug

```swift
interface Test<T, G> { }
// interface Test<T> { }

// public struct Wrapper<T> { }
// public struct Pair<T, G> { }

extend <T, G> Array<(T, G)>       <: Test<T, G> { } // FAILS
extend <T, G> Wrapper<Pair<T, G>> <: Test<T, G> { } // FAILS
extend <T, G> Array<(T, G)>       <: Test<T>    { } // OK
extend <T, G> Array<(T, G)>       <: Test<G>    { } // OK
extend <T, G> Wrapper<Pair<T, G>> <: Test<T, T> { } // OK
extend <T, G> Wrapper<Pair<T, G>> <: Test<G, G> { } // OK
extend <T>    Wrapper<Pair<T, T>> <: Test<T, T> { } // OK
extend <T>    Wrapper<Pair<T, T>> <: Test<T>    { } // OK
```

**Problem**
> We're using extend on `Magical<T>` for exporting of defining of optics.
> Exportation of optics outside package only happens if there's an interface bound to it.
>
> If we're writing an extend for some object with two generic parameters (it's not about polymorphic optics), then we cannot export this optics outside of package.
>
> Also we cannot support tuples because of that. At all

**Solution**
-  Fix `extend`
or
- wait fix
or
- Make `public extend` 
## Default interface implementation bug

```swift
func call(v1: () -> Unit): Unit {
	v1()
}

interface DefaultImpl {
	func test(): Unit { println("Hello, World!") }
}

struct A <: DefaultImpl { }

main(): Unit {
	call(A().test)
}
```

**Problem**
> We're using extend on `Magical<T>` for exporting of defining of optics.
> Exportation of optics outside package only happens if there's an interface bound to it.
>
> If we're writing a default implementation inside of the interface. Then it will cause codegen error.
>
> It's easily avoidable, by generating different code, but still unpleasant

**Solution**
- Fix by itself
or
- wait fix

## Inlining won't inline

**Problem**
> After rewriting everything to the new scheme, which supports composition of different optics inlining simply refused to work. Overhead is somewhat on par with 2 virtual calls
>
> It's probably due to now, existing multiple "Magical" structs `Lenses`, `Prisms`, and `Optionals` and optics implementations existing in different struct, thus they don't really compose

| Case           |   Median |        Err |  Err% |     Mean |   Ratio |
| :------------- | -------: | ---------: | ----: | -------: | ------: |
| nativeBaseline | 2.969 ns | ±0.0122 ns | ±0.4% | 2.965 ns |    100% |
| opticsBaseline | 16.81 ns | ±0.0285 ns | ±0.2% | 16.86 ns | +467.9% |
| reconstruction | 2.535 ns | ±0.0101 ns | ±0.4% | 2.546 ns |  -14.2% |

| Case           |   Median |        Err |  Err% |     Mean |   Ratio |
| :------------- | -------: | ---------: | ----: | -------: | ------: |
| reconstruction | 140.5 ns |  ±0.656 ns | ±0.5% | 143.3 ns |    100% |
| opticsBaseline | 1.304 us | ±0.0128 us | ±1.0% | 1.355 us | +864.4% |
**Solution**
- `@Attribute[ForceInline]`
or
- Making optics a compiler primitive with it's own inlining rules and resolve
or
- Improve existing inlining
or
- Something else?
c
## Optics is mostly static but in fact, it's not

```swift
@DeriveLenses
public struct A {
	...
}

main() {
	@Lucida(A.x.y) // this doesn't work
}
```
**Problem**
> Optics is mostly static entity, so it would be logical for first classness to make it possible to access it by type name. Currently optics is only accessible through an object (which is not always around)

**Solution**
- Separate syntax for statics:
  ```swift
@Lucida(\A.x.y) // Like swift
// or
@Lucida(@Type(A).x.y)
```
- Name resolution in compiler for special syntax for optics
  ```swift
A&.x.y
// same as
let t = A()
x&.x.y
```
