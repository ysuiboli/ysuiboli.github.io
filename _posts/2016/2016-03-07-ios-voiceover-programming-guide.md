---
layout: post
title: iOS VoiceOver Programming Guide
categories:
- Programming
tags:
- iOS
- Accessibility
---

## 前言

VoiceOver是苹果“读屏”技术的名称，属于辅助功能的一部分。VoiceOver可以读出屏幕上的信息，以帮助盲人进行人机交互。
这项技术在苹果的各个系统中都可以看到，OS X，iOS，watchOS，甚至tvOS。
苹果公司的VoiceOver在2015年6月18日获得了美国盲人基金会（American Foundation for the Blind, AFB）颁发的海伦凯勒成就奖，成为全球首家获得此殊荣的科技公司。
单从iOS来说，iOS的VoiceOver功能可以毫不夸张的说是三大移动平台中做的最好的。

虽然说苹果默认的UI组件都已经默认支持VoiceOver功能了，但是通常情况下App还是需要对VoiceOver进行适配和优化的，比如说一些自定义复杂UI组件。

## 基本使用
iPhone上开启VoiceOver功能后，就可以通过 **单指左右轻扫** 来遍历当前界面中的所有的AccessibilityElement（可以被VoiceOver访问的UI元素）,当一个AccessibilityElement被选中后，VoiceOver会将AccessibilityElement的信息读出来。 **单指轻点两次** 能够激活当前元素对应的操作，比如当前AccessibilityElement是一个按钮，那么对应的就是按钮的Action事件。

简单点来说在App开发过程中关于VoiceOver我们需要关注如下几点：

- 界面上的AccessibilityElement有哪些
- AccessibilityElement的位置和形状
- AccessibilityElement的信息是什么（就是Element被选中后，被读出来内容）
- AccessibilityElement所能响应的的事件有哪些

UIKit中的控件基本都是 VoiceOver Ready的，即使是UIView，你也可以通过简单的设置其实变成AccessibilityElement。所以这一小节中所讲的AccessibilityElement其实都是UIView或其子类的实例。   
相关属性和方法基本都在`UIAccessibility.h`这个头文件中进行了声明。

所以简单的情况下通过UIView的`isAccessibilityElement`属性就可以控制某个View是否是AccessibilityElement，在UIKit的控件中，像UILabel，UIButton 这些控件的isAccessibilityElement属性默认就是true的，UIView这个属性默认是false。

一般情况下AccessibilityElement的位置和形状是通过accessibilityFrame进行设置的，默认值是View在屏幕中的位置，形状就是View的矩形形状。如果你想自己设置accessibilityFrame的值，那么得注意下，这边的frame值是相对于设备Screen的坐标系的，当然可以通过UIAccessibilityConvertFrameToScreenCoordinates函数来帮助转换。此函数有两个参数，一个rect，一个是view， 其含义就是将相对于view这个坐标系的rect转换成相对于screen坐标系的值并返回。所以一般情况下 rect可以是目标Element在父View中的frame，view就为其父view。

```
public func UIAccessibilityConvertFrameToScreenCoordinates(rect: CGRect, _ view: UIView) -> CGRect
```

如果你想设置非矩形的形状，你也可以通过给 **accessibilityPath** 属性指定一个UIBezierPath类型的值来自定义AccessibilityElement的形状。


至于AccessibilityElement的信息可以通过下面几个UIAccessibility的属性来决定

- accessibilityLabel  这是什么
- accessibilityHint 这个有什么用，会产生什么样的结果
- accessibilityValue 这个的 **值** 是什么
- accessibilityTraits 这个的类型以及状态，就是通过traits来表征这个Element的特质，数据类型是一个枚举类型，可以通过按位或的方式合并多个特性。

这里有个需要注意的就是，当某个View的是AccessibilityElement的时候 ，其subviews都会被屏蔽掉，这个特性有时候还是有用的，比如一个View中包含多个Label，那么你希望每一个下面的Label不要单独可以访问到，那么你可以将这个View设置成可以访问的，然后将其accessibilityLabel设置为所有子Label的accessibilityLabel的合并值。

至于AccessibilityElement的事件，最简单的莫过于上面提到 **单指轻点两次** 能够激活当前元素对应的操作了,如果当前AccessibilityElement实现的`public func accessibilityActivate() -> Bool`这个方法返回true，那边此逻辑将被调用，否则相当于在AccessibilityElement的accessibilityActivationPoint这个位置点上进行了一次Tap操作。

## 高级特性
#### Accessibility Container
设想下这样的一个场景，一个UIView，内部包含一组用户可以进行交互的内容，每一个内容之间是独立的，但是这些内容不是以子View的形式存在，而是通过Quarz 2D或者Core Text渲染而成，所以这部分内容无法通过上面的方式变成AccessibilityElement。这种情况UIView需要按照UIAccessibilityContainer的方式，来将内部的每一个独立的内容都描述成UIAccessibilityElement的实例，这个时候这个UIView我们称之为Accessibility Container。

接下来讲解具体的实现步骤了。
先说说iOS 8之后如何实现，首先Accessibility Container的isAccessibilityElement值必须设置为false，另外我们需要创建出所有的UIAccessibilityElement的实例，然后赋给accessibilityElements属性（iOS 8.0+）
假设在一个UIView的子类中，通过叫做updateAccessibleElements的方法来更新维护所有的UIAccessibilityElement实例，当界面上的内容发生变化，或者VoiceOver开启关闭状态发生变化时，调用此方法以更新accessibilityElements，相关伪代码如下：

