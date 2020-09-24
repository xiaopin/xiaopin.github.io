---
title: iOS禁用第三方输入法
tags:
  - iOS
abbrlink: 478350d8
date: 2018-10-26 15:28:55
---

当我们需要用户输入数字时，我们会将键盘类型设置为 `UIKeyboardTypeNumberPad` 、 `UIKeyboardTypePhonePad` 或 `UIKeyboardTypeDecimalPad` 等数字类型的键盘。但是如果用户使用了第三方输入法（比如搜狗），那就无法达到强制输入数字的目的了，因为搜狗输入法在这些 UIKeyboardType 枚举类型中，除了常规的 0～9 数字键外，还额外带了其他符号键，用户可以输入一些特殊字符，此时如果想要达到效果，就只能通过重写遵守 `UITextFieldDelegate` 并实现 `- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string` 方法，在这里通过正则判断用户输入的是否为数字。

如果不想这么麻烦，那就干脆直接禁用第三方输入法吧 :)

```ObjC
- (BOOL)application:(UIApplication *)application shouldAllowExtensionPointIdentifier:(UIApplicationExtensionPointIdentifier)extensionPointIdentifier {
    if ([extensionPointIdentifier isEqualToString:UIApplicationKeyboardExtensionPointIdentifier]) {
        return NO;
    }
    return YES;
}
```

PS：如果 `secureTextEntry` 属性为 YES，那么将会强制使用系统键盘，不管你有没有禁用第三方输入法，苹果这是为了安全考虑。

- [https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/CustomKeyboard.html](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/CustomKeyboard.html)