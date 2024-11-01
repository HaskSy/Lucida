- [ ] #TODO The problems start from the beginning... What even is "existential optics"? What are the generalities between different implementations of this embedding?

#PS Due to suggestion to already start writing some text, even the most obvious things will be mentioned.

Different optics entities such as Lens, Prism and Traversals generalize already known notions. See table below

| Optics    | Generalization of         |
| --------- | ------------------------- |
| Lens      | Immutable structure field |
| Prism     | Constructor               |
| Traversal | Iterator?                 |
## Encoding kinds

Article mentions three different ways to implement optics:

### Explicit encoding

```haskell
data Lens s a = Lens 
	{ get :: s -> a
	, set :: s -> a -> s
	}
```

Encoding described in article as very simple, very ad-hoc and very verbose
### Van Laarhoven encoding

```haskell
type Lens s a
	=  forall f. Functor f
	=> (a -> f a) -> s -> f s
 ```

- [ ] #TODO Look at the [[Compositional Data Access and Manipulation|talk]] by Simon Peyton Jones
- [ ] #TODO Look at the [lecture](https://cs-uni.ru/index.php?title=%D0%A4%D0%9F_5SE_%D0%BE%D1%81%D0%B5%D0%BD%D1%8C_2023) by Denis Moskvin