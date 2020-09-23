---
layout: post
title: "Une barre de progression circulaire en Swift"
date: 2020-02-21 09:30:00 +0100
categories: dev
tags: swift swift5 ios ios13
---

Dans ce tutoriel, nous allons créer une jolie barre de progression circulaire.

# Qu’allons-nous réaliser exactement ?

Avant de commencer, laissez-moi vous montrer le rendu souhaité :

<img src="/assets/2020-02-21/progress_bar_animation.gif" alt="Progress bar animation" width="125">

La barre doit être animée et pouvoir être mise à jour.

Comme je n’ai jamais apprécié les tutoriels qui commencent par vous demander d’allumer votre Mac, je ne vous demanderai pas de créer un nouveau projet.

Nous utiliserons deux formes — une pour la bordure, et une pour la barre de progression — et utiliserons la propriété `strokeEnd` pour animer les mises à jour.

# La classe de la barre de progression

Votre temps est précieux, donc je vous donne la classe complète directement. Si vous avez besoin d’explications, vous les trouverez après le bloc de code.

```swift
@IBDesignable public class CircularProgressBar: UIView {
  
  private(set) var progress: Double = 0.0
  
  private var borderLayer = CAShapeLayer()
  private var progressLayer = CAShapeLayer()
  
  // MARK: Life cycle
  
  override public init(frame: CGRect) {
    super.init(frame: frame)
    createLayers()
  }
  
  required public init?(coder: NSCoder) {
    super.init(coder: coder)
    createLayers()
  }
  
  // MARK: Drawing
  
  private func createLayers() {
    let lineWidth = 2.0
    let radius = min(frame.size.width / 2, frame.size.height / 2)
    let viewCenter = CGPoint(x: frame.midX, y: frame.midY)
    let borderPath = UIBezierPath(arcCenter: componentCenter, radius: radius - lineWidth / 2, startAngle: -.pi / 2, endAngle: 3 * .pi / 2, clockwise: true)
    let progressRadius = radius - 2 * lineWidth
    let progressPath = UIBezierPath(arcCenter: componentCenter, radius: progressRadius / 2, startAngle: -.pi / 2, endAngle: 3 * .pi / 2, clockwise: true)
    borderLayer.path = circlePath.cgPath
    borderLayer.lineWidth = lineWidth
    progressLayer.path = progressPath.cgPath
    progressLayer.lineWidth = progressRadius
    progressLayer.strokeEnd = 0
    
    borderLayer.fillColor = UIColor.clear.cgColor
    borderLayer.strokeColor = .blue
    progressLayer.fillColor = UIColor.clear.cgColor
    progressLayer.strokeColor = .blue
    
    layer.addSublayer(borderLayer)
    layer.addSublayer(progressLayer)
  }
  
  // MARK: Component update
  
  func updateProgress(to percentage: Double, animated: Bool) {
    if percentage == progress { return }
    CATransaction.begin()
    if !animated {
      CATransaction.setDisableActions(true)
    }
    progressLayer.strokeEnd = CGFloat(percentage)
    CATransaction.commit()
    progress = percentage
  }
  
}
```

## Layer paths

La première étape de la fonction `createLayers` sert à dessiner les calques.

Dans un premier temps, nous définissons la largeur de ligne que nous souhaitons. Ce sera utilisé pour l’épaisseur de la bordure, et l’espace entre la bordure et la barre de progression.

```swift
let lineWidth = 2.0
```

Ensuite, nous définissons le rayon maximum de notre composant. Il sera limité par la plus petite hauteur ou largeur de la frame. Nous stockons également le centre de la vue.

```swift
let radius = min(frame.size.width / 2, frame.size.height / 2)
let viewCenter = CGPoint(x: frame.midX, y: frame.midY)
```

Maintenant, dessinons la bordure — c’est la partie facile.

Le chemin sera un cercle centré autour de `viewCenter`.

La bordure est *centrée* sur le chemin. Cela signifie qu’elle dépassera à l’intérieur et à l’extérieur. C’est pourquoi le chemin doit être plus petit que le rayon souhaité de `lineWidth / 2`.

