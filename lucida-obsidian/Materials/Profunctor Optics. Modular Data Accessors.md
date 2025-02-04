- [ ] #TODO Make some Excalidraw pictures of optics
## Section 1

**View-update problem** is formulated here as follows:

> A lens onto a component of type `A` within a larger data structure of type `S` consists of a function `view :: S -> A` that extracts the component from its context, and  a function `update :: (A, S) -> S` that takes a new component of type `A` and an old data structure of type `S` and yields a new data structure with the component updated

```haskell
data Lens s t a b = Lens { view :: s -> a, update :: (b, s) -> t }
```

What's also feels nice is **Downcast/Upcast analogy** for `Prisms`:

> Given a compound data structure of type `S`, one of whose possible variants is of type `A`, one can access that variant via a function `match :: S -> S + A` that **downcasts** to an `A`  if possible, and yields the original `S` if not; conversely, one may update the data structure via a function `build :: A -> S` that **upcasts** a new `A` to the compound type `S`. Such a pair of functions if formally dual to a lens (in the mathematical sense of "reversing all arrows")

```haskell
data Prism s t a b = Prism { match :: s -> Either t a, build :: b -> t }
```

An example of why these primitive abstractions are **Broken**:

> We might want to access the `A` component of a compound type `(1 + A) x B` built using both sums and products. We might further hope to be able to construct this accessor onto the optional left-hand component by composing the prism $\textit{the}$ onto an optional value and the lens $\pi_1$  onto a left-hand component. However, the composite accessor is not a lens, because it cannot guarantee a view as an `A`; neither is it a prism, because it cannot build the composite data structure from an `A` alone. We cannot even express this combination; our universe of accessors is not closed under the usual operations for composing data. **The abstraction is clearly broken.**

Paper mentions that the true, closed composability is not achievable in simplistic representations. But **Profunctor optics** is supposed to fix that.

Reading imposed the following questions: 
- [x] #TODO Try to realize what's so lattice about optics. (I guess it's about inheritance, and optics hierarchy) ✅ 2025-02-04
	> Yes, It was just about optics hierarchy

## Section 2

This sections contains some general knowledge about optics I should be already aware of, but there still might be something
### Section 2.1 Adapters

```haskell
data Adapter s t a b = Adapter { from :: s -> a, to :: b -> t }
```
From what I've read, that's basically an `Iso` optics, but for whatever reason they do not call it that way.

It might be, because they explicitly state that they neither will rely on `from` and `to` functions being an inverse of each other nor they will enforce this property

- [x] #QUESTION Is `Adapter` is the same as `Iso`? And if so, why they won't enforce the optical properties of it ✅ 2025-02-04
> Yes, an `Adapter` is simply another name for `Iso`. About the second questing is an obvious question in response: How?....

### Section 2.2 Traversal

#NOTE Am I surprised to see mention of effectful computations in a paper recommended by Andrei? Not in the slightest

```haskell
data FunList a b t = Done t | More a (FunList a b (b -> t))

data Traversal s t a b = Traversal { extract :: s -> FunList a b t }
```

A container datatype is *traversable*, it it's data structure has a finite number of elements, and an ordering on the positions of those elements is specified.

- [x] #QUESTION "Because the ordering on positions is explicit, one may safely apply an effectful operation to each element." Huh?... ✅ 2025-02-04
> I feel very mach disagreeable on that take because in that way we implicitly rely onto the implementation of `Traversable` and if it will change, lots of things can go wrong. If anything then, `Traversable` implementations should be closed of changes then. **BUUUUUUT!!! With `Traversable` being implemented as optics, it becomes first-class citizen thus allowing us not to change the implementation of `Traversable` everywhere, but to change it only in those places where we intend to change the optic.** I guess I managed to answer on that question myself, but i need to make sure

### Section 2.3 Traversals as concrete optics 

>The type $(A \rightarrow F B) \rightarrow (S \rightarrow F\, T)$ is almost equivalent to the pair of functions $\texttt{contents ::}\; S \rightarrow A^n$ and $\texttt{fill ::}\; S \times B^n \rightarrow T$

That's I kinda believe and understand because those "witnesses of traversability" are simply [[Tweag. Existential Optics#Van Laarhoven encoding| Van Laarhoven encoding]] but what I do not understand is that:

- [x]  #QUESTION ✅ 2025-02-04
    > for singleton containers $(n = 1)$ this specializes both to lenses and to prisms. 
	
    Why would that even be the case for prisms? Seems like it's not true or too big of a stretch with $n$ being both 0 and 1

	> I guess the answer is in the fact, that under the constraint `Applicative f` this function becomes a signature of `traverse` method. Which is above in hierarchy than both `Prism` and `Lens`
	- [ ] #TODO Maybe It's worth to provide an example
 
- [ ]  #QUESTION$\text{Traversable } S \cong \exists n. A^n \times (B^n\rightarrow T)$ Why is that? Try look up to [[https://doi.org/10.1145/2578854.2503781|paper]]

## Section 3 Profunctors

Just in case you forgot, dummy:

