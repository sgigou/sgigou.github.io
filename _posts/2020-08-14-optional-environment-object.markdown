---
layout: post
title: "SwiftUI’s optional environment object"
date: 2020-08-14 18:00:00 +0100
categories: [ios, swiftui]
---

Allowing an environment object to be `nil` in a SwiftUI application may be powerful.

For example, let’s say that our app needs to display a log in / sign up screen. An elegant way to do it would be to inject an optional user entity into the environment, as it is a commonly used object. The landing view can then check if the user is nil to display the form.

## The wrapper

Allowing an environment object to be nil is hard and not very effective.

The easiest way to perform this is to use a _wrapper entity_. It will be non-optional, but it will contain the optional entity.

```swift
class UserWrapper {
  @Published var user: UserEntity?
}
```

## How to inject it

The wrapper injection will be performed as usual:

```swift
let userWrapper = UserWrapper()
userWrapper.user = // Load the user or set it to nil
let view = LandingView().environmentObject(userWrapper)

How to use it in views

In your views, you can now check the existence of the user entity:

struct LandingView: View {

  @EnvironmentObject var userWrapper: UserWrapper

  var body: some View {
    if userWrapper.user == nil {
      LoginView()
    } else {
      HomeView()
    }
  }

}
```

After the login is done, you will only have to set `userWrapper.user = loggedInUser`, and the `LandingView` will be reloaded to display the home page.

On the other side, if you want to disconnect the user, you only have to set `userWrapper.user = nil` to get back to the login form.

## How to mock it

You can mock your view easily, and even trigger a change after a few seconds to check the transition: 

```swift
struct MyView_Previews: PreviewProvider {

  static var previews: some View {
    let objectWrapper = MyObjectWrapper()
    DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
      objectWrapper.object = MyObject()
    }
    return MyView().environmentObject(objectWrapper)
  }

}
```
