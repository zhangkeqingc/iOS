# 如何制作一个CocoaPods私有库

最近在学习组件化相关的知识，也准备写个项目练练手。iOS组件化的实现是利用CocoaPods制作Pod库，主工程分别引用这些Pod库。最终想要达到的目标是主工程只是一个壳工程，其它代码都在组件Pod里，主工程只负责加载这些组件，没有其它任何代码。

这篇文章作为组件化开发的前置文章总结一下如何制作一个CocoaPods私有库。

下面通过一个例子来讲解整个步骤流程。

##### 1、打开终端，进入到要建立私有库工程的目录，执行以下命令，DDatePickerView为项目名

```
pod lib create DDatePickerView
```

这里会询问几个问题，答案根据实际情况具体设置

```
What platform do you want to use? [ iOS / macOS ]
 > iOS
What language do you want to use? [ Swift / ObjC ]
 > ObjC 
Would you like to include a demo application with your library? [ Yes / No ]
 > Yes
Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > None
Would you like to do view based testing? [ Yes / No ]
 > Yes 
What is your class prefix?
 > QZ
```

命令执行完成后会创建一个私有库工程。

##### 2、创建私有库Git地址，这里以GitHub为例

省略

##### 3、修改配置文件DDatePickerView.podspec

```
#
# Be sure to run `pod lib lint DDatePickerView.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see https://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = 'DDatePickerView'
  s.version          = '0.0.1'
  s.summary          = '自定义时间选择控件'

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/zhangkeqingc/DDatePickerView'
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'John' => 'zhangkeqinga@gmail.com' }
  s.source           = { :git => 'https://github.com/zhangkeqingc/DDatePickerView.git', :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.ios.deployment_target = '9.0'

  s.source_files = 'DDatePickerView/Classes/**/*'
  
  # s.resource_bundles = {
  #   'DDatePickerView' => ['DDatePickerView/Assets/*.png']
  # }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
end
```

podspec文件的详细说明可以看[Podspec Syntax Reference](http://guides.cocoapods.org/syntax/podspec.html)。

##### 4、打开终端，进入Example文件夹，执行`pod install`，安装依赖项

##### 5、添加库的源码文件

将源码文件放入DDatePickerView/Classes文件下，与podspec文件中的配置要保持一致。

```
ReplaceMe.swift //替换掉
```

运行`pod install`，让工程加载新添加的类。这里需要注意的是，**只要新增加类、资源文件或依赖的第三方库都需要重新运行`pod install`来进行更新。**

```
pod install
```

##### 6、添加测试代码，运行工程测试

添加测试代码，运行工程测试 。

##### 7、验证私有库正确性

```
cd /Users/tianyue/Desktop/DDatePickerView
pod lib lint DDatePickerView.podspec
```

如果出现

```
[!] MMUtils did not pass validation, due to 15 warnings (but you can use `--allow-warnings` to ignore them).
You can use the `--no-clean` option to inspect any issue.
```

忽略警告可以运行如下命令

```
pod lib lint DDatePickerView.podspec --allow-warnings
```

##### 8、提交源码到GitHub，并打tag

在项目工程文件下运行以下命令：

```
$ git remote add origin https://github.com/MyGitHubModule/MMUtils.git
$ git add .
$ git commit -a -m "0.0.1"
$ git pull origin master
$ git push origin master
$ git tag 0.0.1
$ git push origin 0.0.1
```

运行完成后，可以去GitHub上对我们的提交进行查看。

##### 9、发布私有库

对于开源库，podspec文件是放在CocoaPods的Specs项目下的。

![img](https://user-gold-cdn.xitu.io/2018/4/10/162b03a806696148?)

因为我们创建的是私有库，所以我们需要创建自己的Specs管理库。

创建一个Git库命名为MMSpecs，创建过程和第二步一样。

打开终端，运行

```
$ pod repo add MMSpecs https://github.com/MyGitHubModule/MMSpecs.git
```

执行成功后，可以看到在本地生成了MMSpecs目录。

![img](https://user-gold-cdn.xitu.io/2018/4/10/162b03a806745850?)

运行

```
$ pod repo push MMSpecs MMUtils.podspec 
```

进行发布。

如果没通过，可以试试`pod repo push MMSpecs MMUtils.podspec --allow-warnings`忽略警告。

成功之后可以在GitHub上对MMSpecs进行查看。

##### 10、在自己项目中引用私有库

Podfile如下：

```
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/MyGitHubModule/MMSpecs.git'
platform :ios, ‘8.0’

target “TestTest” do
pod 'MMUtils', '~> 0.0.1’
end
```

这里很尴尬，发现开源库中也有个同名的库。

![img](https://user-gold-cdn.xitu.io/2018/4/10/162b03a80679e1d8?)

修改Podfile为：

```
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/MyGitHubModule/MMSpecs.git'
platform :ios, ‘8.0’

target “TestTest” do
pod 'MMUtils', :git => 'https://github.com/MyGitHubModule/MMUtils.git', :tag => '0.0.1'
end
```

读者朋友这个操作就不要学了。

具体步骤流程就是这样了，这是我2018年的第一篇博客，希望自己在2018年接下来的日子里也能继续努力。大家共勉。

本文示例代码下载：[MMUtils](https://github.com/MyGitHubModule/MMUtils)

##### 参考链接

- [www.cnblogs.com/wsyuzx/p/70…](https://www.cnblogs.com/wsyuzx/p/7089564.html)
- [www.cnblogs.com/brycezhang/…](http://www.cnblogs.com/brycezhang/p/4117180.html)
- [www.jianshu.com/p/760d6cd46…](https://www.jianshu.com/p/760d6cd46719)