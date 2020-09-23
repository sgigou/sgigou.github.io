---
layout: post
title: "Afficher une action sheet de confirmation pour rejeter une modale sur iOS 13"
categories: dev
tags: swift swift5 ios ios13
---

Avec iOS 13 est apparu le geste « glisser pour rejeter » sur les modales.

Vous pouvez exécuter du code au moment du rejet, et même intercepter ce geste avant qu’il soit appliqué. Cela vous permet d’afficher une `ActionSheet` pour demander une confirmation — si un changement a été effectué par exemple.

<img src="/assets/2020-03-18/animation.gif" alt="Modal dismiss animation" width="125">


## Étape par étape


### Étape 1: intercepter le geste de rejet

Cette étape est importante si vous souhaitez demander une confirmation à l’utilisateur.

Le framework vous permet de conserver le geste, mais de désactiver le _dismiss_ automatique. À la place, le système appellera une fonction afin que vous puissiez faire ce que bon vous semble.

Pour désactiver le rejet, dans votre `UIViewController`, vous devez assigner la valeur `true` au booléen `isModalInPresentation`.

Vous pouvez même attendre que l’utilisateur ait modifié le formulaire pour le setter !


### Étape 2: Devenir délégué

Nous devons permettre au système d’appeler notre fonction lorsque l’utilisateur essaie de fermer la modale.

Cette opération est réalisée en s’assignant en tant que délégué de `UIPresentationController` de votre contrôleur.

Dans votre contrôleur — dans la fonction `viewDidLoad` par exemple — ajoutez la ligne suivante :

```swift
presentationController?.delegate = self
// ou, si le UIViewController est dans un UINavigationController
navigationController?.presentationController?.delegate = self
```


### Étape 3: Implémenter le protocole

La ligne précédente génèrera une erreur, car votre contrôleur ne correspond pas au protocole :

```swift
extension YourViewController: UIAdaptativePresentationControllerDelegate {
  // Code à exécuter
}
```

Ce protocole vous donne accès à des fonctions pour :

* savoir quand le contrôleur va être fermé,
* savoir quand le contrôleur a été fermé,
* savoir quand l’utilisateur a essayé de fermer le contrôleur,
* dire au système si l’utilisateur a le droit de fermer le contrôleur,
* d’autres choses que vous pouvez trouver dans la [documentation officielle](https://developer.apple.com/documentation/uikit/uiadaptivepresentationcontrollerdelegate).


### Étape 4: Afficher une `UIActionSheet`

La fonction qui nous intéresse est :

```swift
func presentationControllerDidAttemptToDismiss(_ presentationController: UIPresentationController) {
  // Afficher l’ActionSheet, puis appeler dismiss(…)
}
```

Cette fonction sera appelée lorsque l’utilisateur essaie d’abaisser la vue, mais qu’il a été stoppé par la propriété `isModalInPresentation` précédemment assignée à `true`.

Vous pouvez lui demander s’il veut réellement annuler, puis effectuer le _dismiss_.


## Tout ensemble

Si vous avez besoin de répéter ce comportement sur plusieurs formulaires, vous pouvez créer une classe de laquelle hériter.

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

Quand vos champs ont été édités, verrouillez le formulaire en appelant la fonction `formWasEdited()` de la classe parente.

```swift
class MyViewController: DismissableFormViewController {

  @IBAction func textFieldEditingChanged(_ sender: UITextField) {
    formWasEdited()
  }

}
```

Vous pouvez encore améliorer cette classe en lui permettant — par exemple — de gérer les boutons _Cancel_ et _Save_ de la `UINavigationBar`.