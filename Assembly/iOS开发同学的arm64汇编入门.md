# iOS开发同学的arm64汇编入门

在定位某些crash问题的时候，有时候遇到一些问题很诡异。有时候挂在了系统库里面。这个时候定位crash问题往往是比较头疼的。那么这个时候学会一些汇编知识，利用汇编调试技巧进行调试可能会起到意想不到的效果。

学习汇编语言不只是帮助定位crash而已，学习汇编可以帮助你真正的理解计算机。毕竟CPU上跑的就是对应的指令集。

## 0x1 工具

我们面对的要么是源代码，要么是二进制。因此我们需要一些反汇编的工具来辅助我们进行汇编代码查看。推荐工具有： – [Hopper Disassembler](https://www.hopperapp.com/) 收费应用，看汇编代码非常方便 – [MachOView](https://github.com/gdbinit/MachOView) 开源工具，看Mach-o文件结构非常方便。

## 0x2 基本概念

从高级语言过渡到汇编语言，重要的是基本概念的转换。汇编里面要学习的三个重要概念，我认为是 寄存器、栈、指令。 arm64架构又分为2种执行状态： `AArch64 Application Level` 和 `AArch32 Application Level`, 本文只讲AArch64.

### 0x21 寄存器

如果你还不知道什么是寄存器，建议先Google一下。 这里不再详细说明，寄存器是CPU中的高速存储单元，要比内存中存取要快的多。

这里说明一下arm64有哪些寄存器：

- **R0 – R30**

`r0 - r30` 是31个通用整形寄存器。每个寄存器可以存取一个64位大小的数。 当使用 `x0 - x30`访问时，它就是一个64位的数。当使用 `w0 - w30`访问时，访问的是这些寄存器的低32位，如图：

![1.png](https://blog.cnbluebox.com/images/arm64-start/1.png)

其实通用寄存器有32个，第32个寄存器x31，在指令编码中，使用来做 `zero register`, 即`ZR`, `XZR/WZR`分别代表64/32位，`zero register`的作用就是0，写进去代表丢弃结果，拿出来是0.

其中 `r29` 又被叫做 `fp` (frame pointer). `r30` 又被叫做 `lr` (link register)。其用途会在下一节《栈》中讲到。

- **SP**

SP寄存器其实就是 x31，在指令编码中，使用 `SP/WSP`来进行对SP寄存器的访问。

- **PC**

PC寄存器中存的是当前执行的指令的地址。在arm64中，软件是不能改写PC寄存器的。

- **V0 – V31**

`V0 - V31` 是向量寄存器，也可以说是浮点型寄存器。它的特点是每个寄存器的大小是 128 位的。 分别可以用`Bn Hn Sn Dn Qn`的方式来访问不同的位数。如图：

![2.png](https://blog.cnbluebox.com/images/arm64-start/2.png)

`Bn Hn Sn Dn Qn`可以这样理解记忆, 基于一个word是32位，也就是4Byte大小：

> Bn: 一个Byte的大小
> Hn: half word. 就是16位
> Sn: single word. 32位
> Dn: double word. 64位
> Qn: quad word. 128位

- **SPRs**

SPRs是状态寄存器，用于存放程序运行中一些状态标识。不同于编程语言里面的if else.在汇编中就需要根据状态寄存器中的一些状态来控制分支的执行。状态寄存器又分为 `The Current Program Status Register (CPSR)` 和 `The Saved Program Status Registers (SPSRs)`。 一般都是使用`CPSR`， 当发生异常时， `CPSR`会存入`SPSR`。当异常恢复，再拷贝回`CPSR`。

还有一些系统寄存器，还有 `FPSR` `FPCR`是浮点型运算时的状态寄存器等。基本了解上面这些寄存器就可以了。

### 0x22 栈

栈就是指令执行时存放临时变量的内存空间。在学习汇编代码的执行过程中，了解栈的结构非常重要。

先列出一些栈的特性：

- 栈是从高地址到低地址的， 栈低是高地址，栈顶是低地址。
- `fp`指向当前frame的栈底，也就是高地址。
- `sp`指向栈顶，也就是地地址。

下面的图简单的描述了从方法A调用方法B时 栈是如何划分的：

![3.jpeg](https://blog.cnbluebox.com/images/arm64-start/3.jpeg)

其中3行汇编代码就是方法B的前三行汇编指令。它们做的事情就是图中描述的事情 (x29就是fp, x30就是lr)：

- 将`fp, lr`保存到 `sp - 0x10`的地方. 也就是图中 `--> fp_B`的位置。然后将sp设置为 `sp-0x10`
- 将 `fp` 设置为当前 `sp`。也就是 `--> fp_B`的位置。 这一步就设置了`_funcB`的 fp了
- 将 `sp` 设置为 `sp - 0x30`。 也就是将`sp`指向了图中 `--> sp_B` 的位置

> 注： `lr` 是`link register`中的值，它存的是方法`_funcA`的执行的最后一行指令的下一行。它的作用也很好理解：当`_funcB`执行完了之后要返回`_funcA`继续执行，但是计算机要如何知道返回到哪执行呢？ 就是靠`lr`记录了返回的地址，方法才能得以正常返回。

说道这里，那么当 `_funcB`执行完毕后，是如何把栈恢复到`_funcA`的过程的呢？ 我们直接分析 `_funcB`的最后3条指令：



```
mov        sp, fp;              //  sp 设置为fp, 就是图中 -->fp_B 的位置 ldp           fp, lr, [sp], #0x10; //  从sp指向的地址中读取 2个64位，分别存入fp,lr。 然后将sp += 0x10 // 这一步执行完之后，fp就执行了图中 -->fp_A. lr恢复成 _funcA的返回地址。 sp指向了 -->sp_A.  // 这个时候状态已经完全恢复到了 _funcA 的环境 ret;    // 返回指令，这一步直接执行lr的指令。 
```

> 上面描述了方法如何调用的。我们知道在编程语言里面方法都有入参，有返回值的。在汇编里面如何体现呢？

- 一般来说 arm64上 x0 – x7 分别会存放方法的前 8 个参数
- 如果参数个数超过了8个，多余的参数会存在栈上，新方法会通过栈来读取。
- 方法的返回值一般都在 x0 上。
- 如果方法返回值是一个较大的数据结构时，结果会存在 x8 执行的地址上。

### 0x23 指令

在上一级的内容中我们已经看到了一些指令。 汇编指令除了数量较多，其基本原理都是比较简单的，单拎出来一条指令就是很simple的操作。 比如`mov`就是一个赋值。`ldr`就是一个取值。

那汇编指令大概可以分为哪几种呢？我认为了解以下几种基本指令就可以正常阅读汇编代码了。

#### 0x231 **运算**

- 算术运算

算术运算就是像 `ADD` `SUB` `MUL` … 等加减乘除运算，也是很好理解的指令
如：



```
add x0, x1, x2; // 把 x1 + x2 = x0 这样一个操作。 sub sp, sp, 0x30; // 把 sp - 30 存入sp. cmp x11, #4;  // 相当于 subs xzr, x11, #4.                // 如果 x11 - 4 == 0, 那么状态寄存器NZCV.Z = 1              // 如果 x11 - 4 < 0, 那么 NZCV.N = 1 
```

> `NZCV`是状态寄存器中存的几个状态值，分别代表运算过程中产生的状态，其中：
> \* N, negative condition flag，一般代表运算结果是负数
> \* Z, zero condition flag, 运算结果为0
> \* C, carry condition flag, 无符号运算有溢出时，C=1。
> \* V, oVerflow condition flag 有符号运算有溢出时，V=1。

- 逻辑运算指令

有 `LSL`(逻辑左移) `LSR`(逻辑右移) `ASR`(算术右移) `ROR`(循环右移)。
有 `AND`(与) `ORR`(或) `EOR`(异或)

逻辑位移运算通常也可以与算术运算一起用，如：



```
 add  x14, x4, x27, lsl #1; // 意思是把  (x27 << 1) + x4 = x14; 
```

- 拓展位数运算

有 `zero extend`(高位补0) 和 `sign extend`(高位填充和符号位一致，一般有符号数用这个)。 一般用来补齐位数。常和算术运算配合一起.
如：



```
add        w20, w30, w20, uxth  // 取 w20的低16位，无符号补齐到32位后再进行  w30 + w20的运算。 
```

- Mov

#### 0x232 寻址

既然是和内存相关的，那就是两种，一种存，一种取。一般来说
**L打头的基本都是取值指令，如 LDR LDP;**
**S打头的基本都是存值指令，如 STR STP;**

例：



```
ldr x0, [x1]; // 从`x1`指向的地址里面取出一个 64 位大小的数存入 `x0` ldp x1, x2, [x10, #0x10]; // 从 x10 + 0x10 指向的地址里面取出 2个 64位的数，分别存入x1, x2 str x5, [sp, #24]; // 把x5的值（64位数值）存到 sp+24 指向的内存地址上 stp x29, x30, [sp, #-16]!; // 把 x29, x30的值存到 sp-16的地址上，并且把 sp-=16.  ldp x29, x30, [sp], #16;  // 从sp地址取出 16 byte数据，分别存入x29, x30. 然后 sp+=16; 
```

其中寻址的格式由分为下面这3种类型：



```
[x10, #0x10]      // signed offset。 意思是从 x10 + 0x10的地址取值 [sp, #-16]!       // pre-index。  意思是从 sp-16地址取值，取值完后在把 sp-16  writeback 回 sp [sp], #16         // post-index。 意思是从 sp 地址取值，取值完后在把 sp+16 writeback 回 sp 
```

#### 0x233 跳转

跳转氛围有返回跳转`BL`和无返回跳转`B`。 有返回的意思就是会存`lr`,因此 `BL`的`L`也可以理解为`LR`的意思。

> 1.存了`LR`也就意味着可以返回到本方法继续执行。一般用于不同方法直接的调用
> 2.`B`相关的跳转没有`LR`，一般是本方法内的跳转，如`while`循环，`if else`等。

跳转相关的指令还会有种逻辑运算，就是`condition code`。配合状态寄存器中的状态标示，就是代码分支`if else`实现的关键。
`condition code`有以下这些，表格中还标注除了分别是比NZCV的哪个值： ![4.png](https://blog.cnbluebox.com/images/arm64-start/4.png)

如：



```
cmp x2, #0;         // x2 - 0 = 0。  状态寄存器标识zero: PSTATE.NZCV.Z = 1 b.ne  0x1000d48f0;  // ne就是个condition code, 这句的意思是，当判断状态寄存器 NZCV.Z != 1才跳转，因此这句不会跳转 0x1000d4ab0 bl testFuncA;  // 跳转方法，这个时候 lr 设置为 0x1000d4ab4 0x1000d4ab4 orr x8, xzr, #0x1f00000000 // testFuncA执行完之后跳回lr就周到了这一行 
```

## 0x4 小结

本文简单介绍了一些arm64的汇编知识，arm64汇编的学习对于理解iOS代码的执行，计算机的运行都有着不少的好处。我们在日常中利用汇编知识可以定位一些疑难杂症的crash问题。可以从汇编原理出手开一个个脑洞，玩一些黑科技。比如包瘦身，静态扫描等。

汇编指令的执行是简单确定的，不会像我们调试其他代码一眼，有些诡异问题，而汇编每条指令的结果都是确定的，从这一角度来定位问题往往可以定位到根本原因。

在汇编指令执行的世界，你可以对代码执行有更深刻的理解，原来一行代码会被分解成这么多的指令！因此，如果你在看完本文后对于学习汇编有了兴趣，但是有很多细节还不太懂，建议你自己用`hopper`反编译一些代码，自己尝试一行一行理解每一个指令的意义，基本看透几个方法就可以融汇贯通了。

## 0x5 参考

- [ARMv8-A Architecture – ARM](https://developer.arm.com/products/architecture)

Share

# Comments







# ARM64 汇编——寄存器和指令

- iOS 中的 armv7,armv7s,arm64 这些都代表什么？

> ARMv7｜ARM7s｜ARM64都是ARM处理器的指令集
> 真机32位处理器需要ARMv7,或者ARMv7s架构，
> 真机64位处理器需要ARM64架构。

- 处理器和寄存器的位数

> 32位处理器,能同时处理32位的数据，所以对应寄存器为32位的；
> 64位处理器,能同时处理64位的数据，所以对应寄存器为64位的；
> 寄存器位数一般会对应处理器位数，两者一般相等，但也有例外8086

- ARM 指令长度

> ARM处理器用到的指令集分为 ARM 和 THUMB 两种。ARM指令长度固定为32bit，THUMB指令长度固定为16bit。所以 ARM64指令集的指令长度为32bit

- ARM中字的长度

> 字（Word）：32位系统字长就是32,64位系统字长就是64。`stdint.h`文件中定义了宏`__WORDSIZE`表示字的位数:



```cpp
#if __LP64__
#define __WORDSIZE 64
#else
#define __WORDSIZE 32
#endif
//  注：那么 LP64 是什么意思呢？
在64位机器上，如果int是32位，long是64位，pointer也是64位，那么该机器就是LP64的，其中的L表示Long，P表示Pointer，64表示Long和Pointer都是64位的。
```

> 半字（Half-Word）：半字永远是字的一半，双字永远是字的2倍大小
> 字节（Byte）：8位

### 寄存器

ARM64 有34个寄存器，包括31个通用寄存器、SP、PC、CPSR。

| 寄存器  | 位数  | 描述                                                         |
| ------- | :---: | :----------------------------------------------------------- |
| x0-x30  | 64bit | 通用寄存器，如果有需要可以当做32bit使用：WO-W30              |
| FP(x29) | 64bit | 保存栈帧地址(栈底指针)                                       |
| LR(x30) | 64bit | 通常称X30为程序链接寄存器，保存子程序结束后需要执行的下一条指令 |
| SP      | 64bit | 保存栈指针,使用 SP/WSP来进行对SP寄存器的访问。               |
| PC      | 64bit | 程序计数器，俗称PC指针，总是指向即将要执行的下一条指令,在arm64中，软件是不能改写PC寄存器的。 |
| CPSR    | 64bit | 状态寄存器                                                   |

- x0-x7: 用于子程序调用时的参数传递，X0还用于返回值传递

- `x0 - x30` 是31个通用整形寄存器。每个寄存器可以存取一个64位大小的数。 当使用 `x0 - x30` 访问时，它就是一个64位的数。当使用 `w0 - w30` 访问时，访问的是这些寄存器的低32位，如图：

  ![img](https://upload-images.jianshu.io/upload_images/1117042-e3fabfec65c187f5.png?)

  image.png

  

- CPSR(状态寄存器)

> NZCV是状态寄存器的条件标志位，分别代表运算过程中产生的状态，其中：
>
> - N, negative condition flag，一般代表运算结果是负数
> - Z, zero condition flag, 指令结果为0时Z=1，否则Z=0；
> - C, carry condition flag, 无符号运算有溢出时，C=1。
> - V, oVerflow condition flag 有符号运算有溢出时，V=1。

- Xcode在真机中运行项目，添加断点，lldb中查看各寄存器状态register read

  ![img](https://upload-images.jianshu.io/upload_images/1117042-df0905c0c7d8b85e.png?)

  E2A993F9-F400-46C2-9B4C-E38DD2D6E383.png

### 指令

- ARM64经常用到的汇编指令



```css
MOV    X1，X0         ;将寄存器X0的值传送到寄存器X1
ADD    X0，X1，X2     ;寄存器X1和X2的值相加后传送到X0
SUB    X0，X1，X2     ;寄存器X1和X2的值相减后传送到X0

AND    X0，X0，#0xF    ; X0的值与0xF相位与后的值传送到X0
ORR    X0，X0，#9      ; X0的值与9相位或后的值传送到X0
EOR    X0，X0，#0xF    ; X0的值与0xF相异或后的值传送到X0

LDR    X5，[X6，#0x08]        ；ld：load; X6寄存器加0x08的和的地址值内的数据传送到X5
LDP  x29, x30, [sp, #0x10]    ; ldp :load pair ; 一对寄存器, 从内存读取数据到寄存器

STR X0, [SP, #0x8]         ；st:store,str:往内存中写数据（偏移值为正）; X0寄存器的数据传送到SP+0x8地址值指向的存储空间
STUR   w0, [x29, #-0x8]   ;往内存中写数据（偏移值为负）
STP  x29, x30, [sp, #0x10]    ;store pair，存放一对数据, 入栈指令

CBZ  ;比较（Compare），如果结果为零（Zero）就转移（只能跳到后面的指令）
CBNZ ;比较，如果结果非零（Non Zero）就转移（只能跳到后面的指令）
CMP  ;比较指令，相当于SUBS，影响程序状态寄存器CPSR 

B   ;跳转指令，可带条件跳转与cmp配合使用
BL  ;带返回的跳转指令， 返回地址保存到LR（X30）
BLR  ; 带返回的跳转指令，跳转到指令后边跟随寄存器中保存的地址(例：blr    x8 ;跳转到x8保存的地址中去执行)
RET   ;子程序返回指令，返回地址默认保存在LR（X30）
```

其中 `MOV` 指令只能用于寄存器之间传值，寄存器和内存之间传值通过 `LDR` 和 `STR`

- ARM指令又一个重要特点就是所有指令都是带有条件的，就是说汇编中就需要根据状态寄存器中的一些状态来控制分支的执行。例如：

  ![img](https://upload-images.jianshu.io/upload_images/1117042-9f6690220936a523.png?)

  131CC631-51C9-4601-AA45-EAC6EA43BEF3.png

  图中指令`b` 是跳转指令，后边跟着跳转的条件`eq`，那么这个`eq`是什么意思呢？

  

- ARM指令的结构

  ![img](https://upload-images.jianshu.io/upload_images/1117042-b2c177c73a7b92a6.png?)

  ARM指令编码表

  上图列出了不同种类ARM指令的编码格式，文章开头讲过ARM指令长度固定为 32bit即图中的0-31位。

> 28-31位是条件码，21-24为操作码，12-19为寄存器编号

上边提到的跳转条件`eq`实际就是28-31位对应的条件码，但是28-31位都是二进制数据不好记，所以就对二进制的条件码取了好记的助记符，例如 `eq`。

`eq`英文单词`equal`的意思，注意这里`equal`并不是c语言当中`==`的意思，这里根据状态寄存器的条件标志位`Z`来判断，如果`Z = 1`则`eq`成立，如果`Z = 0`则`eq`不成立，就是`NE`。

- ARM指令包含4位的条件码列表：

| 操作码 | 条件码助记符                  | 标志     | 含义               |
| ------ | :---------------------------- | :------- | :----------------- |
| 0000   | EQ                            | Z=1      | 相等               |
| 0001   | NE(Not Equal)                 | Z=0      | 不相等             |
| 0010   | CS/HS(Carry Set/High or Same) | C=1      | 无符号数大于或等于 |
| 0011   | CC/LO(Carry Clear/LOwer)      | C=0      | 无符号数小于       |
| 0100   | MI(MInus)                     | N=1      | 负数               |
| 0101   | PL(PLus)                      | N=0      | 正数或零           |
| 0110   | VS(oVerflow set)              | V=1      | 溢出               |
| 0111   | VC(oVerflow clear)            | V=0      | 没有溢出           |
| 1000   | HI(HIgh)                      | C=1,Z=0  | 无符号数大于       |
| 1001   | LS(Lower or Same)             | C=0,Z=1  | 无符号数小于或等于 |
| 1010   | GE(Greater or Equal)          | N=V      | 有符号数大于或等于 |
| 1011   | LT(Less Than)                 | N!=V     | 有符号数小于       |
| 1100   | GT(Greater Than)              | Z=0,N=V  | 有符号数大于       |
| 1101   | LE(Less or Equal)             | Z=1,N!=V | 有符号数小于或等于 |
| 1110   | AL                            | 任何     | 无条件执行(默认)   |
| 1111   | NV                            | 任何     | 从不执行           |

ARM指令所有指令都是带有条件的，默认是`AL`即无条件执行，当指令带有默认条件时不需要明确写出。

看个例子：
OC代码：



![img](https://upload-images.jianshu.io/upload_images/1117042-1da35ce35264be33.png?imageMogr2/auto-orient/strip|imageView2/2/w/637/format/webp)

D34A79BA-7AAE-471A-BB25-846EF0DBD9D8.png



汇编代码：



![img](https://upload-images.jianshu.io/upload_images/1117042-a9de01a9d58fbd26.png?imageMogr2/auto-orient/strip|imageView2/2/w/907/format/webp)

1FF4BE4B-6C1D-49CB-8DB6-FE6C32C2DA18.png


汇编代码注释：

![img](https://upload-images.jianshu.io/upload_images/1117042-5fac85f84be458dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

AE06E4F8-8CED-4979-A5F1-7DB5A48EA164.png

- 后续添加：

> adrp是计算指定的符号地址到run time PC值的相对偏移



```csharp
 - STR Wt, addr

Store Register: stores word from Wt to memory addressed by 
addr.


 - STR Xt, addr

Store Register (extended): stores doubleword from Xt to
 memory addressed by addr.


 - STUR Wt, [base,#simm9]

Store (Unscaled) Register: stores word from Wt to memory addressed by base+simm9.


 - STUR Xt, [base,#simm9]

Store (Unscaled) Register (extended): stores doubleword 
from Xt to memory addressed by base+simm9.


- SCVTF Sm, Ro  

Converts an integer value to a floating-point value.


- FCVTSty Rn, Sm    

Converts a floating-point value to an integer value (ty specifies type of rounding).


- FMUL Sd, Sn, Sm   

Multiplies two values.


 - LDRSB Wt, addr

Load Signed Byte: loads a byte from memory addressed by 
addr, then sign-extends it into Wt.


 - CBZ Wn, label

Compare and Branch Zero: conditionally jumps to label if
 Wn is equal to zero.
```

### MOVZ 、MOVK 和MOVN

MOVZ 赋值一个16位的立即数到寄存器中，该寄存器中除了立即数占用到的位之外的其他位都设为0，立即数可以设置向左移位0、16、32或者48(lsl：向左移位):



```csharp
instruction                           value of x0
movz    x0, #0x1234           |         0x1234
movz    x0, #0x1234, lsl #16  |         0x12340000
```

MOVK 赋值一个立即数到寄存器，保持立即数没用到的位保持不变。



```csharp
instruction                    value of x0
mov x0, xzr               | 0x0000000000000000
movk x0, #0x0123, lsl #48 | 0x0123000000000000
movk x0, #0x4567, lsl #32 | 0x0123456700000000
movk x0, #0x89ab, lsl #16 | 0x0123456789ab0000
movk x0, #0xcdef          | 0x0123456789abcdef
```

MOVN 用于赋值立即数的位掩码，例如想要将0xffffffff0000ffff赋值给x0,只需要使用MOVN将向左移位16的0xffff赋值位寄存器就可以实现，会自动求移位后的立即数位掩码然后赋值：



```csharp
      instruction                     value of x0
MOVN x0, 0xFFFF, lsl 16       |       0xffffffff0000ffff
```

### 参考

[iOS 中的 armv7,armv7s,arm64,i386,x86_64 都是什么](https://www.jianshu.com/p/3fce0bd6f045)
[ARM 指令的条件码](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Flincyang%2Farticle%2Fdetails%2F13623131)
[ARM视频教程](https://links.jianshu.com/go?to=http%3A%2F%2Fedu.51cto.com%2Fcenter%2Fcourse%2Flesson%2Findex%3Fid%3D46134)
[iOS开发同学的arm64汇编入门](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.cnbluebox.com%2Fblog%2F2017%2F07%2F24%2Farm64-start%2F%3Fhmsr%3Dtoutiao.io%26utm_medium%3Dtoutiao.io%26utm_source%3Dtoutiao.io)
[iOS逆向第五篇(ARM64 汇编)](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FyUHakSmU7zcJvTzjBvQDuQ)
《iOS应用逆向工程》
[LP64是什么意思](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fchaoguo1234%2Fp%2F5306596.html)



36人点赞



[iOS开发](https://www.jianshu.com/nb/3015466)