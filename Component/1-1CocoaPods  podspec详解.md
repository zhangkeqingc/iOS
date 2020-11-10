# iOS podspec详解



## 一、官方文档：

 [传送门](https://guides.cocoapods.org/syntax/podspec.html)

## 二、spec 规范说明

可以通过pod spec create命令创建。
简单使用：

```
Pod::Spec.new do |spec|
  spec.name         = 'Reachability'
  spec.version      = '3.1.0'
  spec.license      = { :type => 'BSD' }
  spec.homepage     = 'https://github.com/tonymillion/Reachability'
  spec.authors      = { 'Tony Million' => 'tonymillion@gmail.com' }
  spec.summary      = 'ARC and GCD Compatible Reachability Class for iOS and OS X.'
  spec.source       = { :git => 'https://github.com/tonymillion/Reachability.git', :tag => 'v3.1.0' }
  spec.source_files = 'Reachability.{h,m}'
  spec.framework    = 'SystemConfiguration'
end
```

详细：

```
Pod::Spec.new do |spec|
  spec.name          = 'Reachability'
  spec.version       = '3.1.0'
  spec.license       = { :type => 'BSD' }
  spec.homepage      = 'https://github.com/tonymillion/Reachability'
  spec.authors       = { 'Tony Million' => 'tonymillion@gmail.com' }
  spec.summary       = 'ARC and GCD Compatible Reachability Class for iOS and OS X.'
  spec.source        = { :git => 'https://github.com/tonymillion/Reachability.git', :tag => 'v3.1.0' }
  spec.module_name   = 'Rich'
  spec.swift_version = '4.0'

  spec.ios.deployment_target  = '9.0'
  spec.osx.deployment_target  = '10.10'

  spec.source_files       = 'Reachability/common/*.swift'
  spec.ios.source_files   = 'Reachability/ios/*.swift', 'Reachability/extensions/*.swift'
  spec.osx.source_files   = 'Reachability/osx/*.swift'

  spec.framework      = 'SystemConfiguration'
  spec.ios.framework  = 'UIKit'
  spec.osx.framework  = 'AppKit'

  spec.dependency 'SomeOtherPod'
end
```

## 三、Root specification

root spec存储特定版本库的信息。

一些属性只能写在root上，而一些细节规范则可以只写在sub spec上面，用于约束细节。
如下为root spec才可以使用的字段（required为必须存在的字段）：

- name(required)           //库的名称
- version(required)        //版本号 CocoaPods的版本信息遵循 [semantic versioning](https://semver.org/)规则
- swift_version                //版本号
- cocoapods_version     //支持的CocoaPods版本
- authors(required)       //作者
- social_media_url         //媒体联系人的URL
- license(required)         //Pod的许可证
- homepage(required)  //Pod主页的URL
- source(required)         //资源链接
- summary(required)    //摘要
- description                   //Pod的说明比摘要更详细
- screenshots                 //屏幕截图
- documentation_url     //Pod文档
- prepare_command     //下载Pod后将执行的bash脚本
- static_framework        //如果用了use_frameworks!，则应该声明为true
- deprecated                   //是否已弃用该库
- deprecated_in_favor_of   //被弃用库的名称



## 四、Platform

- platform
  支持此Pod的平台。空白则表示支持所有平台。当支持多个平台时，应使用deployment_target。

```
spec.platform = :osx, '10.8'
spec.platform = :ios1
spec.platform = :osx1
```

- deployment_target

  支持的平台的最低部署目标。
  与该`platform`属性相反，该`deployment_target` 属性允许指定支持该Pod的多个平台-为每个平台指定不同的部署目标。

```
spec.ios.deployment_target = '6.0'
spec.osx.deployment_target = '10.8'
```



## 五、Build settings

在该组中列出了与应用于构建库的构建环境的配置有关的属性。
如果未在子规范中定义，则此组的属性将继承父级的值。

- dependency
  依赖关系可以指定版本要求。
  `~>`推荐使用乐观版本指示器，因为它可以很好地控制版本，而又不会过于严格。
  例如， `~> 1.0.1`等效于`>= 1.0.1`与`< 1.1`。
  同样， `~> 1.0`将匹配`1.0`，`1.0.1`，`1.1`，但不会升级到`2.0`。

```
spec.dependency 'AFNetworking', '~> 1.0'
spec.dependency 'AFNetworking', '~> 1.0', :configurations => ['Debug']
spec.dependency 'AFNetworking', '~> 1.0', :configurations => :debug
spec.dependency 'RestKit/CoreData', '~> 0.20.0'
spec.ios.dependency 'MBProgressHUD', '~> 0.5'
```

- info_plist

  要添加到生成的键值对`Info.plist`。
  这些值将与CocoaPods生成的默认值合并，从而覆盖所有重复项。
  对于库规范，值将合并到为使用框架集成的库生成的Info.plist中。它对静态库无效。
  不支持子规格（应用和测试规格除外）。
  对于应用程序规范，这些值将合并到应用程序主机的中`Info.plist`。
  对于测试规格，这些值将合并到测试包的中`Info.plist`。

```
spec.info_plist = {
  'CFBundleIdentifier' => 'com.myorg.MyLib',
  'MY_VAR' => 'SOME_VALUE'
}
```

- requires_arc

  `requires_arc`允许特定文件使用ARC。
  true表示所有文件都支持ARC。
  不使用ARC的文件将带有`-fno-objc-arc`编译器标志。
  此属性的默认值为`true`。

```
Defaults to:
spec.requires_arc = true

Examples:
spec.requires_arc = false
spec.requires_arc = 'Classes/Arc'
spec.requires_arc = ['Classes/*ARC.m', 'Classes/ARC.mm']
```

指定某些文件不使用ARC，我们可以将这些文件放在一个spec.requires_arc = false的subspec里面：

```
non_arc_files = 'xxx/xxx.{h,m}'
s.subspec 'no-arc' do |mrc|
    mrc.source_files = non_arc_files
    mrc.requires_arc = false
end
```

- frameworks
  用户的target需要用到的框架。

```
spec.ios.framework = 'CFNetwork'
spec.frameworks = 'QuartzCore', 'CoreData'
```

- weak_frameworks
  需要弱连接的框架。

```
spec.weak_framework = 'Twitter'
spec.weak_frameworks = 'Twitter', 'SafariServices'
```

- libraries
  需要链接的系统库。

```
spec.ios.library = 'xml2'
spec.libraries = 'xml2', 'z'
```

- compiler_flags
  传递给编译器的标志列表。

```
spec.compiler_flags = '-DOS_OBJECT_USE_OBJC=0', '-Wno-format'
```

- pod_target_xcconfig
  要添加到最终专用 pod目标xcconfig文件的任何标志。

```
spec.pod_target_xcconfig = { 'OTHER_LDFLAGS' => '-lObjC' }
```

- user_target_xcconfig
  指定要添加到最终聚合目标xcconfig文件的标志，该文件将传播到非覆盖的目标文件，并将构建设置继承到集成用户目标。
  **不建议使用**此属性，因为Pod不应污染用户项目的构建设置，否则可能导致冲突。

```
spec.user_target_xcconfig = { 'MY_SUBSPEC' => 'YES' }
```

- prefix_header_contents
  任何要插入到pod项目的前缀标头中的内容。
  **不建议使用**此属性，因为Pod不应污染其他库或用户项目的前缀标头。

```
spec.prefix_header_contents = '#import <UIKit/UIKit.h>'
spec.prefix_header_contents = '#import <UIKit/UIKit.h>', '#import <Foundation/Foundation.h>'
```

- prefix_header_file
  前缀头文件的路径，可插入到pod项目的前缀头中。 `false`指示不应生成默认的CocoaPods前缀标头。 `true`是默认值，表示应生成默认的CocoaPods前缀标头。
  **不建议**使用文件路径选项，因为Pod不应污染其他库或用户项目的前缀标头。

```
spec.prefix_header_file = 'iphone/include/prefix.pch'
spec.prefix_header_file = false
```

- module_name 
  用于该规范的框架/ clang模块的名称，而不是默认的名称（如果设置，则为header_dir，否则为规范名称）。

```
spec.module_name = 'Three20'
```

- header_dir 
  头文件的存储目录，以便头文件不会中断。

```
spec.header_dir = 'Three20Core'
```

- header_mappings_dir 

  用于保留头文件的文件夹结构的目录。如果未提供，则将头文件展平。

```
spec.header_mappings_dir = 'src/include'
```

- script_phases

  此属性允许定义脚本阶段，以作为Pod编译的一部分执行。
  与prepare命令不同，脚本阶段作为`xcodebuild`它们的一部分执行，也可以利用在编译期间设置的所有环境变量。

  Pod可以提供要执行的多个脚本阶段，并且将按照声明的顺序添加它们，并考虑它们的执行位置设置。

  **注意**为了提供对所有脚本阶段内容的可见性和意识，如果安装了Pod包含任何脚本阶段，则会在安装时向用户显示警告。

```
spec.script_phase = { :name => 'Hello World', :script => 'echo "Hello World"' }

spec.script_phase = { :name => 'Hello World', :script => 'echo "Hello World"', :execution_position => :before_compile }

spec.script_phase = { :name => 'Hello World', :script => 'puts "Hello World"', :shell_path => '/usr/bin/ruby' }

spec.script_phase = { :name => 'Hello World', :script => 'echo "Hello World"',
  :input_files => ['/path/to/input_file.txt'], :output_files => ['/path/to/output_file.txt']
}

spec.script_phase = { :name => 'Hello World', :script => 'echo "Hello World"',
  :input_file_lists => ['/path/to/input_files.xcfilelist'], :output_file_lists => ['/path/to/output_files.xcfilelist']
}

spec.script_phases = [
    { :name => 'Hello World', :script => 'echo "Hello World"' },
    { :name => 'Hello Ruby World', :script => 'puts "Hello World"', :shell_path => '/usr/bin/ruby' },
  ]
```



## 六、文件符号

Podspecs应该位于存储库的**根目录**，并且还应相对于存储库的根目录指定文件路径。文件格式不支持遍历父目录（`..`）。文件模式可能包含以下通配符模式：：

- 符号：*

  匹配任何文件。可以受全局中的其他值限制。

  - `*`匹配所有文件
  - `c*` 匹配所有c开头的文件
  - `*c` 匹配所有c结尾的文件
  - `*c*` 匹配所有包含c的文件

- 符号：`**`
  递归路径

- 符号：`?`
  匹配任意一个字符

- 符号：`[set]`
  匹配任意set中的一个字符，和正则规则类似，例如否集`[^a-z]`

- 符号：`{p,q}`
  包含p和q的集合

- 符号：`\`
  跳出下一个元字符。

例如在https://github.com/johnezang/JSONKit的文件路径下：

```
"JSONKit.?"    #=> ["JSONKit.h", "JSONKit.m"]
"*.[a-z][a-z]" #=> ["CHANGELOG.md", "README.md"]
"*.[^m]*"      #=> ["JSONKit.h"]
"*.{h,m}"      #=> ["JSONKit.h", "JSONKit.m"]
"*"            #=> ["CHANGELOG.md", "JSONKit.h", "JSONKit.m", "README.md"]
```

#### （1）source_files

pod的源文件：

```
spec.source_files = 'Classes/**/*.{h,m}'
spec.source_files = 'Classes/**/*.{h,m}', 'More_Classes/**/*.{h,m}'
```

#### （2）public_header_files

应用作公共头文件的的文件模式列表。
这些头文件将暴露给用户的工程，当这个库被建立的时候，这些文件也将出现在build目录下，如果没有设定头文件，那么所有source的头文件都将被暴露。

```
spec.public_header_files = 'Headers/Public/*.h'
```

#### （3）private_header_files

私有的头文件。
如果没有设定公有头文件，所有文件被暴露的情况下，设定私有头文件，则这部分文件将不会暴露出来。
没有被私有和公有头文件设定的文件，将被视作私有的。

```
spec.private_header_files = 'Headers/Private/*.h'
```

#### （4）vendored_frameworks

路径下的framework将与pod一同使用

```
spec.ios.vendored_frameworks = 'Frameworks/MyFramework.framework'
spec.vendored_frameworks = 'MyFramework.framework', 'TheirFramework.framework'
```

#### （5）vendored_libraries

路径下的Library将与pod一同使用

```
spec.ios.vendored_library = 'Libraries/libProj4.a'
spec.vendored_libraries = 'libProj4.a', 'libJavaScriptCore.a'
```

#### （6）resource_bundles

该方式会建立name和file对应的bundle文件
强烈建议使用该方式，将所需的资源打成bundle，这样可以避免文件名的冲突，在命名的时候建议加上pod库的相关名称以避免bundle名字的冲突。

```
spec.ios.resource_bundle = { 'MapBox' => 'MapView/Map/Resources/*.png' }
spec.resource_bundles = {
    'MapBox' => ['MapView/Map/Resources/*.png'],
    'OtherResources' => ['MapView/Map/OtherResources/*.png']
  }
```

#### （7）resources

copy至target的资源列表。
强烈建议使用上述的`resource_bundles`方式，来避免命名冲突，并且该方式是直接拷贝到客户端，没有xcode对相关资源的优化措施。

```
spec.resource = 'Resources/HockeySDK.bundle'
spec.resources = ['Images/*.png', 'Sounds/*']
```

#### （8）exclude_files

应该从其他文件模式中排除的文件模式列表。

```
spec.ios.exclude_files = 'Classes/osx'
spec.exclude_files = 'Classes/**/unused.{h,m}'
```

#### （9）preserve_paths

下载后不应删除的文件。
默认情况下，CocoaPods将移除所有其他文件模式不匹配的文件。

```
spec.preserve_path = 'IMPORTANT.txt'
spec.preserve_paths = 'Frameworks/*.framework'
```

#### （10）module_map

当这个pod集成为framework时所需要使用的模块映射文件。

默认情况下，CocoaPods根据spec中的公共头文件创建模块映射文件。

```
spec.module_map = 'source/module.modulemap'
```

#### （11）script_phases

这个属性允许定义一个脚本，作为编译pod的一部分来执行。与prepare command不同，脚本阶段作为xcodebuild的一部分执行，它们还可以利用编译期间设置的所有环境变量。

一个pod可以提供多个脚本来执行，它们将按照声明的顺序来执行

注意，为了提供可见性和意识到执行内容的情况，如果用户安装了你的pod，如果包含任何脚本，将向用户发出警告。

```
spec.script_phase = { :name => 'Hello World', :script => 'echo "Hello World"' }1
spec.script_phase = { :name => 'Hello World', :script => 'echo "Hello World"', :execution_position => :before_compile }1
spec.script_phase = { :name => 'Hello World', :script => 'puts "Hello World"', :shell_path => '/usr/bin/ruby' } }1
spec.script_phases = [
    { :name => 'Hello World', :script => 'echo "Hello World"' },
    { :name => 'Hello Ruby World', :script => 'puts "Hello World"', :shell_path => '/usr/bin/ruby' } },
  ]
