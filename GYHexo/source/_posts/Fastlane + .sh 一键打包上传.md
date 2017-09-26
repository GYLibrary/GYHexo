---
title: Fastlane + .sh 一键打包上传
date:
categories: iOS持续集成
tags: [iOS持续集成] 
description: 记录。
---
# Fastlane + .sh 一键打包上传

## Fastlane的安装、使用

1. 首先检查是否已经安装最新版的 Xcode 命令行工具，运行 xcode-select --install 命令，根据你的情况进行不同处理。
2. 安装 fastlane ，这里使用 brew cask 管理工具对 fastlane 进行安装

   ```
   $ brew cask install fastlane
   ```
3. 如果没有安装 brew cask ，运行上面命令会先安装 brew cask 工具

   ```
   $ Creating Caskroom at /usr/local/Caskroom
   ```
4. 重新安装

   ```
   $ brew cask reinstall fastlane
   ```
5. 在项目目录下，对项目进行初始化。

	```
	$ fastlane init //在 init 过程中会出现 登录开发者账号，输入密码和选中 Targets 类似的交互
	```
6.重新安装 
	```
	$ brew cask reinstall fastlane
	```
7.卸载和安装相对应

  ```
  $ brew cask uninstall fastlane
  ```

    


### 一键自动化打包并上传到fir.m
	
1.安装fir-cil
```
 gem install fir-cli
```
2.登录[fir.im](https://fir.im/)
登录指令非常简单，只需要fir login API Token，这里面的API Token就是你fir帐号下的API Token，然后通过fir me查看你是否登录成功.

```
终端登录:fir login API Token
```
	
	
```
脚本:
#!/bin/bash
#设置超时 export 
FASTLANE_XCODEBUILD_SETTINGS_TIMEOUT=120 

#计时 
SECONDS=0 

#假设脚本放置在与项目相同的路径下 
project_path=$(pwd) 

#取当前时间字符串添加到文件结尾 
now=$(date +"%Y_%m_%d_%H_%M_%S") 

#指定项目的scheme名称 
scheme="Demo" 

#指定要打包的配置名 
configuration="Adhoc" 

#指定打包所使用的输出方式，目前支持app-store, package, ad-hoc, enterprise, development, 和developer-id，即xcodebuild的method参数 
export_method='ad-hoc' 

#指定项目地址 
workspace_path="$project_path/Demo.xcworkspace" 

#指定输出路径 
output_path="$project_path/IPA" 

#指定输出归档文件地址 
archive_path="$output_path/Demo_${now}.xcarchive"
 
#指定输出ipa地址 
ipa_path="$output_path/Demo_${now}.ipa"

#指定输出ipa名称 
ipa_name="Demo_${now}.ipa" 

#获取执行命令时的commit message
 commit_msg="$1" 
 
#输出设定的变量值 
echo "===workspace path: ${workspace_path}===" 
echo "===archive path: ${archive_path}===" 
echo "===ipa path: ${ipa_path}===" 
echo "===export method: ${export_method}===" 
echo "===commit msg: $1===" 

#先清空前一次build 
fastlane gym --workspace ${workspace_path} --scheme ${scheme} --clean --configuration ${configuration} --archive_path ${archive_path} --export_method ${export_method} --output_directory ${output_path} --output_name ${ipa_name} 

#上传到fir 
fir publish ${ipa_path} -T "XXX_YOUR_API_TOKEN_XXX" -c "${commit_msg}" 

#输出总用时 
echo "===Finished. Total time: ${SECONDS}s==="

```








