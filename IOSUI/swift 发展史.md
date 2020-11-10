# swift 发展史  



### 简述

Swift是一种支持多编程范式和编译式的编程语言，是用来撰写macOS/OS X、iOS、watchOS和tvOS的语言之一。 2014年，其在苹果开发者年会（WWDC）发布。设计Swift时，苹果公司有意让Swift与Objective-C共存在苹果公司的操作系统上。

### 历史

- 2010年7月，苹果开发者工具部门总监克里斯·拉特纳开始着手 Swift 编程语言的设计工作，以一年时间，完成基本架构后，他领导了一个设计团队大力参与其中。
- 2014年6月发表， Swift大约历经4年的开发期。苹果宣称Swift的特点是：快速、现代、安全、互动，而且明显优于Objective-C语言。Swift以LLVM编译，可以使用现有的Cocoa和Cocoa Touch框架。Xcode Playgrounds功能是Swift为苹果开发工具带来的最大创新，该功能提供强大的互动效果，能让Swift源代码在撰写过程中能即时显示出其运行结果。拉特纳本人强调，Playgrounds很大程度是受到布雷特·维克多理念的启发。
- 2015年6月8日，苹果于WWDC2015上宣布，Swift将开放源代码，包括编译器和标准库。
- 2015年12月3日，苹果宣布开源swift，并支持Linux，苹果在新网站swift.org和托管网站Github上开源了swift，但苹果的app store并不支持开源的swift，只支持苹果官方的swift版本，官方版本会在新网站swift.org上定期与开源版本同步。

### 类似 Objective-C之处

- 基本数值类型（numeric types）大致相同 (例如Int, UInt, Float, Double)
- 大量的C 运算对象被移出Swift, 但又引入一些新运算对象。
   大括号被用于组群陈述（group statements）。
- 变量之赋值使用等于符号, 但比较则使用“连续两个等于”（==）运算对象。还有一个新的运算对象，“连续三个等于”（===）被用来判断常量或变量之间是否为同一对象之实例（instance）。
- 中括号（[], Square brackets）用于数组的表示, 宣告阵例之后, 可以指派索引值（index）来进行元素（element）之访问。
- 控制陈述（control statement）, for, while, if, switch 与Objective-C都十分类似, 但有延伸功能, 像是 for in 用于集合（collection）的轮询，switch 还可以接受非整数的cases条件值, 诸如此类。

### 不同于 Objective-C之处

- 陈述句（statement）不须再使用分号（;）做为结束，但分号还是可以在一行以内作为两个以上陈述的分隔。
- 头文件（Header files）不再需要。
- 注解方式 /* ... */ 可以为嵌套（nested）注解，意思是指注解内可以再有注解，过去有些C或C++编译器不支持嵌套注解。
- 强类型
- 类型推论或隐含类型（Type inference）
- 支持泛型编程。
- 函数为第一等类型（first-class object），这意味着函数可以作为其他函数的参数与返回值。
- 运算对象可在类别内重新定义 (运算对象重载)，可以生成新的运算对象。
- 字符串全方面支持 Unicode，某些字符甚至可以成为语言的名称。
- **许多C语言家族过去恶名昭彰的怪语法（error-prone behaviors）也被改变**：

1. **不再存在指针。**
2. **指派（Assignments）不再回传值，正确写法是 if (i==0) ，一般容易误写成 if (i=0) 会造成编译时期错误（compile-time error）。**
3. **在switch 的区块内不需要再使用 break 叙述句。另外，case后面都需要有可执行的代码（C或C++可连续使用多个case而不需要额外的代码），否则会发生编译错误。**
4. **变量和常量都要被初始化，而且数组（array）的界限也要确认清楚。**
5. **溢出（overflows）的问题。C语言没有强制整数类型（signed integers）的界限，常常在运行时间发生问题。Swift可以通过整数类型的max或min属性获取最大值或最小值。**



作者：小驴拉磨
链接：https://www.jianshu.com/p/f384a68e5059
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。





**2019年6月3日**

