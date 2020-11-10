# CTMediator 原理详解（一）



> 最近开始用`CTMetidor`来做`App模块化`，顺便研究一下它的实现原理

`CTMetidor` 源码中经常出现如下关键词：`NSSelectorFromString`、`NSClassFromString` 、`SEL` 这些是个啥？？？

在理解CTMediator原理之前我们先弄懂这么几个概念：

### Method

先来看一下Method相关的定义

```
typedef struct objc_method *Method
struct objc_method{
    SEL method_name      OBJC2_UNAVAILABLE; // 方法名
    char *method_types   OBJC2_UNAVAILABLE; // 函数的返回值和参数
    IMP method_imp       OBJC2_UNAVAILABLE; // 方法的具体实现
}
```



![img](https://user-gold-cdn.xitu.io/2019/1/7/168273f0871fda6e?imageslim)



我们可以看到该结构体中包含一个`SEL`和`IMP`，实际上相当于在`SEL`和`IMP`之间作了一个映射，将`SEL`和`IMP`进行了关联，通过`SEL`我们便可以找到对应的`IMP`，从而调用方法的实现代码。

### SEL（selector）

- 方法编号，对方法名hash化的字符串
- 无论什么类里，只要方法名相同，`SEL`就相同。项目里的所有`SEL`都保存在一个NSSet集合里（NSSet集合里的元素不能重复），所以查找对应方法，只要找到对应的`SEL`就可以了。

既然SEL是方法的唯一标识，那不同的类调用名字相同的方法怎么办呢？

> 每个方法名有对应的唯一seletor,其`SEL`相同，但对应的`IMP`函数指针不同。

如何获取SEL？

```
SEL s1  = @selector（test）；
SEL s2 = NSSelectorFromString（@“test”）
复制代码
```

以上两个方法是等价的

### IMP （implement）

- 一个函数指针,保存了方法的地址，内部实现：

```
typedef id (*IMP)(id, SEL, ...); 
复制代码
```

- 包含`id`（消息接受者，也就是对象），`SEL`（方法的名字），`参数`

XX调用XXX方法，参数XX也都确定了

执行对应的方法:

```
[object test];
// @selector(test) 是一个C的字符串
[object performSelector:@selector(test)]];
// 转换成如下实现方式
objc_msgSend(object,@selector(test))
复制代码
```

### 总结

- NSClassFromString 通过字符串的名称来获取一个类，可以根据Target来进行获取
- NSSelectorFromString 通过字符串(已存在的方法名称)获取一个SEL





# CTMediator 原理详解（二）

> 在上一篇文章中我们大概知道了 `CTMetidor` 中的 `NSSelectorFromString`、`NSClassFromString` 、`SEL` 这篇文章主要介绍一下 `respondsToSelector`、 `performSelector` 、`NSInvocation`、`NSMethodSignature` ，了解完这些之后对理解 `CTMediator` 很有帮助

在介绍 `performSelector` 之前，先简单说一下RunTime吧

### RunTime

#### 什么是 RunTime ?

- `RunTime`简称运行时。OC就是运行时机制，也就是在运行时候的一些机制，其中最主要的是消息机制， 消息(方法)传递，如果消息(方法)在对象中找不到，就进行转发

#### RunTime 可以用来做什么 ?

- 通过`RunTime`我们可以 让一个对象发送消息（也就是执行方法）、交换方法（Method Swizzling）、动态添加方法、给分类增加属性、字典转模型等

```
// 创建person对象
Person *p = [[Person alloc] init];
// 调用对象方法
[p eat];
// 本质：让对象发送消息
objc_msgSend(p, @selector(eat));
    
// 调用类方法的方式：两种
// 第一种通过类名调用
[Person eat];
    
// 第二种通过类对象调用
[[Person class] eat];
// 用类名调用类方法，底层会自动把类名转换成类对象调用
// 本质：让类对象发送消息
objc_msgSend([Person class], @selector(eat));
复制代码
```

### respondsToSelector

(BOOL)respondsToSelector:(SEL)aSelector;

> 判断对象是否响应此方法，一般和performSelector 一起使用，防止crash

### performSelector

> `CTMetidor` 主要用到就是`RunTime`中的让对象发送消息

performSelector 本质上就是会转化成 objc_msgSend 来进行实现，其内部实现步骤：



![RunTime](https://user-gold-cdn.xitu.io/2019/1/8/1682b212ce19470d?imageslim)



##### 1、通过obj的isa指针找到它的 class ;

##### 2、在 class 的 method list 找 eat ;

##### 3、如果 class 中没到 eat，继续往它的 superclass 中找 ;

##### 4、一旦找到 eat 这个函数，就去执行它的实现IMP 。

来看一下 `CTMetidor` 的一段代码：

```
[target performSelector:action withObject:params];
复制代码
```

`action`（`SEL`） 我们通过 `NSSelectorFromString` 获取了，`target` 我们通过 `NSClassFromString` 获取，接下来只需要通过 `performSelector`方法 执行 `target`（`Class`） 中的 `action` 即可。