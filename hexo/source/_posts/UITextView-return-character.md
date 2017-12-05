---
title: UITextView禁用回车符
date: 2017-11-17 16:40:36
tags:
	- iOS
---

最近项目有个需求，需要在UITextView中禁止用户输入回车符，因为后台不会过滤回车符号，所以当AFNetworking请求数据时，如果返回的数据中包含了回车符，那么JSON解析将会失败。既然后台不处理，那没办法，只能在UITextView上禁用回车符了。


我们需要解决以下问题：

1. 禁止通过键盘的`return`键输入回车符
2. 粘贴文本时过滤掉文本中的回车符


对于问题1，我们可以通过重写`- (void)insertText:(NSString *)text`方法，判断text是否为`\n`即可；
对于问题2，当用户点击Paste按钮时，系统会调用UITextView的`- (void)paste:(id)sender`方法，我们可以从此处下手。


下面附上示例代码

```ObjC
@interface XPTextView : UITextView

/// 是否禁用回车符
@property (nonatomic, assign, getter=isDisableReturnCharacter) BOOL disableReturnCharacter;

@end


@implementation XPTextView

- (void)insertText:(NSString *)text {
    if (_disableReturnCharacter && [text isEqualToString:@"\n"]) {
        // 禁止输入回车符
        [self resignFirstResponder];
        return;
    }
    [super insertText:text];
}

- (void)paste:(id)sender {
    if (_disableReturnCharacter) {
        NSString *text = [[UIPasteboard generalPasteboard] string];
        if ([text containsString:@"\n"]) {
            // 需要从粘贴的字符串中把回车符过滤掉
            text = [[text componentsSeparatedByString:@"\n"] componentsJoinedByString:@""];
            [self replaceRange:self.selectedTextRange withText:text];
            return;
        }
    }
    [super paste:sender];
}

@end
```


下面安利一下我写的`XPTextView`，不仅可以禁用回车符，还可以设置占位字符：

[👉源码戳这里👈](https://github.com/xiaopin/XPTextView.git)