```

## 七、Subspecs

一个library可以指定对另一个library的依赖性，另一个library的subspec或自身的subspec。

- subspec
  表明库的一个子模块的spec
  一般来说，subspec集成父类的值，并且有自己的设定，比如我们在使用如下库时，当我们庄ShareKit，会将她所有的subspec都安装上，有些我们并不需要

```
pod 'ShareKit', '2.0'
```

而我们可以指定安装的subspec来安装我们想要的库

```
pod 'ShareKit/Twitter',  '2.0'
pod 'ShareKit/Pinboard', '2.0'
```

而这样做的前提是我们设定了ShareKit的诸多subspec。
例如：

- subspec含有不同的source文件

```
subspec 'Twitter' do |sp|
  sp.source_files = 'Classes/Twitter'
end

subspec 'Pinboard' do |sp|
  sp.source_files = 'Classes/Pinboard'
end
```

- 子模块对其子模块的依赖：

```
Pod::Spec.new do |s|
  s.name = 'RestKit'

  s.subspec 'Core' do |cs|
    cs.dependency 'RestKit/ObjectMapping'
    cs.dependency 'RestKit/Network'
    cs.dependency 'RestKit/CoreData'
  end

  s.subspec 'ObjectMapping' do |os|
  end
end
```

- 嵌套subspec

```
Pod::Spec.new do |s|
  s.name = 'Root'

  s.subspec 'Level_1' do |sp|
    sp.subspec 'Level_2' do |ssp|
    end
  end
