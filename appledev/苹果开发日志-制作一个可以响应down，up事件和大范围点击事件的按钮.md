[Home](https://wecache.com)

# 做一个定制化的按钮

[toc]

知道迟早会碰到做一个定制化的按钮的问题，但没想到这么快就碰到了。起因很简单，就是想做若干个按钮，按钮呢，按下有适当的阴影，松开还原，一开始，以为如此简单的需求，swiftUI 应该是有现成的按钮属性我调用一下就好，结果发现最后还得自己去实现各种细节。

## 苹果的原生按钮

苹果的原生按钮限制很多，最大的限制是其点击范围是以按钮的`Text` 大小做为判定的。尽管在网上找了不少方案，想修改这个范围，但效果都不太好，例如使用`frame` 或者`Color.overlay` 方法来修改点击响应范围，但依然受到`Text` 的大小限制，而且不能完美覆盖整个 Button。所以，很明显，苹果的原生 Button 就是只能用于简单场景下使用的，真是没想到一上来就需要开始自己去做定制化的 UI 了。

## 定制化按钮

既然要定制，就开始自己去写一个有 Button 效果的 View（注意，不是用或者继承 Button，而是直接自己撸一个 View）。话说，这个时候 ChatGPT 直接把我给出的需求实现出来了，我根本没去翻 stack overflow，把 ChatGPT 的代码贴过来就能用了，个人觉得，即使 stack overflow 有解决方案，也不可能做到代码直接贴过来就能用的地步。

ChatGPT 的方案其实很简单，但也让我学到很多，其核心如下：

```swift
    var number_text: some View {
        Text("\(self.title)")
            .frame(width: self.width, height: self.height)
            .background(self.bgColor)
            .border(Color.gray, width: self.borderWidth)
            .overlay(Color.gray.opacity(0.00000001))
            .gesture(
                LongPressGesture(minimumDuration: 0.0001)
                    .sequenced(before: DragGesture(minimumDistance: 0)
                        .onEnded { _ in
                            self.pressUp()
                        })
                    .onChanged { value in
                        switch value {
                        case .first(true):
                            self.pressDown()
                            break
                        default:
                            break
                        }
                    }
            )
    }
```

`frame`是确定按钮的大小，`overlay` 是修改其事件响应的范围。有了这两个，就可以让按钮的事件响应完美匹配按钮的大小了。

关键的是后面的`gesture`，其功能按文档描述是 combine 两个事件，代码中，第一个事件是 `LongPressGesture` 即长按事件，它的持续时间很短，这样相当于模拟了 tap 事件。第二个事件是 `DragGesture` 即拖动事件，其目的是能监听到按钮松开的动作，拖动距离设置为0，也就是即使不拖动，也要收到按钮松开的动作事件，`DragGesture` 结束的时候会调用`pressUp` ，从而做一些松开的调用动作。

两个事件被 combined 后，每个动作有反应都会从 `onChanged` 得到，`first` 表示前一个状态，`second` 表示后一个状态，第一次按下，没有前一个状态，因此 `first`送过来的 true 表示第一次按下，此时调用 `pressDown` 。更多细节，可以参看文档。

## 按钮如何更好看

核心的问题解决了，但具体如何让按钮更好看呢？我觉得所谓的好看，其实就是看上去更有点击感，这也是为什么我开始要做到按下去有阴影，松开还原的原因。

可这个阴影指的是什么？开始做的时候，就是背景白变灰，这是没问题的，但总感觉还是少点什么。参看其它 app，发现他们还会在按钮的周围画上更深的阴影，于是，这个也加上。

但又觉得动画效果太快了，不自然，继续询问 ChatGPT，同样给出答案，使用 `withAnimation` 来控制动画时间，一连串微调参数，总算调出比较满意的动画效果了。

接着是白灰组合不是不好看，而是不同种类的功能的按钮有不同的颜色视觉上更好区分，于是打开 N 久没开的 PS，开始微调 RGB 颜色。需要的效果是，某个颜色按下后蒙上灰色，或者说饱和度增加。发现这么一个规律，Red 和 Green 不变的情况下，仅仅调整 Blue 获得的饱和度比较满意，于是另一个颜色的按钮颜色效果也调好了。

总结：

1、按钮想要有点击感，需要有背景明暗变化；

2、按钮的边框最好也有阴影的变化，加强按下去弹起来的感觉；

3、按钮动画默认情况下太快，可以适当修改，使其更有现实感；

4、可以使用 PS 来细调整效果（想当年我也是广东漫展修图师一枚）。

即使一个简单的 app 藏着的都是细节。