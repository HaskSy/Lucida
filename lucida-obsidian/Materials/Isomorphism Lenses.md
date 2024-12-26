#PS blogpost is uninteresting and mostly confirms what's already there.

> Optics is a proof of existence of isomorphism

```haskell
-- Isomorphisms/bijections between type @a@ and @b@
data Iso a b = Iso { fw :: a -> b, bw :: b -> a }

-- Lenses with a data wrapper, in practice you might want to unpack the Iso type
data Lens a b = forall r. Lens (Iso a (b,r))
```

What's not really useful but interesting interesting here are laws which Van Laarhoven mentions and proves:

## Laws

There are two (or one, depending on how you count) obvious laws you want isomorphisms to satisfy:
```
fw i . bw i = bw i . fw i = id
```
On the other hand, there are several less obvious laws for lenses:
```
set l (get l a) a = a
get l (set l b a) a = b
set l c (set l b a) = set l c a
```

And now comes the magic: with isomorphism lenses all of these laws follow from the simple laws of isomorphisms. Here are the quick and dirty proofs. 

**One:**
```
  set l (get l a) a
= {- expanding definitions of get and set -}
  (bw l . first (const ((fst . fw l) a)) . fw l) a
= {- let x = fw l a, rewrite -}
  bw l (first (const (fst x)) x) where x = fw l a
= {- (first (const (fst x)) x) = x -}
  bw l x where x = fw l a
= {- fill in x and rewrite -}
  (bw l . fw l) a
= {- isomorphism law -}
  a
```

**Two:**
```
  get l (set l b a) a
= {- expanding definitions of get and set, rewrite to use composition -}
  fst . fw l . bw l . first (const b) . fw l $ a
= {- isomorphism law -}
  fst . first (const b) . fw l $ a
= {- expanding fst, first and const -}
  (\(x,y) -> x) . (\(x,y) -> (b,y)) . fw l $ a
= {- composing the two lambda terms -}
  const b . fw l $ a
= {- definition of const -}
  b
```

**Three:**
```
set l c (set l b a)
= {- expanding definition of set, rewrite to use composition -}
  bw l . first (const c) . fw l . bw l . first (const b) . fw l $ a
= {- isomorphism law -}
  bw l . first (const c) . first (const b) . fw l $ a
= {- first f . first g = first (f . g) -}
  bw l . first (const c . const b) . fw l $ a
= {- const c . const b = const c -}
  bw l . first (const c) . fw l $ a
= {- definition of set -}
  set l c a
```

