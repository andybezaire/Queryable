
<p align="center">
  <img width="200" height="200" src="https://user-images.githubusercontent.com/7083109/231827191-7472e663-a8f2-42c6-a7aa-77bb38ae484a.png">
</p>

# Queryable - Asynchronous View Presentations in SwiftUI
![GitHub release (latest by date)](https://img.shields.io/github/v/release/SwiftedMind/Queryable?label=Latest%20Release)
![GitHub](https://img.shields.io/github/license/SwiftedMind/Queryable)

`Queryable` is a property wrapper that can trigger a view presentation and `await` its completion from a single `async` function call, while fully hiding the state handling of the presented view.

```swift
struct ContentView: View {
  @Queryable<Void, Bool> var buttonConfirmation

  var body: some View {
    Button("Commit", action: confirm)
      .queryableAlert(controlledBy: buttonConfirmation, title: "Really?") { item, query in
          Button("Yes") { query.answer(with: true) }
          Button("No") { query.answer(with: false) }
      } message: {_ in}
  }

  @MainActor
  private func confirm() {
    Task {
      do {
        let isConfirmed = try await buttonConfirmation.query()
        // Do something with the result
      } catch {}
    }
  }
}
```

Not only does this free the presented view from any kind of context (it simply provides an answer to the query), but you can also pass `buttonConfirmation` down the view hierarchy so that any child view can conveniently trigger the confirmation without needing to deal with the actually displayed UI. It works with `alerts`, `confirmationDialogs`, `sheets`, `fullScreenCover` and fully custom `overlays`.

- [Installation](#installation)
- **[Get Started](#get-started)**
- [Supported Queryable Modifiers](#supported-queryable-modifiers)
- [License](#license)

## Installation

Queryable supports iOS 15+, macOS 12+, watchOS 8+ and tvOS 15+.

### In Swift Package

Add the following line to the dependencies in your `Package.swift` file:

```swift
.package(url: "https://github.com/SwiftedMind/Queryable", from: "1.0.0")
```

### In Xcode project

Go to `File` > `Add Packages...` and enter the URL "https://github.com/SwiftedMind/Queryable" into the search field at the top right. Queryable should appear in the list. Select it and click "Add Package" in the bottom right.

### Usage

To use, simply import the `Queryable` target in your code.

```swift
import SwiftUI
import Queryable

struct ContentView: View {
  @Queryable<Void, Bool> var buttonConfirmation
  /* ... */
}
```

## Get Started

To best explain what `Queryable` does, let's look at an example. Say we have a button whose action needs a confirmation by the user. The confirmation should be presented as an alert with two buttons. 

Usually, you would implement this in a way similar to the following:

```swift
import SwiftUI

struct ContentView: View {
  @State private var isShowingConfirmationAlert = false

  var body: some View {
    Button("Do it!") {
      isShowingConfirmationAlert = true
    }
    .alert(
      "Do you really want to do this?",
      isPresented: $isShowingConfirmationAlert
    ) {
      Button("Yes") { confirmAction(true) }
      Button("No") { confirmAction(false) }
    } message: {}
  }

  @MainActor private func confirmAction(_ confirmed: Bool) {
    print(confirmed)
  }
}
```

The code is fairly simple. We toggle the alert presentation whenever the button is pressed and then call `confirmAction(_:)` with the answer the user has given. There's nothing wrong with this approach, it works perfectly fine.

However, I believe there is a much more convenient way of doing it. If you think about it, triggering the presentation of an alert and waiting for some kind of result – the user's confirmation in this case –is basically just an asynchronous operation. In Swift, there's a mechanism for that: *Swift Concurrency*.

Wouldn't it be awesome if we could simply `await` the confirmation and get the result as the return value of a single `async` function call? Something like this:

```swift
import SwiftUI

struct ContentView: View {
  // Some property that takes care of the view presentation
  var buttonConfirmation: /* ?? */

  var body: some View {
    Button("Do it!") {
      confirm()
    }
    .alert(
      "Do you really want to do this?",
      isPresented: /* ?? */
    ) {
      Button("Yes") { /* ?? */ }
      Button("No") { /* ?? */ }
    } message: {}
  }

  @MainActor private func confirm() {
    Task {
      do {
        // Suspend, show the alert and resume with the user's answer
        let isConfirmed = try await buttonConfirmation.query()
      } catch {}
    }
  }
}
```

The idea is that this `query()` method would suspend the current task, somehow toggle the presentation of the alert and then resume with the result, all without us ever leaving the scope. The entire user interaction with the UI is contained in this single line.

And that is exactly what `Queryable` does. It's a property wrapper that you can add within any SwiftUI `View` to control view presentations from asynchronous contexts. Here's what it looks like:

```swift
import SwiftUI
import Queryable

struct ContentView: View {
  // Since we don't need to provide data with the confirmation, we pass `Void` as the Input.
  // The Result type should be a Bool.
  @Queryable<Void, Bool> var buttonConfirmation

  var body: some View {
    Button("Commit") {
      confirm()
    }
    .queryableAlert( // Special alert modifier whose presentation is controller by a Queryable
      controlledBy: buttonConfirmation,
      title: "Do you really want to do this?"
      ) { item, query in
        // The provided query type lets us return a result
        Button("Yes") { query.answer(with: true) }
        Button("No") { query.answer(with: false) }
      } message: {_ in}
  }

  @MainActor
  private func confirm() {
    Task {
      do {
        let isConfirmed = try await buttonConfirmation.query()
        // Do something with the result
      } catch {}
    }
  }
}
```

## Supported Queryable Modifiers

Currently, these are the view modifiers that support being controlled by a `Queryable`:

- `queryableAlert(controlledBy:title:actions:message)`
- `queryableConfirmationDialog(controlledBy:title:actions:message)`
- `queryableFullScreenCover(controlledBy:onDismiss:content:)`
- `queryableSheet(controlledBy:onDismiss:content:)`
- `queryableOverlay(controlledBy:animation:alignment:content:)`
- `queryableClosure(controlledBy:block:)`


## License

MIT License

Copyright (c) 2023 Dennis Müller and all collaborators

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
