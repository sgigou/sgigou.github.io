---
layout: post
title: "Changer la hauteur d’une extension de clavier sans warnings"
date: 2020-06-05 18:00:00 +0100
categories: dev
tags: ["iOS","iOS 13","Swift","Swift 5","keyboard extension","UIKit"]
---

Si vous avez déjà essayé de modifier la hauteur d’un clavier personnalisé — _keyboard extension_ — avec des contraintes, vous avez dû obtenir le message d’avertissement suivant dans votre console :

```
2020-06-05 15:38:08.839964+0200 keyboard[20168:2238601] [LayoutConstraints] Unable to simultaneously satisfy constraints.
(
    "<NSLayoutConstraint:0x281a5cd20 UIInputView:0x105512e80.height == 250   (active)>",
    "<NSLayoutConstraint:0x281a6ed00 'UIView-Encapsulated-Layout-Height' UIInputView:0x105512e80.height == 216   (active)>"
)

Will attempt to recover by breaking constraint
<NSLayoutConstraint:0x281a5cd20 UIInputView:0x105512e80.height == 250   (active)>
```

Pour obtenir ce warning, il m’a suffit de définir la hauteur de mon clavier dans le contrôleur principal :

```swift
class KeyboardViewController: UIInputViewController {
  override func viewDidLoad() {
    super.viewDidLoad()
    view.heightAnchor.constraint(equalToConstant: 250.0).isActive = true
  }
}
```


## Pourquoi ce message ?

Ce warning apparaît dans les logs parce que la vue par défaut de `KeyboardViewController` transforme les masques de redimensionnement en contraintes (`translatesAutoresizingMaskIntoConstraints`). La hauteur initiale de la frame (ici 216) est convertie en contrainte, qui entrera en conflit avec la hauteur personnalisée définie (ici 250).

Le problème principal ici n’est pas le warning en lui-même, mais la décision du système de faire sauter notre contrainte — c’est précisément la pire chose à faire, car notre valeur est tout simplement ignorée.


## Comment corriger ça ?

Pour simplement supprimer le warning, vous pouvez désactiver la transformation du masque en contrainte :

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  view.translatesAutoresizingMaskIntoConstraints = false
  view.heightAnchor.constraint(equalToConstant: 250.0).isActive = true
}
```

MAIS… ce n’est pas suffisant. Lorsque vous la désactivez, cela annulera également des contraintes utiles : celles de gauche, du bas et de droite. Le clavier ne remplira plus toute la largeur de l’écran.

Vous devez ajouter ces contraintes manuellement pour remplacer celles du système. Le meilleur endroit pour ce faire est dans `viewWillAppear`, puisqu’il vous permet d’accéder à la _superview_ pour accrocher votre clavier. Mais comme elle peut être appelée plusieurs fois dans un seul cycle de vie, il vous faut vous assurer de ne pas ajouter plusieurs fois les mêmes contraintes.

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


## La solution complète

Votre contrôleur principal devrait ressembler à ceci :

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

Avec cette solution, vous pouvez même laisser les _subviews_ décider de la hauteur du clavier avec leurs contraintes !