- 针对Swift 5.1进行了更新。
- 添加了有关指定其返回值符合的协议的函数的信息，而不是向“ [不透明类型”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FOpaqueTypes.html)一章提供特定的命名返回类型。
- 添加了[带隐式返回](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FFunctions.html%23ID607)和[速记吸气器声明的](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProperties.html%23ID608)功能部分，其中包含有关省略的功能的信息`return`。
- 添加了有关使用类型上的标信息[类型下标](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FSubscripts.html%23ID609)部分。
- 更新了[结构类型的](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID214)成员初始化程序部分，现在成员初始化程序支持省略具有默认值的属性的参数。
- 添加了有关动态成员的信息，这些成员在运行时通过关键路径查找到[dynamicMemberLookup](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID585)部分。
- 更新了“ [自我类型”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID610)部分，现在`Self`可用于引用当前类，结构或枚举声明引入的类型。

**2019年3月25日**

- 更新了Swift 5.0。
- 添加了“ [扩展字符串分隔符”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID606)部分，并使用有关扩展字符串分隔符的信息更新了“ [字符串文字”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID417)部分。
- 添加了[dynamicCallable](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID603)部分，其中包含有关使用该`dynamicCallable`属性动态调用实例作为函数的信息。
- 添加了[unknown](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID605)和[Switching Over Future Enumeration Cases](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID602)部分，其中包含有关使用`unknown`switch case属性处理switch语句中的未来枚举情况的信息。
- `\.self`向[Key-Path Expression](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID563)部分添加了有关身份密钥路径（）的信息。
- 添加了有关`<`在平台条件中使用小于（）运算符到[条件编译块](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID539)部分的信息。

**2018年9月17日**

