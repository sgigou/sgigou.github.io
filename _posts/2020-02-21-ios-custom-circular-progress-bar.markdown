---
layout: post
title: "A circular progress bar in iOS using Swift 5"
date: 2020-02-21 09:30:00 +0100
categories: iOS
---

In this tutorial, we will create a nice circular progress bar to display progress on your apps.

# What will we do?

Before starting, let me show you the desired output:

![Progress bar animation](/assets/2020-02-21/progress_bar_animation.gif)

The progress bar should be animated, and should be updatable.

As I have never liked tutorials that explain how to create a new project, I will start from the assumption that your project already exists.

We will use two shapes — one for the border and one for the progress bar — and use the strokeEnd property to animate the progress change.

# The progress bar class

You don't have any time to lose, so let's display the complete class directly. If you needs explanations, you will find them after the code block.

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

The first step of the `createLayers` function is to draw the layers.

Firstly, we will define the line width we want. It will be used to draw the border thickness, and the inner space between the border and the progress bar.

```swift
let lineWidth = 2.0
```

Next, we will define the max radius of our component. It will be limited by the smallest width or height of the frame. We will also store the center of the view.

```swift
let radius = min(frame.size.width / 2, frame.size.height / 2)
let viewCenter = CGPoint(x: frame.midX, y: frame.midY)
```

Now, let’s talk about the border drawing — the easiest part.

The path will be a circle around `viewCenter`.

The border is *centered* on the path. This means that it will protrude inside and outside. That is why the path must be smaller by `lineWidth / 2`.

![Border structure](/assets/2020-02-21/border_structure.png)

The angles are expressed in radians; you can take a look at the [Wikipedia page](https://en.wikipedia.org/wiki/Radian) if you do not know this concept. To make a complete circle, you should begin at -π and end at 3×π/2. I hate hadians.

```swift
let borderPath = UIBezierPath(arcCenter: componentCenter, radius: radius - lineWidth / 2, startAngle: -.pi / 2, endAngle: 3 * .pi / 2, clockwise: true)
borderLayer.path = circlePath.cgPath
borderLayer.lineWidth = lineWidth
```

Next, let’s talk about the progress bar. In order to use the `strokeEnd` property to animate our transitions, we will need to use a massive stroke instead of a fill color.

We will define the `progressRadius`, and the progress bar structure will be the same than the border, but the stroke width will be equal to `progressRadius`. The shape will be rolled up, and will look like a circle shape, but will be only composed of a stroke.

Next, we will init the `strokeEnd` property to 0.0.

```swift
let progressRadius = radius - 2 * lineWidth
let progressPath = UIBezierPath(arcCenter: componentCenter, radius: progressRadius / 2, startAngle: -.pi / 2, endAngle: 3 * .pi / 2, clockwise: true)
progressLayer.path = progressPath.cgPath
progressLayer.lineWidth = progressRadius
progressLayer.strokeEnd = 0
```

## Color management

The color management of the border is simple: you have to set the fill color to `.clear` and choose the stroke color.

```swift
borderLayer.fillColor = UIColor.clear.cgColor
borderLayer.strokeColor = .blue
```

The only point you need to pay attention to is the `progressLayer`. As said before, even if it looks like a circle shape filled with a color, it is not! It is only composed of a border rolled-up.

That is why you need to set its colors the same way as the `borderLayer`.

```swift
progressLayer.fillColor = UIColor.clear.cgColor
progressLayer.strokeColor = .blue
```

## Animating the updates

We will use `CATransaction` to perform our animations. It will allow to stay simple, and to disable them if needed.

The main animation is code is:

```swift
CATransaction.begin()
progressLayer.strokeEnd = CGFloat(percentage)
CATransaction.commit()
```

When changing the `strokeEnd` property, the transaction will automatically animate it.

If you want to skip the animation, you can ask the `CATransaction` to disable them.

```
CATransaction.setDisableActions(true)
```

# To go further

`CATransaction` is easily customizable. You can adjust the animation duration, the timing function… I will let you take a look at the [Apple documentation](https://developer.apple.com/documentation/quartzcore/catransaction).

The hard-coded values can also be turned into variables: the color, the line width, for example.

You can also watch the frame of the view to update the circle if the size is changed; you can create a function to update the color, and even animate the color change to match the progress value.

# The turnkey solution

Il you like this component, you can use the [NoveCircularProgressView library](https://cocoapods.org/pods/NoveLogger). You can also take a look at the [source code](https://github.com/sgigou/NoveCircularProgressView) to find some inspiration for your own implementation.