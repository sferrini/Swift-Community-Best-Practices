# Swift-Best-Practices

Best practices for software development with Swift.

## Preface

This document grew from an set of notes I produced while working on [SwiftGraphics][SwiftGraphics]. Most of the recommendations in this guide are definitely considered opinions and arguments could be made for other approaches. That's fine. When other approaches make sense they should be presented in addition.

These best practices do not dictate or recommend whether Swift should be used in an procedural, object-oriented or functional manner. Instead a pragmatic approach is taken. Individual recommendations might be focused on OOP or functional solutions as needed.

The scope of this document is mostly aimed at the Swift language and Swift standard library. That said specific recommendations on how to use Swift with Mac OS, iOS, WatchOS and TVOS might be provided if a unique Swift angle or insight can be provided. Hints & tips style recommendations on how to use Swift effectively with Xcode and LLDB might also be provided.

This is very much a work in progress. Contributions are very much appreciated in the form of pull requests or filing of issues.

Discussion can be found on the [Swift-Lang slack][slack] (in the #bestpractices channel)

### Note to Contributors

Please make sure all examples are runnable (which may not be the case for existing examples). This markdown will be converted to a Mac OS X playground.

### Golden Rules

* Apple is generally right. Defer to Apple's preferred or demonstrated way of doing things. However Apple is a large corporation and we should be prepared to see discrepancies in their example code.
* Never write code merely to attempt to reduce the number of keystrokes you need to type. Rely on autocompletion, autosuggestion, copy and paste, etc instead.

## Style Guide

This document is not intended to be a style guide. Although some style advice is given, it's clear that the world doesn't need more language style debate and argument. Use the style Apple has defined within their “[The Swift Programming Language][Swift_Programming_Language]” book. This document will contain further clarifications where necessary.

### Whitespace

Tabs are four spaces. It's the Xcode default. Deal with it.

### Naming

As per the “Swift Programming Language” types should be [upper camel case][Studly_caps] (example: “`VehicleController`”).

[Studly_caps]: https://en.wikipedia.org/wiki/Studly_caps

Variables and constants should be lower camel case (example “`vehicleName`”).

You should rely on Swift modules to help namespace your code and not use Objective-C style class prefixes for Swift code (unless of course interfacing with Objective-C).

Do not use any form of [Hungarian notation][Hungarian_notation] (e.g. k for constants, m for methods), instead use short concise names and use Xcode's type Quick Help (⌥ + click) to discover a variable's type. Similarly do not use [`SNAKE_CASE`][Snake_case].

[Hungarian_notation]: https://en.wikipedia.org/wiki/Hungarian_notation
[Snake_case]: https://en.wikipedia.org/wiki/Snake_case

The only exception to this general rule are enum values, which should be uppercase (this follows Apple's "Swift Programming Language" style):

```swift
enum Planet {
    case Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}
```
Needless contractions and abbreviations should be avoided where at all possible, you can actually type out the characters "ViewController" without any harm, rely on Xcode's autocompletion to save you typing in the future. Extremely common abbreviations such as URL are fine. Abbreviations should be represented all uppercase ("URL") or all lowercase "url" as appropriate: use the same rule for types and variables: if url was a type it would be uppercase, if url was a variable it would be lower case.

### Comments

Comments should _not_ be used to disable code. Commented out code is dead code and pollutes your source. If you want to remove code but keep it around in case it's useful in the future you should be relying on git and/or your bug tracker.

## Best Practices

### Type inference

Where possible use Swift’s type inference to help reduce redundant type information. For example, prefer:

```swift
var currentLocation = Location()
```

to:

```swift
var currentLocation: Location = Location()
```

### Self Inference

Let the compiler infer self in all cases where it is able to. Areas where self should be explicitly used includes setting parameters in init, and non-escaping closures. For example:

```swift
struct Example {
    let name:String

    init(name:String) {
        self.name = name
    }
}
```

### Capture List Inference

Failure to infer type inside a capture list could lead to a rather verbose line of code. Only specify types if needed.

```swift
let people = [
    ("Mary", 42),
    ("Susan", 27),
    ("Charlie", 18),
]

let strings = people.map() {
    (name:String, age:Int) -> String in
    return "\(name) is \(age) years old"
}
```

If at all possible remove the types if the compiler can infer them:

```swift
let strings = people.map() {
    (name, age) in
    return "\(name) is \(age) years old"
}
```

Using the numbered parameter names ("`$0`") further reduces verbosity, often eliminating capture lists completely. Only use the numbered form when the parameter names add no further information to the closure (e.g. very simple maps and filters).

Apple can and will change the parameter types of closures (optionals removed or changed to auto-unwrapping etc). Intentionally under-specifying your optionals and relying on Swift to infer the information it needs, reduces the risk of the code breaking under these circumstances.

(see also: http://www.russbishop.net/swift-capture-lists)

### Constants

Constants used within type definitions should be declared static. For example:

```swift
class PhysicsModel {
    static var speedOfLightInAVacuum = 299_792_458
}

class Spaceship {
    static let topSpeed = PhysicsModel.speedOfLightInAVacuum
    var speed:Double

    func fullSpeedAhead() {
        speed = Spaceship.topSpeed
    }
}
```

Making the constants static allow them to be referred to without instances of the type. They also allow other code to access the constants.

Constants at global level should be avoided except for singletons.

### Computed Properties

Use the short version of computed properties if you only need to implement a getter. For example, prefer this:

```swift
class Example {
    var age: UInt32 {
        return arc4random()
    }
}
```

to this:

```swift
class Example {
    var age: UInt32 {
        get {
            return arc4random()
        }
    }
}
```

If you add a `set` or a `didSet` to the property then you will need to explicitly provide a `get`.

```swift
class Person {
    var age: UInt32 {
        get {
            return arc4random()
        }
        set {
            print("Time flies like a banana so I'm throwing away your age")
        }
    }
}
```

### Converting Instances

When creating code to convert instances from one type to another use either "to" methods, e.g.:

```swift
struct Mood {
    func toColor() -> NSColor {
        return NSColor.blueColor()
  }
}
```

Or `init()` methods:

```swift
extension NSColor {
    convenience init(_ mood:Mood) {
        super.init(color:NSColor.blueColor)
    }
}
```

(NOTE: Apple has gone down the path of `init()` methods for conversions. It seems generally best to follow Apple's lead.)

While you might be tempted to use a getter, e.g:

```swift
struct Mood {
    var color: NSColor {
        return NSColor.blueColor()
    }
}
```

getters should generally be limited to returning components of the receiving type. For example returning the area of a `Circle` instance is well suited to be a getter, but converting a `Circle` to a `CGPath` is better as a "to" function or an `init()` extension on `CGPath`.

### Singletons

Singletons are simple in Swift:

```swift
class ControversyManager {
    static let sharedInstance = ControversyManager()
}
```

The Swift runtime will make sure that the singleton is created and accessed in a thread-safe manner.

Singletons should generally just be named "`sharedInstance`" unless you have a compelling reason to name it otherwise

Because singletons are so easy in Swift and because consistent naming saves you so much time you will have even more chances to complain about how singletons are an anti-pattern and should be avoided at all costs. Your fellow developers will thank you.

### Extensions for Code Organisation

Extensions should be used to help organise code.

Methods and properties that are peripheral to an instance should be moved to an extension. Note that not all property types can be moved to an extension - do the best you can within this limitation.

You should use extensions to help organise your instance definitions. One good example of this is a view controller that implements table view data source and delegate protocols. Instead of mixing all that table view code into one class, put the data source and delegate methods onto extensions that adopt the relevant protocol.

Inside a single source file feel free to break down a definition into whatever extensions you feel best organise the code in question. Don't worry about methods in the main class or struct definition referring to methods or properties inside extensions. As long as it's all contained within one swift file it is all good.

Conversely the main instance definition should not refer to elements defined in extensions outside of the main Swift file..

### Chained Setters

Do not use chained methods as a more "convenient" replacement for property setters:

Prefer:

```swift
instance.foo = 42
instance.bar = "xyzzy"
```

to:

```swift
instance.setFoo(42).setBar("xyzzy")
```

Traditional setters are far easier and require far less boilerplate code than chain-able setters.

### Error Handling

Swift 2's `do`/`try`/`catch` mechanism is fantastic. Use it. (TODO: provide examples)

### Avoid `try!`

In general prefer:

```swift
do {
    try somethingThatMightThrow()
}
catch {
    fatalError("Something bad happened.")
}
```

to:

```swift
try! somethingThatMightThrow()
```

Even though this form is far more verbose it provides context to other developers and hopefully useful information in the log.

It is okay to use `try!` as a temporary error handler until a more comprehensive error handling strategy is evolved. But suggest you periodically sweep your code for any errant `try!` that might have snuck past your code reviews and convert them to the more verbose syntax.

### Avoid `try?` where possible

`try?` is used to "squelch" errors and is only useful if you truly don't care if the error is generated. In general though, you might want to catch the error and at least log the failure.

### Early Returns & Guards

When possible, use `guard` statements to handle early returns or other exits (e.g. fatal errors or thrown errors).

Prefer:

```swift
guard let safeValue = criticalValue else {
    fatalError("criticalValue cannot be nil here")
}
someNecessaryOperation(safeValue)
```

to:

```swift
if let safeValue = criticalValue {
    someNecessaryOperation(safeValue)
} else {
    fatalError("criticalValue cannot be nil here")
}
```

or:

```swift
if criticalValue == nil {
    fatalError("criticalValue cannot be nil here")
}
someNecessaryOperation(criticalValue!)
```

This flattens code otherwise tucked into an `if let` block, and keeps early exits near their relevant condition instead of down in an `else` block.

Even when you're not capturing a value (`guard let`), this pattern enforces the early exit at compile time. In the second `if` example, though code is flattened like with `guard`, accidentally changing from a fatal error or other return to some non-exiting operation will cause a crash (or invalid state depending on the exact case). Removing an early exit from the `else` block of a `guard` statement would immediately reveal the mistake.

## TODO Section

This is a list of headings for possible future expansion.

### Protocols & Protocol Driven Development

### Implicitly Unwrapped Optionals

### Reference vs Value Types

### Async Closures

### `unowned` vs `weak`

### Cocoa Delegates

### Immutable Structs

### Instance Initialisation

### Logging & Printing

### Computed Properties vs Functions

### Value Types and Equality

[SwiftGraphics]: https://github.com/schwa/SwiftGraphics/blob/develop/Documentation/Notes.markdown
[slack]: http://swift-lang.schwa.io
[Swift_Programming_Language]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/index.html
