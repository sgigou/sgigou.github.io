---
layout: post
title: "Set a keyboard extension’s height without warnings"
date: 2020-06-05 18:00:00 +0100
categories: [ios]
---

If you ever tried to set the height of a custom keyboard extension with constraints, you may have seen the following warning message in your console:

```
2020-06-05 15:38:08.839964+0200 keyboard[20168:2238601] [LayoutConstraints] Unable to simultaneously satisfy constraints.
(
    "<NSLayoutConstraint:0x281a5cd20 UIInputView:0x105512e80.height == 250   (active)>",
    "<NSLayoutConstraint:0x281a6ed00 'UIView-Encapsulated-Layout-Height' UIInputView:0x105512e80.height == 216   (active)>"
)

Will attempt to recover by breaking constraint
<NSLayoutConstraint:0x281a5cd20 UIInputView:0x105512e80.height == 250   (active)>
```

To get this warning, I just had to set the height of the keyboard extension’s main view controller:

```swift
class KeyboardViewController: UIInputViewController {
  override func viewDidLoad() {
    super.viewDidLoad()
    view.heightAnchor.constraint(equalToConstant: 250.0).isActive = true
  }
}
```

## Why do we see this?

This warning is logged because the default `KeyboardViewController`’s view translates autoresizing mask into constraints. The initial height of the frame (216 here) is converted as a constraint, and will conflict with the custom height (250).

The main problem here is not the warning itself, but the system’s decision to remove the custom constraint — it is precisely the worst thing to do, because your value is ignored.

## How to fix that

To remove the warning, as it is related to the autoresizing mask constraints, you only have to disable it:

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  view.translatesAutoresizingMaskIntoConstraints = false
  view.heightAnchor.constraint(equalToConstant: 250.0).isActive = true
}
```

BUT… it is not enough. When you remove the translation into constraints, it will also remove useful constraints: the left, the bottom and the right one. The keyboard will not fill the width of the screen.

You need to add those constraints programmatically to replace the system. The best place to do it is the `viewWillAppear` function, as it allows you to get the superview to stick your keyboard. But as it can be called several time in a single lifecycle, you need to be sure to add your constraints only once.

```swift
override func viewWillAppear(_ animated: Bool) {
  super.viewWillAppear(animated)
  // Only add the constraints once.
  if constraintsHaveBeenAdded { return }
  // Be sure that the superview exists.
  guard let superview = view.superview else { return }
  // Add the missing constraints.
  view.leftAnchor.constraint(equalTo: superview.leftAnchor).isActive = true
  view.bottomAnchor.constraint(equalTo: superview.bottomAnchor).isActive = true
  view.rightAnchor.constraint(equalTo: superview.rightAnchor).isActive = true
  // Remember that the constraints have been added.
  constraintsHaveBeenAdded = true
}
```

## The whole solution

Your main view controller should contain look like this:

```swift
class KeyboardViewController: UIInputViewController {

  private var constraintsHaveBeenAdded = false

  override func viewDidLoad() {
    super.viewDidLoad()
    view.translatesAutoresizingMaskIntoConstraints = false
    view.heightAnchor.constraint(equalToConstant: 250.0).isActive = true
  }

  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    if constraintsHaveBeenAdded { return }
    guard let superview = view.superview else { return Logger.error("Keyboard has no superview.") }
    view.leftAnchor.constraint(equalTo: superview.leftAnchor).isActive = true
    view.bottomAnchor.constraint(equalTo: superview.bottomAnchor).isActive = true
    view.rightAnchor.constraint(equalTo: superview.rightAnchor).isActive = true
    constraintsHaveBeenAdded = true
  }
}
```

With this solution, you can even let the subviews set the height of the keyboard with their constraints!