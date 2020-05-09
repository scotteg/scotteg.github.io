---
title: Protocol-Oriented Programming
excerpt: "Protocol-oriented programming enables writing code that is more modular, decoupled, and flexible, compared to traditional object-oriented programming. Not surprisingly, protocols are at the heart of protocol-oriented programming. In this article, I'll explain what protocol-oriented programming is and how to adopt this paradigm using protocols to define requirements and provide default implementations, while also ensuring optimum performance."
tags: [swift, ios, protocol-oriented-programming, protocols, dynamic-dispatch, static-dispatch]
date: 2020-04-22
---

[Back]({% link index.md %})

# [Protocol-Oriented Programming](#capture-lists)

#### Posted by [scotteg](https://www.linkedin.com/in/scotteg/) on April 22, 2020

#### Contents
- [Introduction](#introduction)
- [Understanding Protocols](#understanding-protocols)
- [Defining and Adopting Protocols](#defining-and-adopting-protocols)
- [Extending Protocols](#extending-protocols)
- [Dynamic vs. Static Dispatch](#dynamic-vs-static-dispatch)
- [Dispatch with Protocols](#dispatch-with-protocols)
- [Dispatch Type Flowchart](#dispatch-type-flowchart)
- [Conclusion](#conclusion)

---

## [Introduction](#introduction)

Protocol-oriented programming enables writing code that is more modular, decoupled, and flexible, compared to traditional object-oriented programming. Not surprisingly, protocols are at the heart of protocol-oriented programming. In this article, I'll explain what protocol-oriented programming is and how to adopt this paradigm using protocols to define requirements and provide default implementations, while also ensuring optimum performance.

## [Understanding Protocols](#understanding-protocols)

In traditional object-oriented programming, classes are primarily used to model objects. Objects can inherit state and behavior — e.g., properties and methods — from a parent class. Swift is single-inheritance, meaning, a class can only inherit from a single parent class. This approach can lead to an extensive class hierarchy, and code that is tightly coupled and rigid.

For example, in the following diagram, the `Vehicle` parent class is subclassed by `LandVehicle`, `WaterVehicle`, and `AirVehicle`. Each of these subclasses will inherit common features from the parent class, and add features specific to their unique implementation. Each of those subclasses are also subclassed, and those subclasses maybe be further subclassed, and so on, until the desired model structure can be defined.

# ![](Assets/SingleInheritanceObjectOrientedProgramming.png)

Expanding upon this example, `GasCar` might inherit a `milesPerGallon` property from `GasVehicle`, which inherits a `numberOfWheels` from `LandVehicle`, which inherits `passengerCapacity` from `Vehicle`. This is a reasonable implementation, because `GasCar` can be linearly derived within a single-inheritance model.

However, from which parent class would you subclass `HybridCar`? It needs to inherit state and behavior from both `ElecVehicle` and `GasVehicle`, so you would need to duplicate that state and behavior in a new subclass of `LandVehicle`. And how would you define `AmphiCar`, i.e., a car that can travel on land or water? What if `AmphiCar` could also be powered via gas or electricity?

This approach would quickly become unwieldily and lead to excessive duplication and rigidity. Additionally, type inheritance is not available for value types, such as structures and enumerations.

Protocols can help create a more flattened model hierarchy. They can be used with structures and enumerations, in addition to classes. And a type can adopt multiple protocols, similar to multiple class inheritance found in other languages.

The following animation shows how each concrete model type can adopt and conform to one or more protocols — i.e., `Electric`, `Gas`, `Wheeled`, etc. — to holistically define the requirements for that type. This approach keeps your model definitions modular and decoupled.

# ![](Assets/ProtocolOrientedProgramming.gif)

Unlike with class inheritance, in which the actual properties and methods are implemented, protocols only define requirements. This is still highly beneficial, because it enables you to expressively define types that are certain to have the expected state and behavior. This also aids in unit testing protocol-defined types, e.g., using mock objects that conform to the same protocols.

Protocols can themselves declare adoption of one or more parent protocols, and by doing so inherit requirements from multiple parent protocols.

Additionally, protocols can be extended to provide default implementations. This enables you to avoid duplicating code, and it keeps your model definitions flexible to be extended or modified later.

Protocol-oriented programming is not exclusive. It is common to use class inheritance and protocol-oriented programming techniques in the same project, and even in the same type. One reason why you may do this is that protocol extensions cannot implement stored state, i.e., it is not possible to set a property to a default stored value in a protocol extension. Therefore, a class may be subclassed to inherit a stored property, and then use protocols to add new methods to provide the behavior required for instances of that class.

## [Defining and Adopting Protocols](#defining-and-adopting-protocols)

Protocols are defined similarly to concrete types such as classes and structures, using the `protocol` keyword. Protocol type names should indicate either the requirements of the protocol or the behavior of a type that conforms to the protocol. For example, `BidirectionalCollection` or `Equatable`.

Required properties must indicate if they are read-only (`{ get }`) or read-write (`{ get set }`)<span id="a1">[¹](#1)</span>. Property requirements are defined using the `var` keyword, however the actual implementation may opt to define the property as a `let` constant if the requirement does not specify it must be read-write.

Method requirements only define the signature, not the actual implementation body. They also cannot define default parameter values. Associated types can be defined and referred to in the protocol definition, to be implemented via a type alias by the adopting type if the associated type cannot be inferred.

#### Example: Defining a protocol

```swift
protocol Drivable {
    associatedtype Direction
    var topSpeed: Double { get }
    func turn(direction: Direction, percent: Double)
}
```

In the above code, an associated type `Direction` is required, and a read-only `topSpeed` property and method `turn(direction:percent:)` are defined.

#### Example: Adopting and conforming to a protocol

```swift
// 1
struct Car: Drivable {
    // 2
    enum TurnDirection: String {
        case left, right
    }
    
    typealias Direction = TurnDirection
    var topSpeed = 180.0
    
    func turn(direction: TurnDirection, percent: Double) {
        print("Turning \(direction.rawValue) \(percent)%")
    }
}
```

In the above code:

1. A `Car` structure is defined and it declares adoption of the `Drivable` protocol from the previous example.
2. `Car` conforms to the protocol by defining an enumeration `TurnDirection` and assigning it to the `Direction` type alias, followed by implementing the additional property and method requirements.

As mentioned earlier, protocols can inherit from multiple parent protocols.

#### Example: Protocol multiple inheritance

```swift
protocol A { }

protocol B { }

protocol C: A, B { }
```

In the above code, protocol `C` inherits the requirements of protocols `A` and `B`.

Another way to declare inheritance of multiple protocols is by using *protocol composition*.

#### Example: Protocol composition
```swift
protocol A { }

protocol B: A { }

protocol C { }

protocol D: B & C { }
```

In the above code, protocol `D` adopts a *temporary unnamed protocol* that is composed of protocols `B` and `C` using the `&` operator. Because protocol B` adopts `A`, `D` will inherit all the requirements of `A`, `B`, and `C`. It is common to create type aliases when composing protocols, to give the composed protocol a meaningful name, e.g., `typealias BAndC: B & C`.

> **Note:** There are many additional aspects to working with protocols in Swift that go beyond the scope of this article. Consult the <a href="https://docs.swift.org/swift-book/LanguageGuide/Protocols.html" target="_blank">Swift language guide Protocols chapter</a> for more information.*

## [Extending Protocols](#extending-protocols)

Earlier you learned that extensions may be defined on protocols to provide default implementations. Extensions can only contain computed properties, not stored properties. Defining a computed property for something that only needs to be initialized once should be avoided, especially if its initialization is expensive. However, it's reasonable to provide a computed property to return a literal value without performance implications.

#### Example: Extending a protocol to provide default implementations

```swift
// 1
enum TurnDirection: String {
    case left, right
}

// 2
extension Drivable {
    var topSpeed: Double { 100 }
    func turn(direction: TurnDirection, percent: Double) {
        print("Turning \(direction.rawValue) \(percent)%")
    }
}

// 3
struct Car: Drivable { }

// 4
class MotorCycleViewModel: Drivable {
    var topSpeed = 200
}
```

In the above code:

1. `TurnDirection` is moved to the global space, because enumerations cannot be defined in a protocol extension.
2. An extension on the previously-defined `Drivable` protocol provides default implementations. The type alias was inferred and did not need to be explicitly defined.
3. The `Car` definition is empty. It will receive the default implementations defined in the extension.
4. A `MotorCycleViewModel` class is defined that adopts `Drivable`. It implements its own value for `topSpeed` and will receive the default implementation of `turn(direction:percent:)`.

## [Dynamic vs. Static Dispatch](#dynamic-vs-static-dispatch)

When a class is subclassed and a member — e.g., a property or method — is called on an instance of that class at runtime, the program needs to determine whether the member is being called on that class or on an overridden member in the subclass. This is called *dynamic dispatch*, and this is how polymorphism is made possible. **Dynamic dispatch involves looking up the function in a method table at runtime**, and then performing an indirect call to it. In performance-sensitive code this additional work can have an adverse effect.

Marking a method as `final` in the parent class will prevent that method from being overridden. As a result, dynamic dispatch can be avoided, and instead a *static dispatch* can be made directly to the parent class' implementation. **Static dispatch is determined at compile time.** An entire class can also be marked `final`, which will prevent dynamic dispatch on any of its member calls.

Additionally, marking a member or the entire class `private` will allow the compiler to determine if the member or entire class is final, and if so also avoid dynamic dispatch. 

## [Dispatch with Protocols](#dispatch-with-protocols)

> **Remember:** "Member" refers to a property or method.

When a type adopts a protocol and an instance of that type is defined explicitly or can be inferred to be a concrete type, all  the protocol members are statically dispatched at compile time.

#### Example: Concrete instance of type conforming to a protocol

```swift
protocol A {
    var a: Int { get }
}

extension A {
    var a: Int { 123 }
    func doA() {
        print("A doA()")
    }
}

final class ClassA: A { }

let concreteTypeInstance = ClassA()

concreteTypeInstance.a
concreteTypeInstance.doA()
```

In the above code, `concreteTypeInstance` is inferred to be of type `ClassA`. Therefore, both members `a` and `doA()` are statically dispatched.

When an instance is defined to be of a *protocol* type, each member requirement that is declared in the protocol will be dynamically dispatched at runtime.

However, if a member is *only* provided in a protocol extension — i.e., *not* declared in the protocol itself — it is statically dispatched.

#### Example: Instance defined as a protocol type

```swift
protocol B {
    var b: Int { get }
}

extension B {
    var b: Int { 123 }
    func doB() {
        print("B doB()")
    }
}

final class ClassB: B { }

let protocolTypeInstance: B = ClassB()

protocolTypeInstance.b
protocolTypeInstance.doB()
```

In the above code, `b` is defined as a requirement in the protocol, so it will be dynamically dispatched. And `doB()` is *only* provided in the extension to the protocol, so it will be statically dispatched.

Structuring your protocols and protocol extensions to ensure static dispatch when feasible will yield optimum run-time performance.

A member requirement that is defined in a protocol along with a default implementation in an extension is referred to as a *customization point*. The compiler will know that a customization may *potentially* be provided, so this will be dynamically dispatched to determine if a customization is actually provided.

> **Note:** There is one caveat: Without a customization point, you will lose any customization in instances that are declared as of the protocol type.

#### Example: Customization is lost in protocol-typed instance
```swift
protocol A { }

extension A {
    var a: Int { 1 }
}

protocol B: A { }

struct SomeStruct: B {
    var a = 2
}

let concreteTypeInstance = SomeStruct()
print(concreteTypeInstance.a)
let protocolTypeInstance: B = SomeStruct()
print(protocolTypeInstance.a)
```

In the above code, `SomeStruct` provides a customized implementation of `a`. However, because `protocolTypeInstance` is declared to be of a protocol type, even though it was assigned an instance of `SomeStruct`, it will not receive the customization.

This will print:
```
2
1
```

If the customization point is included, even protocol-typed instances will receive the customization.

#### Example: Include the customization point to ensure customization

```swift
protocol A {
    var a: Int { get }
}

extension A {
    var a: Int { 1 }
}

protocol B: A { }

struct SomeStruct: B {
    var a = 2
}

let concreteTypeInstance = SomeStruct()
print(concreteTypeInstance.a)
let protocolTypeInstance: B = SomeStruct()
print(protocolTypeInstance.a)
```

The above code is identical to the previous example, except the customization point is included in protocol `A`'s definition.

This will print:
```
2
2
```

## [Dispatch Type Flowchart](#dispatch-type-flowchart)

Use this flowchart to determine if a member will be statically or dynamically dispatched.
# ![](Assets/DispatchTypeFlowchart.png)

## [Conclusion](#conclusion)

In this article, you learned about:

1. How protocol-oriented programming differs from traditional object-oriented programming.
2. How to define and adopt protocols.
3. How to provide default implementations in protocol extensions.
4. Dynamic vs. static dispatch.
5. How to determine the dispatch type for each property or method.
6. How to ensure static dispatch when feasible, to achieve optimum performance.

---
<span id="1">¹</span> A read-only property requirement can be satisfied with a read-write property, however, a read-write property requirement cannot be satisfied with a read-only property.[⏎](#a1)<br>

[Back]({% link index.md %})

<sub><a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.</sub>
<br><sub>© 2020 Scott Gardner</sub><br>
