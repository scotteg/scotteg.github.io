---
title: Debounce vs. Throttle in Combine
excerpt: ""
tags: [swift, combine, reactive-programming, ios]
date: 2019-08-15
---

[Back]({% link index.md %})

# [Debounce vs. Throttle in Combine](#debounce-vs-throttle-in-combine)

#### Posted by [scotteg](https://www.linkedin.com/in/scotteg/) on August 15, 2019

---

## Introduction

The `debounce` and `throttle` operators exist in most reactive implementations, such as RxSwift, yet they continue to elicit confusion about what each one does and how to use them effectively.

### Setup

The examples will use the following setup code:

```swift
// 1
let values = "abcdefghijklmnopqrstuvwxyz"
    .map { String(describing: $0) }
    .reduce([String]()) { values, next in
        let new = (values.last ?? "") + next
        return values + [new]
}

// 2
let subject = PassthroughSubject<(Double, String), Never>()

// 3
values.enumerated().forEach { i, value in
    let delay = TimeInterval(i) / 10.0
    
    DispatchQueue.main.asyncAfter(deadline: .now() + delay) {
        subject.send((delay, value))
    }
}
```
1. Create an array of incremental values `["a", "ab", "abc"...]`.
1. Create a passthrough subject to send values at specified time intervals.
1. Iterate over the array, sending each value to the passthrough subject after an increasing delay.

This code results in each value being sent at a rate of one character every tenth of a second.

## `debounce`

Essentially, `debounce` will **pause** for a specified time **after each value is received**, and then at the end of that pause it will **send the last value** through.

The following code creates a subscription to the `subject` created in the setup code:

```swift
let debounceSubscription = subject
    .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
    .sink(receiveValue: { print($0.1) })
```

This code will produce the following output:

![](Assets/debounce.gif)

Each sequential value triggers a half-second pause, during which time nothing is sent. After the last value is sent (`a...z`), the half-second pause completes and that last value is sent and printed.

## `throttle`

Conversely, `throttle` **does not pause** after receiving values. Instead, it **waits for the specified interval** repeatedly, and at the end of each interval it will **send either the first or the last value** depending on what is passed for its `latest` parameter. 

When `true` is passed for `latest`, at the end of each interval it will send the _last_ value received during the time interval.

The following code creates a subscription to the `subject` created in the setup code:

```swift
let throttleLatestSubscription = subject
    .throttle(for: .milliseconds(500), scheduler: DispatchQueue.main, latest: true)
    .sink(receiveValue: { print($0.1) })
```

This code will produce the following output:

![](Assets/throttleLatest.gif)

After each half-second interval, `throttle` sends the latest value it received during that interval.

If `latest` is `false`, `throttle` will send the _first_ value received during that time interval.

The following code creates a subscription to the `subject` created in the setup code:

```swift
let throttleFirstSubscription = subject
    .throttle(for: .milliseconds(500), scheduler: DispatchQueue.main, latest: false)
    .sink(receiveValue: { print($0.1) })
```

This code will produce the following output:

![](Assets/throttleFirst.gif)

After each half-second interval, `throttle` sends the first value it received during that interval.

## To Learn More

If you'd like to learn more about Combine, check out this book I co-authored:

[Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/a/742/link/27)

[![](Assets/CombineBook.png)](https://store.raywenderlich.com/a/742/link/27" alt="Combine: Asynchronous Programming with Swift)

It's packed with coverage of Combine concepts, hands-on exercises in playgrounds, and complete iOS app projects. Everything you need to _transform yourself_ from novice to expert with Combine — and have fun while doing it!

---

[Back]({% link index.md %})

<sub><a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.</sub>
<br><sub>© 2020 Scott Gardner</sub><br>
