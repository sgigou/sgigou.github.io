---
layout: post
title: "Gérer efficacement le dismiss des formulaires sur iOS 13"
date: 2020-03-23 11:00:00 +0100
categories: dev
tags: ios ios13 swift swift5
---

Les applications d’Apple sur iOS 13 ne suivent pas les mêmes règles en ce qui concerne l’annulation des formulaires. Je vais vous expliquer les guidelines que je suis et comment les mettre en place.

## Ce que fait Apple

Il semblerait que les équipes ne se soient pas concertées sur une manière de gérer le geste « swipe to dismiss ».

Jetons un œil à quelques apps pour voir comment elles gèrent la situation et essayer d’en tirer des lignes directrices.

### Le bouton _Done_

Ce bouton a un comportement assez régulier sur les différentes apps :

* dans presque toutes les applications, il est intitulé _Done_,
* il est désactivé jusqu’à ce qu’une modification soit apportée au formulaire,
* quand il est pressé, les modifications sont sauvegardées et le formulaire est écarté.

<img src="/assets/2020-03-23/done.png" alt="Greyed-out done button" width="256">

### Le bouton _Cancel_

Ce bouton a un comportement plus variable :

* il est toujours appelé _Cancel_,
* il est présent dans les vieilles apps comme _Contacts_ et _Agenda_, mais il n’apparaît plus dans celles qui ont été refaites — comme _Reminders_,
* lorsqu’il est pressé, une action sheet est affichée pour demander une confirmation à l’utilisateur — dans le cas où le formulaire a été édité.

<img src="/assets/2020-03-23/cancel.png" alt="Cancel button action sheet" width="256">

### Le geste _swipe_

Il fonctionne quasiment comme le bouton _Cancel_ :

* il est supporté par presque toutes les apps, mais certaines ne l’implémentent pas, comme l’app _Contact_,
* lorsqu’il est effectué sans changement du formulaire, le formulaire est écarté,
* lorsqu’il est effectué après avoir apporté des modifications au formulaire, une action sheet demande confirmation à l’utilisateur de les annuler.

## Mes lignes directrices

J’essaie de rester consistent d’une application à l’autre. C’est pourquoi j’ai défini quelques lignes directrices et essaie de m’y tenir.

### Le bouton _Done_

Il est toujours présent, en haut à droite du formulaire. Il est grisé jusqu’à ce qu’une modification soit apportée au formulaire.

Il ne demande pas de confirmation : lorsqu’il est pressé, il sauvegarde les données et ferme le formulaire.

### Le bouton _Cancel_

Il est toujours présent, en haut à gauche du formulaire.

Si l’utilisateur n’a pas effectué de changement, alors un appui ferme le contrôleur.

Si des changements ont été apportés, l’app demandera une confirmation de l’utilisateur par une action sheet. Si l’utilisateur confirme, les modifications sont annulées et le formulaire est clos.

### Le geste _swipe_

Sur ce point, je n’applique pas les quelques lignes d’Apple. Je pense que ce geste est fantastique pour éviter à l’utilisateur d’aller chercher les boutons en haut de l’écran, puisqu’il peut être déclenché depuis le bas de l’écran.

Le comportement que j’ai défini pour ce geste est le suivant :

* si aucune modification n’est en attente, alors il ferme le formulaire,
* si des changements ont été effectués, il affiche une action sheet qui demande à l’utilisateur ce qu’il préfère : sauvegarder les modifications ou les annuler.

Avec ce comportement, l’utilisateur a la possibilité de sauvegarder ou annuler depuis le bas de l’écran — donc avec une main.

<img src="/assets/2020-03-23/swipe.png" alt="Swipe action sheet" width="256">

## Comment implémenter ce comportement

L’implémentation est plutôt simple, donc je vais vous laisser avec cette classe commentée.

Vous pouvez également trouver plus d’informations concernant la gestion de ce geste sur [cet autre article](https://sgigou.github.io/dev/2020/03/18/display-action-sheet-on-ios-13-modal-dismiss.html).

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

  /// Dismisses the view controller in the main thread
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
