Here I'm trying to figure out what kind of optics can be provided to used by default. Then I'm going to; ask myself a question of how it should look. Not necessarily each of those features will have a separate syntax to support it, but all of them better be listed in thought of making one.

Here's the list of options:
- Copy on write mutation of `public var` field of `struct` and `class`.
	- Meaning the mechanism of mutation of variables for those entities will be the same, thus allowing for things, which were limited before. (look in CoW and closures)
	- [ ] #TODO Write an extensive explanation of usability of this feature
- Generation of optics for `public var` fields. (Same thing, but other words)
- Composition of these field accessors
- Prisms for enum constructors
	- A simple example of usecase. The necessity to verify that match did not pass to get else bracket. Simple example to achieve it - coalescing over Prism, applied to value
	- An other example might be reconstruction of enum with different value. Or a completely different enum
	- [ ] #TODO Write an extensive explanation of usability of this feature
- Composition of prisms for nested, recursive and mutually recursive enums
- Composition of prisms and lenses
- Prisms for `sealed` and `open` classes (same as with `enums`)
- Prisms feel very like pattern matching. So, why not allow for using them inside `match-case` expressions
	- [ ] #TODO Look whether language allows for pattern matching inside the `case` of `match-case` without statement expression. If not, that's a use case
- We can also say that cast is a prism but I can't come up anything useful to that
	
 That feels like that's all that we can predefine.... We'll see

> While thinking about all of those "basic" usecases I felt like willing to use the optics in 2 very different ways.
> - An pair of functions which didn't capture the value it should operate on yet
> - An object, which captured an object to operate, and the only thing left is to use it
> I'm not sure whether this usecase can harmonically coexist

> The second thing I noticed in my head that I rarely want to walk in both directions of optics.
> For example, if my feature is to use the optics in `match` expression then I would use `match()` part of optics (Duh).
> So I want to suggest new classification for features, which will use optics, which is `direction`:
> - `forward-directional` (read?)
> - `backwards-directional`(write?)
> - `uni-directional` (both?)
> 
> But this begs me to ask whether that's the same as:
> - Construction syntax
> - Composition syntax
> - Elimination syntax
> 
> Feels like it is.

Ok, after this thought process I feel like I'm finally ready to design at least something. The main thing is to design it very quick so I will not forget