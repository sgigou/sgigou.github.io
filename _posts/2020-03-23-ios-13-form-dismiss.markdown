---
layout: post
title: "The best way to manage form dismiss on iOS 13"
date: 2020-03-23 11:00:00 +0100
categories: [ios]
---

Native iOS 13 apps are quite inconsistent when it comes to form dismiss. Let me explain the guidelines I follow and show you how I do it.

## What Apple do

It seems that teams didn’t agree on the proper way to handle the new “swipe to dismiss” gesture.

Let’s take a look at some apps, to see how they manage their forms, and try to extract some guidelines.

### The *Done* button

The *Done* button is quite consistent:

- in almost all apps, it is called `Done`,
- it is greyed out until any there’s been a change,
- when tapped, changes are persisted and the form is dismissed.

- [ ] IMG: Greyed-out done button

### The *Cancel* button
The *Cancel* button is less consistent:
- it is always called `Cancel`,
- it is present in old apps like *Contacts* and *Agenda*, but it does not appear anymore in redesigned ones like *Reminders*,
- when tapped, an action sheet is displayed to request confirmation from the user, only if the form has changes.

- [ ] IMG: Action sheet in an app with the button

### The swipe gesture
The swipe button act almost like the *Cancel* button:
* it is present on most apps, but some do not implement it, like the contact edition form,
* when swiping with no form changes, the form is dismissed,
* when swiping after updates, an action sheet is displayed to request confirmation from the user before dismissing.

## What I do
I like to stay consistent between my apps. That’s why I defined some guidelines, and try to stick to it.

### The *Done* button
It is always present, on the top right of the form. It is greyed out util a change is made.
There is no confirmation needed: when the user taps on it, it saves the data and dismisses the form.

### The *Cancel* button
It is always present, on the top left of the form.
If the user did not make any change to the form, then a tap on it dismisses the view controller.
If there was any change, the button will ask for the user’s confirmation with an action sheet. If the user confirms, it reverses the changes and dismisses the form.

### The swipe gesture
On this point, I differ from Apple guidelines. I think this gesture is awesome to avoid having to reach for the buttons at the top, as it is can be triggered from the form itself.
The behaviour I defined for this gesture is as follows:
* if there is no changes, then it only dismisses the form,
* if there are updates, it displays an action sheet that will ask the user what he prefers: saving or cancelling.

With this behaviour, the user is able to save or cancel from the bottom of the screen, so with one hand.

- [ ] IMG: Action sheet with both questions

## How to do it
The implementation is rather simple, so I will just let you read this commented class. The tricky lines will be commented at the end, so don’t hesitate to slide the code to the right.

```swift
import UIKit

class FormViewController: UIViewController {

  private(set) var wasEdited = false

  // MARK: Life cycle

  override func viewDidLoad() {
    super.viewDidLoad()
    // Let the system call us when the user tries to dismiss the view controller
    navigationController?.presentationController?.delegate = self
    // Add the *Done* button to the navigation bar.
    let doneButton = UIBarButtonItem(barButtonSystemItem: .done, target: self, action: #selector(doneAction))
    doneButton.isEnabled = false // The button is disabled while the form is not edited.
    navigationItem.rightBarButtonItem = doneButton
    // Add the *Cancel* button to the navigation bar.
    let cancelButton = UIBarButtonItem(barButtonSystemItem: .cancel, target: self, action: #selector(cancelAction))
    navigationItem.leftBarButtonItem = cancelButton
  }

  // MARK: User interactions

  /// Lock the swipe gesture and enable *Done* button.
  /// Call this function when a field is edited.
  func formWasEdited() {
    if wasEdited { return }
    wasEdited = true
    // Disable swipe-to-cancel gesture
    isModalInPresentation = true
    // Enable the *Done* button
    navigationItem.rightBarButtonItem?.isEnabled = true
  }

  /// Selector for the *Done* button
  @objc private func doneAction() {
    // As it will only be enabled when the form was edited, if it is tapped, we can save and quit.
    saveForm()
    dismiss()
  }

  /// Selector for the *Cancel* button
  @objc private func cancelAction() {
    if !wasEdited {
      dismiss()
    } else {
      displayConfirmationActionSheet()
    }
  }

  // MARK: Display

  /// Dismiss the view controller in the main thread
  func dismiss() {
    DispatchQueue.main.async {
      [weak self] in
      self?.dismiss(animated: true)
    }
  }
  
  /// Asks the user if he wants to cancel changes
  private func displayConfirmationActionSheet() {
    let alertController = UIAlertController(title: "Do you want to cancel?", message: "The changes will be reversed.", preferredStyle: .actionSheet)
    let confirmAction = UIAlertAction(title: "Cancel changes", style: .destructive) {
      [weak self] (_) in
      self?.dismiss() // Dismiss without saving first
    }
    let cancelAction = UIAlertAction(title: "Cancel", style: .cancel)
    alertController.addAction(confirmAction)
    alertController.addAction(cancelAction)
    present(alertController, animated: true, completion: nil)
  }
  
  /// Asks the user if he prefers to save or cancel
  private func askForCancelOrSave() {
    let alertController = UIAlertController(title: "You have unsaved changes!", message: "If you cancel, the changes will be reversed.", preferredStyle: .actionSheet)
    let saveAction = UIAlertAction(title: "Save changes", style: .default) {
      [weak self] (_) in
      self?.saveForm()
      self?.dismiss()
    }
    let deleteAction = UIAlertAction(title: "Cancel changes", style: .destructive) {
      [weak self] (_) in
      self?.dismiss() // Dismiss without saving first
    }
    let cancelAction = UIAlertAction(title: "Cancel", style: .cancel)
    alertController.addAction(saveAction)
    alertController.addAction(deleteAction)
    alertController.addAction(cancelAction)
    present(alertController, animated: true, completion: nil)
  }

}

// MARK: - UIAdaptivePresentationControllerDelegate

extension FormViewController: UIAdaptivePresentationControllerDelegate {

  /// Ask if the user wants to cancel or save changes
  /// This function will be called if the user tried to swipe the form while `isModalInPresentation` is set to `true`.
  func presentationControllerDidAttemptToDismiss(_ presentationController: UIPresentationController) {
    askForCancelOrSave()
  }

}
```