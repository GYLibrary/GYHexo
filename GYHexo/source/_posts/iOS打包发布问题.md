---
title: Xcode 8.0+自动打包并发布脚本
date: 2016-12-15 20:37:16
categories: 打包
tags: [Archive] 
description: 使用最新Xcode 8.0 打包发布遇到的问题收集。
---
# Xcode 8+ 打包发布问题
## 1.问题简介
> 
ERROR ITMS-90209: "Invalid Segment Alignment. The app binary at 'XXX.app/Frameworks/WebImage.framework/WebImage' does not have proper segment alignment. Try rebuilding the app with the latest Xcode version."
ERROR ITMS-90209: "Invalid Segment Alignment. The app binary at 'XXX.app/Frameworks/WebImage.framework/WebImage' does not have proper segment alignment. Try rebuilding the app with the latest Xcode version."
ERROR ITMS-90125: "The binary is invalid. The encryption info in the LC_ENCRYPTION_INFO load command is either missing or invalid, or the binary is already encrypted. This binary does not seem to have been built with Apple's linker."
ERROR ITMS-90125: "The binary is invalid. The encryption info in the LC_ENCRYPTION_INFO load command is either missing or invalid, or the binary is already encrypted. This binary does not seem to have been built with Apple's linker."

如图：
![1.png](http://upload-images.jianshu.io/upload_images/2082481-6408e609100223da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.png](http://upload-images.jianshu.io/upload_images/2082481-09038c7ef07aabe3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 解决方案：
> 项目->Build Phases-> +New Run Script -> 添加下方脚本

APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"

# This script loops through the frameworks embedded in the application and
# removes unused architectures.
find "$APP_PATH" -name '*.framework' -type d | while read -r FRAMEWORK
do
FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"

EXTRACTED_ARCHS=()

for ARCH in $ARCHS
do
echo "Extracting $ARCH from $FRAMEWORK_EXECUTABLE_NAME"
lipo -extract "$ARCH" "$FRAMEWORK_EXECUTABLE_PATH" -o "$FRAMEWORK_EXECUTABLE_PATH-$ARCH"
EXTRACTED_ARCHS+=("$FRAMEWORK_EXECUTABLE_PATH-$ARCH")
done

echo "Merging extracted architectures: ${ARCHS}"
lipo -o "$FRAMEWORK_EXECUTABLE_PATH-merged" -create "${EXTRACTED_ARCHS[@]}"
rm "${EXTRACTED_ARCHS[@]}"

echo "Replacing original executable with thinned version"
rm "$FRAMEWORK_EXECUTABLE_PATH"
mv "$FRAMEWORK_EXECUTABLE_PATH-merged" "$FRAMEWORK_EXECUTABLE_PATH"

done

![3.png](http://upload-images.jianshu.io/upload_images/2082481-db27f9533f68e919.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.问题简介

> 打包上传到iTunes显示Upload Success 之后，邮箱收到一封驳回邮件，大概就是一些原因显示不通过之类的。

>> Missing Info.plist key - This app attempts to access privacy-sensitive data without a usage description. The app's Info.plist must contain an NSContactsUsageDescription key with a string value explaining to the user how the app uses this data.
Missing Info.plist key - This app attempts to access privacy-sensitive data without a usage description. The app's Info.plist must contain an NSPhotoLibraryUsageDescription key with a string value explaining to the user how the app uses this data.

这是因为Xcode8.0之后要在info.plist中添加一些权限描述,必须有文字描述，而不是只添加字段就可以的。

![4.png](http://upload-images.jianshu.io/upload_images/2082481-40dd7940f8f75fad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)










