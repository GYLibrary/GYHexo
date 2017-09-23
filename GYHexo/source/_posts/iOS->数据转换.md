---
title: iOS - 数据转化
date: 2015-12-12 20:35:16
categories: iOS
tags: [iOS] 
description: iOS相关的数据转换，NSData与Bytes等。
---

# iOS 数据转化

## NSData 与 NSString相互转化

###     NSString -> NSData
```
NSString *str = @"你最帅";
NSData *data = [str dataUsingEncoding:NSUTF8StringEncoding];
```
###     NSData -> NSString
```
NSString *str  = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
```

## NSData 与 Byte
```
Byte *byte = (Byte *)[data bytes];
NSData *data = [[NSData alloc] initWithBytes:byte length:10];
```

## Byte数组 -> 16进制数
```
    Byte byte[] = {0,1,2};//000102

    NSString *hexStr = @"";
    
    for (int i = 0; i < 3 ; i++) {
        
        NSString *newHexStr = [NSString stringWithFormat:@"%x",byte[i] & 0xff];
        if ([newHexStr length] == 1)
            hexStr = [NSString stringWithFormat:@"%@0%@",hexStr,newHexStr];
        else
            hexStr = [NSString stringWithFormat:@"%@%@",hexStr,newHexStr];
        
    }
    
```

## 16进制数 -> Byte数组

```
int j=0;
Byte bytes[hexString.length / 2];  
for(int i=0;i<[hexStr length];i++)
{
    int int_ch;  /// 两位16进制数转化后的10进制数
    unichar hex_char1 = [hexStr characterAtIndex:i]; ////两位16进制数中的第一位(高位*16)
    int int_ch1;
    if(hex_char1 >= '0' && hex_char1 <='9')
    {
        int_ch1 = (hex_char1-48)*16;   //// 0 的Ascll - 48
    }
    else if(hex_char1 >= 'A' && hex_char1 <='F')
    {
        int_ch1 = (hex_char1-55)*16; //// A 的Ascll - 65
    }
    else
    {
        int_ch1 = (hex_char1-87)*16; //// a 的Ascll - 97
    }
    i++;
    unichar hex_char2 = [hexStr characterAtIndex:i]; ///两位16进制数中的第二位(低位)
    int int_ch2;
    if(hex_char2 >= '0' && hex_char2 <='9')
    {
        int_ch2 = (hex_char2-48); //// 0 的Ascll - 48
    }
    else if(hex_char1 >= 'A' && hex_char1 <='F')
    {
        int_ch2 = hex_char2-55; //// A 的Ascll - 65
    }
    else   
    {
        int_ch2 = hex_char2-87; //// a 的Ascll - 97  
    }
    int_ch = int_ch1+int_ch2;
    NSLog(@"int_ch=%d",int_ch);
    bytes[j] = int_ch;  ///将转化后的数放入Byte数组里
    j++;
  }
```









