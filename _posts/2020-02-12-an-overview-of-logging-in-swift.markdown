---
layout: post
title: "An overview of logging in Swift"
date: 2020-02-12
categories: mobile
---

Les logs sont un outil puissant. Ils peuvent vous aider à détecter des problèmes et à les résoudre. Néanmoins, ils peuvent également être dangereux.


## Méthodes de log

Plusieurs méthodes existent, avec chacune ses spécificités.
Pour illustrer les sorties de ces fonctions, j’utiliserai une `struct` commune :
``` swift
struct Human {
  let name: String
  let age: Int
}
```


### `print`

``` swift
func print(_ items: Any..., separator: String = " ", terminator: String = "\n")
```
C’est la plus simple des fonctions de log en Swift. Elle affichera votre message sur la console Xcode lorsque l’appareil est connecté.
Cette fonction ne supporte pas le _formatting_, mais vous pouvez utiliser l’interpolation de _strings_ — par exemple : `print("My human: \(human).")`.

``` swift
print(human)
// Human(name: "Steve", age: 30)
```

Dans Swift 1, vous aviez deux fonctions : `print()` qui n’ajoutait pas de retour chariot, et `println()` qui en ajoutait un.
Ces fonctions ont été fusionnées en une seule fonction `print()` qui ajoute un `\n` en fin de message. Vous pouvez toujours retirer le retour chariot en spécifiant `""` comme `terminator` :
``` swift
print("A first print", terminator: "")
print("A second print")
// Outputs: A first printA second print
```

