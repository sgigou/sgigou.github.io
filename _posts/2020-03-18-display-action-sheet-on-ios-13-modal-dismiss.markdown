---
layout: post
title: "Display an action sheet to confirm modal dismiss on iOS 13"
categories: [ios]
---

With iOS 13 appeared the new “swipe to dismiss” gesture on modales.

You can have your functions called on dismiss, and even catch a dismiss swipe before it is done. This can allow you to display an ActionSheet to confirm the dismiss if there was any change.

## Step by step

### Step 1: disabling the swipe gesture

This step is important if you wish to ask a confirmation to the user.

The framework allows you to keep the gesture, but to disable the automatic dismiss. Instead, the system will call a function so you can do whatever you want.

To disable the dismiss, in your `UIViewController`, you have to set the [`isModalInPresentation`](https://developer.apple.com/documentation/uikit/uiviewcontroller/3229894-ismodalinpresentation) boolean to `true`.

You can wait until the user has made a change to do so.

### Step 2: Become the delegate

The next step is to allow the system to call you when the user try to dismiss the view controller.

This operation is done by setting yourself as the delegate of the [`UIPresentationController`](https://developer.apple.com/documentation/uikit/uipresentationcontroller) of your `UIViewController`.

In your controller — in the `viewDidLoad` function for example — add the following line:

```swift
presentationController?.delegate = self
// or, if the UIViewController is in a UINavigationController
navigationController?.presentationController?.delegate = self
```

### Step 3: Implement the protocol

The previous line will generate an error, because your view controller does not conform to the delegate protocol:

```swift
extension YourViewController: UIAdaptativePresentationControllerDelegate {
  // Implement any function in the protocol
}
```

This protocol will give you access to functions to:

* know when the controller will be dismissed,
* know when the controller has been dismissed,
* know when the user tried to dismiss the controller,
* tell the system if the user can dismiss the controller or not,
* other things you can find in [the official documentation](https://developer.apple.com/documentation/uikit/uiadaptivepresentationcontrollerdelegate).

### Step 4: Display an `UIActionSheet`

The function we are interested in is:

```swift
func presentationControllerDidAttemptToDismiss(_ presentationController: UIPresentationController) {
  // Display your action sheet, and then call the dismiss(…) function
}
```

This function will be called when the user tried to swipe the view controller down, but was stopped by the `isModalInPresentation` property.

You can ask him if he really wants to cancel, and then dismiss the view controller for real.

## Put it all together

If you need to implement several form sheets, you can create a class to inherit.

```swift
import UIKit

class DismissableFormViewController: UIViewController {

  // MARK: Life cycle

  override func viewDidLoad() {
    super.viewDidLoad()
    navigationController?.presentationController?.delegate = self
    presentationController?.delegate = self
  }

  // MARK: Configuration

  /// Lock the form dismiss
  func formWasEdited() {
    isModalInPresentation = true
  }

  // MARK: Display

  private func displayConfirmationActionSheet() {
    let alertController = UIAlertController(title: "Do you want to cancel?", message: "The changes will be reversed.", preferredStyle: .actionSheet)
    let confirmAction = UIAlertAction(title: "Cancel changes", style: .destructive) {
      [weak self] (_) in
      self?.dismiss(animated: true, completion: nil)
    }
    alertController.addAction(confirmAction)
    let cancelAction = UIAlertAction(title: "Cancel", style: .cancel)
    alertController.addAction(cancelAction)
    present(alertController, animated: true, completion: nil)
  }

}

// MARK: - UIAdaptivePresentationControllerDelegate

extension DismissableFormViewController: UIAdaptivePresentationControllerDelegate {

  func presentationControllerDidAttemptToDismiss(_ presentationController: UIPresentationController) {
    displayConfirmationActionSheet()
  }

}

```

Then you can use this class by inheriting it. When your fields have been edited, lock your form by calling the `formWasEdited()` function of the parent class.

```swift
class MyViewController: DismissableFormViewController {

  @IBAction func textFieldEditingChanged(_ sender: UITextField) {
    formWasEdited()
  }

}
```

You are free to improve this class to allow it, for example, to manage the _Cancel_ and _Save_ buttons of the `UINavigationBar`.

#Blog/Pro