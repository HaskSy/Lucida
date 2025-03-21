After being pleasantly surprised by benchmarking results for high-order functions inlining, if everything is marked with `@Frozen` we need to push language into somehow better way of composing things.

For now, I want to stick with only lenses. If it will work well, I'll try to introduce the whole Profunctor optics to also support `Prism` and `Option`