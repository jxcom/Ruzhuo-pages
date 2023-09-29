[Home](https://wecache.com)

# 【swift基础】增删查改之删（更多核心概念的理解）

[toc]

前面说了通过绑定，只写少量代码就可以把数据显示在 UI 上，其中的核心是 binding inspector 的 bind to 这个功能，以及用代码动态实现 bind to。

这篇文章通过实现删除功能，进一步理解 UI 开发的核心概念 —— sender 和 receiver，以及 Controller Key。当然还是老样子，不会讲太多操作细节，更多是集中在思路，核心概念上。

## 删除的思路

假设我们现在要做一个删除按钮，选中一个 student 后，点击删除按钮，就删去选中的 student。怎么实现呢？

如图，之前实现的绑定关系如下：

![](https://wecache.com/appledev/bindtomore.svg)

因此，按照前面“无代码”的思路，应该是这个按钮绑定 Array controller，点击按钮后，调用 Array controller 的删除接口，由于 Array controller 和 self.students 绑定，因此，self.students 也会删除相应的数据，同样，table view 上也会反应出删除的数据被从视图上删除了。

那么这里就引入一个核心的概念，即“删除按钮调用 Array controller 的删除接口”——这个动作，抽象一点叫 “删除按钮发消息给 Array controller 删除”，再抽象，叫 “ 删除按钮发删除消息给 Array controller 执行删除操作”，再抽象呢，叫”sender 发 remove 消息给 receiver“， sender 即删除按钮， receiver 即 Array controller，消息即 remove。

那么，我们怎么知道一个控件有什么消息可以接收的呢？上篇介绍的 connection inspector 中就有详细的展示，例如 Array controller 的 connection inspector 就有 received actions 展示：

![](https://wecache.com/appledev/bindtomore2.png)

Årray controller 对应的类 NSArrayController 的文档中，有对应 action 函数的说明：

![](https://wecache.com/appledev/bindtomore3.png)

因此，我们上一节学会用 content bind 来绑定数据，最终在 UI 上展示数据，这里就要学会懂得发 action 消息给 receiver，实现对 receiver 的操作。

根据经典操作，我们 control 拖动删除按钮控件到 Array controller 上，然后在弹出的菜单中选择 remove 就可以不写任何一行代码删除 UI 以及其对应绑定的数据集（即 self.students），这个操作也可以在 connection inspector 中通过拖动 + 号来实现（别忘了 connection inspector 除了可以监视到各种绑定关系，还是拖拉操作的地方）

![](https://wecache.com/appledev/bindtomore4.png)

![](https://wecache.com/appledev/bindtomore5.png)

### 总结

通过 sender-receiver 的 action 操作，可以实现简单的删除等操作。一个控件（receicer）能接受什么样的 action 可以在 connection inspector 这里看到，通过拖拉操作，可以实现 `receiver.action(sender)`这样的管理操作。另外，可以通过查阅相应的类的文档看到一些有益的介绍。



## 更精细的操作

上面的删除动作不需要写任何一行代码就可以实现，但有一个缺点，比如我们要在删除本地数据之后或之前向服务器发送删除指令，这样服务器才知道需要执行删除操作删除服务器的数据，那么，光靠上面的操作是无法实现的。这就需要我们自己写代码实现。

同之前的设想一样，若我们要定制化的操作，则需要在代码中自己去实现拖拉操作中的消息发送。回顾之前我们在 storyboard 中都做了什么：

1、绑定数据和控件；

2、控件发送消息；

对于第一个，由于我们不需要动态绑定（因为这些绑定关系在编译前就确定了），因此不需要代码实现，直接放在 storyboard 中做即可。

对于第二个，我们就需要动手写代码了。和之前不一样的是，这回我们不是把 button control 拖到 Array controller 上，而是拖到 view contorller 的代码上，并且在弹出的对话框上填写函数名，表明若点击这个 button 就会调用这个函数：

![](https://wecache.com/appledev/bindtomore6.png)

在函数中，我们需要向 Array controller 发送 remove 消息，但 view controller 中并没有 Array controller 对象，因此，再次使用拖动的方法把拖出 Array contorller 对象，这样，我们就有了响应点击的函数和 receiver 对象，于是可以写代码了：

```swift
    @IBOutlet var myArrayController: NSArrayController!
    @IBAction func pushDeleteButton(_ sender: Any) {
        let deleted = self.students[self.myTableView.selectedRow]
        print("\(deleted) is being deleted")
        // add the code to delete the data from the server
        // ...
        // ...
        
        // send the remove to the receiver
        self.myArrayController.remove(sender)
    }
```

这样我们就完成了删除时顺便做些什么的功能。

## 回到 Controller key 和 Model key path

前面说到可以通过 connection inspector 查看控件都支持什么 action，且可以通过对应类的文档查到 action 的文档描述。那么，之前我们绑定某个控件时，指定 controller key 下面的选项是不是也可以查到是什么呢？答案是的，前面 table view 绑定到 array controller 的时候，content 栏选择了 array controller，controller key 则使用了 arrangedObjects，这个 arrangedObjects 是什么来头呢？查看 array controller 对应的类 NSArrayController，会发现，其实 arrangedObjects 是 NSArrayController 的一个字段，其类型是一个数组，数组是其成员函数 arrange 处理的结果。因此在 binding inspector 的 content 栏中，绑定 array controller 后，指定 controller key 为 arrangedObjects，其实就是读取这个类的 arrangedObjects 字段内容。

也就是说， controller key 这个设置其实就是设置 cocoa 中定义好的某个字段值。而下面的 model key path 则是用户自定义的某个字段值。



## 思维导图

![](https://wecache.com/appledev/swift-binding2.svg)