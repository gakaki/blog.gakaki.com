---
title: 使用XCFramework的注意点
date: 2019-11-28 19:20:18
categories:
- iOS
tags:
- iOS
- Swift
---

* 这篇文章翻译自 https://github.com/bielikb/xcframeworks 感谢作者

此项目演示了 在同一个Xcode项目中,创建与集成xcframeworks以及静态库与Swift packages的协作。

## 目录 
* [介绍XCFramework新格式](#介绍XCFramework新格式)
* [如何创建IOS和IOS模拟器的XCFramework](#如何创建IOS和IOS模拟器的XCFramework)
* [使用create_xcframeworks.sh生成ios和ios模拟器的xcframeworks](#使用create_xcframeworks.sh生成ios和ios模拟器的xcframeworks)
* [测试与故障排除](#测试与故障排除)
* [xcframeworks的分发](#xcframeworks的分发)
* [如何在项目中集成XCFramework](#如何在项目中集成XCFramework)
* [XCFrameworks的工作目录结构](#XCFrameworks的工作目录结构)
* [其他资料](#其他资料)

## 前置要求
- Xcode 11
- Swift 5.1 toolchain - 终端里运行 `sudo xcode-select -s path/to/Xcode11`
- Github/Gitlab/Bitbucket 在Xcode的账号设置里 预先设置

# 介绍XCFramework新格式

## 要求
- Xcode11
- Swift 5.1 及以上吗

## 动机 & 结果
- 引入标准格式以获得Swift框架和库的模块稳定性。库作者和库的使用者不再需要使用相同版本的编译器（注：应该是Swift的ABI和Module稳定化的原因了）
- 在创建和集成模块稳定框架时 提供无缝的体验（注：和过去的Framework一样拖进去就是了，但实际上大部分人用的还是cocospods 而cocospods 1.9才会加入xcframework）
- 支持所有的苹果平台和架构

注：虽然 "胖框架 fat framework" 可以支持模块的稳定性，但Apple并未正式支持用于将框架融合在一起的"lipo"命令行工具。在将具有相似架构的两个平台融合在一起的情况下，也使用lipo工具不足。在iOS + watchOS中可以找到arm6体系结构。

- **STOP** 创建和使用 `fat frameworks` == 无需使用 `lipo`.
- **STOP** 通过在项目的 自定义 `build-phase` （构建阶段）剥离架构 从而切分出 frameworks

## xcframwork的内容
此格式打包了 module 稳定的 frameworks (.swiftinterface) )

[Info.plist]（./ Products / xcframeworks / DynamicFramework.xcframework）bundle 包含了所有可用框架frameworks。 Xcode在链接期间使用此信息=> xcodebuild会为我们要构建的平台选择正确的框架framework

xcframework的结构看起来如下
![xcframework](https://midu.studio515.cn/2019_12_xcframework/xcframework.png)

## xcframework的大小
“xcframework”的大小小于相应的“ fat framework”的大小。我测试了纯swift和 oc混合框架。
通常“lipo”命令行工具会为所有包含的体系结构增加一些开销（会大一些）

## Platforms
xcframework 支持所有的苹果平台 - `iOS`, `macOS`, `tvOS`, `watchOS`, `iPadOS`, `carPlayOS`.

## List of destinations
| Platform  |  Destination |
|---|---|
| iOS  | generic/platform=iOS  |
| iOS Simulator  | generic/platform=iOS Simulator |
| iPadOS  | generic/platform=iPadOS |
| iPadOS Simulator  | generic/platform=iPadOS Simulator|
| macOS  | generic/platform=macOS  |
| tvOS  | generic/platform=tvOS  |
| watchOS | generic/platform=watchOS |
| watchOS Simulator | generic/platform=watchOS Simulator |
| carPlayOS | generic/platform=carPlayOS
| carPlayOS Simulator | generic/platform=carPlayOS Simulator

---

# 如何创建IOS和IOS模拟器的XCFramework

## 1. 为所需平台 归档
1.1 （命令行）传入 `SKIP_INSTALL=NO` && `BUILD_LIBRARY_FOR_DISTRIBUTION=YES` 进行打包

```swift
xcodebuild archive \
-workspace MyWorkspace.xcworkspace \
-scheme MyScheme \
-destination destination="generic/platform=iOS" \
-archivePath "archives/MyScheme-iOS" \
SKIP_INSTALL=NO \
BUILD_LIBRARY_FOR_DISTRIBUTION=YES
```

1.2 **iOS 模拟器** - 为ios模拟器平台打包 指定对应的destion参数 `destination="generic/platform=iOS Simulator"`并指定对应的路径, 例如. `archives/MyScheme-iOS-Simulator`.

```swift
xcodebuild archive \
..
-destination destination="generic/platform=iOS Simulator" \
-archivePath "archives/MyScheme-iOS-Simulator" \
..
```

1.3 **iOS** - （ios真机） 为ios真机打包 指定对应的ios平台 `destination="generic/platform=iOS"`并给出对应的路径。架构路径同刚才的1.2，修改为`MyScheme-iOS`

```
xcodebuild archive \
..
-destination destination="generic/platform=iOS" \
-archivePath "archives/MyScheme-iOS" \
..
```

### 所在目录
二进制产物文件在 `.xcarchive` 在：
* `Products/Library/Frameworks` 动态库framework目录
* `Products/usr/local/lib` 静态库目录

## 2. 创建xcframework 从产物中
`xcodebuild` Xcodebuild 允许您通过指定框架、库来创建 xcframework，甚至可以将header添加到库中。

![-create-xcframework](https://midu.studio515.cn/2019_12_xcframework/xcodebuild_create_xc_framework.png)

###### 1. 指定要添加到. xcframework 中的所有框架或库
###### 2. 指定输出路径,使用-output参数. 不要忘记加上.xcframework 添加到您的输出路径

```
xcodebuild -create-xcframework \
           -framework My-iOS.framework \
           -framework My-iOS_Simulator.framework \
           -output My.xcframework
```

模块的稳定性来自于Xcode11 + Swift 5.1。
一旦你的模块声明 `.swiftinterface` (描述框架的公共接口以及链接器标志、使用的工具链和其他信息)文件。
Swift interface 文件 ，可以在框架的 swiftmodule 文件夹下找到 Swiftinterface 文件是在 xcframework 创建时自动生成的。

![swift-interface](https://midu.studio515.cn/2019_12_xcframework/swiftinterface.png)

## 使用create_xcframeworks.sh生成ios和ios模拟器的xcframeworks

.xcframework的归档和创建由[create_xcframeworks.sh]（/ scripts / create_xcframeworks.sh）脚本执行。

此脚本接受 1个参数(输出目录)。 `Output directory` 输出目录将为 `archives` 和 `xcframeworks` 创建子文件夹。

此脚本将会：

- 根据 scheme `StaticLibrary` 归档 并 创建. xcframework           （从静态库生成.xcframework)
- 根据 scheme `DynamicFramework` 归档 并 创建. xcframework        （从动态库生成.xcframework)

![生成的 xcframework](https://midu.studio515.cn/2019_12_xcframework/generated_xcframework.png)

`使用方法`

```
./scripts/create_xcframeworks.sh 输出目录
```

例如.
```
./scripts/create_xcframeworks.sh Products
```

---

# 测试与故障排除

在 分发给客户之前，请确保构建并运行生成的xcframework。 很少有问题会在编译或运行时提示，所以不要仅仅依赖于 xcframework 创建的成功（不要以为xcframwork创建成功就是正常的 请多做测试）

下面是我 构建xcframework并集成到 Xcode 项目中时遇到的编译器错误列表。


| 问题  | 严重程度 |  描述  | 解决方案 |
|---|---|---|---|
| Redundant conformance of `x` to `NSObjectProtocol` | error - thrown at dynamic linking time | Your class is already subclassed from `NSObject`, which conforms to `NSObjectProtocol`  | Check protocol conformances of your classes and remove redundant conformance to `NSObjectProtocol` |
| Use of unimplemented initializer 'init()' for class | error - thrown at dynamic linking time | Objective-C ABI public classes need to provide `public` init | Provide `public` init override for your public class:  `override public init()` |
| @objc' class method in extension of subclass of `Class X` requires iOS 13.0.0 | error | Rules for interoperability with Objective-C has changed since iOS 13.0.0. and currently doesn't support `@objc` interoperability in class extensions. There's open question on [Swift forums](https://forums.swift.org/t/xcframework-requires-to-target-ios-13-for-inter-framework-dependencies-with-objective-c-compatibility-tested-with-xcode-11-beta-7/28539) | Move/Remove `@objc` declaration from your Swift class extension |
| scoped imports are not yet supported in module interfaces | warning | Read more about Swift import declarations here: https://nshipster.com/import/ | Import the module instead of specific declaration. For example: change `import class MyModule.MyClass` to `import MyModule` |

---

# xcframeworks的分发
* 手动集成 - 从今天起可用（现在就可以用）

* **Carthage**  
    - xcframework 作为二进制发布——从今天起可用
    - [Roadmap to provide support for xcframeworks 2019/2020](https://github.com/Carthage/Carthage/issues/2890)

* **CocoaPods**
    - （提了个pr，v1.9可能会支持）[Feature request to bring support for new xcframework format in v1.9.0](https://github.com/CocoaPods/CocoaPods/issues/9148) + (PR)[https://github.com/CocoaPods/CocoaPods/pull/9334]

* **Swift Package Manager**
    - [SE-0272 Package Manager Binary Dependencies (returned for revision)](https://forums.swift.org/t/se-0272-package-manager-binary-dependencies/30753)

---

# 如何在项目中集成XCFramework
1. 手动将 xcframework 拖放到项目目标中
![Drag & drop xcframework](https://midu.studio515.cn/2019_12_xcframework/xcframework_drag_n_drop.gif)

2. 在项目目标中 嵌入 & 签名 .xcframework 到你的项目targert中
![Embed & sign .xcframework](https://midu.studio515.cn/2019_12_xcframework/xcframework_embed_sign.png)

---

# XCFrameworks的工作目录结构
`XCFrameworks`  工作区包括:
- `StaticLibrary` project - 静态库项目
- `DynamicFramework` project - 动态库dylib项目
- `Swift Package` - Swift Package 项目（含有例子项目）
- `TextAttributes` - 外部引用的Swift Package
- `Sample` - 例子项目 宝航了上面所有的依赖

![swift-interface](https://midu.studio515.cn/2019_12_xcframework/project.png)

---

# 其他资料
## Binary Frameworks in Swift
https://developer.apple.com/videos/play/wwdc2019/416/

## ABI Stability & Module Stability - swift.org
https://swift.org/blog/abi-stability-and-more/

## Library evolution for stable ABIs
https://github.com/apple/swift-evolution/blob/master/proposals/0260-library-evolution.md

## Library evolution - Docs
https://github.com/apple/swift/blob/master/docs/LibraryEvolution.rst

## Swift Unwrapped - Swift 5.1 with Doug Gregor (Library evolution, ...)
https://spec.fm/podcasts/swift-unwrapped/308610

## Alexis Beingessner- How Swift Achieved Dynamic Linking Where Rust Couldn't
https://gankra.github.io/blah/swift-abi/

## Presentation about Dependency management in Xcode 11
https://www.slideshare.net/BorisBielik/dependency-management-in-xcode-11-153424888