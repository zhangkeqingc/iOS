# Swift 命名规范



### 一、命名规范

1.  常量，变量，函数，方法的命名规则使用小驼峰规则，首字母小写，类型名使用大驼峰规则，首字母大写。
2.  当命名里出现缩写词时，缩写词要么全部大写，要么全部小写，以首字母大小写为准。
3.  bool类型命名时，使用is作为前缀。
4.  一般情况下，在逗号后面加一个空格。
5.  常量定义，建议尽可能定义在Type类型里面，避免污染全局命名空间
6.  Swift中类别(类，结构体)在编译时会把模块设置为默认的命名空间，所以不用为了区分类别而添加前缀，比如XYHomeViewController，但是为了和引用的第三方库作区分，建议可以继续使用前缀，以作为规范化处理，结构更清晰。
7.  懒加载用来细致地控制对象的生命周期，这对于想实现延迟加载视图的UIViewController特别有用。
8.  当函数第一个参数构成整个语句的介词时（如:at, by, for, in, to, with 等），为第一个参数添加介词参数标签。例外，当后面所有参数构成独立短语时，则介词提前。

7. 变量命名需要以变量类型结尾

11. 省略所有的冗余的参数标签

12. 进行安全值类型转换的构造方法可以省略参数标签，非安全类型转换则需要添加参数标签以表示类型转换方法

```
extension UInt32 {   

    /// 安全值类型转换，16位转32位，可省略参数标签   
    init(_ value: Int16)     
    
    /// 非安全类型转换，64位转32位，不可省略参数标签   
    /// 截断显示   
    init(truncating source: UInt64)     
    
    /// 非安全类型转换，64位转32位，不可省略参数标签   
    /// 显示最接近的近似值   
    init(saturating valueToApproximate: UInt64) 
}
```





### 二、语法规范

1. 可选类型拆包取值时，使用 if let 判断

2. 多个可选类型拆包取值时，将多个if let 判断合并

3. 尽量不要使用 as! 或 try!，对于可选类型Optional多使用as?，?? 可以给变量设置默认值

4. 类型推断：推荐用紧凑方式写代码，让编辑器推断实例中常量和变量的类型。类型推断也适用于小型非空数组/字典。如有需要，请指定特定类型，例如：CGFloat或者Int16等。

   ```jsx
   // 推荐写法
   let message = "Click the button"
   let currentBounds = computeViewBounds()
   var names = ["Mic", "Sam", "Christine"]
   let maximumWidth: CGFloat = 106.5
   
   // 不推荐写法
   let message: String = "Click the button"
   let currentBounds: CGRect = computeViewBounds()
   var names: [String] = ["Mic", "Sam", "Christine"]
   ```

   针对空数组和空字典的类型推断写法，推荐使用类型注释。此准则也意味着选择描述性名称比以前更加重要。

   ```dart
   // 推荐写法
   var names: [String] = []
   var lookup: [String: Int] = [:]
     
   // 不推荐写法
   var names = [String]()
   var lookup = [String: Int]()
   ```

   

5. 常量定义，建议尽可能定义在类型里面，避免污染全局命名空间，如果是其他地方有可能复用的cell可以定义在类型外面。

   ```csharp
   static let homeListCell = "HomeListCell"
   
   class HomeListCell: UITableViewCell {
       static let kHomeCellHeight = 80.0
       //
   }
   ```

6. 当方法最后一个参数是Closure类型，调用时建议使用尾随闭包语法

   ```objectivec
   UIView.animateWithDuration(1.0) {
        self.myView.alpha=0
   }
   ```

7. 最短路径规则：当编码遇到条件判断时，左边的距离是黄金路径或幸福路径，因为路径越短，速度越快。guard 就为此而生的。

   ```swift
   func login(with username: String?, password: String?) throws -> LoginError {
     guard let username = username else { 
       throw .noUsername  
     }
     guard let password = password else { 
       throw .noPassword
     }
     // 处理登录
   }
   ```

8. 循环遍历使用for-in表达式

   ```swift
   // 循环
   for _ in 0..<list.count {
     print("items")
   }
   // 遍历
   for(index, person) in personList.enumerate() {
       print("\(person)is at position #\(index)")
   }
   // 间隔2位循环
   for index in 0.stride(from: 0, to: items.count, by: 2) {
     print(index)
   }
   // 翻转
   for index in (0...3).reversed() {
       print(index)
   }
   ```


9. 使用二元运算符(+, -，==, 或->)的前后都需要添加空格

   ```csharp
   let value = 1 + 2
   ```

10. 在逗号后面加一个空格

    ```bash
    let titleArray = [1, 2, 3, 4, 5]
    ```

11. 方法的左大括号不要另起，并和方法名之间留有空格，注释空格

12. 判断语句不用加括号

13. 尽量不使用self. 除非方法参数与属性同名

14. 在访问枚举类型时，使用更简洁的点语法

15. 

### 三、协议

##### （1）协议描述的是 “做的事情”，命名为名词

```
protocol TableViewSectionProvider {
    func rowHeight(at row: Int) -> CGFloat
    var numberOfRows: Int { get }
    /* ... */
}
```

##### （2）协议描述的是 “能力”，需添加后缀`able`或 ing 。 (如 Equatable、 ProgressReporting)

```
protocol Loggable {  
    func logCurrentState()
    /* ... */
}  

protocol Equatable {
    func ==(lhs: Self, rhs: Self) -> bool {
    /* ... */
    }
}
 
```

