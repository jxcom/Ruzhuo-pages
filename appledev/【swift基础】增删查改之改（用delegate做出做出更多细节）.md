[Home](https://wecache.com)

# 【swift基础】增删查改之改（用delegate做出做出更多细节）

[toc]

前面两篇文章主要讲的是：

1、控件之间和控件和代码之间的绑定从而做出 value 展示的方法；

2、用控件之间的 action 消息发送的方式实现控件和数据之间的操作。

这篇将会讲 delegate。

如果说 action 消息机制是 sender 和 receiver 之间的调用，那么 delegate 就是系统给我们的 UI （一般是 view controller）的一个回调。

回顾一下之前实现删除的例子：拖动删除按钮到 view controller 类上，创造出一个 `@IBAction` 函数 `pushDeleteButton`，当点击删除按钮的时候，删除按钮（sender）会发消息到 view controller 上，在这个函数中调用 `self.myArrayController.remove(sender)`，即处罚删除按钮发 remove 消息到 array controller 上，从而实现删除功能，当然也可以直接在 storyboard 上直接通过拖动的方式实现，之所以在代码中实现，是因为在 array controller 删除前，我们可能还要做一些前置动作，比如发消息给服务器告诉删除对应的数据。

## 用 action 消息实现改操作

以上就是通过 sender 和 receiver 的方式实现删除的过程。实现修改也可以一样，例如前面的例子中我们要修改 name 这个字段：

1）把对应的 column 改成可编辑（在 attribute inspector 的 behavior 修改）；

2）control 拖动 column 下的 cell 控件到 view controller 类上，创建函数 `clickNameToEdit` ，其入参是 cell 中的控件 NSTextField，即 sender，receiver 当然是 view controller，当完成编辑某一行的 name 字段时，就会发送这个 `clickNameToEdit` 消息，进而从 sender 处获得编辑后的新值，从而可以写入 self.students 对应的字段值中：

![](https://wecache.com/appledev/swift-editable1.png)

```swift
    @IBAction func clickNameToEdit(_ sender: NSTextField) {
        // send the action to the server to update the name
        // ...
      
        // update the name in the local
        self.students[self.myTableView.row(for: sender)].name = sender.stringValue
    }
```

顺带一提的是，table view 的 row(for) 方法可以通过 table view 的子控件获得对应的行号，很方便。

## 用 delegate 实现更细节的东西

用 action 消息实现改还是蛮方便的，一般情况下也够了。但设想有这么个需求：

在修改某些 name 的时候，我们需要检查权限，若有权限，则可以修改，否则不准修改。

如果使用上面的 action 消息去做，是很难做到的，此时有两种方法：

### 使用值绑定

如之前的 bind to 一样，我们把 table view 的 cell 绑定到 student 的一个新建字段 isEditable：

![](https://wecache.com/appledev/swift-editable2.png)

然后在每次读取这个字段时去判断是否真的 editable 即可：

```swift
    @objc var isEditable: Bool {
        get {
            // check if the previlege exist
            // ...
            
            return true
        }
    }
```

但这么做有一个非常不好的地方，如果 student 的数量很多，那么若每次显示 10 个 students，则需要判断 10 次，但实际只选择其中一个编辑而已。这当然是非常低效的做法（因为权限判断很有可能需要联网请求）。

### 使用 delegate

使用 delegate 的思路是，当我试图去编辑某一个 cell 的时候，系统回调我的函数，我此时再去 check 是否有权限，这样，如果用户不编辑某个 cell，就不会触发权限判断逻辑。

这也就是 delegate 的核心思路：系统要做什么之前或者之时或者之后都有可能会通知我一下，让我决定或者知晓。

那么怎么实现 delegate 呢？至少有三件事情要做

1、处理 delegate 的回调类需要实现对应的 protocol；

2、在 storyboard 绑定 delegate 关系（也可以在代码中绑定）；

3、实现需要关注的回调方法。

#### 查询控件提供的 delegate

对于一个控件，其 delegate 是控件名 + Delegate，例如上面说的 cell，其类名是 `NSTextField`那么对应的 delegate 就是 `NSTextFieldDelegate` ，翻阅文档即可知道，系统都提供了什么回调方法给我们：

![](https://wecache.com/appledev/swift-editable3.png)

看上去并没有什么可以让我们在编辑的时候检查权限的方法。但我们发现 `NSTextFieldDelegate` 继承了 `NSControlTextEditingDelegate`，因此，我们继续翻阅：

![](https://wecache.com/appledev/swift-editable4.png)

至此，找到了一个合适的回调，在编辑时检查权限，从而可以拦截住没有权限的操作。

#### NS 类体系下的层级关系

如果我们继续查看继承关系，会发现 `NSTextFieldDelegate`最终其 protocol 的根是 `NSObjectProtocol`，而 `NSTextField` 的最终父类是 `NSObject`。这里就想讲一下关于继承的问题。

1、为什么有的类是继承至 `NSObjectProtocol`而有的是`NSObject`？

目的上，都是一样的，都是为了让我们去自定义一个 NS 类，但手法上`NSObject` 是使用传统的类继承，即把 `NSObject` 的所有属性都继承下来，从而实现 NS 类。而`NSObjectProtocol` 则是用 protocol 得方法，即只要实现其方法就可以让自己的类看上去是一个 NS 类。前者是重量级的实现方式，因为必须把`NSObject` 的实现都继承下来，后者则是轻量级实现方式，因为只要想办法把相关的`NSObjectProtocol` 方法实现出来即可，如何实现没有要求。

对于 Delegate 类，因为都是回调，具体实现 cocoa 是不管的，因此都是使用 `NSObjectProtocol` 作为根 protocol。

2、要留意继承关系。

很多 delegate 类都是多重继承的，因此，如果某个想要的回调函数找不到的话，要么它本身不支持，要么就是它放在了父类上。因此阅读苹果文档时，我们一般从 NSXXXXXDelegate 开始，但也要注意其继承的父类，看父类都给了什么更多的回调方法。

### 实现编辑时检查权限

了解以上原理·后，实现编辑时检查就变得非常容易了。

#### 把 cell 的delegate 绑定给 view controller

首先，必须继承 `NSTextFieldDelegate`：

```swift
extension ViewController: NSTableViewDelegate,
                          NSTableViewDataSource,
                          NSTextFieldDelegate {
                            // ...
                          }
```

然后在 connection inspector 中拖动 delegate 到 view controller：

![](https://wecache.com/appledev/swift-editable5.png)

也可以用代码实现，即 control 拖动 cell 控件到 view controller 类上获得 cell 控件对象（即增加 @IBOutlet），例如名字叫 myCell，则绑定代码为

```swift
self.myCell.delegagte = self
```

可以在 view controller 中的 `viewDidLoad`中实现。

#### 实现回调函数

实现以下回调函数即可：

```swift
    func control(
        _ control: NSControl,
        textShouldBeginEditing fieldEditor: NSText
    ) -> Bool {
        guard let tableview = control.superview?.superview?.superview as? NSTableView else {
            logger.debug("the parent not the table view")
            return false
        }
        
        let student = self.students[tableview.row(for: control)]
        
        logger.debug("check student.id( = \(student.id) % 2 == 0")
        return student.id % 2 == 0
    }
```

这个回调函数在编辑一个 cell 时，cell 作为 sender 会回调 view controller，注意第 5 行，其获得父类控件对象 table view 的方式是不停的取 `supperview` ，那么怎么知道到底哪一个才是 table view 对象呢？只要看 storyboard 即可：

![](https://wecache.com/appledev/swift-editable6.png)

可以看到，cell 的第三个父类对象是 table view 类，因此取第三个父类即可。

后面的 id 是否可以被 2 整除则是业务逻辑，无需关心。

这样我们就可以在编辑的时候才去检查权限了。

## 更新脑图

![](https://wecache.com/appledev/swift-binding3.svg)

