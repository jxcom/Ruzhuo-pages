[Home](https://wecache.com)

# 【swift基础】在 UIKit 中使用 swiftUI

[toc]

使用 swiftUI 制作一些自定义控件还是非常方便的，但某些时候又不得不需要 UIKit。

## 先有一个 swiftUI 的view

假设我们有一个 swiftUI view 如下：

```swift
import SwiftUI

struct SettingsView: View {
    @AppStorage("defaultViewMode") var defaultViewMode = ViewMode.allMovies
    @AppStorage("highRatingLimit") var highRatingLimit = 9.0
    
    var body: some View {
        // 1
        VStack(alignment: .leading, spacing: 30) {
          // 2
          Picker("Default View Mode:", selection: $defaultViewMode) {
            // 3
            Text("All Movies").tag(ViewMode.allMovies)
            Text("Favorites Only").tag(ViewMode.favsOnly)
            Text("Highest Rated").tag(ViewMode.highRating)
          }
          // 4
          .pickerStyle(.radioGroup)

          // 5
          Slider(value: $highRatingLimit, in: 7.5 ... 10.0) {
            // 6
            HStack {
              Text("High Rating Limit:")
              // 7
              Text(highRatingLimit, format: .number.precision(.fractionLength(1)))
            }
          }
        }
        // 8
        .frame(width: 300, height: 120)
        .padding()
    }
}

#Preview {
    SettingsView()
}

```

 这是一个完整的 swiftUI。

## 使用NSHostingController

### 引入 NSHostingController

要想显示 swiftUI view，需要一个 Hosting controller 去承载，因此需要在 storyboard 中的 library 中拉一个 hosting controller 控件：



这个控件将会代表 swfitUI 存在于 storyboard 中。

### 创建 hosting controller 的类

此处当然需要一个类和 hosting controller 关联：

```swift
import SwiftUI

// 2
class SettingsHostingController: NSHostingController<SettingsView> {
  // 3
  required init?(coder: NSCoder) {
    // 4
    super.init(coder: coder, rootView: SettingsView())
  }
}

```

注意 `NSHostingController` 的模板类正是我们的 swiftUI 类 `SettingsView`，其 `required init?(coder: NSCoder)` 函数会调用 `SettingsView` 的构造函数初始化。

最后在 property 面板中将控件和类关联起来：

![](https://wecache.com/appledev/swiftuiincocoa-2.png)

这样我们就可以在 UIKit 的代码中通过  `SettingsHostingController` 来控制一个用 swiftUI 编写的 `SettingsView` 了。

### 显示 hosting controller

显示 hosting controller 其实就是现实其关联的 swiftUI 对象。如果我们打开面板，会发现 hosting controller 有一个 action 消息叫 show 或者 modal，因此只要有人给他发 show 或者 modal 消息就可以显示 hosting controller。

![](https://wecache.com/appledev/swiftuiincocoa-3.png)

这里我们把它绑定到一个 menu 上，只需要 control 拉动即可发送 show 或者 modal 消息：

![](https://wecache.com/appledev/swiftuiincocoa-5.png)

![](https://wecache.com/appledev/swiftuiincocoa-4.png)

show 和 modal 的区别就是前者不阻塞父 view 但后者会。

## UIKit 和 swiftUI 之间的消息通信

解决了显示问题，接下来就要解决消息通讯的问题了，有了 swift 对象，应该就可以相互调用，但考虑到 UI 交互的问题，比如调整了某个变量值，这个调整动作即改变了 UI 显示也改变了后台数据，因此如果直接用对象调用的方式传递数据，有可能会阻塞 UI 显示或者有并发冲突的问题。因此， UI 之间的数据通信，应该使用 swift 提供的一些绑定或者通知框架。

常见的绑定方法当然是 `@Binding` 和 `@State` ，当然这个方法仅限于 swiftUI，那么，在 UIKit 中显示了 swiftUI view 的情况下，怎么绑定呢？这里就用到 NotificationCenter。

### NotificationCenter

假设我们有一个 Student 类，若其名字被修改，则通知 StudentObserver，按照设计原则，Student 应该是不知道谁在观察它的，因此，Student 需要解藕 StudentObserver，此时就需要 NotificationCenter：

```swift
import Foundation

@objc class Student: NSObject {
    var age: Int
    private var inner_name: String
    var name: String {
        get {
            self.inner_name
        }
        set(name) {
            print("the name is going to be changed from \(self.inner_name) to \(name)")
            self.inner_name = name
            NotificationCenter.default.post(name: .studentNotification, object: self)
        }
    }
    
    init(age: Int, name: String) {
        self.age = age
        self.inner_name = name
    }
    
    override var description: String {
        return "name = \(self.name), age = \(self.age)"
    }
}

extension Notification.Name {
    static let studentNotification = Notification.Name("studentNotification")
}

class StudentObserver {
    func changeStudent() {
        let s = Student.init(age: 40, name: "jack")
        NotificationCenter.default.addObserver(self,
                                               selector: #selector(self.observeStudent(student:)),
                                               name: .studentNotification,
                                               object: nil)
        s.age = 899
        s.name = "rose"
        let s1 = Student.init(age: 50, name: "claire")
        s1.name = "jane"
    }
    
    @objc func observeStudent(student: Student) {
        print("student name is changed: \(student)")
    }
}

var s = StudentObserver()
s.changeStudent()

```

 上面的代码可以用下面的图来总结：

![](https://wecache.com/appledev/swiftuiincocoa.svg)

主要有以下几点需要注意：

1、访问系统的`NotificationCenter` 对象是通过访问 `default` 成员来实现的。

2、`addObserver` 这个函数需要使用 selector 传入被调函数，即实现 studentObserver.observerStudent(student) 逻辑。 关于 selector 的话题之前谈过。

3、`addObserver` 的 `object` 参数的意思是如果传入 nil，则任何一个对象都应可以 ` post` ，而若传入某个对象，则只有这个传入的对象的 `post` 调用才有作用。例如，如果上面的例子传入的 s 对象，那么 s1 的 `s1.name = "jane"`是不会触发 `observeStudent` 调用的。

4、上面的例子中我们对 `Notification.Name` 做了一个 extension，注意 extension 只可以增加不再用类对象内存的属性，比如 static 和计算属性。

5、若确定不再观察，及时调用  `removeObserver` 。

在 UIKit 中使用 swiftUI 建议是使用 NotificationCenter 来相互交互通信的。

### UserDefaults

UserDefaults 也可以做到 UIKit 和 swiftUI 共享数据。但要注意，UserDefaults 其实是一个轻量数据库，也就是会有持久化动作，这不可避免会有：1、持久化数据的问题；2、因为持久化，而且 UserDefaults 定位是数据库，因此效率肯定没有 NotificationCenter 高。正确的做法应该是使用 Notification 观察 UserDefaults ：

```swift
        NotificationCenter.default.addObserver(
          forName: UserDefaults.didChangeNotification,
          object: nil,
          queue: OperationQueue.main) { _ in
            self.userDefaultsChanged()
        }
```

这里不过多解释，具体用法详看文档。

唯一要说的，是 UserDefaults 既然持久化，那么其持久化文件放在哪？实际上每个苹果 app 都有自己的沙盒环境，其环境路径是 ~/Library/containers/xxxxx/Data，其中 xxxxx 是 app 的名称，在 Data 目录中，是 app 访问的主要目录，其中的 Library/Prereferences 存放了一些 plist 文件。苹果不推荐去直接修改这些文件，应该使用 UserDefaults 的接口来进行操作。

## 使用 NSHostingView 快速嵌入 cocoa

前面说了，使用`NSHostingView` 可以快速的吧 swiftUI 嵌套入 cocoa 框架中，这里再介绍一下使用 `NSHostingView` 快速把一个定制的 swiftUI view 嵌入 table view 中。

首先，下载 SF symbol app，这个工具可以方便浏览苹果提供的标准图标。

例如之前的例子中，我们使用 star 这个图标来标星：

![](https://wecache.com/appledev/swiftuiincocoa-6.png)

首先创建一个 swiftUI view：

```swift
import SwiftUI

struct RatingView: View {
    let rating: Double
    let ladybugImage = Image(systemName: "star")
      .symbolRenderingMode(.multicolor)

    var body: some View {
        HStack(spacing: 2) {
          ladybugImage.symbolVariant(rating >= 8 ? .fill : .none)
          ladybugImage.symbolVariant(rating >= 8.5 ? .fill : .none)
          ladybugImage.symbolVariant(rating >= 9 ? .fill : .none)
          ladybugImage.symbolVariant(rating >= 9.5 ? .fill : .none)
          ladybugImage.symbolVariant(rating >= 10 ? .fill : .none)
        }
        .font(.title3)
        .foregroundColor(.secondary)
    }
}

#Preview {
    VStack {
      RatingView(rating: 7.5)
      RatingView(rating: 8)
      RatingView(rating: 8.5)
      RatingView(rating: 9)
      RatingView(rating: 9.5)
      RatingView(rating: 10)
    }
}
```

这个是一个很简单 view，现在在 table 显示的时候，直接创建 `NSHostingView` ：

```swift
 func tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView? {
   // ...
           case "RatingColumn":
            // 1
            let view = RatingView(rating: movie.rating)
            // 2
            let host = NSHostingView(rootView: view)
            // 3
            return host
   // ...
 }
```

 上面的代码中，直接用了 `NSHostingView` 来封装 swfitUI view，相比之前的  `NSHostingController` ，它没有属性设置面板，可以作为 cocoa 的子 UI 使用。

## 总结

本节主要是 cocoa 嵌入 swiftUI，主要使用 `NSHostingController`(作为独立框) 或 `NSHostingView` （作为子 UI），可以使用 NotificationCenter 来实现 cocoa 和 swfitUI 之间的通信，可以用 UserDefaults 来存储轻变量配置。

实际上，反过来 swiftUI 也可以嵌入 cocoa 框架，是使用 `NSViewRepresentable` ，但考虑到未来更多是在 cocoa 上开发，swiftUI 作为自定义控件，因此可能暂时用不到。
