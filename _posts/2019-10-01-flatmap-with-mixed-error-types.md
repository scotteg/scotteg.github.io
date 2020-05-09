---
title: FlatMap with Mixed Error Types in Combine
excerpt: ""
tags: [swift, combine, reactive-programming, ios]
date: 2019-10-01
---

[Back]({% link index.md %})

# [FlatMap with Mixed Error Types in Combine](#flatmap-with-mixed-error-types-in-combine)

#### Posted by [scotteg](https://www.linkedin.com/in/scotteg/) on October 1, 2019

---

# Introduction


The `flatMap` operator in Combine enables you to flatten multiple upstream publishers into a single publisher. That flattened publisher does not need to be the same type as _any_ of the upstream publishers — and it often won't be the same.

This works fine with publishers that do not emit errors, or publishers that all emit the _same_ error type.

However, when you're using `flatMap` with publishers that can emit errors of _different_ types, you'll need to coordinate things so that `flatMap` can receive and work with a single error type.

## Setup

The examples will use the following setup code:

```swift
var subscriptions = Set<AnyCancellable>()
```

## Using `flatMap` with No Errors

This example demonstrates a typical use of `flatMap` in which no errors will be emitted:

```swift
struct Player {
    let score = CurrentValueSubject<Int, Never>(0)
}

let scott = Player()
let jenn = Player()

let players = CurrentValueSubject<Player, Never>(scott)

players
    .flatMap { $0.score }
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

scott.score.send(50)
players.send(jenn)
jenn.score.send(100)
```

This will print:

```
0
50
0
100
```

Each player is initialized with a `score` of `0`, and their `score` is printed when they are added to `players` and when their `score` is changed.

The `players` subject is of type `<Player, Never>` — i.e., it will never emit errors. What if that was _not_ the case?

## Using `flatMap` with Mixed Error Types

What if players could have errors of one type, and `score`s could also have errors of _another_ type?

This example is a modified version of the previous one in which players and scores can each have their own error types.

```swift
enum PlayerError: Error {
    case somePlayerError
}

enum ScoreError: Error {
    case someScoreError
}

struct Player {
    let score = CurrentValueSubject<Int, ScoreError>(0)
}

let scott = Player()
let jenn = Player()
let charlotte = Player()

let players = CurrentValueSubject<Player, PlayerError>(scott)

players
    .flatMap { $0.score }
    .sink(
        receiveCompletion: { print("sink receive completion:", $0) },
        receiveValue: { print("sink receive value:", $0) }
    )
    .store(in: &subscriptions)
```

This code produces an error at the `flatMap` line of code: `Instance method 'flatMap(maxPublishers:_:)' requires the types 'PlayerError' and 'ScoreError' be equivalent`.

One way to resolve this issue is to encapsulate the errors under a single parent error type. This enables you to still work with the original error types, while also fitting into the Combine paradigm.

The following code begins to re-implement the above example:

```swift
enum PlayerError: Error {
    case somePlayerError
}

enum ScoreError: Error {
    case someScoreError
}

enum GameError: Error {
    case player(PlayerError)
    case score(ScoreError)
}

struct Player {
    let score = CurrentValueSubject<Int, ScoreError>(0)
}

let scott = Player()
let jenn = Player()
let charlotte = Player()

let players = CurrentValueSubject<Player, PlayerError>(scott)
```

A `GameError` enum conforming to the `Error` protocol is defined, which encapsulates the `PlayerError` and `ScoreError` types.

Continuing this example:

```swift
players
    .print()
    .mapError { GameError.player($0) }
    .flatMap { $0.score.mapError { GameError.score($0) }}
    .sink(
        receiveCompletion: { print("sink receive completion:", $0) },
        receiveValue: { print("sink receive value:", $0) }
    )
    .store(in: &subscriptions)

players.value.score.send(completion: .failure(.someScoreError))
scott.score.send(50)
players.send(jenn)
jenn.score.send(100)
players.send(completion: .failure(.somePlayerError))
```

The `mapError` operator is used twice, to convert both `PlayerError` and `ScoreError` into their counterpart `GameError` representations.

The `print` operator is used to log all publishing events, and `sink` now also includes handling received completion events.

This will print (errors truncated):

```
receive subscription: (CurrentValueSubject)
request unlimited
receive value: (Player(score: ...ScoreError>))
sink receive value: 0
sink receive completion: failure(...GameError.score)
receive value: (Player(score: ...ScoreError>))
sink receive value: 0
sink receive value: 100
receive error: (somePlayerError)
```

You may notice that you can now handle receiving `ScoreError`s in the `sink`, but you do not have a way to handle if a `PlayerError` is emitted, because you are `flatMap`-ing into the `score`.

So to _also_ handle if a `PlayerError` is emitted, you could insert the following code _before_ the first `mapError` line:

```swift
.handleEvents(receiveCompletion: { print("handleEvents receive completion:", $0) })
```

Doing so would then print the following (errors truncated):

```
receive subscription: (CurrentValueSubject)
request unlimited
receive value: (Player(score: ...ScoreError>))
sink receive value: 0
sink receive completion: failure(...GameError.score)
receive value: (Player(score: ...ScoreError>))
sink receive value: 0
sink receive value: 100
receive error: (somePlayerError)
handleEvents receive completion: failure(...PlayerError.somePlayerError)
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
