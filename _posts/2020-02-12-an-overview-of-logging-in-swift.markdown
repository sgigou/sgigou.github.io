---
layout: post
title: "An overview of logging in Swift"
date: 2020-02-12
categories: mobile
---

Logging is a powerful tool. It can help you detect problems and fix them. However, you must use it correctly.


## Logging methods

Several methods exist, each with its own specificities.
To illustrate the outputs of those functions, I will use a common `struct`:
``` swift
struct Human {
  let name: String
  let age: Int
}
```


### print

``` swift
func print(_ items: Any..., separator: String = " ", terminator: String = "\n")
```
It is the easiest logging function in Swift. It will only display your output on the Xcode console when the device is plugged-in.
This function does not support formatting, but you can use string interpolation — like `print("My human: \(human).")`.

``` swift
print(human)
```
Will output:
```
Human(name: "Steve", age: 30)
```

Back in Swift 1, you had two functions: `print()` that did not add a line break at the end of the message, and `println()` that did. Those functions were merged in a single `print()` function, but you still can remove the line break by specifying `""` as `terminator` parameter:
``` swift
print("A first print", terminator: "")
print("A second print")
// Outputs: A first printA second print
```

For more information, you can refer to the [Apple Developer Documentation](https://developer.apple.com/documentation/swift/1541053-print).

### debugPrint
``` swift
func debugPrint(_ items: Any..., separator: String = " ", terminator: String = "\n")
```
It works almost like `print()`. The difference is the function called to display an object. `print()` will call `description()` function, and `debugPrint()` will call `debugDescription()` that will display more informations.

``` swift
debugPrint(human)
```
Will output:
```
__lldb_expr_6.Human(name: "Steve", age: 30)
```

For more information, you can refer to the [Apple Developer Documentation](https://developer.apple.com/documentation/swift/1541053-print).

### dump
``` swift
@discardableResult func dump<T>(_ value: T, name: String? = nil, indent: Int = 0, maxDepth: Int = .max, maxItems: Int = .max) -> T
```
This function allows to display an object. You can specify the wanted depth, the number of items displayed for a collection, and even the number of spaces used for indentation.

``` swift
dump(human)
```
Will produce:
```
▿ __lldb_expr_6.Human
  - name: "Steve"
  - age: 30
```

For more information, you can refer to the [Apple Developer Documentation](https://developer.apple.com/documentation/swift/1541053-print).

### NSLog
``` swift
func NSLog(_ format: String, _ args: CVarArg...)
```
It is a lot slower than `print()` function.
It will automatically add informations to your output: 
```
<Date> <Time> <Program name>[<Process ID>:<Thread ID>] <Message>
2016-07-16 08:58:04.681 test[46259:1244773] NSLog message
```
It allows you to use String formatting:
``` swift
NSLog("%0.4f", CGFloat.pi)
```

The biggest difference with the previous functions is that your logs will be printed on the Xcode console _and_ the device console.

For more information, you can ~~try to~~ refer to the [Apple Developer Documentation](https://developer.apple.com/documentation/foundation/1409759-nslog).

### os_log

``` swift
#define os_log(log, format, ...)
```

This is the new logging standard and it is available since iOS 10 and macOS 10.12.
It must be imported, and you can control the subsystem and the category of your log message. You can also choose the log level: `.default`, `.info`, `.debug`, `.error` or `.fault`.

You can use format, but not string interpolation.

``` swift
import os.log

let log = OSLog(subsystem: Bundle.main.bundleIdentifier!, category: "network")
os_log("url = %@", log: log, url.absoluteString)
```

It will only output your logs on your device, so you have to use `Console.app` to display them.

It is more complex to use than the others, so I will not elaborate on it since this article is an overview. It deserves a dedicated article.

If you are interested, you can read the [Apple Developer Documentation](https://developer.apple.com/documentation/os/os_log?language=occ) or watch the [WWDC 2016 video about Unified Logging and Activity Tracing](https://developer.apple.com/videos/play/wwdc2016/721/).


## Customize output

Like I said before, `print()` and `debugPrint()` relie on system functions to display objects: `description()` and `debugDescription()`.
You can conform to `CustomStringConvertible` (or `CustomDebugStringConvertible`) to override their implementations and display your objects as you want:

``` swift
struct Human: CustomStringConvertible {
  let name: String
  let age: Int

  var description: String {
    return "Human named \(name) (\(age) years old)"
  }
}

let human = Human(name: "Steve", age: 30)

print("Current human: \(human)")
```

Will output:

```
Current human: Human named Steve (30 years old)
```


## Logs issues

Logs suffer from performance and security problems.


### Performances issues

As silly as that sentence may sound: logs consume resources.

You may be tempted to add logs everywhere during development. This will become an issue in production, where those logs are not necessary anymore, but will slow down your app.

If you want to read more about performances issues, you can check this interesting article: [Why print is dangerous](https://medium.com/ios-os-x-development/swift-log-devil-or-why-println-is-dangerous-46390453353d).


### Security issues

As said before, `NSLog()` and `os_log` will output log messages on the device. This means that any user can see them if they know how to use Xcode and the Console app.

You can easily imagine the security issues that can arise when logging confidential or sensitive informations.


### A common fix

A simple solution is to keep some logs for debug mode only:

``` swift
#if DEBUG
  print("Log not wanted in production.")
#endif
print("Production-safe log")
```

With this solution, you can choose the logs you wish to display on the device. I suggest you only keep critical logs for production mode — logs you would have classified as errors or warnings.

Verbose logs will not slow down your app, and sensitive informations will not be used against your app or your users.


## NoveLogger library

To make my work easier, I created a lightweight library.

It supports logs levels and displays the file, the function and even the line number of the log. It also allows you to define which log levels should be printed in production mode.

It is available for everyone on [CocoaPods](https://cocoapods.org/pods/NoveLogger) and you can check the source code on [GitHub](https://github.com/sgigou/NoveLogger).