```
    internal func updateAccessibleElements() {
        guard UIAccessibilityIsVoiceOverRunning() else {
            self.accessibilityElements = nil
            return
        }
        
        self.isAccessibilityElement = false
        var elements = [AnyObject]()

        let element1 = UIAccessibilityElement(accessibilityContainer: self)
        element1.accessibilityLabel = "element1"
        element1.accessibilityTraits = UIAccessibilityTraitStaticText
        element1.accessibilityFrame = UIAccessibilityConvertFrameToScreenCoordinates(element1FrameInSelf, self)
        elements.append(element1)
        ...
        self.accessibilityElements = elements
    }

```

如果是iOS 8以下，那么就需要实现下面的几个方法来实现了，比较繁琐：

```
// accessibilityElement的个数
public func accessibilityElementCount() -> Int

// 返回指定Index的accessibilityElement
public func accessibilityElementAtIndex(index: Int) -> AnyObject?

// 返回指定accessibilityElement的Index
public func indexOfAccessibilityElement(element: AnyObject) -> Int
```

使用上面第二种方式的时候，往往需要自己维护一个包含所有accessibilityElements的数组，然后通过数组来完成上面的这三个方法的返回。这种方法比较累赘，而且当界面更新需要刷新内容的时候，还需要发送通知系统，告诉系统界面上的accessibilityElements有变动需要更新。所以最小版本定位于iOS 8的情况下还是直接设置accessibilityElements属性的方式比较科学。

关于UIAccessibilityElement这个类的设计还是存在一些疑惑的，既然NSObject的UIAccessibility扩展已经包含了诸如`accessibilityLabel`,`accessibilityHint`这些属性，为何UIAccessibilityElement类中还需要重复声明这些属性，有点重复的感觉。

#### Actions 
之前只讲到了最简单的事件，就是单指轻点两下，其实常见的Actions有下面这些，每一个Action都会对应一个方法，可以通过覆盖方法的方式来自定义Action对应的逻辑：

- Activate 单指轻点两次    
   `public func accessibilityActivate() -> Bool`
- Escape. 单指 Z-shaped 手势一般用于退出模态界面或者返回导航的上一页界面    
  `public func accessibilityPerformEscape() -> Bool`
- Magic Tap. 双指轻点两次触发 most-intended action.      
   `public func accessibilityPerformMagicTap() -> Bool`
- Three-Finger Scroll. 三指滑动触发界面水平或者垂直的滚动    
   `public func accessibilityScroll(direction: UIAccessibilityScrollDirection) -> Bool`
- Increment. 单指向上滑动，需要设置accessibilityTraits为UIAccessibilityTraitAdjustable，否则对应的方法不会被调用    
   `public func accessibilityIncrement()`
- Decrement. 单指向下滑动，需要设置accessibilityTraits为UIAccessibilityTraitAdjustable，否则对应的方法不会被调用   
   `public func accessibilityDecrement()`


这些方法中，其中Escape，Magic Tap，Three-Finger Scroll这几种手势支持在响应链中向上寻找对应的Action方法，首先用户在屏幕上进行相应的手势，系统检查当前VoiceOver的Focus的Element有无实现对应的方法，没有实现的话，则向响应链的上一级寻找。比如有些全局性的操作，对应的方法写在上层View或者ViewController中比较合适。
其实很多系统提供的组件都默认实现了一些VoiceOver的手势Action，比如UINavigationController, UIAlertController 都提供了对Escape手势的支持。

#### Accessibility Notification
Accessibility提供了一系列的通知，可以完成一些特定的需求。比如你可以监听`UIAccessibilityVoiceOverStatusChanged通知，来监控Voice Over功能开启关闭的实时通知 `。

或者是你在App中主动发送一些通知，来让系统做出一些变化，比如当你界面上的AccessibilityElement有变动的时候你可以发送`UIAccessibilityLayoutChangedNotification`通知，通知发送时使用UIAccessibility的专用的函数,UIAccessibilityPostNotification函数有两个参数，第一个是通知名，第二个是你想让VoiceOver读出来的字符串或者是新的VoiceOver的焦点对应的元素。   

```
UIAccessibilityPostNotification(UIAccessibilityLayoutChangedNotification, self.myFirstElement)
```                             

## 建议

- AccessibilityElement的信息尽量简洁，accessibilityLabel不要包含提示性质的文案，避免信息干扰
- TableView的每一个Cell的信息尽量合并，使得Cell变成一个整体的AccessibilityElement，避免无意义的冗余元素之间切换的操作。Cell中有多个按钮的时候，可以考虑使用Magic Tap的方式，Magic Tap的Action中弹出sheet样式的UIAlertController来供用户操作。
- 自定义的模态页面注意设置accessibilityViewIsModal为true，最好支持Escape手势
的方式退出模态页面。
- 将页面中装饰用的没有实际意义的元素的accessibilityElementsHidden设置成true，减少操作过程中的干扰


-----

参考文档:
[UIAccessibility Protocol](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIAccessibility_Protocol/index.html)
[Supporting Accessibility](https://developer.apple.com/library/prerelease/ios/featuredarticles/ViewControllerPGforiPhoneOS/SupportingAccessibility.html)
[Accessibility Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/iPhoneAccessibility/Accessibility_on_iPhone/Accessibility_on_iPhone.html)
