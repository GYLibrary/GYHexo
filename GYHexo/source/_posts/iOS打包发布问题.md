---
title: Xcode 8.0+自动打包并发布脚本
date: 2016-12-15 20:37:16
categories: 打包
tags: [Archive] 
description: 使用最新Xcode 8.0 打包发布遇到的问题收集。
---
# Xcode 8+ 打包发布问题
## 问题简介

`xcodebuild` 是苹果提供的打包项目或者工程的命令，了解该命令最好的方式就是使用 `man xcodebuild` 查看其 man page. 尽管是英文，一定要老老实实的读一遍就好了。
> DESCRIPTION
xcodebuild builds one or more targets contained in an Xcode project, or builds a scheme contained in an Xcode workspace or Xcode project.
Usage
To build an Xcode project, run xcodebuild from the directory containing your project (i.e. the directory containing the name.xcodeproj package). If you have multiple projects in the this directory you will need to use -project to indicate which project should be built. By default, xcodebuild builds the first target listed in the project, with the default build configuration. The order of the targets is a property of the project and is the same for all users of the project.
To build an Xcode workspace, you must pass both the -workspace and -scheme options to define the build. The parameters of the scheme will control which targets are built and how they are built, although you may pass other options to xcodebuild to override some parameters of the scheme.
There are also several options that display info about the installed version of Xcode or about projects or workspaces in the local directory, but which do not initiate an action. These include -list, -showBuildSettings, -showsdks, -usage, and -version.

总结:
1. 需要在包含 name.xcodeproj 的目录下执行 `xcodebuild` 命令，且如果该目录下有多个 projects，那么需要使用 `-project` 指定需要 build 的项目。
2. 在不指定 build 的 target 的时候，默认情况下会 build project 下的第一个 target
3. 当 build workspace 时，需要同时指定 `-workspace` 和 `-scheme` 参数，scheme 参数控制了哪些 targets 会被 build 以及以怎样的方式 build。
4. 有一些诸如 `-list`,`-showBuildSettings`, `-showsdks` 的参数可以查看项目或者工程的信息，不会对 build action 造成任何影响，放心使用。

那么，`xcodebuild` 究竟如何使用呢？ 继续看文档:
>NAME
xcodebuild – build Xcode projects and workspaces
SYNOPSIS
1. xcodebuild [-project name.xcodeproj] [[-target targetname] … | -alltargets] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action …] [buildsetting=value …] [-userdefault=value …]
2. xcodebuild [-project name.xcodeproj] -scheme schemename [[-destination destinationspecifier] …] [-destination-timeout value] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action …] [buildsetting=value …] [-userdefault=value …]
3. xcodebuild -workspace name.xcworkspace -scheme schemename [[-destination destinationspecifier] …] [-destination-timeout value] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action …] [buildsetting=value …] [-userdefault=value …]
4. xcodebuild -version [-sdk [sdkfullpath | sdkname]] [infoitem]
5. xcodebuild -showsdks
6. xcodebuild -showBuildSettings [-project name.xcodeproj | [-workspace name.xcworkspace -scheme schemename]]
7. xcodebuild -list [-project name.xcodeproj | -workspace name.xcworkspace]
8. xcodebuild -exportArchive -archivePath xcarchivepath -exportPath destinationpath -exportOptionsPlist path
9. xcodebuild -exportLocalizations -project name.xcodeproj -localizationPath path [[-exportLanguage language] …]
10. xcodebuild -importLocalizations -project name.xcodeproj -localizationPath path

挑几个我常用的形式介绍一下，较长的使用方式以序列号代替:

* xcodebuild -showsdks: 列出 Xcode 所有可用的 SDKs

* xcodebuild -showBuildSettings: 上述序号6的使用方式，查看当前工程 build setting 的配置参数，Xcode 详细的 build setting 参数参考官方文档 Xcode Build Setting Reference， 已有的配置参数可以在终端中以 buildsetting=value 的形式进行覆盖重新设置.

* xcodebuild -list: 上述序号7的使用方式，查看 project 中的 targets 和 configurations，或者 workspace 中 schemes, 输出如下:

