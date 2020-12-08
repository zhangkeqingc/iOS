# Macdown

[参考](https://www.runoob.com/markdown/md-advance.html)

Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档。 
Markdown 语言在 2004 由约翰·格鲁伯（英语：John Gruber）创建。 
Markdown 编写的文档可以导出 HTML 、Word、图像、PDF、Epub 等多种格式的文档。 
Markdown 编写的文档后缀为 .md, .markdown。 
Markdown 能被使用来撰写电子书，如：Gitbook。 
当前许多网站都广泛使用 Markdown 来撰写帮助文档或是用于论坛上发表消息。例如：GitHub、简书、reddit、Diaspora、Stack Exchange、OpenStreetMap 、SourceForge等。


## 编辑器
本教程将使用 Typora 编辑器来讲解 Markdown 的语法，Typora 支持 MacOS 、Windows、Linux 平台，且包含多种主题，编辑后直接渲染出效果。 
支持导出HTML、PDF、Word、图片等多种类型文件。 
Typora 官网：https://typora.io/ 
你也可以使用我们的在线编辑器来测试：https://c.runoob.com/front-end/712。 
参考：《了不起的Markdown》：

### 1、使用 # 号标记  
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

### 2、Markdown 段落  
Markdown 段落没有特殊的格式，直接编写文字就好, 
段落的换行是使用两个以上空格加上回车。(shift+enter)

2-1、字体 
*斜体文本* 
_斜体文本_ 
**粗体文本** 
__粗体文本__ 
***粗斜体文本*** 
___粗斜体文本___  

2-2、分隔线  
***

一一一一一一一

* * *

二二二二二二二

*****

三三三三三三三

- - -

四四四四四四四
     
------

2-3、删除线 
~~BAIDU.COM~~

2-4、下划线 
<u>带下划线文本</u>

2-5、脚注 
[^要注明的文本]


### 3、Markdown 列表   
Markdown 支持有序列表和无序列表。 
无序列表使用星号(*)、加号(+)或是减号(-)作为列表标记， 
这些标记后面要添加一个空格，然后再填写内容：

* 第一项   
* 第二项   
* 第三项   

+ 第一项
+ 第二项
+ 第三项


- 第一项   
- 第二项   
- 第三项  

1. 第一项   
2. 第二项   
3. 第三项   

3-1、列表嵌套  
1.  第一项：  
    -  第一项嵌套的第一个元素   
    -  第一项嵌套的第二个元素   
2.  第二项：  
    -  第二项嵌套的第一个元素   
    -  第二项嵌套的第二个元素   


### 4、Markdown 区块   
> 区块引用  
> 菜鸟教程  
> 学的不仅是技术更是梦想  

> 最外层  
> > 第一层嵌套  
> >
> > > 第二层嵌套  


> 区块中使用列表
> 1. 第一项  
> 2. 第二项  
> + 第一项  
> + 第二项  
> + 第三项  


* 第一项   
    > 菜鸟教程   
    > 学的不仅是技术更是梦想  
* 第二项  

### 5、Markdown 代码   
`printf()` 
代码区块 
代码区块使用 4 个空格或者一个制表符（Tab 键）  

	$(document).ready(function () {
	    alert('RUNOOB');
	});

``` javascript 备注
$(document).ready(function () {
    alert('RUNOOB');
});
```

### 6、Markdown 链接  
[链接名称](https:ww.baidu.com) 或者 <https:ww.baidu.com> 
<https://www.runoob.com>

6-1、高级链接 
这个链接用 1 作为网址变量 [Google][1] 
这个链接用 runoob 作为网址变量 [Runoob][runoob] 
然后在文档的结尾为变量赋值（网址）  

[1]: http://www.google.com/w
[runoob]: http://www.runoob.com/


### 7、Markdown 图片   

<img src="https://img1.3s78.com/codercto/797e62f79ad12c6bb13c4464c20ebffd" alt="图片10" style="zoom:25%;" />  

![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png) 
![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png "RUNOOB")   

这个链接用 1 作为网址变量 [RUNOOB][1]. 
然后在文档的结尾为变量赋值（网址）

[1]: http://static.runoob.com/images/runoob-logo.png

<img src="http://static.runoob.com/images/runoob-logo.png" width="20%">

### 8、Markdown 表格  

|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |
我们可以设置表格的对齐方式： 
-: 设置内容和标题栏居右对齐。 
:- 设置内容和标题栏居左对齐。 
:-: 设置内容和标题栏居中对齐。  

| 左对齐左对齐 | 右对齐右对齐 | 居中对齐居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |


### 9、Markdown 高级技巧   
支持的 HTML 元素
不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。 
目前支持的 HTML 元素有：<kbd> <b> <i> <em> <sup> <sub> <br>等 ，如：   

使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑

**文本加粗** 
\*\* 正常显示星号 \*\*   

\   反斜线 
`   反引号  

*   星号 
_   下划线 
{}  花括号 
[]  方括号
()  小括号
#   井字号
+   加号
-   减号
.   英文句点
!   感叹号

9-1、公式   

$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
${$tep1}{\style{visibility:hidden}{(x+1)(x+1)}}
$$



加粗： Ctrl/Cmd + B 
标题： Ctrl/Cmd + H 
插入链接： Ctrl/Cmd + K 
插入代码： Ctrl/Cmd + Shift + C 
行内代码： Ctrl/Cmd + Shift + K 
插入图片： Ctrl/Cmd + Shift + I 
无序列表： Ctrl/Cmd + Shift + L 
撤销： Ctrl/Cmd + Z   







# typora 快捷键



## Mac中的快捷键：

1. 最大标题：command + 1 或者：#
2. 大标题：command + 2 或者：##
3. 标准标题：command + 3 或者：###
4. 中标题：command + 4 或者：####
5. 小标题：command + 5 或者：#####
6. 插入表格：command + T
7. 插入代码：command + alt +c
8. 行间公式： command + Alt + b
9. 段落：command + 0
10. 竖线 ： command + Alt +q
11. 有序列表（1. 2.） ：输入数字+“.”之后输入空格 或者：command + Alt + o
12. 黑点标记：command + Alt + u
13. 隔离线shift + command + -
14. 超链接：command + Alt + l
15. 插入链接：command +k
16. 下划线：command +u
17. 加粗：command +b
18. 搜索：command +f

## 图片：

[![img](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170159282-378811511.png)](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170159282-378811511.png)
[![img](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170211920-1988294604.png)](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170211920-1988294604.png)

## 表情

输出表情需要借助 `：`符号。

栗子：`:smile` 显示为 😄,记住是左右两边都要冒号。

使用者可以通过使用`ESC`键触发表情建议补全功能，也可在功能面板启用后自动触发此功能。同时，直接从菜单栏`Edit` -> `Emoji & Symbols`插入UTF8表情符号也是可以的。

或者使用下面的方法

访问网站 https://emojikeyboard.org/，找到需要的符号，鼠标左键单击，然后粘贴到需要的地方就行了！🆗

## 数学公式

你可以通过使用**MathJax**来实现*LaTeX*的数学符号的表达。

输入`$$`，然后按下`Enter`键就会弹出一个支持TeX/LaTeX语法的输入框，下面是一个栗子：



**𝐕**1×**𝐕**2=∣∣**𝐢****𝐣****𝐤** ∂𝑋∂𝑢∂𝑌∂𝑢0 ∂𝑋∂𝑣∂𝑌∂𝑣0 ∣∣V1×V2=|ijk ∂X∂u∂Y∂u0 ∂X∂v∂Y∂v0 |



在Markdown源文件中，数学的公式块是通过利用`$$`标记借用*LaTeX*语言来实现的：

```text
Copy$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```

## HTML

Typora不能使用HTML元素，但是Typora可以解析和编译非常有限的HTML元素，作为Markdown功能的补充，这些有限的功能包括：

- 下划线： `<u>underline</u>`
- 图片：`<img src="http://www.w3.org/html/logo/img/mark-word-icon.png" width="200px" />`（HTML标签中的`width`, `height` 以及属于样式的`width`, `height`, `zoom`样式可以被识别和应用。）
- 评论：`<!-- This is some comments -->`
- 超链接： `<a href="http://typora.io" target="_blank">link</a>` 。

大多数这些属性、样式或分类会被忽略。对其他的标签，Typora会将它们以HTML片段的形式表达。

## 行内嵌数学符号

想要使用这个功能，需要在设置面板的 `Markdown`栏启用它。然后使用`$`来启动TeX命令，栗如：`$\lim_{x \to \infty} \exp(-x) = 0$` 会以LaTeX的命令形式表达出来。

为了触发行内内嵌数学符号的实时编译你需要：输入`$`然后按下`ESC`键之后输入TeX命令，之后就会弹出一个如图所示的工具提示栏：

[![img](https://pic3.zhimg.com/v2-4033508b043cad96c59ec4edbca92f36_b.gif)](https://pic3.zhimg.com/v2-4033508b043cad96c59ec4edbca92f36_b.gif)

## 下标[#](https://www.cnblogs.com/hongdada/p/9776547.html#550907507)

想要使用这个功能，需要在设置面板的 `Markdown` 栏启动它，之后使用`~`来修饰下标文本。栗如：

`H~2~O` 和`X~long\ text~` 显示为 H~2~O 和X~long\ text~ 。

\#### 13.上标

想要使用这个功能，需要在设置面板的 `Markdown` 栏启动它，之后使用`^`来修饰下标文本。栗如：

`X^2^` 显示为 X^2^ 。

## 高亮[#](https://www.cnblogs.com/hongdada/p/9776547.html#2958014103)

想要使用这个功能，需要在设置面板的`Markdown` 栏启动它，之后使用`==`来修饰高亮文本，栗如：

`==highlight==` 显示为 highlight 。



## 参考：[#](https://www.cnblogs.com/hongdada/p/9776547.html#3852665210)

https://baka943.coding.me/2018/02/08/2018-02-08-TyporaSimpleDoc/

[在markdown中使用HTML中的特殊符号：](https://blog.csdn.net/vola9527/article/details/69948411)

[Markdown输入Latex公式的特殊符号](https://blog.csdn.net/full_speed_turbo/article/details/69951768)

[Markdown For Typora 中文版使用指南](https://zhuanlan.zhihu.com/p/39872673)

[Cmd Markdown 公式指导手册](https://www.zybuluo.com/codeep/note/163962)

[Markdown 简介](https://bookdown.org/baydap/steemh/wzbjp.html)

作者： hongdada

出处：https://www.cnblogs.com/hongdada/p/9776547.html

版权：本文采用「[署名-非商业性使用-相同方式共享 4.0 国际](https://creativecommons.org/licenses/by-nc-sa/4.0/)」知识共享许可协议进行许可。