- 针对Swift 4.2进行了更新。
- 添加了有关访问所有枚举案例的信息到“ [迭代枚举案例”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FEnumerations.html%23ID581)部分。
- 添加了有关信息`#error`，并`#warning`在[编译时诊断的声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID582)部分。
- 添加了有关内联到和属性下的[声明属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分的信息。`inlinable``usableFromInline`
- 添加了有关在运行时按名称查找[属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)下的[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分的成员的信息`dynamicMemberLookup`。
- 添加了有关[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分的[属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)`requires_stored_property_inits`和`warn_unqualified_access`属性的信息。
- 添加了有关如何根据用于[条件编译块](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID539)部分的Swift编译器版本有条件地编译代码的信息。
- 添加了有关信息`#dsohandle`的[文字表达的](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID390)部分。

**2018年3月29日**

- 更新了Swift 4.1。
- 向[Equivalence Operators](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID45)部分添加了有关等价运算符的综合实现的信息。
- 加入约有条件协议一致性的信息[扩展声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID378)所述的部分[声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html)章，向[有条件符合的协议](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID574)的部分[协议](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html)章。
- [在“关联类型的约束”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html%23ID575)部分中为“ [使用协议”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html%23ID575)添加了有关递归协议约束的信息。
- 添加了有关信息`canImport()`和`targetEnvironment()`平台的条件，[条件编译块](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID539)。

**2017年12月4日**

- 更新了Swift 4.0.3。
- 现在，关键路径支持下标组件，更新了“ [关键路径表达式”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID563)部分。

**2017年9月19日**

- 针对Swift 4.0进行了更新。
- 在[Memory Safety](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FMemorySafety.html)章节中添加了有关内存独占访问的信息。
- 添加了[关联类型和通用子句](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html%23ID557)部分，现在您可以使用通用`where`子句来约束关联类型。
- 添加了有关多字符串文字的信息[字符串文字](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID286)的部分[字符串和字符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html)章节，并以[字符串文字](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID417)的第[词法结构](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html)篇章。
- 更新了的讨论`objc`属性的[声明属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)，现在，这个属性是在更少的地方推断。
- 添加了[通用下标](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html%23ID558)部分，现在下标可以是通用的。
- 更新在讨论[协议组合](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID282)的的部分[协议](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html)章节，并且在[协议组合类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID454)的的部分[类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html)章，现在协议组合物类型可包含一个超类的要求。
- 现在更新了[扩展声明中](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID378)的协议扩展的讨论，这`final`是不允许的。
- 向[断言和前提条件](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID335)部分添加了有关前置条件和致命错误的信息。

**2017年3月27日**

- 更新了Swift 3.1。
- 添加了[带有Generic Where子句的](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html%23ID553)扩展部分，其中包含有关包含要求的扩展的信息。
- 添加了一个范围迭代到[For-In Loops](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FControlFlow.html%23ID121)部分的示例。
- 向[Failable Initializers](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID224)部分添加了可用数字转换的[示例](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID224)。
- 向[声明属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分添加了有关将`available`属性与Swift语言版本一起使用的信息。
- 更新了“ [函数类型”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID449)部分中的讨论，以注意在编写函数类型时不允许使用参数标签。
- 现在，在[条件编译块](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID539)部分更新了对Swift语言版本号的讨论，现在允许使用可选的补丁号。
- 更新了“ [函数类型”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID449)部分中的讨论，现在Swift区分了采用多个参数的函数和采用元组类型的单个参数的函数。
- 从[表达式](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html)章节中删除了动态类型表达式部分，现在这`type(of:)`是一个Swift标准库函数。

**2016年10月27日**

- 更新了Swift 3.0.1。
- 更新了“ [自动引用计数”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAutomaticReferenceCounting.html)一章中对弱引用和无引用引用的讨论。
- 添加了有关信息`unowned`，`unowned(safe)`以及`unowned(unsafe)`在声明修饰符[的声明修饰语](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID381)部分。
- 在[Type Casting for Any和AnyObject](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTypeCasting.html%23ID342)部分添加了关于在`Any`期望type值时使用可选值的[注释](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTypeCasting.html%23ID342)。
- 更新了“ [表达式”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html)一章，以分隔对括号表达式和元组表达式的讨论。

**2016年9月13日**

- 更新了Swift 3.0。
- 更新了[函数](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FFunctions.html)章节和[函数声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID362)部分中函数的讨论，注意默认情况下所有参数都获得参数标签。
- 在[Advanced Operators](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html)章节中更新了对运算符的讨论，现在您将它们实现为类型方法而不是全局函数。
- 添加了有关信息`open`和`fileprivate`访问级别修饰符的[访问控制](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAccessControl.html)一章。
- 更新了`inout`“ [功能声明”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID362)部分中的讨论，注意它出现在参数类型的前面而不是参数名称的前面。
- 更新了对[Escaping Closures](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID546)和[Autoclosures](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID543)部分和[属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html)章节中的[属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html)`@noescape`和`@autoclosure`属性的讨论，因为它们是类型属性，而不是声明属性。
- 添加了有关运算符优先级组信息的[优先级自定义中缀运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID47)中的部分[高级操作员](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html)章，并以[优先级组宣言](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID550)中的部分[声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html)篇章。
- 整个更新的讨论使用macOS而不是OS X，`Error`而不是`ErrorProtocol`和协议名称，`ExpressibleByStringLiteral`而不是`StringLiteralConvertible`。
- 更新在讨论[WHERE子句通用](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html%23ID192)的部分[泛型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html)的章节和[通用参数和参数](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FGenericParametersAndArguments.html)章，现在，通用`where`条款在声明的结尾写的。
- 更新了[Escaping Closures](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID546)部分中的讨论，现在默认情况下闭包是非脱节的。
- 更新的讨论在[可选绑定](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID333)一节的[基础知识](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html)章和[虽然声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID432)中的部分[陈述](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html)章，现在`if`，`while`和`guard`语句中使用的条件下不使用逗号分隔的列表`where`条款。
- 添加了有关具有多个模式的开关情况下的信息[交换](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FControlFlow.html%23ID129)的部分[控制流](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FControlFlow.html)章和[switch语句](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID436)中的部分[陈述](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html)章。
- 由于函数参数标签不再是函数类型的一部分，因此更新了[函数类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID449)部分中函数类型的讨论。
- 更新协议组合物类型的讨论在[协议组合](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID282)的的部分[协议](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html)章节和在[协议组合类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID454)的的部分[类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html)章以使用新的语法。`Protocol1 & Protocol2`
- 更新了“动态类型表达式”部分中的讨论，以使用动态类型表达式的新`type(of:)`语法。
- 更新了对行控制语句的讨论，以使用`#sourceLocation(file:line:)`“ [行控制语句”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID540)部分中的语法。
- 更新了“ [永不返回的函数”中](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID551)的讨论以使用新`Never`类型。
- 向[Literal Expression](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID390)部分添加了有关游乐场文字的信息。
- 更新了[In-Out Parameters](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID545)部分中的讨论，注意只有非转义闭包才能捕获输入输出参数。
- 在“ [默认参数值”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FFunctions.html%23ID169)部分更新了有关默认参数的讨论，现在它们无法在函数调用中重新排序。
- 更新了属性参数以在“ [属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html)一章中使用冒号。
- 添加了有关将重新抛出函数的catch块内的错误抛出到[Rethrowing Functions and Methods](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID531)部分的信息。
- 添加了有关访问Objective-C属性的getter或setter [选择](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID547)器到[Selector Expression](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID547)部分的信息。
- 向[Type Alias Declaration](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID361)部分添加了有关泛型类别别名和在协议内使用类型别名的信息。
- 更新了“ [函数类型”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID449)部分中函数类型的讨论，注意参数类型周围的括号是必需的。
- 更新了[属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html)章节需要注意的是`@IBAction`，`@IBOutlet`和`@NSManaged`属性意味着`@objc`属性。
- `@GKInspectable`在[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分中添加了有关该属性的信息。
- 更新了“可选协议要求[”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID284)部分中对[可选协议要求](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID284)的讨论，以阐明它们仅用于与Objective-C互操作的代码中。
- 删除了`let`在[函数声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID362)部分中明确使用函数参数的讨论。
- 现在该协议已从Swift标准库中删除，从[语句](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html)章节中删除了对`Boolean`协议的讨论。
- 更正了[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分中对`@NSApplicationMain`属性的讨论。

**2016年3月21日**

- 更新了Swift 2.2。
- 添加了有关如何根据用于[条件编译块](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID539)部分的Swift版本有条件地编译代码的信息。
- 添加了有关如何区分名称仅与[Explicit Member Expression](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID400)部分的参数名称不同的方法或初始值设定项的信息。
- `#selector`在“ [选择器表达式”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID547)部分添加了有关Objective-C选择器语法的信息。
- 更新了关联类型的讨论，以`associatedtype`在[关联类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html%23ID189)和[协议关联类型声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID374)部分中使用关键字。
- 更新了有关`nil`在[Failable Initializers](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID224)部分中完全初始化实例之前返回的初始值设定[项的信息](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID224)。
- 添加了有关将元组[与比较运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FBasicOperators.html%23ID70)部分进行比较的[信息](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FBasicOperators.html%23ID70)。
- 添加了有关将关键字用作[关键字和标点符号](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID413)部分的外部参数名称的信息。
- 更新了[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分中对`@objc`属性的讨论，以指出枚举和枚举情况可以使用此属性。
- 通过讨论包含点的自定义运算符更新了“ [运算符”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID418)部分。
- 在[Rethrowing Functions and Methods](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID531)部分添加了一个注释，重新抛出函数不能直接抛出错误。
- 向[Property Observers](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProperties.html%23ID262)部分添加了一个注释，说明在将属性作为输入输出参数传递时调用的属性观察者。
- 在[A Swift Tour](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FGuidedTour%2FGuidedTour.html)章节中添加了有关错误处理的部分。
- 更新了“ [弱参考”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAutomaticReferenceCounting.html%23ID53)部分中的数字，以更清楚地显示重新分配过程。
- 删除了对C风格`for`循环，`++`前缀和后缀运算符以及`--`前缀和后缀运算符的讨论。
- 删除了对变量函数参数的讨论以及curried函数的特殊语法。

**2015年10月20日**

- 更新了Swift 2.1。
- 更新了[字符串插值](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID292)和[字符串字面](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID417)现在该字符串插值可以包含字符串文字部分。
- 添加了[Escaping Closures](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID546)部分，其中包含有关该`@noescape`属性的信息。
- 使用有关tvOS的信息更新了[声明属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)和[条件编译块](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID539)部分。
- 向[In-Out Parameters](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID545)部分添加了有关in-out参数行为的信息。
- 向[Capture Lists](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID544)部分添加了有关如何捕获闭包捕获列表中指定的值的信息。
- [通过可选链接](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FOptionalChaining.html%23ID248)更新了“ [访问属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FOptionalChaining.html%23ID248)部分，以阐明[通过可选链接](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FOptionalChaining.html%23ID248)进行的分配的行为方式。
- 改进了[Autoclosures](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID543)部分中对自动[爆破](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID543)的讨论。
- 添加了一个将`??`操作符用于[A Swift Tour](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FGuidedTour%2FGuidedTour.html)章节的示例。

**2015年9月16日**

- 针对Swift 2.0进行了更新。
- 向[错误处理](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FErrorHandling.html)章节，[Do语句](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID533)部分，[Throw Statement](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID518)部分，[Defer Statement](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID532)部分和[Try Operator](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID516)部分添加了有关错误处理的信息。
- 现在所有类型都符合协议，更新了“ [表示和投掷错误”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FErrorHandling.html%23ID509)部分`ErrorType`。
- `try?`向“ [将错误转换为可选值”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FErrorHandling.html%23ID542)部分添加了有关新关键字的信息。
- 在“ [枚举”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FEnumerations.html)一章的“ [递归枚举”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FEnumerations.html%23ID536)部分和[“声明”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html)一章中的[“ ](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html)[任意类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID365)的[枚举的](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FEnumerations.html)[枚举”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FEnumerations.html%23ID536)部分中添加了有关递归枚举的信息。
- 将有关API可用性检查的信息添加到“ [控制流”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FControlFlow.html)一章的“ [检查API可用性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FControlFlow.html%23ID523)部分和“ [语句”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html)一章的“ [可用性条件”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID522)部分。
- 添加了有关新的信息`guard`语句将[提前退出](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FControlFlow.html%23ID525)的部分[控制流](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FControlFlow.html)章和[卫队声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID524)中的部分[陈述](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html)章。
- 添加了有关协议扩展到信息[协议扩展](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID521)了部分[协议](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html)的篇章。
- 向[访问控制](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAccessControl.html)章节的“ [单元测试目标](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAccessControl.html%23ID519)的[访问级别”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAccessControl.html%23ID519)部分添加了有关单元测试的[访问控制的信息](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAccessControl.html)。
- 将有关新可选模式的信息添加到[Patterns](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FPatterns.html)章节的[Optional Pattern](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FPatterns.html%23ID520)部分。
- 更新了[Repeat-While](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FControlFlow.html%23ID126)部分，其中包含有关`repeat`- `while`循环的信息。
- 更新了[字符串和字符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html)章节，现在`String`不再符合`CollectionType`Swift标准库中的协议。
- `print(_:separator:terminator)`在“ [打印常量和变量”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID314)部分添加了有关新Swift标准库函数的信息。
- 在“ [枚举”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FEnumerations.html)一章`String`的“ [隐式分配的原始值”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FEnumerations.html%23ID535)部分和[“声明”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html)一章的[“ ](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html)[包含原始值类型的案例](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID366)的[枚举”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID366)部分中添加了有关具有原始值的枚举个案行为的信息。
- 添加了有关`@autoclosure`属性（包括其`@autoclosure(escaping)`形式）的信息到[Autoclosures](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID543)部分。
- 使用有关和属性的信息更新了[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分。`@available``@warn_unused_result`
- 使用有关属性的信息更新了“ [类型属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID350)部分`@convention`。
- 添加了一个使用带有`where`子句的多个可选绑定到[Optional Binding](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID333)部分的示例。
- 向[String Literals](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID417)部分添加了有关如何`+`在编译时使用运算符连接字符串文字的信息。
- 向[元类型类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID455)部分添加了有关比较[元类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID455)值并使用它们构造具有初始化表达式的实例的信息。
- 在“ [使用断言调试”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID336)部分添加了关于何时禁用用户定义断言的注释。
- 现在，可以将属性应用于某些实例方法，更新了[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分中对`@NSManaged`属性的讨论。
- 更新了[Variadic Parameters](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FFunctions.html%23ID171)部分，现在可以在函数参数列表的任何位置声明可变参数。
- 向[Overriding a Failable Initializer](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID229)部分添加了有关如何通过强制解包超类初始值设定项的结果，将不可用的初始化程序委托给可用的初始化程序的信息。
- 添加了有关将枚举案例用作[“任何类型的案例](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID365)的[枚举”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID365)部分的函数的信息。
- 添加了有关将初始化程序显式引用到[Initializer Expression](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID399)部分的信息。
- 将有关构建配置和行控制语句的信息添加到“ [编译器控制语句”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID538)部分。
- 在[元类型类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FTypes.html%23ID455)部分添加了一个关于从元类型值构造类实例的注释。
- 在[弱引用](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAutomaticReferenceCounting.html%23ID53)部分添加了一个关于弱引用不适合缓存的注释。
- 更新了“ [类型属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProperties.html%23ID264)部分中的注释，以提及存储的类型属性已延迟初始化。
- 更新了[捕获值](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID103)部分，以阐明如何在闭包中捕获变量和常量。
- 更新了[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分，以描述何时可以将`@objc`属性应用于类。
- 在[处理错误](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FErrorHandling.html%23ID512)部分添加了关于执行`throw`语句性能的注释。`do`在[Do Statement](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID533)部分添加了有关该语句的类似信息。
- 更新了“ [类型属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProperties.html%23ID264)部分，其中包含有关类，结构和枚举的存储和计算类型属性的信息。
- 使用有关标记的break语句的信息更新了[Break Statement](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FStatements.html%23ID441)部分。
- 在更新后的一记[地产观察家](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProperties.html%23ID262)部分澄清的行为`willSet`和`didSet`观察员。
- 在“ [访问级别”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAccessControl.html%23ID5)部分添加了一个注释，其中包含有关`private`访问范围的信息。
- 在[Weak References](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAutomaticReferenceCounting.html%23ID53)部分添加了一个注释，说明垃圾收集系统和ARC之间弱引用的差异。
- 使用更精确的Unicode标量定义更新了“ [字符串文字”中](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID295)的“ [特殊字符”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID295)。

**2015-04-09**

- 针对Swift 1.2进行了更新。
- Swift现在有一个本机`Set`集合类型。有关更多信息，请参阅[集](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID484)。
- `@autoclosure`现在是参数声明的属性，而不是其类型。还有一个新的`@noescape`参数声明属性。有关更多信息，请参阅[声明属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)。
- 类型方法和属性现在使用`static`关键字作为声明修饰符。欲了解更多信息，请参阅[类型变量属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID483)。
- 斯威夫特现在包括`as?`和`as!`failable沮丧的运营商。有关更多信息，请参阅[检查协议一致性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID283)。
- 添加了有关[字符串索引](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID534)的新指南部分。
- 从[溢出运算符中](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID37)删除溢出除法（`&/`）和溢出余数（`&%`）运算[符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID37)。
- 更新了常量和常量属性声明和初始化的规则。有关更多信息，请参阅[常量声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID355)。
- 更新了字符串文字中Unicode标量的定义。请参阅[字符串文字中的特殊字符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID295)。
- 更新[范围运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FBasicOperators.html%23ID73)以注意具有相同开始和结束索引的半开范围将为空。
- 更新的[闭包是参考类型，](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID104)以阐明变量的捕获规则。
- 更新[值溢出](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID38)以阐明有符号和无符号整数的溢出行为
- 更新[协议声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID369)以阐明协议声明范围和成员。
- 更新[定义捕获列表](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAutomaticReferenceCounting.html%23ID58)以阐明闭包捕获列表中弱和无主引用的语法。
- 更新的[运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID418)以明确提及自定义运算符支持的字符的示例，例如数学运算符，杂项符号和标志Unicode块中的字符。
- 现在可以声明常量而不在本地函数范围中初始化。首次使用前，它们必须具有设定值。有关更多信息，请参阅[常量声明](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID355)。
- 在初始化程序中，常量属性现在只能分配一次值。有关更多信息，请参阅[初始化期间分配常量属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID212)。
- 现在，多个可选绑定可以`if`作为逗号分隔的赋值表达式列表出现在单个语句中。有关更多信息，请参阅[可选绑定](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID333)。
- 一个[可选的链式表达](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID405)必须后缀表达式中出现。
- 协议强制转换不再局限于`@objc`协议。
- 现在可以在运行时失败的类型转换使用`as?`or `as!`运算符，并且使用运算符键入保证不会失败的转换`as`。有关更多信息，请参阅[类型转换运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FExpressions.html%23ID388)。

**2014年10月16日**

- 更新了Swift 1.1。
- 添加了[Failable Initializers](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID224)的完整指南。
- 添加了协议的[Failable Initializer要求](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID274)的说明。
- 类型的常量和变量`Any`现在可以包含函数实例。更新了[Type Casting for Any和AnyObject中](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTypeCasting.html%23ID342)的示例，以显示如何检查并转换为`switch`语句中的函数类型。
- 具有原始值的枚举现在具有`rawValue`属性而不是`toRaw()`方法，并且具有`rawValue`参数而不是`fromRaw()`方法的可用初始化程序。有关更多信息，请参阅[具有原始值类型的案例的](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID366)[原始值](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FEnumerations.html%23ID149)和[枚举](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID366)。
- 添加了一个关于[Failable Initializers](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID376)的新参考部分，它可以触发初始化失败。
- 自定义运算符现在可以包含该`?`字符。更新了[运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID418)参考以描述修订的规则。从[Custom Operators中](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID46)删除了有效运算符字符集的重复描述。

**2014年8月18日**

- 描述Swift 1.0的新文档，这是Apple用于构建iOS和OS X应用程序的新编程语言。
- 在协议中添加了有关[初始化程序要求](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID272)的新部分。
- 添加了有关[仅使用类的协议](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FProtocols.html%23ID281)的新部分。
- [断言和前置条件](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID335)现在可以使用字符串插值。删除了相反的说明。
- 更新了“ [连接字符串和字符”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID291)部分，以反映这样的事实：值`String`和`Character`值不能再与加法运算符（`+`）或加法赋值运算符（`+=`）组合。这些运算符现在仅用于`String`值。使用`String`type的`append(_:)`方法将单个`Character`值附加到字符串的末尾。
- `availability`在[“声明属性”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FAttributes.html%23ID348)部分中添加了有关该属性的信息。
- [Optionals](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID330)不再隐含地评估`true`它们何时具有值以及`false`何时不具有值，以避免在使用可选`Bool`值时出现混淆。相反，`nil`使用`==`或`!=`运算符进行显式检查，以确定可选项是否包含值。
- Swift现在有一个[Nil-Coalescing Operator](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FBasicOperators.html%23ID72)（），如果它存在，它会解包一个可选的值，如果是可选的，则返回一个默认值。`a ?? b``nil`
- 更新并扩展了“ [比较字符串”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID298)部分，以反映和演示字符串和字符比较以及前缀/后缀比较现在基于扩展字形集群的Unicode规范等效性。
- 您现在可以尝试设置属性的值，分配给下标，或通过[Optional Chaining](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FOptionalChaining.html)调用变异方法或运算符。有关[通过可选链接访问属性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FOptionalChaining.html%23ID248)的信息已相应更新，并且已扩展了[通过可选链接调用方法](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FOptionalChaining.html%23ID249)检查方法调用成功的示例，以显示如何检查属性设置是否成功。
- 添加了有关通过可选链接[访问可选类型的下](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FOptionalChaining.html%23ID251)标的新部分。
- 更新了“ [访问和修改阵列”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID110)部分，注意您不能再使用`+=`运算符将单个项目附加到数组。而是使用该`append(_:)`方法，或者使用`+=`运算符附加单项数组。
- 添加了一条说明该初始值`a`的[范围运营商](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FBasicOperators.html%23ID73) `a...b`和`a..<b`不得超过终值越大`b`。
- 重写了[继承](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInheritance.html)章节，删除了对初始化程序覆盖的介绍性介绍。本章现在更多地关注在子类中添加新功能，以及使用覆盖修改现有功能。本章的[Overriding Property Getters和Setters](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInheritance.html%23ID200)示例已被重写，以显示如何覆盖`description`属性。（在子类初始化程序中修改继承属性的默认值的示例已移至“ [初始化”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html)一章。）
- 更新了“ [初始化程序继承和覆盖”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID221)部分，注意现在必须使用`override`修饰符标记指定初始值设定项的覆盖。
- 更新了[Required Initializers](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID231)部分，注意`required`现在在所需初始化程序的每个子类实现之前编写修饰符，并且现在可以通过自动继承的初始化程序满足所需初始化程序的要求。
- 中缀[运算符方法](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID42)不再需要该`@infix`属性。
- [前缀和后缀运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID43)的`@prefix`和`@postfix`属性已被和声明修饰符替换。`prefix``postfix`
- 添加了有关在该命令的说明[前缀和后缀运算](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID43)当两个前缀和后缀运算符应用于同一操作应用。
- [复合赋值运算](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID44)符的运算符函数`@assignment`在定义函数时不再使用该属性。
- 定义[自定义运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID46)时指定修饰符的顺序已更改。你现在写，而不是，例如。`prefix operator``operator prefix`
- 添加了有关`dynamic`声明修饰符中的[声明修饰符的信息](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID381)。
- 添加了有关类型推断如何与[Literals一起使用的信息](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID414)。
- 添加了有关curried函数的更多信息。
- 添加了有关[访问控制](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAccessControl.html)的新章节。
- 更新了[字符串和字符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html)章节，以反映Swift的`Character`类型现在代表单个Unicode扩展字形集群的事实。包括有关[Extended Grapheme Clusters](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID296)的新部分以及有关[Unicode标量值](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID294)和[比较字符串的](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID298)更多信息。
- 更新了“ [字符串文字”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html%23ID286)部分，注意字符串文字中的Unicode标量现在写为`\u{n}`，其中`n`是0到10FFFF之间的十六进制数，即Unicode代码空间的范围。
- 该`NSString` `length`属性现在映射到Swift的本机`String`类型`utf16Count`，而不是`utf16count`。
- 斯威夫特的原生`String`类型将不再有一个`uppercaseString`或`lowercaseString`财产。已删除[字符串和字符中](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FStringsAndCharacters.html)的相应部分，并且已更新各种代码示例。
- 添加了有关[没有参数标签的初始化参数](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID210)的新部分。
- 添加了有关[必需初始化程序](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID231)的新部分。
- 添加了有关[可选元组返回类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FFunctions.html%23ID165)的新部分。
- 更新了“ [类型注释”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FTheBasics.html%23ID312)部分，注意可以在一行中使用一种类型注释定义多个相关变量。
- 在`@optional`，`@lazy`，`@final`，和`@required`属性现在是`optional`，`lazy`，`final`，和`required` [的声明修饰语](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FDeclarations.html%23ID381)。
- 更新了整本书，将其`..<`称为[半开放式操作员](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FBasicOperators.html%23ID75)（而不是“半封闭式操作员”）。
- 更新了“ [访问和修改字典”](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID116)部分以注意`Dictionary`现在具有布尔`isEmpty`属性。
- 澄清了定义[自定义运算符](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FAdvancedOperators.html%23ID46)时可以使用的完整字符列表。
- `nil`和布尔人`true`，`false`现在是[文学](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FLexicalStructure.html%23ID414)。
- Swift的`Array`类型现在具有完整的值语义。更新了有关[集合](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID106)和[数组](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID107)[可变性](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID106)的信息，以反映新方法。还澄清了字符串数组和字典的赋值和复制行为。
- [数组类型速记语法](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID108)现在编写为`[SomeType]`而不是`SomeType[]`。
- 添加了一个关于[字典类型速记语法](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID114)的新部分，编写为。`[KeyType: ValueType]`
- 添加了有关[集类型的哈希值](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FCollectionTypes.html%23ID493)的新部分。
- [Closure Expressions的](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FClosures.html%23ID95)示例现在使用全局`sorted(_:_:)`函数而不是全局`sort(_:_:)`函数来反映新的数组值语义。
- 更新了有关[结构类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FInitialization.html%23ID214)的成员初始化程序的信息，以阐明即使结构的存储属性没有默认值，成员结构初始化程序也可用。
- 已更新为`..<`，而不是`..`对[半开区间操作](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FBasicOperators.html%23ID75)。
- 添加了[扩展通用类型](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FLanguageGuide%2FGenerics.html%23ID185)的示例。

</article>

[语法概述](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.swift.org%2Fswift-book%2FReferenceManual%2FzzSummaryOfTheGrammar.html)

BETA软件

本文档包含有关正在开发的API或技术的初步信息。此信息可能会发生变化，根据本文档实施的软件应使用最终操作系统软件进行测试。

[了解有关使用Apple测试版软件的更多信息](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.apple.com%2Fsupport%2Fbeta-software%2F)





作者：超级卡布达
链接：https://www.jianshu.com/p/2daa2d438ff8
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。