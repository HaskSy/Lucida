Here I'm trying to figure out what kind of optics can be provided to used by default. Then I'm going to; ask myself a question of how it should look. Not necessarily each of those features will have a separate syntax to support it, but all of them better be listed in thought of making one.

Here's the list of options:
- Copy on write mutation of `public var` field of `struct` and `class`.
	- Meaning the mechanism of mutation of variables for those entities will be the same, thus allowing for things, which were limited before. (look in CoW and closures)
	- [ ] #TODO Write an extensive explanation of usability of this feature