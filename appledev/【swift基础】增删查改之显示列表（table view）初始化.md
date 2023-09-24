[Home](https://wecache.com)

# 【swift基础】增删查改之显示列表（table view）初始化

[toc]

增删查改当然是最无聊最无技术含量的工作，但实际又是最基础最核心的编程入门操作。去星巴克买咖啡，打开 app，显示咖啡列表（查咖啡列表），点击拿铁进入下单页面（增加一条订单），选择低脂（修改订单属性），选择付款，最核心的操作其实都是增删查改。今天花时间去总结一下这些最基本的操作，减少日后遇到类似问题时的停留时间，因为我们应该把更多的时间花在产品体验和深挖更多垂直功能上。

当然这里不会只记录操作步骤（操作细节属于枝节末节，教材有很详细的描述，这里不多讲），更多是学习 UI 开发的设计思想和核心操作，日后遇到类似问题可以复用今天的思路，做到举一反三。对操作细节比较熟悉但又觉得很混乱需要理清脉络的开发者（比如我），可以参考本文。

swift 里面制作一个 table view 可以使用 swiftUI 或 UIkit，swiftUI 是最简单最快捷的，在很短时间内就能出很好的效果，但这里用 UIkit，因为还是有很多地方不得不使用 UIkit，可以说，能用 swiftUI 就用swiftUI，但一旦 swiftUI 不能满足时，UIkit 就派上用场了，所以 UIkit 目前还是绕不开。

## 绑定

UIkit 核心的思想其实和当年 MFC 很像即“data model - user interface”。data model 和 user interface 是通过绑定的方式来进行操作的。例如有一个类：

```swift
@objc class Student: NSObject {
    @objc let id: UInt64
    @objc var name: String
    @objc var age: UInt32
    @objc var city: String
    @objc var address: String
    
    init(id: UInt64, name: String, age: UInt32, city: String, address: String) {
        self.id = id
        self.name = name
        self.age = age
        self.city = city
        self.address = address
    }
}

```

我们有这个类的列表（数组）students，它是某个 view controller 的成员。现在的任务是把这个 students 展示到 table view 上。也就是 students 就是我们的 data model，table view 就是 user interface。它们的绑定是使用 storyboard 的 connection inspector 和 binding inspector 来绑定的。



## 建立 data 到 控件的绑定关系

### connection inspector

主要有两个功能：

1、查看各种绑定关系；

2、拖拉操作建立绑定关系。绑定关系主要有两种：数据和控件之间，控件和代码之间。

### binding inspector

编辑绑定关系。它和 connection inspector 一样都可以操作绑定关系，相比 connection inspector的拖拉操作，它使用的是编辑的方式，更细节，更进一步。

这两个 inspector 主要就是绑定操作的主要界面。

因此，想在 table view 展示我们的数据，我们首先需要建立 students 和 UI 之间的绑定关系。这里需要一个中间控件（像 text field 这样的控件就不需要）：Array Controller。也就是说我们的绑定关系是：students - Array Controller - Table View。

我们先把 Array Controller 拖到对应的 View Controller Scene 上：

![](https://wecache.com/appledev/swift-bindinspector3.png)

此时，用 connection inspector 是无法建立和 students 的绑定关系的，只能用 binding inspector了，在 binding inspector 中找到 controller content，里面的 bind to 就可以绑定到对应的 View Controller 上，然后，model key path 填入 View Controller 的 students 数组即可建立好绑定关系：

![](https://wecache.com/appledev/swift-bindinspector1.png)

绑定好后，在 connection inspector 可以看到刚才设置的绑定关系：

![](https://wecache.com/appledev/swift-bindinspector2.png)

### 总结

可见，一般情况下使用 connection inspector 和 binding inspector 来进行绑定，connection inspector 更多是拖拉进行绑定，而且还可以查看各种绑定关系，binding inspector 则可以用编辑的方式绑定。binding inspector 中控件被绑定到某个 view controller 之后，就可以用 self.xxxx 的方式绑定 data 了。

## 控件之间的绑定

现在我们来绑定 Array Controller 和 Table View 之间的关系。之前是把 Array Controller 绑定给 students，现在是把 Table View 绑定给 Array Controller。

选择 Table View 后，来到 binding inspector，同样也是来到 Content 这一栏（因为我们都是为了显示内容），在 bind to 那里选择 Array Controller，即我们刚才拖出来的控件，此时会发现 controller key 会自动变成 arrangedObjects，目前这些都是自动选择好的，不需要去修改，这就完成了 Array Controller 和 Table View 之间的绑定。

![](https://wecache.com/appledev/swift-bindinspector4.png)

去看 connection inspector，可以看到多了一个 bindings：

![](https://wecache.com/appledev/swift-bindinspector5.png)

### 总结

这里的思路和之前一样，都是通过 binding inspector 进行绑定，通过 bind to 去绑定到某个控件，然后通过下面的 Controller Key 或者 Model Key Path 对值进行指定。

这里 Controller Key 和 Model Key Path 区别是，前者是 Cocoa 定义好了一些操作，例如这里的 arrangedObjects 就表示显示数组，而 后者则是绑定代码中的某些值或者字段。

若不理解也没关系，修改 bind to 的时候，会自动修改 Controller Key 和  Model Key Path，使用默认自动修改的值即可，当然  Model Key Path 需要填更进一步的字段名。

总之，**使用 binding inspector 的 bind to 可以建立或者修改绑定关系，而 connection inspector 则是可以查看所有的绑定关系，也可以通过拖拉进行绑定**



## 绑定 cell 到 struct 字段

上面两步做好后，students - Array Controller - Table View 就绑定起来了，但 table view 的每一个 cell 显示什么具体字段还需要我们进一步操作。这里的操作竟然是绑定自己，同时设置 Model Key Path 为 objectValue.id，objectValue.name 和 objectValue.age：

![](https://wecache.com/appledev/swift-bindinspector6.png)

做完这些绑定后，运行程序，可以看到此时 table view 会显示 student 的内容了：

![](https://wecache.com/appledev/swift-bindinspector7.png)

**可以看到我们没有写太多的代码，尤其是 UI 相关的代码，更多是在绑定UI 控件和代码数据，其核心操作是使用 binding inspector 的 bind to 去进行绑定，因为我们是为了展示内容，所以使用的是里面的 Content 一栏下的 bind to。另外，Connection inspector 的 Binding 栏下详细展示了绑定关系。**



## 动态显示列

前面我们几乎没写什么 UI 相关的代码，也说了 Connection inspector 可以通过拖拉进行绑定，但还未如此操作过。现在我们需要写一些 UI 代码了，会发现，这些 UI 相关的代码，其实也是在做绑定，我们不使用之前的绑定操作，是因为我们需要动态显示，程序还未跑起来之前，是不知道会展示哪些列的。

为了能动态显示列，首先代码中需要有一个字段和我们的 table view 绑定起来，这样我们才可以在代码中去动态设置 table view 的列。

### 通过 Connection inspector 拖拉绑定

以下操作比较经典：

1、打开 storyboard；

2、在 project navigator 中 option 点击 view controller swift 文件；

3、此时，会显示 storyboard 和 view controller swift 文件在文件浏览区域；

4、按住 control 拖动 table view 到 view controller swift 文件，此时弹出对话框需要填入成员变量名，假设变量名为 myTableView。

这样就完成了控件到代码之间的绑定。其中第四步，也可以先事先定义好代码的成员，再拖动的时候选中成员代码。

### 设置 dataSource 和 delegate

我们有了绑定好的 myTableView 对象后，就可以动态的展示列了。但还需要做一件微妙饿事情，就是把 view controller 设置为 table view 的 data source 和 delegate。为什么要做这件事情呢**？因为需要有一个地方去处理 table view 的数据和事件，当 table view 的数据发生变化或者有 UI 事件发生时，需要让系统知道通知谁，我们把 table view 绑定到 view controller 的 myTableView 成员对象上，就需要告诉系统，如果有什么数据或者 UI 事件，那就通知 view controller，也就是回调通知，这样再回调通知的函数中，就可以操控 table view 了**。

设置的方法也很经典，打开 storyboard，选中 table view，打开 connection inspector，分别选中 dataSource 和 delegate 右边的 + 号，拖到 view controller 控件上，这就完成了 UI 绑定。

当然代码也要加上，因为此时回调需要，因此 controller view 对应的类需要实现  dataSource 和 delegate protocol：

```swift
class ViewController: NSViewController, NSTableViewDelegate, NSTableViewDataSource {
    
    @IBOutlet weak var myTableView: NSTableView!
}
```

 这样就完成了 dataSource 和 delegate 的设置。

### 生成新的列和在代码中绑定内容

有了上面的准备，现在可以在代码中动态生成一个列了，假设我们在初始化的时候生成：

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Do any additional setup after loading the view.
        let newColumn = NSTableColumn(identifier: NSUserInterfaceItemIdentifier(rawValue: "addressIdentifier"))
        newColumn.title = "address"
        newColumn.width = 100.0
        
        self.myTableView.addTableColumn(newColumn)
        self.myTableView.reloadData()
    }
```

生成好列后，就需要绑定其内容到数据上，跟之前一样，我们需要把 cell 绑定给自己，其实就是把之前在 xcode 上的操作用代码实现一遍。而且这个时候就需要用到回调了，当需要展示 table view 的内容时，会回调一个叫 tableView 的函数给到 view controller，因为我们之前已经绑定好，设置好，所以 view controller 会收到这个回调，回调中，我们设置绑定：

```swift
    func tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView? {
        guard let column = tableColumn else { return nil }

         // Varying the binding based on what column is being requested
         switch column.identifier {
         case NSUserInterfaceItemIdentifier(rawValue: "addressIdentifier"):
             let cell = makeCell(frame: NSRect(x: 0, y: 0, width: column.width, height: self.myTableView.rowHeight))
             cell.textField?.bind(NSBindingName.value, to: cell, withKeyPath: "objectValue.address")
             return cell
            
         default:
             let existingCell = self.myTableView.makeView(withIdentifier: column.identifier, owner: nil) as? NSTableCellView
             return existingCell
         }
     }
```

可以看到，上面的逻辑是，若是我们动态生成的列，其 id 是 addressIdentifier，那么则生成一个 cell，然后调用 bind 函数，其中的 withKeyPath 参数就是我们之前在 xcode 设置的 objectValue.xxx。对于不是 addressIdentifier 的列，直接 makeView，走在 xcode 设置的流程。

makeCell 代码如下：

```swift
    private func makeCell(frame: CGRect) -> NSTableCellView {
        // Reuse an existing cell if possible
        let cell_id = NSUserInterfaceItemIdentifier.init(rawValue: "addressCell")
        if let existingCell = self.myTableView.makeView(withIdentifier: cell_id, owner: nil) as? NSTableCellView {
            return existingCell
        }

        // Make a new cell
        let textField = NSTextField(frame: frame)
        textField.drawsBackground = false
        textField.isBordered = false
        textField.isEditable = false

        let cell = NSTableCellView(frame: frame)
        cell.identifier = cell_id
        cell.textField = textField
        cell.addSubview(textField)
        return cell
    }
```

这些代码只需要询问 ChatGPT 即可得到，关键还是学会其底层的原理，架构。

## 本文的思维导图

![](https://wecache.com/appledev/swift-binding.svg)
