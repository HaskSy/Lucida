## First Iteration

> https://github.com/swiftlang/swift-evolution/blob/main/proposals/0062-objc-keypaths.md

`#keyPath(chain.of.accessors)` is an expression which represents `String` literal which is being compile time checked for it's validity. Not much there and not really viable for `CJ` because it doesn't have `Prototypes` and field accessing by `String`

## Second Iteration 

> https://github.com/swiftlang/swift-evolution/blob/main/proposals/0161-key-paths.md

`\Type.chain.of.accessors` is an expression which represents `KeyPath` object which is itself, simply a reference to a field accessor.

These references are note made equal due to them not possessing the same information in different context of their creation:

- **Unknown Path / Unknown Root Type** - Backwards compatibility with `ObjC` which does not store any information about validity of provided key path
- **Unknown Path / Known Root Type** - Matches with any route from known root type
- **Known Path / Known Root Type** - We know exactly where to seek the value of the field
- **Value/Reference Mutation Semantics Mutation** - Allows not only getting the value like the previous one, but also modifying it

It's worth mentioning, that people from **Swift** had some concerns about ABI stability which I'm not really understanding now:

> This feature adds the following requirements to ABI stability:
 - mechanism to access key paths of public properties
 >
> We think a protocol-based design would be preferable once the language has sufficient support for generalized existentials to make that ergonomic. By keeping the class hierarchy closed and the concrete implementations private to the implementation it should be tractable to provide compatibility with an open protocol-based design in the future.

## Third Iteration

> https://github.com/swiftlang/swift-evolution/blob/main/proposals/0210-key-path-offset.md

They mention somewhat useful API for accessing an access of Memory layout offset.

- [ ] #TODO Check whether our API for unsafe has something similar. That might be a good use-case

Also they suggest the following API. And I'd also like to look for presence of similar API in the language
```swift
extension UnsafePointer {
  subscript<Field>(field: KeyPath<Pointee, Field>) -> UnsafePointer<Field> {
    return (UnsafeRawPointer(self) + MemoryLayout<Pointee>.offset(of: field))
      .assumingMemoryBound(to: Field.self)
  }
}

extension UnsafeMutablePointer {
  subscript<Field>(field: KeyPath<Pointee, Field>) -> UnsafePointer<Field> {
    return (UnsafeRawPointer(self) + MemoryLayout<Pointee>.offset(of: field))
      .assumingMemoryBound(to: Field.self)
  }

  subscript<Field>(field: WritableKeyPath<Pointee, Field>) -> UnsafeMutablePointer<Field> {
    return (UnsafeMutableRawPointer(self) + MemoryLayout<Pointee>.offset(of: field))
      .assumingMemoryBound(to: Field.self)
  }
}
```

## Fourth Iteration

https://github.com/swiftlang/swift-evolution/blob/main/proposals/0227-identity-keypath.md

A neutral element over concatenation of paths

- [ ] #TODO Think about this concatenation, and how does it connected with `Self` in the language spec

##  Fifth Iteration

https://github.com/swiftlang/swift-evolution/blob/main/proposals/0249-key-path-literal-function-expressions.md

That feels like a big one. They want to allow `KeyPath` being used as Static getter, thus making them a functions `(Root) -> Value`.

- [ ] #TODO Think about side effects, which are encapsulated inside of closure. When and how they should be called

## Sixth Iteration

https://github.com/swiftlang/swift-evolution/blob/main/proposals/0252-keypath-dynamic-member-lookup.md

`@dynamicMemberLookup` over `KeyPath` in addition to `String`

> That's as lens, as Lens can get.

- [ ] #TODO It's said for the main alternative to not doing this at all. Why if String based dynamic lookup is already present

## Seventh Iteration

https://github.com/swiftlang/swift-evolution/blob/main/proposals/0369-add-customdebugdescription-conformance-to-anykeypath.md

- [ ] #NOTSTARTED It doesn't seems to be really interesting right now. I'll skip this one

## Eighth Iteration

https://github.com/swiftlang/swift-evolution/blob/main/proposals/0416-keypath-function-subtyping.md

I guess that's self apparent If we'll manage to make them first class, behaving similar to function
## Ninth Iteration

https://github.com/swiftlang/swift-evolution/blob/main/proposals/0438-metatype-keypath.md

- [ ] #TODO They've encountered some problem, related to static prorepties. Look into possibly ocurring problems in our language