```
Information about project "NavTabBar":
    Targets:
        NavTabBar
        NavTabBarTests
        NavTabBarUITests

    Build Configurations:
        Debug
        Release
        Ad-hoc

    If no build configuration is specified and -scheme is not passed then "Release" is used.

    Schemes:
        NavTabBar
```
* `xcodebuild [-project name.xcodeproj] [[-target targetname] ... | -alltargets] build`: 上述序号1的使用方式，会 build 指定 project，其中 `-target` 和 `-configuration` 参数可以使用 `xcodebuild -list` 获得，`-sdk` 参数可由 `xcodebuild` `-showsdks` 获得，`[buildsetting=value ...]` 用来覆盖工程中已有的配置。可覆盖的参数参考官方文档 [Xcode Build Setting Reference](https://developer.apple.com/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/), `action...` 的可用选项如下, 打包的话当然用 build，这也是默认选项。

> build
Build the target in the build root (SYMROOT). This is the default action, and is used if no action is given.

> analyze
Build and analyze a target or scheme from the build root (SYMROOT). This requires specifying a scheme.

> archive
Archive a scheme from the build root (SYMROOT). This requires specifying a scheme.

> test
Test a scheme from the build root (SYMROOT). This requires specifying a scheme and optionally a destination.

> installsrc
Copy the source of the project to the source root (SRCROOT).

> install
Build the target and install it into the target’s installation directory in the distribution root (DSTROOT).

> clean
Remove build products and intermediate files from the build root (SYMROOT).

* `xcodebuild -workspace name.xcworkspace -scheme schemename build`: 上述序号3的使用方式，build 指定 workspace，当我们使用 CocoaPods 来管理第三方库时，会生成 xcworkspace 文件，这样就会用到这种打包方式.

## 使用xcodebuild和xcrun打包签名
开始之前，可以新建一个测试工程 TestImg 来练习打包，在使用终端命令打包之前，请确认该工程也可以直接使用 Xcode 真机调试成功。

然后，打开终端，进入包含 TestImg.xcodeproj 的目录下，运行以下命令:

```
xcodebuild -project TestImg.xcodeproj -target TestImg -configuration Release
```
如果 build 成功，会看到 ** BUILD SUCCEEDED ** 字样，且在终端会打印出这次 build 的签名信息，如下:
> Signing Identity: “iPhone Developer: xxx(59xxxxxx)”
Provisioning Profile: “iOS Team Provisioning Profile: *"

且在该目录下会多出一个 `build` 目录，该目录下有 `Release-iphoneos` 和 `TestImg.build` 文件，根据我们 `build -configuration` 配置的参数不同，Release-iphoneos 的文件名会不同。

在 `Release-iphoneos` 文件夹下，有我们需要的`TestImg.app`文件，但是要安装到真机上，我们需要将该文件导出为ipa文件，这里使用 xcrun 命令。

```
xcrun -sdk iphoneos -v PackageApplication ./build/Release-iphoneos/TestImg.app -o ~/Desktop/TestImg.ipa
```

这里又冒出一个 `PackageApplication`, 我刚开始也不知道这是个什么玩意儿，万能的google告诉我，这是 Xcode 包里自带的工具，使用 `xcrun -sdk iphoneos -v PackageApplication -help` 查看帮助信息.

> Usage:
PackageApplication [-s signature] application [-o output_directory] [-verbose] [-plugin plugin] || -man || -help

> Options:

> `[-s signature]`: certificate name to resign application before packaging
`[-o output_directory]`: specify output filename
`[-plugin plugin]`: specify an optional plugin
`-help`: brief help message
`-man`: full documentation
`-v[erbose]`: provide details during operation

如果执行成功，则会在你的桌面生成 TestImg.ipa 文件，这样就可以发布测试了。如果你遇到以下警告信息:

> Warning: –resource-rules has been deprecated in Mac OS X >= 10.10! ResourceRules.plist: cannot read resources

请参考 [stackoverflow](http://stackoverflow.com/questions/32504355/error-itms-90339-this-bundle-is-invalid-the-info-plist-contains-an-invalid-ke/32762413#32762413)

## 将打包过程脚本化
工作中，特别是所做项目进入测试阶段，肯定会经常打 Ad-hoc 包给测试人员进行测试，但是我们肯定不想每次进行打包的时候都要进行一些工程的设置修改，以及一系列的 next 按钮点击操作，现在就让这些操作都交给脚本化吧。

1.脚本化中使用如下的命令打包:

```
xcodebuild -project name.xcodeproj -target targetname -configuration Release -sdk iphoneos build CODE_SIGN_IDENTITY="$(CODE_SIGN_IDENTITY)" PROVISIONING_PROFILE="$(PROVISIONING_PROFILE)"
```
或者

```
xcodebuild -workspace name.xcworkspace -scheme schemename -configuration Release -sdk iphoneos build CODE_SIGN_IDENTITY="$(CODE_SIGN_IDENTITY)" PROVISIONING_PROFILE="$(PROVISIONING_PROFILE)"
```
2.然后使用 xcrun 生成 ipa 文件:

```
xcrun -sdk iphoneos -v PackageApplication ./build/Release-iphoneos/$(target|scheme).app
```
3.清除 build 过程中产生的中间文件
4.结合蒲公英分发平台，将 ipa 文件上传至蒲公英分发平台，同时在终端会打印上传结果以及上传应用后该应用的 URL。蒲公英分发平台能够方便地将 ipa 文件尽快分发到测试人员，该平台有开放 API，可避免人工上传。

该脚本的使用可使用 `python autobuild.py -h` 查看，与 `xcodebuild` 的使用相似:
>Usage: autobuild.py [options]

>Options:
`-h, --help`: show this help message and exit
`-w name.xcworkspace, --workspace=name.xcworkspace`: Build the workspace name.xcworkspace.
`-p name.xcodeproj, --project=name.xcodeproj`: Build the project name.xcodeproj.
`-s schemename, --scheme=schemename`: Build the scheme specified by schemename. Required if building a workspace.
`-t targetname, --target=targetname`: Build the target specified by targetname. Required if building a project.
`-o output_filename, --output=output_filename`: specify output filename

在脚本顶部，有几个全局变量，根据自己的项目情况修改。

```
CODE_SIGN_IDENTITY = "iPhone Distribution: companyname (9xxxxxxx9A)"
PROVISIONING_PROFILE = "xxxxx-xxxx-xxx-xxxx-xxxxxxxxx"
CONFIGURATION = "Release"
SDK = "iphoneos"

USER_KEY = "15d6xxxxxxxxxxxxxxxxxx"
API_KEY = "efxxxxxxxxxxxxxxxxxxxx"
```
其中，`CODE_SIGN_IDENTITY` 为开发者证书标识，可以在 Keychain Access -> Certificates -> 选中证书右键弹出菜单 -> Get Info -> Common Name 获取，类似 `iPhone Distribution: Company name Co. Ltd (xxxxxxxx9A)`, 包括括号内的内容。

`PROVISIONING_PROFILE`: 这个是 mobileprovision 文件的 identifier，获取方式：

Xcode -> Preferences -> 选中申请开发者证书的 Apple ID -> 选中开发者证书 -> View Details… -> 根据 `Provisioning Profiles` 的名字选中打包所需的 mobileprovision 文件 -> 右键菜单 -> Show in Finder -> 找到该文件后，除了该文件后缀名的字符串就是 `PROVISIONING_PROFILE` 字段的内容。

当然也可以使用脚本获取, 此处参考 命令行获取[mobileprovision文件的UUID](https://my.oschina.net/ioslighter/blog/494342):

```
#!/bin/bash
if [ $# -ne 1 ]
then
  echo "Usage: getmobileuuid the-mobileprovision-file-path"
  exit 1
fi

mobileprovision_uuid=`/usr/libexec/PlistBuddy -c "Print UUID" /dev/stdin <<< $(/usr/bin/security cms -D -i $1)`
echo "UUID is:"
echo ${mobileprovision_uuid}
```
`USER_KEY`, `API_KEY`: 是蒲公英开放 API 的密钥。

将`autobuild.py`脚本放在你项目的根目录下，进入根目录，如下使用：

```
./autobuild.py -w yourname.xcworkspace -s schemename -o ~/Desktop/yourname.ipa
```
或者

```
./autobuild.py -p yourname.xcodeproj -t targetname -o ~/Desktop/yourname.ipa
```
该脚本可在[github](https://github.com/carya/Util)查看，如有任何问题，请留言回复。

常见问题:
1.找不到request module

```
import requests
ImportError: No module named requests
```
参考[stackoverflow](http://stackoverflow.com/questions/17309288/importerror-no-module-named-requests), 使用 `$ sudo pip install requests`或者`sudo easy_install -U requests`;

2.如果使用了上传蒲公英，且安装需要密码，请打开脚本，搜一下脚本里的password，将其值设置为空。














