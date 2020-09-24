---
layout: post
title: "Rendre un objet d’environnement SwiftUI optionnel"
date: 2020-08-14 18:00:00 +0100
categories: dev
tags: ["iOS","iOS 13","Swift","Swift 5", "SwiftUI"]
---

Avoir la possibilité d’assigner `nil` à un _environment object_ en SwiftUI est très pratique.

Par exemple, imaginons que notre application ait besoin d’afficher un écran de connexion. Un manière élégante de le réaliser serait d’injecter une propriété `user` optionnelle dans l’environnement. La vue principale de l’application pourrait alors vérifier si l’utilisateur est nul pour afficher le formulaire.


## Le _wrapper_

Permettre directement à un _environment object_ d’être nul est difficile et peu pratique.

La manière la plus simple d’arriver à nos fins est d’utiliser un _wrapper_. Il ne sera pas optionnel, mais il contiendra notre propriété _nullable_.

```swift
class UserWrapper {
  @Published var user: UserEntity?
}
```


## Comment l’injecter ?

L’injection du _wrapper_ se fera comme d’habitude :

```swift
let userWrapper = UserWrapper()
userWrapper.user = // Load the user or set it to nil
let view = LandingView().environmentObject(userWrapper)
```


## Comment l’utiliser dans les vues ?

Dans vos vues, vous pouvez dorénavant vérifier l’existence de l’entité `user` :

```swift
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

Après que le login doit effectué, vous n’aurez qu’à assigner `userWrapper.user = loggedInUser`, et la `LandingView` sera rechargée pour afficher la page d’accueil au lieu du formulaire.

D’un autre côté, si vous souhaitez déconnecter l’utilisateur, vous n’avez qu’à effectuer `userWrapper.user = nil` pour afficher le formulaire de connexion à nouveau.


## Comment simuler le comportement ?

Vous pouvez simuler votre vue aisément, et même déclencher une connexion / déconnexion après quelques secondes pour tester la transition :

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
