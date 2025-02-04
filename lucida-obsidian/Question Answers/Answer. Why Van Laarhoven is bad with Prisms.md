Van Laarhoven's encoding is displayed like so
```haskell
type Lens a b s t
	=  forall f. Functor f
	=> (a -> f b) -> s -> f t
 ```
 
 Profunctor encoding looks similar, simply generalizing over function arrow
```haskell
type Optic p a b s t 
	= p a b -> p s t
```

All of the existing libraries which rely on Van Laarhoven's encoding are using the second encoding, for Prisms, in which they're looking like
- [ ] #TODO Cite libraries

```haskell
class Profunctor p => Cocartesian p where
	left  :: p a b -> p (a + c) (b + c)
	right :: p a b -> p (c + a) (c + b)

type PrismP a b s t = forall p. Cocartesian p => Optic p a b s t
```

In Van Laarhoven's encoding, traverse is looking good:

```haskell
type Traverse a b s t
	= forall f. Applicative f
	=> (a -> f b) -> s -> f t
```

And we're fairly close to encoding affine traverse, but there's a catch:
```haskell
type AffineTraverse a b s t
	= forall f. Functor f
	=> (a -> f b) -> s -> Either t (f t)
```
This encoding of affine traverse makes apparent why it's second name is Optional

- [ ] #TODO Finish off the experimentation with Van Laarhoven's encoding for Prism and Affine Traverse