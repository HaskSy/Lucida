> Посмотрел на KeyPath. Выглядит как куча костылей, но забавно.
> 
> - [swift-evolution/proposals/0062-objc-keypaths.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0062-objc-keypaths.md)
> - [swift-evolution/proposals/0161-key-paths.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0161-key-paths.md)
> - [swift-evolution/proposals/0210-key-path-offset.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0210-key-path-offset.md)
> - [swift-evolution/proposals/0227-identity-keypath.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0227-identity-keypath.md)
> - [swift-evolution/proposals/0249-key-path-literal-function-expressions.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0249-key-path-literal-function-expressions.md)
> - [swift-evolution/proposals/0252-keypath-dynamic-member-lookup.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0252-keypath-dynamic-member-lookup.md)
> - [swift-evolution/proposals/0369-add-customdebugdescription-conformance-to-anykeypath.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0369-add-customdebugdescription-conformance-to-anykeypath.md)
> - [swift-evolution/proposals/0416-keypath-function-subtyping.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0416-keypath-function-subtyping.md)
> - [swift-evolution/proposals/0438-metatype-keypath.md at main · swiftlang/swift-evolution · GitHub](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0438-metatype-keypath.md)
> 
> Иерархия не имеет ничего общего с интерфейсом для работы (поддерживаются только линзы), вместо этого наследники просто обладают большим количеством типовой информации. Зачем базовые классы — загадка.
> `PartialKeyPath<Root> <: AnyKeyPath`
> `KeyPath<Root, Value> <: PartialKeyPath<Root>`
> 
> Применение `KeyPath` только через магически сгенерированные для всех типов subscript'ы.
> 
> Чтобы `KeyPath` можно было использовать как функции, используется сахар завязанный на expected type.