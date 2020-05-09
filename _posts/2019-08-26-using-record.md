---
title: Using Record in Combine
excerpt: ""
tags: [swift, combine, reactive-programming, ios]
date: 2019-08-26
---

[Back]({% link index.md %})

# [Using Record in Combine](#using-record-in-combine)

#### Posted by [scotteg](https://www.linkedin.com/in/scotteg/) on August 26, 2019

---

## A Common Approach to Exploring Operators

The examples will use the following setup code:

```swift
enum SomeError: Swift.Error {
    case test
}
```

When learning about operators in Combine, you may find yourself writing a lot of boilerplate code to create publishers and send values, in order to explore how a particular operator works.

For example, here is an example of how you might explore how the `scan(_:_:)` operator works:

```swift
let publisher = PassthroughSubject<Int, SomeError>()

_ = publisher
    .scan(0) { $0 + $1 }
    .sink(
        receiveCompletion: { print($0) },
        receiveValue: { print($0) }
)

publisher.send(1)
publisher.send(2)
publisher.send(3)
publisher.send(completion: .failure(.test))
```

This will print:

```none
1
3
6
failure(__lldb_expr_48.SomeError.test)
```

If you don't need to observe when an error event is emitted, you can access the `publisher` computed property on sequences.

However, this publisher will not be able to emit an error event — its error type is `Never`.

For example:

```swift
_ = (1...3).publisher
    .scan(0) { $0 + $1 }
    .sink(
        receiveCompletion: { print($0) },
        receiveValue: { print($0) }
)
```

This will print:

```none
1
3
6
finished
```

## Use `Record` to Streamline Exploring Operators

There is a lesser-known publisher type in Combine called `Record` that lets you pre-specify the values and completion event that will be emitted.

The first example could be implemented using `Record` as follows:

```swift
_ = Record(
        output: Array(1...3),
        completion: .failure(SomeError.test)
    )
    .scan(0) { $0 + $1 }
    .sink(
        receiveCompletion: { print($0) },
        receiveValue: { print($0) }
    )
```

This will print the exact same output:

```none
1
3
6
failure(__lldb_expr_48.SomeError.test)
```

## To Learn More

If you'd like to learn more about Combine, check out this book I co-authored:

[Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/a/742/link/27)

[![](Assets/CombineBook.png)](https://store.raywenderlich.com/a/742/link/27" alt="Combine: Asynchronous Programming with Swift)

It's packed with coverage of Combine concepts, hands-on exercises in playgrounds, and complete iOS app projects. Everything you need to _transform yourself_ from novice to expert with Combine — and have fun while doing it!

---

[Back]({% link index.md %})

<sub><a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.</sub>
<br><sub>© 2020 Scott Gardner</sub><br>
