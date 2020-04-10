---
layout: post
title: "Add the slide to delete action on UITableView rows"
date: 2020-04-10 18:00:00 +0100
categories: [ios]
---

iOS 11.0 added a new way to manage cells swipe actions. It is now very simple to implement a _Delete_ button.

<img src="/assets/2020-04-10/intro.png" alt="Swipe to delete illustration" width="256">


## Display the _Delete_ button

The only thing you have to implement is the result of the [`tableView(_:trailingSwipeActionsConfigurationForRowAt:)`](https://developer.apple.com/documentation/uikit/uitableviewdelegate/2902367-tableview) function.

It returns a `UISwipeActionsConfiguration`, which is a container of `UIContextualAction`s.

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

Don’t forget to set `self` as a weak reference to avoid retain cycles!


## Perform the deletion animation

Now, you display a beautiful _Delete_ button, that is tappable, and which calls you. Let’s perform a deletion animation of our row!


### The table view deletion animation

To display a deletion animation for a given row is quite simple; you only have to call the table view’s `deleteRows(at: [indexPath], with: .automatic)` function. You can even choose the animation you want: `.fade`, `.right`, `.left`, `.top`, `.bottom`, `.none`, `.middle` or `.automatic`.

The `.automatic` one is the one used by the system.

```swift
private func deleteItem(at indexPath: IndexPath) {
  tableView.beginUpdates()
  tableView.deleteRows(at: [indexPath], with: .automatic)
  tableView.endUpdates()
}
```


### The data reloading

The tricky part is here. If you just reload your data, then perform the tableview updates, you will end with an error telling you that the number of rows before and after the animation is the same, and that it should not (as you deleted one row).

You should organize you data update like this:

```swift
private func deleteItem(at indexPath: IndexPath) {
  let store = ItemStore() // Pre-load everything you can
  store.deleteItem(at: indexPath.row) // You can perform the deletion here and persist, but do not update the data in your controller!
  tableView.beginUpdates()
  items = store.load() // Update your controller’s data content inside the table view updates block.
  tableView.deleteRows(at: [indexPath], with: .automatic)
  tableView.endUpdates()
}
```


## Full example

Here is an excerpt of a class implementing this solution.

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