end
```

- requires_app_host
  测试spec是否要求应用程序主机运行测试。这仅适用于测试spec。

```
test_spec.requires_app_host = true
```

- test_spec
  表示库的测试spec规范。在这里，可以为podspec提供所有的测试以及测试依赖项。

```
Pod::Spec.new do |spec|
  spec.name = 'NSAttributedString+CCLFormat'

  spec.test_spec do |test_spec|
    test_spec.source_files = 'NSAttributedString+CCLFormatTests.m'
    test_spec.dependency 'Expecta'
  end
end
```

- default_subspecs
  一个subspec的数组，该数组将作为优先依赖的subspec进行提供，如果没有指定，则spec要求全部subspec为依赖项。
  默认情况下，pod应该可以提供完整的库。当用户的要求是已知的，用户可以微调他们的依赖关系，并排除不需要的subspec。因此，很少需要这种属性。它是用来选择默认subspec的，例如有subspec提供替代一些subspec不兼容的实现，或很少需要的模块（特别是如果它们依赖于其他库）

```
spec.default_subspec = 'Core'
spec.default_subspecs = 'Core', 'UI'
```

## 八、多平台支持

规范可以存储仅特定于一个平台的值。
例如，可能要存储仅适用于iOS项目的资源。

```
spec.resources = 'Resources/**/*.png'
spec.ios.resources = 'Resources_ios/**/*.png'
```

为不同的平台提供不同的资源。

- ios

```
spec.ios.source_files = 'Classes/ios/**/*.{h,m}'
```

- osx

```
spec.osx.source_files = 'Classes/osx/**/*.{h,m}'
```

- macos

```
spec.osx.source_files = 'Classes/osx/**/*.{h,m}'
```

- tvos

```
spec.tvos.source_files = 'Classes/tvos/**/*.{h,m}'
```

- watchos

```
spec.watchos.source_files = 'Classes/watchos/**/*.{h,m}'
```