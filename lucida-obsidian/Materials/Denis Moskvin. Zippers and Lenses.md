Mostly interested in the second part

### Naive Lenses (Explicit Encoding)

```
lens :: (s -> a) -> (s -> a -> s) -> Lens s a
lens :: (s -> a) -> (s -> b -> t) -> Lens s t a b
```

Explanation of the laws:

- `view l (set l v s) = v`
	If we see some data structure `s` and store a value `v` through the lens `l`, then the result of `view` through the same lens `l` is `v`
- `set l (view l s) s = s` 
	If we see some value inside of structure `s` using lens `l` then storing that value in same structure `s` using lens `l` won't change the structure
- `set l v' (set l v s) = set l v' s`
	**"Forgetting law"**
	If we use multiple consecutive `set`'s on the same data structure using the same lens `l` it should be equivalent to applying just the last `set` 
	
then the same laws for lenses are mentioned in [[Isomorphism Lenses|Van Laarhoven's blog]] with proofs

> Negatives:
> - Ineffective. Requires some additional data structure existing in runtime
> - Problematic to compose
### Lan Vaarhoven encoding

```haskell
{-# LANGUAGE Rank2Types #-}
type Lens s a = forall f. Functor f => 
	(a -> f a) -> s -> f s
```

```haskell
lens :: (s -> a) -> (s -> a -> s) -> Lens s a
lens get set = \ret s -> fmap (set s) (ret $ get s)
```

No runtime functions overhead but getters and setters are not stored explicitly and should be obtained through special functor:

```haskell
newtype Const a x = Const { getConst :: a }

instance Functor (Const a) where
	fmap _ (Const v) = Const v
```

Thus allowing us to get values like so:
```haskell
-- ((a -> Const a a) -> s -> Const a s) -> s -> a -> s -> a
view :: Lens s a
view l s = getConst $ l Const s
```

#PS maybe it can be coerced

Setter are used through `Identity`
```haskell
newtype Identity a = Identity { runIdentity :: a }

instance Functor Identity where
	fmap f (Identity x) = Identity (f x)
```

Implementing `over` like so:
```haskell
over :: Lens s a -> (a -> a) -> a -> s
over l f s = runIdentity $ l (Identity . f) s
```

Worth noticing that Lenses work mostly only with product types, but it's not so clear how it should work for sum types.

**Prism** is a dual entity towards the **Lens**. Working exactly the same for sum-types like lens for product types. In cases the field does not exist

| Optics    | Foci count     |
| --------- | -------------- |
| Lens      | 1              |
| Prism     | 0-1            |
| Traversal | Natural number |
- [ ] #TODO Here's mentioned derivation of optics using Template Haskell. Maybe should look into it

### Summary

Not much to mention here. Only got a feeling that existential typing will flip me over when I man up to make first class optics user interface