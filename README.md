### Post & Receive without combine

```swift
// Post
NotificationCenter.default.post(name: NSNotification.Name("Detail"), object: nil)

// Receive
NotificationCenter.default.addObserver(forName: NSNotification.Name("Detail"), object: nil, queue: .main) { _ in

}
```

### Post & Receive with combine

```swift
let notification = Notification.Name("MyNotification")
let publisher = NotificationCenter.default.publisher(for: notification, object: nil)

let subscription = publisher.sink { notification in
    print("Notification Received")
}

NotificationCenter.default.post(name: notification, object: nil)
```

### combine custom subscriber

```swift
/// String Subscriber
class StringSubscriber: Subscriber {

    func receive(subscription: Subscription) {
        print("Received Subscription")
        // how many items do we want to receive?
        subscription.request(.max(3)) // backpressure, don't send me more than `3` items
    }

    // Do you want to demand more?
    // We dont want to really change the demands
    func receive(_ input: String) -> Subscribers.Demand {
        print("Received Value", input)
        return .none // just keep it like that
    }

    func receive(completion: Subscribers.Completion<Never>) {
        print("Completed")
    }

    typealias Input = String
    typealias Failure = Never

}

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        let publisher = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K"].publisher

        let subscriber = StringSubscriber()
        publisher.subscribe(subscriber)

    }

}
```

### Subjects

```swift
import UIKit
import Combine

enum MyError: Error {
    case subscriberError
}

class StringSubscriber: Subscriber {

    func receive(subscription: Subscription) {
        subscription.request(.max(2))
    }

    func receive(_ input: Input) -> Subscribers.Demand {
        print(input)
        return .none
    }

    func receive(completion: Subscribers.Completion<MyError>) {
        print("Completion")
    }

    typealias Input = String
    typealias Failure = MyError

}

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Subjects
            // - Publisher
            // - Subscriber
        let subscriber = StringSubscriber()
        let subject = PassthroughSubject<String, MyError>()
        subject.subscribe(subscriber)

        // any cancellable
        let subscription = subject.sink { error in
            print("error")
        } receiveValue: { value in
            print("Received value from sink")
        }


        subject.send("A")
        subject.send("B")

        subscription.cancel()

        subject.send("C")
        subject.send("D")
    }

}
```
