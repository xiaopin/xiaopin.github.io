---
title: 十六进制字符串转UIColor
date: 2018-06-03 16:02:06
tags:
	- iOS
---

支持十六进制字符串的格式有：

- #FFF
- #FFFF
- #FFFFFF
- #FFFFFFFF
- 0xFFF
- 0xFFFF
- 0xFFFFFF
- 0xFFFFFFFF
- 可省略`#`、`0x`符号
- 不区分大小写


```ObjC
@interface UIColor (XP)

+ (instancetype)hexColor:(NSString *)hex;

@end
```

```ObjC
@implementation UIColor (XP)

+ (instancetype)hexColor:(NSString *)hex {
    NSString *str = hex;
    if ([hex hasPrefix:@"#"]) {
        str = [hex substringFromIndex:1];
    } else if ([hex.lowercaseString hasPrefix:@"0x"]) {
        str = [hex substringFromIndex:2];
    }
    
    NSScanner *scanner = [[NSScanner alloc] initWithString:str];
    unsigned int hexValue;
    if ([scanner scanHexInt:&hexValue]) {
        CGFloat r, g, b, a = 1.0;
        switch (str.length) {
            case 3:
                r = ((hexValue & 0xF00) >> 8)       / 15.0;
                g = ((hexValue & 0x0F0) >> 4)       / 15.0;
                b = (hexValue & 0x00F)              / 15.0;
            break;
            case 4:
                r = ((hexValue & 0xF000) >> 12)     / 15.0;
                g = ((hexValue & 0x0F00) >> 8)      / 15.0;
                b = ((hexValue & 0x00F0) >> 4)      / 15.0;
                a = (hexValue & 0x000F)             / 15.0;
            break;
            case 6:
                r = ((hexValue & 0xFF0000) >> 16)   / 255.0;
                g = ((hexValue & 0x00FF00) >> 8)    / 255.0;
                b = (hexValue & 0x0000FF)           / 255.0;
            break;
            case 8:
                r = ((hexValue & 0xFF000000) >> 24) / 255.0;
                g = ((hexValue & 0x00FF0000) >> 16) / 255.0;
                b = ((hexValue & 0x0000FF00) >> 8)  / 255.0;
                a = (hexValue & 0x000000FF)         / 255.0;
            break;
            default:
                return nil;
        }
        if (@available(iOS 10.0, *)) {
            return [UIColor colorWithDisplayP3Red:r green:g blue:b alpha:a];
        }
        return [UIColor colorWithRed:r green:g blue:b alpha:a];
    }
    return nil;
}

@end
```
