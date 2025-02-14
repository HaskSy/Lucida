Well, this meeting felt great actually. Got back the trust into that something actually good might occur from this diploma.

So. Here's the what we came to

### 1. Engine
The optical engine is not of the most importance. So the current designing won't be thinking about it. But still that's worth to mention that engine defines 2 things:
	- Performance (Who asked?...)
	- Possible to implement features. (We'll come up with lot's of them, so that's shouldn't be an issue)
  Due to **engine** being hidden behind the user interface, it's better to think of that.

### 2. User Interface
The interface that I've used in demonstration was labeled as horrendous... And I concur. This "clusterf\*ck" of `&.` and `&>` makes the language look more like Haskell or OCaml that human-readable language.

But I believe that the places where new syntax occured are right. It is just the examples, and the idea behind them is what should be slightly changed. And maybe should use different symbols, but that's not important.

We came to the naming 3 types of syntax which I insist on keeping as official and try to classify everything according to that:
- Construction syntax
- Composition syntax
- Elimination syntax
### 3. Optics Hierarchy
It is an open question whether to include some optics or not, and whether every part of interface of optical device is needed. Meaning exactly:
- Should `Traverseable` be considered as it is requires existance of applicative functor in the language
- Do we actually need `build` part for prism? We already have constructions
### 4. Resolve
- [ ] #TODO Finish the resolve section. There might be a lot to say
### 5. Construction Interface
There's 2 kinds of interfaces for the construction.
- Derived
- User-defined
The question is how each should look
### 6. Features
- Befriend `mut` keyword of struct with that feature
- \*Ownership
- \*User-defined patterns
- \*uniplate (derivfe foldable, traversable, ...)

## Summary

There's lots of things to do and to think of:
- [ ] #DESIGNQ Decide over construction syntax
- [ ] #DESIGNQ Decide over elimination syntax
- [ ] #DESIGNQ Decide over composition syntax
- [ ] #DESIGNQ Decide over optics to be supported
	> For now `Iso`, `Lens`, `Prism`, `AffineTraverse`, `Setter`, `Fold`
- [ ] #DESIGNQ Decide over derived optics syntax
- [ ] #DESIGNQ Decide over user-defined optics syntax
- [ ] #DESIGNQ Decide over  befriending with `mut` 
- [ ] #DESIGNQ Decide over process of storage of optics in language
- [ ] #TODO Look more into uniplate
- [ ] #DESIGNQ Think about user-defined patterns and exaustiveness