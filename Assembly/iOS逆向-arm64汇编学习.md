# iOS逆向-arm64汇编学习



参考：[Mach-O Programming Topics](https://link.zhihu.com/?target=https%3A//developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html)



### CPU架构

```dart
指令集对应的机型：
2018 A12芯片arm64e ： iphone XS、 iphone XS Max、 iphoneXR
2017 A11芯片arm64： iPhone 8, iPhone 8 Plus, and iPhone X
2016 A10芯片arm64：iPhone 7 , 7 Plus, iPad (2018)
2015 A9芯片arm64： iPhone 6S , 6S Plus 
2014 A8芯片arm64： iPhone 6 , iPhone 6 Plus
2013 A7芯片arm64： iPhone 5S
armv7s：iPhone5｜iPhone5C｜iPad4(iPad with Retina Display)
armv7：iPhone4｜iPhone4S｜iPad｜iPad2｜iPad3(The New iPad)｜iPad mini｜iPod Touch 3G｜iPod Touch4

模拟器32位处理器测试需要i386架构，
模拟器64位处理器测试需要x86_64架构，
真机32位处理器需要armv7,或者armv7s架构，
真机64位处理器需要arm64架构。
```





捣鼓了一段时间的iOS逆向相关的东西，在动态分析过程中会阅读汇编代码，分析代码的执行流程，在此记录下阅读汇编代码过程中经常遇到的一些指令。

> 当然如果不玩逆向也有必要学习汇编，在定位某些crash问题的时候会很有帮助，比如有时候程序挂在系统库里面，如果能读懂汇编代码，再利用一些调试技巧，可会达到意想不到的效果。
>
> 除此之外，学习汇编有利于帮助自己更深层次的理解程序，理解计算机。

说明：下面所描述的都是在**arm64**架构的汇编指令，在**arm32**或者**x86**架构会有所区别。

------

# 寄存器

> 在**arm64**架构下，所有的寄存器都是64位，并且每个寄存器都有名字的，按照功能来划分，可分为一下几类，分别为：
>
> 1. 通用寄存器
> 2. 程序计数器
> 3. 堆栈指针
> 4. 链接寄存器
> 5. 程序状态寄存器
> 6. 零寄存器

下面分别说明这些寄存器的作用

### 通用寄存器

**64bit** : x0 ~ x28 ，每个寄存器都是64bit
**32bit**：w0 ~ w28，实际上是x0~x28寄存器的低32bit
**x0~x7**：通过用来存储函数的参数，如果函数有更多的参数使用堆栈来传递
**x0**：通常用来存放函数的返回值

![img](https://upload-images.jianshu.io/upload_images/5936457-49acc76d096cb502.png)

### 程序计数器

程序计数器叫**Program Counter**，俗称**PC**，也就是**x32**寄存器，存储着CPU当前正在执行的地址。

### 堆栈指针

**SP (Stack Pointer)**，就是**x31**寄存器，存储的是**栈顶**的地址
**FP (Frame Pointer)**，**FP**也就是**x29**寄存器，存储着**栈底**的地址
随着函数的调用，**SP**、**FP**会不断的变化。

### 链接寄存器

**LR**（Link Register），也就是**x30**寄存器，存储着函数的返回地址。
当函数结束时，就是通过**LR**寄存器的值，跳转到调用函数的位置继续往下执行。

### 程序状态寄存器

**CPSR (Current Program Status Register)**，各个bit的含义如下图：

![img](https://upload-images.jianshu.io/upload_images/5936457-8575138773e1760d.jpeg?)



**SPSR (Saved Program Status Register)**，在异常状态下使用，当发生异常时，会把**CPSR**的内容写入**SPSR**, 等异常恢复之后，又会把**SPSR**写会到**CPSR**中。

### 零寄存器

**WZR**
**XZR**
里面存储的值都是0。

------

# 常用指令

### 算术运算指令

**mov** 赋值指令

```cpp
mov x0, #2  // 把2这个值赋值给x0寄存器
mov x0, x1  // 把x1寄存器中的值赋值给x0寄存器中
```

**add**
两个操作数相加，相加的结果存放到一个寄存器中

```csharp
add, x2, x0, x1 // 把x0的值与x1的值相加，得到的结果存放到x2寄存器中
add, x2, x0, #3 // 把x0的值与3相加，得到的结果存放到x2寄存器中
```

**sub**
第一个操作数减第一个操作数，得到的结果存放到一个寄存器中

```cpp
sub, x2, x1, x0 // x1的值减去x0的值，得到的结果存放到*x2*寄存器中
sub, x2, x1, #4 // x1的值减去4，得到的结果存放到x2寄存器中
```

**mul** 乘法指令

```cpp
mul x3, x1, x2  // x1 乘以 x2 的结果存放在 x3 中
```

**sdiv** 除法指令

```cpp
sdiv    w0, w0, w1 // w0 除以 w1 的结果存放在 w0 中
```

### 逻辑运算指令

> 这里的运算是指位运算

1. **LSL** 逻辑左移
   按操作数所指定的数量向左移位，低位用零来填充
2. **ASL** 算术左移
   通逻辑左移，**ASL** 与 **LSL**等价

```bash
lsl x0, x0, #1
asl x1, x1, x0
```

1. **LSR** 逻辑右移
   按操作数所指定的数量向右移位，左端用*零*来填充。
2. **ASR** 算术右移
   按操作数所指定的数量向右移位，左端用最高位位的值来填充 ，如果是负数，最高位为1

```bash
lsr w1, w2, #1
asr x1, x2, #2
```

1. **ROR** 循环右移
   按操作数所指定的数量向右循环移位， 左端用右端移出的位来填充。其中，操作数可以是通用寄存器，也可以是立即数。 当进行*寄存器bit位数*的循环右移操作时，通用寄存器中的值不改变。

```bash
ror x0, #6
ror w0, #32  // 循环移动了32位，w0的值不变
```

### 跳转指令

**ret**
相当于高级编程语言的**return**，函数返回。

**cmp**
将两个操作数相减，相减的结果会影响**cpsr 寄存器**的标志位，当结果小于0时，**CPSR**寄存器的**N**位为1， 等于0时， **CPSR**寄存器的**Z**为位1。
`cmp x0, x1`

**b**
跳转指令，跳转找指定的**标记处**执行；可以带条件跳转，一般跟**cmp**配合使用，使用到的条件域如下：

1. **EQ**：equal
2. **NE**：not equal
3. **GT**：great than
4. **GE**：great equal
5. **LT**：less than
6. **LE**：less equal

**普通跳转**
`b testCode`，**testCode**是汇编代码中的一个标记

**条件跳转**，当**x0**和**x1**的值相等时，才跳转到testCode标记处执行代码

```undefined
cmp x0, x1
beq testCode
```

**bl**
带返回值的跳转指令，这个指令会做两个操作

1. 将下一条指令的地址存储到**lr （x30）**寄存器中
2. 跳转到标记处开始执行代码
   `bl testCode`，当执行完**testCode**标记处代码后，又会返回来执行**bl**指令下面的指令。

------

# 内存操作

**load从内存中读取数据**

1. **ldr** 地址**没有**偏移或者偏移为**正数**时使用
2. **ldur** 地址偏移为**负数**时使用
   a) `str x5 [x0]` `x0`中是内存的地址，读取的值存放在`x5`寄存器中, x寄存器读取**8**个字节
   b）`str w6 [x0]` `x0`中是内存的地址，读取的值存放在`w5`寄存器中, w寄存器读取**4**个字节

**说明:** 地址还可以偏移 `str x5 [x0, #0x4]` , `stur x5 [x0, #-0x4]` , 偏移量为**正数**往**高地址**偏移，使用`str`指令、偏移量为**负数**往**低地址**偏移，使用`stur`指令。

1. **ldp** 从指定内存中读取数据到一对寄存器中， **p** 是 **pair**的意思，这一对寄存器必须是同类型的，要么**x**类型, 要么**w**类型。其中低位读取到第一个寄存器、高位读取到第二个寄存器
   `ldp w5, w6, [x0]` , 地址可以偏移 `ldp x5, x6, [x0, #-x04]`

**store 往指定的内存写入数据**

1. **str**

```css
str x1, [x0]
str w2, [x1]
str w3, [x1, #4]
```

1. **stur**

```bash
stur x3, [x0, #-4]
stur w2, [x1, #-4]
```

1. **stp**

```css
stp x2, x3, [x0]
stp w4, w5, [x0]
```

使用方法与从内存中读取数据类似，只不过是往内存写入数据。

------

# 总结

本文整理了一些在逆向iOS程序时常见的一些汇编指令，当然在实际逆向的过程所看到的汇编指令更加复杂，比如还有函数调用栈，这是下篇的内容。如有错误请指正。

# Refrence

1. [iOS开发同学的arm64汇编入门 - 刘坤的技术博客](https://blog.cnbluebox.com/blog/2017/07/24/arm64-start/)
2. ARM汇编电子书



12人点赞



[日记本](https://www.jianshu.com/nb/12351574)