Pour plus d’informations, vous pouvez consulter la [documentation officielle](https://developer.apple.com/documentation/swift/1541053-print).

### `debugPrint`
``` swift
func debugPrint(_ items: Any..., separator: String = " ", terminator: String = "\n")
```
Elle fonctionne quasiment comme `print()`. La différence réside en la méthode appelée pour afficher les objets passés en argument. `print()` appellera la fonction `description()`, et `debugPrint()` appellera la fonction `debugDescription()`, qui affiche plus d’informations par défaut.

``` swift
debugPrint(human)
// __lldb_expr_6.Human(name: "Steve", age: 30)
```

Pour plus d’informations, vous pouvez consulter la [documentation officielle](https://developer.apple.com/documentation/swift/1541053-print).

### `dump`
``` swift
@discardableResult func dump<T>(_ value: T, name: String? = nil, indent: Int = 0, maxDepth: Int = .max, maxItems: Int = .max) -> T
```
Cette fonction permet d’afficher un objet. Vous pouvez spécifier la profondeur désirée, le nombre d’éléments affichés dans une collection, et même le nombre d’espaces utilisées pour l’indentation.

``` swift
dump(human)
```
Sortira :
```
▿ __lldb_expr_6.Human
  - name: "Steve"
  - age: 30
```

Pour plus d’informations, vous pouvez consulter la [documentation officielle](https://developer.apple.com/documentation/swift/1541053-print).

### `NSLog`
``` swift
func NSLog(_ format: String, _ args: CVarArg...)
```
Elle est bien plus lente que la fonction `print()`.
Elle ajoutera automatiquement certaines informations à votre message : 
```
<Date> <Time> <Program name>[<Process ID>:<Thread ID>] <Message>
2016-07-16 08:58:04.681 test[46259:1244773] NSLog message
```
Elle permet également d’utiliser des formats :
``` swift
NSLog("%0.4f", CGFloat.pi)
```

La plus grande différence avec les fonctions précédentes est que vos logs seront affichés sur la console Xcode _et_ la console présente sur l’appareil.

Pour plus d’informations, vous pouvez — essayer de — vous référer à la [documentation officielle](https://developer.apple.com/documentation/foundation/1409759-nslog).

### `os_log`

``` swift
#define os_log(log, format, ...)
```

C’est le nouveau standard de logs, disponible depuis iOS 10 et macOS 10.12.
Le système doit être importé, et vous pouvez contrôler le sous-système et la catégorie de votre message. Vous pouvez également choisir le niveau de log entre `.default`, `.info`, `.debug`, `.error` et `.fault`.

Vous pouvez utiliser les formats, mais pas l’interpolation.

``` swift
import os.log

let log = OSLog(subsystem: Bundle.main.bundleIdentifier!, category: "network")
os_log("url = %@", log: log, url.absoluteString)
```

La sortie sera effectuée sur la console Xcode, mais également sur la console de l’appareil. Il est conseillé d’utiliser l’application `Console.app` pour les afficher — vous pourrez ainsi les filtrer, et profiter de toute la puissance de ce système.

`os_log` est plus complexe à utiliser que les autres méthodes, donc je n’irai pas plus loin puisque cet article n’est qu’un aperçu. Il mériterait un article complet.

Si cela vous intéresse, vous pouvez lire la [documentation officielle](https://developer.apple.com/documentation/os/os_log?language=occ) ou regarder la vidéo suivante : [WWDC 2016 video about Unified Logging and Activity Tracing](https://developer.apple.com/videos/play/wwdc2016/721/).


## Sortie personnalisée

Comme je l’ai dit précédemment, `print()` et `debugPrint()` s’appuient sur des fonctions prédéfinies pour afficher les objets : `description()` et `debugDescription()`.
Vous pouvez implémenter le protocol `CustomStringConvertible` (ou `CustomDebugStringConvertible `) pour écraser les implémentations par défaut et afficher les objets comme vous le souhaitez :

``` swift
struct Human: CustomStringConvertible {
  let name: String
  let age: Int

  var description: String {
    return "Human named \(name) (\(age) years old)"
  }
}

let human = Human(name: "Steve", age: 30)

print("Current human: \(human)")
```

Aura pour sortie :

```
Current human: Human named Steve (30 years old)
```


## Les problèmes des logs

Les logs souffrent de problèmes de performance et de sécurité.


### Les problèmes de performance

Aussi évident que cela puisse paraître : les logs consomment des ressources.

Vous pourriez être tenté d’ajouter des logs partout durant le développement. Ainsi, vous pourriez retracer très exactement le cheminement de chaque action de l’utilisateur.
Cela deviendrait néanmoins un problème en production, où ces logs ne sont plus forcément nécessaires, mais ralentiront votre application.

Si vous voulez en lire plus sur les problèmes de performance, vous pouvez jeter un œil à cet article : [Why print is dangerous](https://medium.com/ios-os-x-development/swift-log-devil-or-why-println-is-dangerous-46390453353d).


### Les problèmes de sécurité

Comme je l’ai dit auparavant, `NSLog()` et `os_log` auront une sortie sur les logs de l’appareil. Cela signifie que l’utilisateur est capable de les voir si il sait comment utiliser Xcode ou l’application Console. Pire encore, cela veut dire que ces logs peuvent être volés.

Vous imaginez aisément les problèmes de sécurité qui peuvent survenir si vous logguez des informations confidentielles ou à risque.


### Une solution commune

Une solution simple consiste à ne conserver certains logs que pour le mode `DEBUG` :

``` swift
#if DEBUG
  print("Log not wanted in production.")
#endif
NSLog("Production-safe log")
```

Avec cette solution, vous pouvez filtrer les logs qui seront présents sur l’appareil. Je vous conseille de conserver les logs critiques uniquement en production — les logs que vous classifiez en `ERROR` ou `WARNING`.

Les logs de debug ne ralentiront pas votre app, et les informations sensibles ne seront ainsi affichées qu’en développement.


## La bibliothèque NoveLogger

Pour faciliter mon travail, j’ai créé une bibliothèque simple pour gérer mes logs.

Elle supporte les niveaux de logs, et affiche le fichier, la fonction et même la ligne à laquelle est effectuée le log. Elle vous permet également de définir quels logs doivent être conservés en production.

Elle est disponible sur [CocoaPods](https://cocoapods.org/pods/NoveLogger) et vous pouvez jeter un œil aux sources sur [GitHub](https://github.com/sgigou/NoveLogger).