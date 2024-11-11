
#PS Due to suggestion to already start writing some text, even the most obvious things will be mentioned.

Different optics entities such as Lens, Prism and Traversals generalize already known notions. See table below

| Optics    | Generalization of           |
| --------- | --------------------------- |
| Lens      | Immutable structure field   |
| Prism     | Constructor / Tagged Union? |
| Traversal | Iterator? / Containers?     |
## Encoding kinds

Article mentions three different ways to implement optics:

### Explicit encoding

```haskell
data Lens s a = Lens 
	{ get :: s -> a
	, set :: s -> a -> s
	}
```

Encoding described in article as very simple, very ad-hoc and very verbose. Which is very much believable, there's no composability
### Van Laarhoven encoding

```haskell
type Lens s a
	=  forall f. Functor f
	=> (a -> f a) -> s -> f s
 ```

- [ ] #TODO Look at the [[Compositional Data Access and Manipulation|talk]] by Simon Peyton Jones
- [x] #TODO Look at the [lecture](https://cs-uni.ru/index.php?title=%D0%A4%D0%9F_5SE_%D0%BE%D1%81%D0%B5%D0%BD%D1%8C_2023) by [[Denis Moskvin. Zippers and Lenses|Denis Moskvin]] ✅ 2024-11-11

Article states Van Laarhoven encoding works well with Traversals, but not so great with Prisms.
- [ ] #TODO Find out why Van Laarhoven encoding does not work good with Prisms and what's "well"

### Profunctor encoding

- [ ] #TODO Read up about profunctors.

```haskell
type Lens s a
  =  forall p. Strong p
  => p a a -> p s s
```

From what's stated in the article, only thing which is determines the type of optics are typeclass constraints imposed on profunctor `p` 

| Optics    | Typeclasses  |
| --------- | ------------ |
| Lens      | `Strong`     |
| Iso       | `Profunctor` |
| Prism     | `Choice`     |
| Traversal | -            |

- [ ] #TODO Finish up the table which maps kind of optic to the set of constraints
### Existential encoding

#### Lens
This is the primary section of the article. 
```haskell
data Lens s a
  = forall c. Lens (s -> (c, a)) ((c, a) -> s)
```
Same encoding is described in [[Isomorphism Lenses|Laarhoven's blog]].

 > A `Lens s a` is a proof that there exists a `c` such that `s` is isomorphic to `(c, a)`

Very explicit, very precise, but so what? My current feeling is that `c` is just some arbitrary, maybe randomly generated identifier by which we can understand how to return backwards. Some additional information to guarantee bijection.

This one is obvious way to get `view`
```haskell
view :: Lens s a -> s -> a
view (Lens f _) = snd . f
```

`over` is also straightforward. Get value, apply and go back?...
```haskell
over :: Lens s a -> (a -> a) -> s -> s
over (Lens f g) h = g . fmap h . f
```

Alright right now my previous assumption about `c` being some identifier does not make any sense... I'll better look it up

I see now. `c` is some type, which represents all of the information our lens is not "focused" on.

Now it's all coming together:

1. `f :: s -> (c, a)` takes a structure `s` and returns a pair `(c, a)`. Here, `c` represents context, and `a` is the part we want to modify.
2. Then we apply modifier to our pair, luckily:
   `fmap h (c, a) = (c, h a)`
3. And now we're collecting the structure back together with:
   `g :: (c, a) -> s`

#### Prism
Article also mentions how to build `Prism`:
```haskell
data Prism s a
  = forall c. Prism (s -> Either c a) (Either c a -> s)
```

> `Prism s a` is just a proof that there exists a `c` such that `s` is isomorphic to `Either c a`


```haskell
preview :: Prism s a -> s -> Maybe a
preview (Prism f _) = rightToMaybe . f
```

```haskell
review :: Prism s a -> a -> s
review (Prism _ g) = g . Right
```
Trying to reason about what's said...

Alright, I guess it's the following:

1. `s -> Either c a`: This function attempts to extract a value of type a from a structure of type `s`. If `s` does not contain an `a`, it returns a value of type `c` instead, indicating the failure to extract an `a`.
2. `Either c a -> s`: This function allows for the construction of an `s` either from the failed alternative `c` or directly from the a component. This allows `Prism` to reassemble `s` after modifying or viewing the `a` part.
#### Optic in general
```haskell
data Optic f s a
  = forall c. Optic (s -> f c a) (f c a -> s)
```

This section mentions a bunch of optics through some of data classes some of which I'm not aware:

```haskell
type Lens = Optic (,)

type Prism = Optic Either

type Iso = Optic Tagged

type Grate = Optic (->)

type AffineTraversal = Optic Affine

type Traversal = Optic PowerSeries
```

### Composing and optic types

With existential optics: the [[Optics Hierarchy|hierarchy]] of different kinds of optics is very explicit. Due to the fact of working with data classes instead of functions composability is very low.

You have to specify `Morph` typeclass:
```haskell
instance Morph Tagged (,) where
  type M Tagged (,) c = ()

  f2g :: Tagged c a -> ((), a)
  f2g (Tagged a) = ((), a)

  g2f :: ((), a) -> Tagged c a
  g2f ((), a) = Tagged a
```

- [ ] #TODO Is it worth trying to implement what's described here?

### Summary

Felt like pretty good and basic overview on what's out there