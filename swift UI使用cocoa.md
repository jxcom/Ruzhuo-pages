绑定：绑定两个值，或者说是 x 引用 y，cocoa 中可以随意绑定或者引用

@Binding：绑定者，即引用者

@State： 私有成员被绑定者，即被引用者

@Environment ：绑定系统属性

@Environment(\.openWindow) 系统注册的WindowGroup，可以显示制定的 view


1、swift ui 使用 appkit
在swiftUI中显示 NSView，可以在swift UI 中创建一个 swift UI view 继承 NSViewRepresentable，
实现其中 makeNSView（初始化cocoa view） 和 updateNSView（刷新，显示）。
这个swift UI view 相当于封装了 cocoa view，这样cocoa view就可以在 swift ui 中使用了。
swiftUI 中可以用  makeCoordinator 返回一个 名叫 Coordinator 的对象来实现 swiftUI， cocoa 与 delegate 之间的 交互。

2、appkit 使用swift ui
先有一个swiftUI view 类 xxxx
再在library 中拉一个hosting view controller 控件，此对应的类继承 NSHostingController<xxxx>
hosting类即 NS view，

