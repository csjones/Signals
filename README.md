# Signals

Signals is a micro-framework for creating and observing events. It replaces delegates and NSNotificationCenter with something much more powerful and beautiful.

### Features

- [x] Attach-and-forget observation
- [x] Type-safety
- [x] Filtered observation
- [x] Delayed and queued observation
- [x] Comprehensive Unit Test Coverage

### Requirements

- iOS 7.0+ / Mac OS X 10.9+
- Xcode 6.1+ (compatible with Swift 1.1 and 1.2)

### Installation

> **CocoaPods uses embedded frameworks that require a minimum deployment target of iOS 8 or OS X Mavericks.**
>
> To use Signals with a project targeting iOS 7, simply copy `Signals.swift` into your project.

### CocoaPods

CocoaPods 0.36 adds supports for Swift and embedded frameworks. To integrate Signals into your project add the following to your `Podfile`:

```ruby
platform :ios, '8.0'
use_frameworks!

pod 'Signals', '~> 1.0'
```

### Carthage

To integrate Signals into your project add the following to your `Cartfile`:

```ruby
github "artman/Signals" ~> 1.0
```

### Quick start

Make events on a class observable by creating one or more signals:
```swift
class NetworkLoader {

    // Creates a number of signals that can be subscribed to
    let onData = Signal<(data:NSData, error:NSError)>()
    let onProgress = Signal<Float>()

    ...

    func receivedData(receivedData:NSData, receivedError:NSError) {
        // Whenever appropriate, fire off any of the signals
        self.onProgress.fire(1.0)
        self.onData.fire(data:receivedData, error:receivedError)
    }
}
```

Subscribe to these signals from elsewhere in your application

```swift
let networkLoader = NetworkLoader("http://artman.fi")

networkLoader.onProgress.listen(self) { (progress) in
    println("Loading progress: \(progress*100)%")
}

networkLoader.onData.listen(self) { (data, error) in
    // Do something with the data
}
```

Adding listeners to signals is a attach-and-forget operation. If your listener is deallocated, the Signal removes the listener from it's list of listeners. If the Signal emitter is deallocated, so is the closure that was supposed to fire on the listener, so you don't need to explicitly manage the removal of listeners.

Singals aren't restricted to one listener, so multiple objects can listen on the same Signal.

You can also subscribe to events after they have occurred:
```swift
networkLoader.onProgress.listenPast(self) { (progress) in
    // This will immediately fire with last progress that was reported
    // by the onProgress signal
    println("Loading progress: \(progress*100)%")
}
```

Signal listeners can apply filters:
```swift
networkLoader.onProgress.listen(self) { (progress) in
    // This fires when progress is done
}.filter { $0 == 1.0 }
```

You can queue up listener dispatches for a set amount of time and fire them only once:
```swift
networkLoader.onProgress.listen(self) { (progress) in
    // Executed once per second while progress changes
}.queueAndDelayBy(1.0)
```

### Replacing delegates

Signals is simple and modern and greatly reduce the amount of boilerplate that is required to set up event delegation.

Would you rather implement a callback using a delegate:
- Create a protocol that defines what is delegated
- Create a delegate property on the class that wants to provide delegation
- Mark each class that wants to become a delegate as comforming to the delegate protocol
- Implement the delegate methods on the class that want to become a delegate
- Set the delegate property to become a delegate of the instance
- Check that your delegate implements each delegate method before invoking it

Or do the same thing with Signals:
- Create a signal for the class that wants to provide an event
- Subscribe to the signal as a listener from any instance you want

Signals can have multiple listeners and they therefore don't provide a way for the observer to return data to the signal invoker. The delegate pattern should still be used for data requests (e.g. UITableViewDataSource).

#### Replace NSNotificationCenter

To replace global notifications via the NSNotificationCenter with Signals, just create a Singleton with a number of public signals that anybody can subscribe to or fire. You'll gain type safety, refactorability and attach-and-forget observation.

### Communication

- If you **found a bug**, open an issue or submit a fix via a pull request.
- If you **have a feature request**, open an issue or submit a implementation via a pull request or hit me up on Twitter [@artman](http://twitter.com/artman)
- If you **want to contribute**, submit a pull request.

### License

Signals is released under an MIT license. See the LICENSE file for more information
