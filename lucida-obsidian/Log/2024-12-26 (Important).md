After a significant pause in all of that, finally found the courage to organize some meeting. Contrary to my expectations that was much more fruitful than I anticipated.

Added Andrew to the repo. Thanks to him I have some log of what's been discussed. ([[Meeting 2024.12.26|See here]]).

Want to compose some thought while they're fresh...
- First thing which came to my mind is to simply take Mikhail's DSL Kit and to finish the implementation of most basic optics. Meaning: 
	- Syntax for array indices `[0]`
	- Syntax for prisms `.?`, `as`, `enum`
	  - [ ] #TODO Make sure that's correct way of interpreting that
	- Maybe some traversal with ranges, but that feels unnecessary for future progress
  - [ ] #TODO Still feels somewhat useless. Whether it actually bring me closer to something
- An open question I currently have is about first-classness and locality. I almost do not have any doubts about optics being first-class entity but the question is where:
	- The language as a whole?
	- Some scope in which this optics is derived syntactically and passed around there
  Primitive approach definitely allows for the first one, but that would be incredibly slow (allegedly, these talks do not have any sense before something will be actually implemented)

A small look further is the question of composability. A very painful I'd say. After implementing the primitive representation there should occur some benchmarking of performance to see how slow those optical devices are.
- [ ] #TODO If implemented primitive version. Benchmark it to see what to do further

Even now I feel like composability is the biggest bottleneck. There's multiple ideas what to do with that:
	- Nothing. Believe in language optimizer
	- CPS-based (Look into DSL kit)
	- "Macro".
		- If explaining in smart words. Instead of constructing function we construct some Intermediate Representation (aka. Deep Embedding) of optics which can be reduced later via performing optimization passes (Exact rules of which are yet to be defined)
		- It can be done either during Macro expansion or right inside the compiler. Or maybe mix of both, where Macro generates embedding and compiler reduces it

If we thing even further. A concern about user defined optics getting raised. These implementations are very different in their extensibility:
	- Doing nothing and CPS-based are probably equally extendable
	- "Macro" somewhat worrying. 
		- If we only provide fixed set of optics. We can derive the rules for them and use in optimizations without any trouble
		- Allowing user specified optics raises the questions of verification of it's well-defined. So our optimization passes won't change the behavior of the code.

And there's special approach which is worth looking into - Swift KeyPath:
- [ ] #TODO Lookup about Swift KeyPath