```haskell
-- Profunctor is defined by the following typeclass
class Profunctor p where
	dimap :: (a' -> a) -> (b -> b') -> p a b -> p a' b'

-- While simultaneously satisfy the following laws:
-- 
-- dimap id id = id
-- dimap (f' . f) (g . g') = dimap f g . dimap f' g'
-- 
```

> Think of $P\,A\,B$ as a type of "transformers that consume As and produce Bs"

Another, analogy worth to look back upon:

> ... think of `dimap f g h` as 'wrapping' the transformer h in a preprocessor 'f' and a postprocessor 'g' 

**Stating the obvious:** profunctor is covariant by what it produces (aka. 'b') and contravariant by what it consumes (aka. 'a')

- [x] #QUESTION What's the importance of the statement above for lecture? ✅ 2025-02-04
> Not much

An obvious example of profunctor is (Obviously function):
```haskell
instance Profunctor (->) where
	dimap f g h = g . h . f
```

### Cartesian Profunctor
A next step in modifying how profunctor should behave is to create a profunctor with context.

```haskell
class Profunctor p => Cartesian p where
	first  :: p a b -> p (a, c) (b, c)
	second :: p a b -> p (c, a) (c, b)
```

Laws are getting weirder and oddly specific, almost unrealistic to some extent?

```haskell
runit  :: (a, ()) -> a -- A x 1 -> A
runit  (a, ()) = a

runit' :: a -> (a, ()) -- A -> A x 1
runit' a = (a, ())

assoc  :: (a, (b, c)) -> ((a, b), c) -- A x (B x C) -> (A x B) x C
assoc' :: ((a, b), c) -> (a, (b, c)) -- (A x B) x C -> A x (B x C)

dimap runit runit' h  = first h
dimap assoc assoc' (first (first h))  = first h

-- Same laws for the 'second' but symmetrical
```

Looking back to our old friend `(->)` instance of `Cartesian` for it would look something like that:

```haskell
cross :: (a -> c) -> (b -> d) -> (a, b) -> (c, d)
cross f g (x, y) = (f x, g y)

instance Cartesian (->) where
	first  h = cross h id
	second h = cross id h
```

### Cocartesian Profunctor

Everything is very much similar to the above

```haskell
class Profunctor p => Cocartesian p where
	left  :: p a b -> p (a + c) (b + c)
	right :: p a b -> p (c + a) (c + b)

-- Laws
--
-- rzero :: a + 0 -> a; rzero :: a -> a + 0
--
-- dimap rzero rzero' h = left h
-- dimap coassoc' coassoc (left (left h)) = left h
--
-- Symmething thing for right
```

example instance for an arrow is:
```haskell
instance Cocartesian (->) where
	left h = plus h id
	right h = plus id h

plus :: (a -> c) -> (b -> d) -> Either a b -> Either c d
```

>So too are functions with structured results, provided that there is an injection $A \rightarrow F A$ of pure values into that structure
 - [x] #QUESTION  what does that even mean? This leads to constraint `Applicative` instead of `Functor` with bonuses. Which seems unnecessary, because of `<*>` ✅ 2025-02-04
 > The answer is that, if classical hierarchy was more fine grained, then we'd see a typeclass named `Pointed` which has the following signature:
 ```haskell
 class Functor f => Pointed f where
	 pure :: a -> f a
 ```

```haskell
instance Applicative f => Cocartesian (UpStar f) where
  left (UpStar unUpStar) = UpStar (either (fmap Left . unUpStar) (pure . Right))
  right (UpStar unUpStar) = UpStar (either (pure . Left) (fmap Right . unUpStar))
```

### Monoidal Profunctor

```haskell
class Profunctor p => Modoindal p where
  par :: p a b -> p c d -> p (a, c) (b, d)
  empty :: p 1 1
  
-- Laws
--
-- dimap assoc assoc' (par (par h j) k) = par h (par j k)
-- dimap runit runit' h = par h empty
-- dimap lunit lunit' h = par empty h
--
-- Symmething thing for right
```

```haskell
instance Monoidal (->) where
  par = cross
  empty = id
```

```haskell
instance Applicative f => Monoidal (UpStar f) where
  empty = UpStar pure
  par h k = UpStar (pair (unUpStar h) (unUpStar k))

pair :: Applicative f => (a -> f b) -> (c -> f d) -> (a, c) -> f (b, d)
```

## Section 4. Optics in terms of profunctors

```haskell
type Optic p a b s t = p a b -> p s t
```

### Adapter profunctor

```haskell
type AdapterP a b s t = forall p. Profunctor p => Optic p a b s t

adapretC2P :: Adapret a b s t -> AdapterP a b s t
adapretC2P (Adapret o i) = dimap o i
```


- [ ] #FINISH  Finish section
## Section 5. Composing profunctor optics
- [ ] #FINISH   Finish section
## Section 6. Related work
- [ ] #FINISH   Finish section
## Section 7. Discussion

Paper also subtly mentions `AffineTraversal` but they question it's usability as is.
- [ ] #TODO Consider doing it

> Ill-behaved optics fits under constraints just as good as well-behaved one.

That's seems like true for any optics engine
- [ ] #FINISH   Finish section