<img src="/assets/2020-02-21/border_structure.png" alt="Border structure" width="126">

Les angles sont exprimés en radians ; vous pouvez jeter un œil à la [page Wikipedia](https://en.wikipedia.org/wiki/Radian) si vous n’êtes pas familiers avec ce concept. Pour réaliser un cercle complet, vous devez commencer à -π et terminer à 3×π/2. Je hais les radians.

```swift
let borderPath = UIBezierPath(arcCenter: componentCenter, radius: radius - lineWidth / 2, startAngle: -.pi / 2, endAngle: 3 * .pi / 2, clockwise: true)
borderLayer.path = circlePath.cgPath
borderLayer.lineWidth = lineWidth
```

Ensuite, parlons de la barre de progression. Pour utiliser la propriété `strokeEnd` pour animer nos transitions — ce qui est extrêmement pratique — nous allons devoir utilise une ligne très épaisse plutôt qu’une couleur de remplissage.

Nous allons définir le rayon de la barre de progression, et la structure sera au final la même que la bordure ; simplement, l’épaisseur du trait sera égale au rayon de la barre. La forme sera enroulée sur elle-même, et ressemblera à un cercle rempli, alors qu’elle ne sera qu’un trait très épais.

Ensuit, nous allons initialiser la propriété `strokeEnd` à 0.0 — ce qui correspond à ne rien dessiner.

```swift
let progressRadius = radius - 2 * lineWidth
let progressPath = UIBezierPath(arcCenter: componentCenter, radius: progressRadius / 2, startAngle: -.pi / 2, endAngle: 3 * .pi / 2, clockwise: true)
progressLayer.path = progressPath.cgPath
progressLayer.lineWidth = progressRadius
progressLayer.strokeEnd = 0
```


## La gestion des couleurs

La gestion des couleurs pour la bordure extérieure est simple : vous devez assigner la couleur de remplissage à `.clear` et choisir uniquement la couleur de la bordure.

```swift
borderLayer.fillColor = UIColor.clear.cgColor
borderLayer.strokeColor = .blue
```

Le seul point auquel vous devez prêter attention est au `progressLayer`. Comme je l’ai dit précédemment, bien qu’il ressemble à un cercle rempli, il ne s’agit que d’une ligne très épaisse et enroulée sur elle-même.

C’est pourquoi pour lui également, il ne faut définir que la couleur de la bordure.

```swift
progressLayer.fillColor = UIColor.clear.cgColor
progressLayer.strokeColor = .blue
```


## Animer les mises à jour

Nous utiliserons `CATransaction` pour réaliser les animations. C’est très simple et ça peut être désactivé si besoin.

Le code principal des animations est :

```swift
CATransaction.begin()
progressLayer.strokeEnd = CGFloat(percentage)
CATransaction.commit()
```

Lorsque nous changeons la valeur de `strokeEnd`, la transaction va _automatiquement_ l’animer — et oui.

Si vous souhaitez sauter l’animation, vous pouvez demander à `CATransaction` de les désactiver.

```
CATransaction.setDisableActions(true)
```


## Pour aller plus loin

`CATransaction` est simple à personnaliser. Vous pouvez changer la durée de l’animation, la fonction temporelle… je vous laisser regarder la [documentation d’Apple](https://developer.apple.com/documentation/quartzcore/catransaction).

Les valeurs codées en dur dans le tutoriel peuvent également être changées en variables : la couleur, la taille, l’épaisseur…

Vous pouvez enfin vous abonner aux changement de la frame de la vue pour mettre à jour le composant si la taille est modifiée ; vous pouvez également créer une fonction pour animer le changement de couleur, voire changer la couleur en fonction de la valeur affichée.


## La solution clef en main

Si vous aimez ce composant, vous pouvez utiliser la bibliothèque [NoveCircularProgressBar](https://cocoapods.org/pods/NoveCircularProgressBar). Elle fait tout ce qui est décrit dans ce tutoriel, et même plus…

Vous pouvez également jeter un œil au [code source](https://github.com/sgigou/NoveCircularProgressBar) pour trouver de l’inspiration pour votre propre implémentation.