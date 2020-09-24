---
layout: post
title: "Glisser pour supprimer une ligne d’UITableView"
date: 2020-04-10 18:00:00 +0100
categories: dev
tags: ["iOS","iOS 11","Swift","Swift 5"]
---

iOS 11.0 a introduit une nouvelle manière de gérer les gestes sur les cellules. Implémenter un bouton _Supprimer_ est maintenant très simple.

<img src="/assets/2020-04-10/intro.png" alt="Swipe to delete illustration" width="256">


## Affiche le bouton _Supprimer_

La seule chose que vous ayez à implémenter est la fonction [`tableView(_:trailingSwipeActionsConfigurationForRowAt:)`](https://developer.apple.com/documentation/uikit/uitableviewdelegate/2902367-tableview).

Elle retourne un objet `UISwipeActionsConfiguration`, qui est un conteneur de `UIContextualAction`. 

```swift
func tableView(_ tableView: UITableView, trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
  let deleteAction = UIContextualAction(style: .destructive, title: "Delete") {
    [weak self] (_, _, completion) in
    self?.deleteItem(at: indexPath) // Your deletion function
    completion(true)
  }
  return UISwipeActionsConfiguration(actions: [deleteAction])
}
```

N’oubliez pas de définir la propriété `self` en `weak` afin d’éviter un cycle de rétention !


## Jouer l’animation de suppression

Vous affichez dorénavant un magnifique bouton _Supprimer_, qui est tappable et qui appelle votre code. L’étape suivante consiste à jouer une jolie animation de suppression !


### L’animation de suppression de UITableView

Une animation de suppression s’implémente très simplement, puisqu’il suffit d’appeler la fonction `deleteRows(at:with:)` de votre _table view_. Vous pouvez même [choisir quelle animation jouer](https://developer.apple.com/documentation/uikit/uitableview/rowanimation): `.fade`, `.right`, `.left`, `.top`, `.bottom`, `.none`, `.middle` ou `.automatic`.

La valeur `.automatic` est celle utilisée par défaut par le système.

```swift
private func deleteItem(at indexPath: IndexPath) {
  tableView.beginUpdates()
  tableView.deleteRows(at: [indexPath], with: .automatic)
  tableView.endUpdates()
}
```


### Recharger les données

La particularité arrive ici. Si vous vous contentez de recharger vos données, puis de jouer l’animation, vous obtiendrez une erreur vous informant que le nombre de lignes avant et après l’animation est le même. La table s’attend à ce que la quantité de lignes soit égale à `oldCount - 1`, puisqu’une a été supprimée.

Vous devez organiser votre mise à jour de la manière suivante :

```swift
private func deleteItem(at indexPath: IndexPath) {
  let store = ItemStore() // Pre-load everything you can
  store.deleteItem(at: indexPath.row) // You can perform the deletion here and persist, but do not update the data in your controller!
  // Here, items.count == n
  tableView.beginUpdates()
  items = store.load() // Update your controller’s data content inside the table view updates block.
  tableView.deleteRows(at: [indexPath], with: .automatic)
  // Now, items.count == n - 1
  tableView.endUpdates()
}
```


## Exemple complet

Voici un exemple de `UIViewController` implémentant cette solution.

```swift
class MyViewController: UIViewController {

  private items = [Item]() // Your table view data

  private func deleteItem(at indexPath: IndexPath) {
    let store = ItemStore() // Pre-load everything you can
    store.deleteItem(at: indexPath.row) // You can perform the deletion here and persist, but do not update the data in your controller!
    tableView.beginUpdates()
    items = store.load() // Update your controller’s data content inside the table view updates block.
    tableView.deleteRows(at: [indexPath], with: .automatic)
    tableView.endUpdates()
  }

}

extension MyViewController: UITableViewDelegate {

  func tableView(_ tableView: UITableView, trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
    let deleteAction = UIContextualAction(style: .destructive, title: "Delete") {
      [weak self] (_, _, completion) in
      self?.deleteItem(at: indexPath) // Your deletion function
      completion(true)
    }
    return UISwipeActionsConfiguration(actions: [deleteAction])
  }

}
```
