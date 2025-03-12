First things first. I've never seen Swift language. So before going further I'll list the cheat sheet of explanation and links for unknown to me, constructions.

- [`@inlinable` attribute](https://swiftrocks.com/understanding-inlinable-in-swift)
  An attribute, is helping with inlining calls outside the module, where function declaration is contained other.
  > Cross-module inlining only makes sense for public declarations. Which is obvious
  
  Another interesting thing:
  >If the `@inlinable` method happens to be calling other methods internally, the compiler will ask you to also expose these methods. This can be done either by also making them `@inlinable` or by marking them with `@usableFromInline`, which indicates that althoug the method is being used from an inlined method, it itseld is not really supposed to be inlined

- [`@_show_in_interface` attribute](https://github.com/swiftlang/swift/blob/main/docs/ReferenceGuides/UnderscoredAttributes.md#_show_in_interface)
  #GUESS - generated interface is public interface for `ObjC` to be called.