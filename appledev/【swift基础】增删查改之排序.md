[Home](https://wecache.com)

# 【swift基础】增删查改之排序和selector

[toc]

## 按之前的思路去想

现在我们要对 table view 中的某一列进行排序，比如年龄这一列，那么怎么办呢？比如用户点击年龄这一列，就会按年龄排序，重新展示列表。

按之前的总结的经验，应该是这样的：

1、在 delegate 中找到点击 column 的回调函数；

2、再回调函数中，对 NSArrayController 进行排序；

3、然后调用 table view 对 reload data 函数重新显示列表。

看上去应该是这样，那么，实际去做的时候会遇到什么问题呢？

## 排序的方法

### NSSortDescriptor

进入 UI 设置之前，先简单了解一下 swift 的 sort descriptor。这个如同 C++ 的谓词概念， compare 的时候根据谓词逻辑得出谁先谁后，因此，我们只需要关心“谁先谁后”即可，排序的细节由 swift 操心即可。下面就是一个简单的使用谓词逻辑进行排序的例子：

```swift
class Student: NSObject {
    var id: uint64
    @objc var name: String
    
    init(id: uint64, name: String) {
        self.id = id
        self.name = name
    }
    
    override var description: String {
        return "Student(id = \(self.id), name = \(self.name))\n"
    }
}

var myarray = NSArray.init(array: [
    Student(id: 1, name: "jack"),
    Student(id: 2, name: "rose"),
    Student(id: 3, name: "claire"),
    Student(id: 4, name: "jenny"),
    Student(id: 5, name: "sally"),
    Student(id: 6, name: "nina"),
])

let nameSortDescriptor = NSSortDescriptor.init(key: "name", ascending: true)

print(myarray.sortedArray(using: [nameSortDescriptor]))
```

需要注意的是，`name` 字段是加上了 `@objc` 关键字的，因为如果 swift 代码需要和 objc  框架互动，比如让 cocoa 知道 `name` 这个字段，那么就需要加上标识符 `@objc` 否则会引起运行时崩溃。

有了 `@objc` 标识，`NSSortDescriptor` 才可以初始化，进而使用这个 descriptor 进行排序。

上面的 key 也可以用 keypath 来代替，也似乎更 swifty：

```swift
let nameSortDescriptor = NSSortDescriptor.init(keyPath: \Student.name, ascending: true)
```

那么，如果有一个 VipStudent，且其包含了一个 student 字段，若对 VipStudent 进行排序则依赖 student 字段排序，且 name 字段一样的情况下，按照 id 来排序怎么写呢？可以这样：

```swift
class Student: NSObject {
    @objc var id: uint64
    @objc var name: String
    
    init(id: uint64, name: String) {
        self.id = id
        self.name = name
    }
    
    override var description: String {
        return "Student(id = \(self.id), name = \(self.name))\n"
    }
}

class VipStudent: NSObject {
    @objc let student: Student
    
    init(student: Student) {
        self.student = student
    }
    
    override var description: String {
        return "in vip student, Student(id = \(self.student.id), name = \(self.student.name))\n"
    }
}

var myarray = NSArray.init(array: [
    VipStudent(student: Student(id: 1, name: "jack")),
    VipStudent(student: Student(id: 4, name: "rose")),
    VipStudent(student: Student(id: 3, name: "claire")),
    VipStudent(student: Student(id: 2, name: "rose")),
    VipStudent(student: Student(id: 5, name: "sally")),
    VipStudent(student: Student(id: 6, name: "nina")),
])

let nameSortDescriptor = NSSortDescriptor.init(keyPath: \VipStudent.student.name, ascending: true)
let idSortDescriptor = NSSortDescriptor.init(keyPath: \VipStudent.student.id, ascending: true)

print(myarray.sortedArray(using: [nameSortDescriptor, idSortDescriptor]))
```

上面主要有这几个地方值得关注：

1、使用了 `keyPath` 来指定排序字段；这里对 `keyPath` 的理解又进一步了，看起来像个对字段的引用。

2、排序的时候，传入的 descriptor 是有顺序的，若前一项相同，则用后一项排。

3、swift 的字段给到 cocoa 引用的话，则必须加上 `@objc` 标识。

### compare 方法

NSSortDescriptor 给了一个很简单的解决方案，但有个缺点，就是如果 class 或者 struct 嵌套很深的话， `keyPath` 会变得很长，或者有时候我们并不想关心 Student 的排序方式，这时怎么办呢？那么就可以用 `compare` 方法来实现，一方面不需要写很长的 `keyPath`，另一方面也隐藏了 Student 的排序方式，以后有什么变动外界无需感知。

```swift
@objc class Student: NSObject {
    @objc var id: uint64
    @objc var name: String
    
    init(id: uint64, name: String) {
        self.id = id
        self.name = name
    }
    
    override var description: String {
        return "Student(id = \(self.id), name = \(self.name))\n"
    }
    
    @objc func compare(_ other: Student) -> ComparisonResult {
        if self.name == other.name {
            return self.id < other.id ? .orderedAscending : .orderedDescending
        }
        return self.name.compare(other.name)
    }
}

class VipStudent: NSObject {
    @objc let student: Student
    
    init(student: Student) {
        self.student = student
    }
    
    override var description: String {
        return "in vip student, Student(id = \(self.student.id), name = \(self.student.name))\n"
    }
}

var myarray = NSArray.init(array: [
    VipStudent(student: Student(id: 1, name: "jack")),
    VipStudent(student: Student(id: 4, name: "rose")),
    VipStudent(student: Student(id: 3, name: "claire")),
    VipStudent(student: Student(id: 2, name: "rose")),
    VipStudent(student: Student(id: 5, name: "sally")),
    VipStudent(student: Student(id: 6, name: "nina")),
])

let nameSortDescriptor = NSSortDescriptor.init(keyPath: \VipStudent.student, ascending: true)

print(myarray.sortedArray(using: [nameSortDescriptor]))
```

和之前相比，增加了 `compare` 方法，返回一个枚举 `ComparisonResult` ，表示是升序还是降序，后面的 VipStudent 则直接用 `student` 字段排序即可，这样就可以隐藏排序的内部逻辑和字段，减少耦合， `keyPath` 也不需要写太长的字段了。

这里同样注意 `compare` 函数之前有 `@obj` 标识，因为这是需要给 cocoa 引用的函数，因此需要加上这个标识。



## 用 delegate 触发 column 点击事件

回到 UI 上，那么有没有上面 delegate 回调函数可以监听到点击 column 的事件呢？这个监听回调实际是放在 dataSource delegate 上的，即事件的 delegate 是 NSXXXXDelegate，而关于数据变化的 deleagte 是 NSXXXXDataSource。在 table view 的 dataSource delegate 上就有点击 column 触发数据变化的回调，在这个回调中，我们如之前，设置其 sortDescriptor，然后调用相关的函数获得 sorted 后的数据即可：

```swift
    func tableView(
       _ tableView: NSTableView,
       sortDescriptorsDidChange oldDescriptors: [NSSortDescriptor]
    ) {
        self.myArrayController.sortDescriptors = tableView.sortDescriptors
        self.students = self.myArrayController.arrangedObjects as! [Student]
        
        tableView.reloadData()
    }
```

上面的代码中，鼠标点击 column 就会调用到 `NSTableViewDataSource` 的这个 `tablewView` 回调函数，其中把 `tableView` 的 `sortDescriptors` 给到 `myArrayController` 的 `sortDescriptors`，然后其`arrangedObjects` 数组对象就是排序后（或 filtered）后的对象，赋值给后台的 students 对象即可。最后 `tableView` 需要 reload 一下触发 UI 变化。

那么我们怎么知道点击了哪个 column呢？ 做法是初始化的时候，给这个 column 赋值 sort descriptor，这样，点击的时候，若 column 有设置，则触发上面的回调，一般是在 `viewDidLoad` 中初始化的：

```swift
        if let column = self.myTableView.tableColumn(withIdentifier: NSUserInterfaceItemIdentifier(rawValue: "age")) {
            column.sortDescriptorPrototype = NSSortDescriptor.init(key: "age", ascending: true)
            logger.debug("set the sort descriptor successfully")
        }
```

上面的代码中，可以看到若我们需要引用一个 UI 对象中的对象，需要使用一个 identifier（即 `NSUserInterfaceItemIdentifier`） 对象，而初始化一个 identifier 对象则需要它有一个 String 类型的 rawValue，其是在属性面板中设置的，比如 age，就是在 storyboard 中，对应的 column 中设置：

![](https://wecache.com/appledev/bind-datasource.png)

前面的代码中，获得 age 对应的 column 对象后，则可以设置它的 `sortDescriptorPrototype` 这样，在点击这个 column 的时候就可以相应对应的回调，然后设置给 array controller 对象的 `sortDescriptors`对象了。

##  selector 

讲一个和排序无关的但很重要的特性，selector。之前讲过，swift 中通知的方式有两种： 1、delegate；2、在 storyboard 上建立事件响应关系。那么，这里的 selector 算是第三种通知方式。当然，storyboard 上的事件响应关系绑定，底层实现很可能就是 selector。只不过相较于在 storyboard 中通过拖拉建立关系，直接使用 selecor 是在代码中实现。

### 基本思想和用法

本质上，其实就是建立 sender 和 receiver 之间的关系，伪代码：

```swift
class receiver {
   func doSomething(sender) {
			// ...
   }
}

class sender {
  var selector: Selector?
}

sender.selector = receiver.doSomething
receiver.perfrom(selector, sender)
```

那么，为什么不直接这么做呢？

```swift
receiver.doSomething(sender)
```

关键之处就在于前面伪代码的第 11 行：写 sender 的人不知道 receiver 到底是谁。比如 sender 是一个 cocoa 的 UI 控件，那么很明显， UI 控件的作者不知道谁会使用这个控件，因此只有使用者，也就是 receiver 才能写出第 11 行这段代码。UI 框架才能有机会调用第 12 行这段代码。

### 实现双击事件响应

现在我们实现双击 table view 的响应。cocoa 给 table view UI 控件提供了一个  `doubleAction` 的 `Selector`。即若有谁想监听双击事件，只需要注册到这个 doubleAction即可。

```swift
extension ViewController: NSTableViewDelegate,
                          NSTableViewDataSource,
                          NSTextFieldDelegate { 
		override func viewDidLoad() {
        super.viewDidLoad()
        
        // Do any additional setup after loading the view.
        self.myTableView.doubleAction = #selector(self.doubleClick)
        
        logger.debug("finish view loading")
    }

    @objc func doubleClick(sender: NSTableView) {
        let s = self.students[sender.selectedRow]
        logger.debug("clicked: \(s)")
    }
}
```

上面的代码中，在初始化完成后的函数回调 `viewDidLoad` 中设置了 `doubleAction` ，可以看到设置 selector 的时候使用的是 #selector，其会把 `doubleClick` 引用赋值给 `doubleAction`（实际上，是传了 `ViewController` 对象引用和其成员函数`doubleClick`的地址），cocoa 系统框架会保证，在 table view 被双击后回调 `doubleClick`，即在双击的情况 table view 对象在其合适的地方调用（伪代码）：

```swift
aController.perform(self.doubleAction, with: self)
```

上面的 aController 即一个抽象的 view controller，self 即 table view 对象。注意 doubleClick 是有 `@objc` 标识的，只要与 cocoa 交互，就必须有这个标识。

另外，selector 的用法并不止于 UI 交互，还可以用于 timer，此处不展开讲。

## 总结与脑图

![](https://wecache.com/appledev/swift-binding4.svg)