---
title: Capture Lists
excerpt: "Capture lists have been available in Swift since its debut at WWDC 2014. They play a crucial role in ensuring your asynchronous code does not leak memory or cause exceptions at runtime. In order to use capture lists correctly, you'll need a good grasp of related topics including value vs. reference types, strong reference cycles, and escaping vs. nonescaping closures. In this article, I'll cover these supporting topics first, and then explain why, when, and how to use capture lists effectively."
tags: [swift, ios, closures, strong-reference-cycles, capture-lists]
date: 2020-04-15
redirect_from:
    - /conquering-capture-lists
    - /Conquering-Capture-Lists
---

[Back]({% link index.md %})

# [Capture Lists](#capture-lists)

#### Posted by [scotteg](https://www.linkedin.com/in/scotteg/) on April 15, 2020

#### Contents
- [Introduction](#introduction)
- [Value and Reference Types](#value-and-reference-types)
- [Strong, Weak, and Unowned References](#strong-weak-and-unowned-references)
- [Strong Reference Cycles](#strong-reference-cycles)
- [Preventing Strong Reference Cycles](#preventing-strong-reference-cycles)
- [Escaping vs. Nonescaping Closures](#escaping-vs-nonescaping-closures)
- [Defining Capture Lists](#defining-capture-lists)
- [Preventing Strong Reference Cycles in Closures](#preventing-strong-reference-cycles-in-closures)
- [Capturing Multiple Class References](#capturing-multiple-class-references)
- [Capture List Decision Flowchart](#capture-list-decision-flowchart)
- [Conclusion](#conclusion)

---

## [Introduction](#introduction)

Capture lists have been available in Swift since its debut at WWDC 2014. They play a crucial role in ensuring your asynchronous code does not leak memory or cause exceptions at runtime. In order to use capture lists correctly, you'll need a good grasp of related topics including value vs. reference types, strong reference cycles, and escaping vs. nonescaping closures. In this article, I'll cover these supporting topics first, and then explain why, when, and how to use capture lists effectively.

## [Value and Reference Types](#value-and-reference-types)

Value types include all of the basic data types in the Swift Standard Library, such as `String` and `Int`. Collections such as `Array` are also value types. These types are defined as structures. In other words, structures are value types. Enumerations are also value types.

**Value types are passed by copy**<span id="a1">[¹](#1)</span>. This means when you assign a value type to another variable or constant, or pass a value type to a function, the value is copied and the original value is left untouched.

#### Example of value types

```swift
var a = 1
let b = a
a = 2
print(a, b) // Prints 2, 1
```

Reference type include classes, functions, and closures<span id="a2">[²](#2)</span>.

**Reference types are passed by reference**. When you assign a reference to a variable or constant, or pass a reference type to a function, what you are *actually* passing is a pointer to the same instance of that reference type in memory.

#### Example: Reference types

```swift
class SomeClass {
    var value: String
    init(_ value: String) {
        self.value = value
    }
}

let a = SomeClass("Hello, world!")
let b = a
a.value = "Changed"
print(a.value, b.value) // Prints Changed Changed
```

## [Strong, Weak, and Unowned References](#strong-weak-and-unowned-references)

> **Note:** This section introduces some preliminary terminology and theory. These topics will be expanded upon and used in examples throughout the rest of the article.*

A *strong* reference will refuse to release or "let go" of the instance it is referring to. When you define a property of a reference type, it will be a strong reference by default. A strong reference property, once assigned, will not release the instance it refers to until either that property is set to `nil`<span id="a3">[³](#3)</span> or the property's owner is deallocated.

Within the definition of a class, the keyword `self` is a strong reference to the instance of that class.

**Only class reference types may be marked as `weak` or `unowned`.**  In other words, a closure property cannot be marked `weak` or `unowned`.

A *weak* reference will only hold on to the class instance it is referring to until there are no longer any strong references to that instance. Once there are no longer any strong references, a weak reference will be set to `nil` and release its hold so that the instance it referred to can be deallocated and its memory freed. For this reason, weak properties must be defined as `Optional`s.

Similar to a weak reference, an *unowned* reference does not keep a strong hold on the class instance it refers to. However, an unowned reference expects that the instance it refers to will always have a value, and will not be deallocated before the parent of the unowned instance. Therefore, unowned properties should not be defined as an `Optional`.

## [Strong Reference Cycles](#strong-reference-cycles)

A *strong reference cycle* occurs when two class reference type instances hold a strong reference to each other. In this scenario, each instance will refuse to let go of the other instance. The result is, neither instance is deallocated and their memory is leaked.

#### Example: Strong reference cycle
```swift
// 1
class Band {
    var singer: Singer?
    
    init(singer: Singer? = nil) {
        self.singer = singer
    }
    
    deinit {
        print("\(Self.self)", #function)
    }
}

class Singer {
    let name: String
    let band: Band
    
    init(name: String, band: Band) {
        self.name = name
        self.band = band
    }
    
    deinit {
        print("\(Self.self)", #function)
    }
}

// 2
var steelDragon: Band! = Band()
var bobby: Singer! = Singer(name: "Bobby Beers", band: steelDragon)

// 3
steelDragon.singer = bobby

// 4
bobby = nil
steelDragon = nil
```

In the above code:

1. `Band` and `Singer` classes are defined. Each class has a property of the type of the other class. And each class will print a descriptive message in their `deinit` method.
2. Instances of both classes are created.
3. The `Singer` instance is assigned to the `Band` instance's `singer` property. **At this point a strong reference cycle is created**, because each instance holds a strong reference to the other instance.
4. Each instance is set to `nil`, but neither instance is deallocated, because it still has a strong reference to it.

Nothing will be printed because neither `deinit` is called.

## [Preventing Strong Reference Cycles](#preventing-strong-reference-cycles)

To prevent a strong reference cycle, at least one of the properties between two class instances that reference each other must either be marked as `weak` or `unowned`.

> **Remember:**<br>1. A weak property must be of an `Optional` type, because it will be set to `nil` when there are no other strong references to that instance.<br>2. An unowned property should not be an `Optional`, because it is expected to never be `nil` for the life of the instance that owns that property.

#### Example: Breaking a strong reference cycle with a weak reference
```swift
// 1
class Band {
    weak var singer: Singer?
    
    init(singer: Singer? = nil) {
        self.singer = singer
    }
    
    deinit {
        print("\(Self.self)", #function)
    }
}

// 2
class Singer { ... }

var steelDragon: Band! = Band()
var bobby: Singer! = Singer(name: "Bobby Beers", band: steelDragon)

steelDragon.singer = bobby

// 3
bobby = nil
steelDragon = nil
```

In the above code:

1. Replaces previous definition of `Band`.
2. Same definition of `Singer` as in previous example.
3. The weak property will break the strong reference cycle and both instances will be deallocated.

This will print:

```
Singer deinit
Band deinit
```

#### Example: Breaking a strong reference cycle with an unowned reference
```swift
// 1
class Band {
    var singer: Singer?
    
    init(singer: Singer? = nil) {
        self.singer = singer
    }
    
    deinit {
        print("\(Self.self)", #function)
    }
}

// 2
class Singer {
    let name: String
    unowned let band: Band
    
    init(name: String, band: Band) {
        self.name = name
        self.band = band
    }
    
    deinit {
        print("\(Self.self)", #function)
    }
}

var steelDragon: Band! = Band()
var bobby: Singer! = Singer(name: "Bobby Beers", band: steelDragon)

steelDragon.singer = bobby

// 3
bobby = nil
steelDragon = nil
```

In the above code:

1. Replaces previous definition of `Band`. In this version, the `singer` property is a regular (strong) reference to an `Optional`.
2. Replaces the definition of `Singer`. In this version, the `band` property is an unowned reference.
3. The unowned property will break the strong reference cycle and both instances will be deallocated.

This will print:

```
Singer deinit
Band deinit
```

## [Escaping vs. Nonescaping Closures](#escaping-vs-nonescaping-closures)

When a closure is passed as a parameter to a function, that closure can be defined to behave in one of two ways:

1. Nonescaping - program flow is not allowed to exit the scope of the function until the closure parameter has finished execution. This is synchronous behavior.
2. Escaping - program flow will exit the scope of the function before the closure parameter has finished execution. This is asynchronous behavior.

A closure parameter is nonescaping by default. In order to define a closure parameter as escaping, mark the parameter using the `@escaping` attribute.

#### Example: Defining an escaping closure
```swift
func execute(action: @escaping () -> Void) {
    action()
}
```

When an escaping closure parameter of a method<span id="a4">[⁴](#4)</span> references any member of that type, including properties and methods, `self` must be explicitly written. This is enforced by the compiler to make the capture semantics explicit. In other words, the compiler is reminding you of your responsibility to properly handle captures of `self` or any of its members.

## [Defining Capture Lists](#defining-capture-lists)

**Capture lists are primarily used to prevent strong reference cycles in closures.**

A capture list is defined at the beginning of the body of a closure, before argument names and the return type are defined. As implied by its name, a capture list can contain more than one value. Each capture is defined following this format:

`weak/unowned captureName = classReferenceTypeInstance`

If the capture name is `self`, or it matches a property name of the class in which it is defined, you can omit explicitly defining the assignment and it will be inferred.

#### Example: Defining a capture list
```swift
class SomeClass {
    let value = "Hello, world!"
    lazy var printValue: () -> Void = { [unowned self] in
        print(self.value)
    }
}
```

In the above code, an unowned capture of `self` is created. Because the capture name is `self`, it is not necessary to explicitly assign it (e.g., `[unowned self = self]`).

> **Tip:** A lazy closure property is treated as an escaping.

Multiple captures are separated by commas.

#### Example: Defining a capture list with multiple captures
```swift
class SomeClass {
    // 1
    typealias PrintableObject = AnyObject & CustomStringConvertible
    var value: PrintableObject?
    
    lazy var printValue: () -> Void = {
        // 2
        [unowned self, weak theValue = self.value] in
        guard let theValue = theValue else { return }
        print(theValue)
    }
    
    init(value: PrintableObject) {
        self.value = value
    }
    
    func execute(action: @escaping () -> Void) {
        action()
    }
}
```

In the above code:

1. A type alias is defined to represent class types that conform to the `CustomStringConvertible` protocol by providing a textual representation.
2. The capture list defines an unowned capture of `self` and weak capture of `self.value`, using explicit assignment for the second capture because it uses a different capture name.

## [Preventing Strong Reference Cycles in Closures](#preventing-strong-reference-cycles-in-closures)

> **Remember:**<br>1. A closure is a reference type.<br>2. The `self` keyword used in a class definition is a strong reference to the instance of that class.

For this section, an iOS app project will be referenced. The app displays a *Present Alert* button, and tapping it presents an alert. In the alert's definition, an action is performed asynchronously after a three-second delay. The action is an escaping closure parameter of the `DispatchQueue.asyncAfter` method. To test for the occurrence of a strong reference cycle, dismiss the alert within three seconds.

If you'd like to follow along with the examples in this section, you can download a starter version of this project <a href="https://github.com/scotteg/CaptureLists/tree/starter" target="_blank">here</a>.

In each of the following examples, the following actions are presumed:

1. The alert view controller will be presented.
2. The alert view controller will be dismissed before three seconds have elapsed.

#### Example: No strong reference cycle, closure is executed
```swift
class AlertViewController: UIAlertController {
    var value = "Hello, world!"

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
            print(self.value)
        }
    }

    deinit {
        print(#function)
    }
}
```

In the above code, the `print` statement will be executed after three seconds, even if the alert is dismissed beforehand. The trailing closure of `asyncAfter` holds a strong reference to `self` until the `print` statement has executed. At that point, `self` is released and the alert instance is deallocated. This will print:

```
Hello, world!
deinit
```

#### Example: No strong reference cycle, closure is not executed, exception because self is nil
```swift
class AlertViewController: UIAlertController {

    var value = "Hello, world!"

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        DispatchQueue.main.asyncAfter(deadline: .now() + 3) { [unowned self] in
            print(self.value)
        }
    }

    deinit {
        print(#function)
    }
}
```

In the above code, `self` will be `nil` before the three-second delay has elapsed. However, creating an unowned capture of `self` requires that `self` will never be `nil`. So, when the `print` statement is executed and it attempts to reference `self.value`, an exception will occur.

There are three situations where creating an unowned capture of `self` is advisable:

1. A view controller that will never be dismissed.
2. The class is implemented as a singleton, i.e., only a single instance of the class will be initialized and shared globally, and the instance will not be deallocated during the app's life-cycle.
3. `self` is referenced in a nonescaping closure.

> **Tip:** Presented view controllers that can be dismissed and classes that can be deallocated should not contain unowned captures.

#### Example: No strong reference cycle, closure is not executed
```swift
class AlertViewController: UIAlertController {
    var value = "Hello, world!"

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        DispatchQueue.main.asyncAfter(deadline: .now() + 3) { [weak self] in
            guard let self = self else { return }
            print(self.value)
        }
    }

    deinit {
        print(#function)
    }
}
```

In the above code, a weak capture of `self` is made inside the closure, and the `guard` statement attempts to create a local  temporary strong reference to `self` — i.e., `self` is not `nil`— before executing the `print` statement. If the alert is dismissed *before* the three-second delay elapses, `self` will be `nil` and the `print` statement will not be executed. This will print:

```
deinit
```

> **Tip:** It is also common to use a different name for the strong reference to `self` in a closure, e.g., `guard let strongSelf = self else { return }`.

#### Example: Strong reference cycle, closure is executed
```swift
class AlertViewController: UIAlertController {
    var value = "Hello, world!"
    var action: (() -> Void)?

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        action = {
            print(self.value)
        }

        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
            self.action!()
        }
    }

    deinit {
        print(#function)
    }
}
```

In the above code, an `action` property is added that is a closure. `self` holds a strong reference to its `action` property, and the value assigned to it holds a strong reference to `self`. **This creates a strong reference cycle.** Neither the closure or the instance of the alert will be deallocated, and their memory is leaked. This will print:

```
Hello, world!
```

#### Example: No strong reference cycle, closure is executed
```swift
class AlertViewController: UIAlertController {
    var value = "Hello, world!"
    var action: (() -> Void)?

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        action = { [weak self] in
            guard let self = self else { return }
            print(self.value)
        }

        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
            self.action!()
        }
    }

    deinit {
        print(#function)
    }
}
```

In the above code, the `action` closure property is assigned a value that first creates a weak capture of `self`, and then attempts to create a temporary strong reference to `self`. If it's successful, it executes the `print` statement that references `self`. The weak capture prevents a strong reference cycle. However, the `asyncAfter` closure will hold a strong reference to `self` until it has finished executing. This will print:

```
Hello, world!
deinit
```

This may be desired behavior in some situations, where you want the closure to keep a strong reference to `self` until it can finish its work. However if the work performed in the closure is unnecessary, and especially if that work is expensive, it would be more performant to avoid executing that closure if `self` is `nil`. You'll see how to do this in the last example.

#### Example: Strong reference cycle, closure is executed
```swift
class AlertViewController: UIAlertController {
    var value = "Hello, world!"
    var action: (() -> Void)?

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        action = {
            print(self.value)
        }

        DispatchQueue.main.asyncAfter(deadline: .now() + 3) { [weak self] in
            guard let self = self else { return }
            self.action!()
        }
    }

    deinit {
        print(#function)
    }
}
```

In the above code, it might seem at first that the weak capture of `self` in the closure will prevent a strong reference cycle. However, even though the `asyncAfter` closure only weakly captures `self`, the `action` property of `self` strongly captures `self`. Therefore, `self` strongly holds `action`, and `action` strongly holds `self`. **This creates a strong reference cycle.** This will print:

```
Hello, world!
```

#### Example: No strong reference cycle, closure is not executed
```swift
class AlertViewController: UIAlertController {
    var value = "Hello, world!"
    var action: (() -> Void)?

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        action = { [weak self] in
            guard let self = self else { return }
            print(self.value)
        }

        DispatchQueue.main.asyncAfter(deadline: .now() + 3) { [weak self] in
            guard let self = self else { return }
            self.action!()
        }
    }

    deinit {
        print(#function)
    }
}
```

In the above code, both the `action` closure and the `asyncAfter` closure create weak captures of `self` and then temporary strong references of `self` before referencing `self`'s members. This will print:

```
deinit
```

This example demonstrates preventing a strong reference cycle, and it prevents executing the `asyncAfter` closure if `self` is `nil`, i.e., the alert was already dismissed and is deallocated. This would be advisable if the result of executing that closure is not needed.

> **Tip:** This also applies to nonescaping closures. If the only remaining strong reference to `self` is in a nonescaping closure, creating a weak capture and then using a `guard` statement at the top to attempt to create a temporary strong reference will fail and the closure will not be executed.

## [Capturing Multiple Class References](#capturing-multiple-class-references)

Creating capture lists of `self` is by far the most common technique to prevent strong reference cycles. However, it is also possible to create capture lists that capture other class references beside `self`, or that capture more than one class reference.

#### Example: Capturing multiple class references
```swift
class AlertViewController: UIAlertController {
    // 1
    class Action {
        let execute: () -> Void
        
        init(action: @escaping () -> Void) {
            self.execute = action
        }
    }
    
    var value = "Hello, world!"
    var action1: Action?
    var action2: Action?

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        // 2
        action1 = Action() { [weak self] in
            self?.value = "Changed"
        }
        
        action2 = Action() { [weak self] in
            guard let self = self else { return }
            print(self.value)
        }
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
            // 3
            [weak action1, weak action2] in
            guard let action1 = action1,
                let action2 = action2
                else { return }
            
            action1.execute()
            action2.execute()
        }
    }

    deinit {
        print(#function)
    }
}
```

In the above code:

1. An `Action` class is defined to encapsulate a closure property.
2. Both the `action1` and `action2` properties are assigned instances of `Action` that weakly capture `self`.
3. A capture list is created that create weak captures of `action1` and `action2`, and then strong references to those class instances before calling their `execute` method.

This will print:

```
deinit
```

## [Capture List Decision Flowchart](#capture-list-decision-flowchart)

Use this flowchart to determine if a capture list should be defined in a closure in a class type.
# ![](Assets/CaptureListDecisionFlowchart.png)

## [Conclusion](#conclusion)

In this article, you learned about:

1. Value vs. reference types.
2. Strong, weak, and unowned references.
3. Strong reference cycles between class instances and how to prevent them by creating weak and unowned properties.
4. Escaping vs. Nonescaping closures.
5. Capture lists and how to define them with one or more captures.
6. Using capture lists in escaping closures to prevent strong reference cycles.

You can download the final version of the project in this article <a href="https://github.com/scotteg/CaptureLists/" target="_blank">here</a>.

---
<span id="1">¹</span> Technically, value types are passed by copy-on-write, which means they won't actually be copied until they need to be mutated.[⏎](#a1)<br>
<span id="2">²</span> Functions are actually just named closures.[⏎](#a2)<br>
<span id="3">³</span> This would require the property to be an `Optional`.[⏎](#a3)<br>
<span id="4">⁴</span> A function within a type definition is referred to as a method.[⏎](#a4)<br>

[Back]({% link index.md %})

<sub><a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.</sub>
<br><sub>© 2020 Scott Gardner</sub><br>