##### （3）如果已经定义类，需要给类定义相关协议，则添加`Protocol`后缀

```
protocol InputTextViewProtocol {
     func sendTrackingEvent()
     func inputText() -> String 
     /* ... */
}
```

**（4）如果确定protocol的实现不会被重写，建议用extension将protocol实现分离　**

```
推荐：
class MyViewController: UIViewController {
     // class stuff here
}

// MARK: - UITableViewDataSource
extension MyViewController: UITableViewDataSource {
    // table view data source methods
}

// MARK: - UIScrollViewDelegate
extension MyViewController: UIScrollViewDelegate {
    // scroll view delegate methods
}


不推荐：
class MyViewController: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
    // all methods
}
```



### 四、注释 

**（1）添加有必要的注释，尽可能使用Xcode注释快捷键（⌘⌥/）**

```swift
/// <#Description#>
///
/// - Parameters:
///   - tableView: <#tableView description#>
///   - section: <#section description#>
/// - Returns: <#return value description#>
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return dataList.count
}
```



**（2）使用 // MARK: -，按功能、协议、代理等分组**

```cpp
// MARK: - UITableViewDelegate

// MARK: - Action

// MARK: - Request
```



**（3）协议一致性：当对象要实现协议一致性时，推荐使用 extension 隔离协议中的方法集，这样让相关方法和协议集中在一起，方便归类和查找**

```swift
// MARK: - UICollectionViewDelegate, UICollectionViewDataSource
extension XMHomeViewController: UICollectionViewDelegate, UICollectionViewDataSource {
    // 代理方法
}

// MARK: - HttpsRequest
extension XMHomeViewController {
    // 网络请求方法
}
```



**（4）当对外接口不兼容时，使用@available(iOS x.0, *) 标明接口适配的起始系统版本号**

```swift
@available(iOS 8.0, *)
func myFunction() {
    //
}
```



### 五、关于闭包

##### （1）在Closures中使用self时避免循环引用

```
推荐
resource.request().onComplete { [weak self] response in
    guard let strongSelf = self else {
        return
    }
    let model = strongSelf.updateModel(response)
    strongSelf.updateUI(model)
}


不推荐
// 不推荐使用unowned
// might crash if self is released before response returns
resource.request().onComplete { [unowned self] response in
    let model = self.updateModel(response)
    self.updateUI(model)
}

不推荐
// deallocate could happen between updating the model and updating UI
resource.request().onComplete { [weak self] response in
    let model = self?.updateModel(response)
    self?.updateUI(model)
}
```



**（2）Golden Path，最短路径原则**　　

```
推荐
func login(with username: String?, password: String?) throws -> LoginError {
  guard let username = contextusername else {
    throw .noUsername
  }
  guard let password = password else {
    throw .noPassword
  }
 
  /* login code */
}


不推荐
func login(with username: String?, password: String?) throws -> LoginError {
  if let username = username {
    if let password = inputDatapassword {
        /* login code */
    } else {
        throw .noPassword
    }
  } else {
      throw .noUsername
  }
}
```

　　 

### 六、单例

尽量少使用单例

```
class TestManager {
    static let shared = TestManager()
 
    /* ... */
}
```

　　

 



# 其他语法规范 Objective-C兼容

1 多使用let，少使用var

2 少用`!`去强制解包

3 可选类型拆包取值时，使用`if let`判断

4 不要使用 as! 或 try!

5 数组访问尽可能使用 .first 或 .last, 推荐使用 for item in items 而不是 for i in 0..

6 如果变量能够推断出类型，则不建议声明变量时指明类型

7 如果变量能够推断出类型，则不建议声明变量时指明类型

8 switch case选项不需要使用`break`关键词

9 访问控制

10 对于私有访问，如果在文件内不能被修改，则标记为`private`；如果在文件内可修改，则标记为`fileprivate`

11 对于公有访问，如果不希望在外面继承或者override，则标记为`public`，否则标明为`open`

12 访问控制权限关键字应该写在最前面，除了`@IBOutlet`、`IBAction`、`@discardableResult`、`static` 等关键字在最前面

13 如调用者可以不使用方法的返回值，则需要使用`@discardableResult`标明

14 使用`==`和`!=`判断`内容`上是否一致

15 使用`===`和`!==`判断class类型对象是否同一个引用，而不是用 `==`和`!=`

16 Runtime兼容

17 Swift语言本身对Runtime并不支持，需要在属性或者方法前添加dynamic修饰符才能获取动态型，继承自NSObject的类其继承的父类的方法也具有动态型，子类的属性和方法也需要加dynamic才能获取动态性





#  Objective-C兼容

1 Swift接口不对Objective-C兼容，在编译器或者Coding中就会出现错误

2 暴漏给Objective-C的任何接口，需要添加@objc关键字，如果定义的类继承自NSObject则不需要添加

3 如果方法参数或者返回值为空，则需要标明为可选类型





[参考文献]：

[Swift Programming Language](https://link.jianshu.com/?t=https://developer.apple.com/library/prerelease/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID309)

[Swift API Design Guidelines](https://link.jianshu.com/?t=https://swift.org/documentation/api-design-guidelines/)

[Raywenderlich Swift Style Guide](https://link.jianshu.com/?t=https://github.com/raywenderlich/swift-style-guide#spacing)

[Linkedin Swift Style Guide](https://link.jianshu.com/?t=https://github.com/linkedin/swift-style-guide#1-code-formatting)