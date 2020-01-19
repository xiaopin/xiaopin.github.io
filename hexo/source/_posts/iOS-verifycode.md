---
title: iOS 验证码输入框
date: 2019-11-12 18:03:53
tags:
	- iOS
---

## 特性

- 支持自定义验证码长度(默认长度为6)
- 支持切换键盘类型，适应纯数字、数字+字母等组合(默认为系统数字键盘)
- 验证码输入完毕后自动收起键盘并通过block回调调用者

## 效果图

![](/images/2020/verify-code.jpg)

## 源码

```ObjC
NS_AVAILABLE_IOS(9_0)
@interface VCVerifyCodeInputView : UIView

/// 验证码长度,默认`6`
@property (nonatomic, assign) NSInteger length;
/// 键盘类型,默认`UIKeyboardTypeNumberPad`
@property (nonatomic, assign) UIKeyboardType keyboardType;
/// 验证码输入完毕后的回调
@property (nonatomic, copy) void(^didInputCompletionHandler)(NSString *verifycode);
/// 文本框高亮颜色,默认(30,144,255)
@property (nonatomic, strong) UIColor *focusColor;

/// 获取用户输入的验证码
@property (nonatomic, copy, readonly) NSString *verifycode;

@end
```

```ObjC
@interface VCVerifyCodeInputView ()<UITextFieldDelegate>

@end

@implementation VCVerifyCodeInputView
{
    UIStackView *stackView;
    UITextField *textField;
}

#pragma mark - Lifecycle

- (instancetype)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        [self setup];
    }
    return self;
}

- (instancetype)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super initWithCoder:aDecoder]) {
        [self setup];
    }
    return self;
}

- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UITextFieldTextDidChangeNotification object:textField];
}

#pragma mark - <UITextFieldDelegate>

- (void)textFieldDidBeginEditing:(UITextField *)textField {
    [self updateInputIndicator];
}

- (void)textFieldDidEndEditing:(UITextField *)textField {
    for (UIView *view in stackView.arrangedSubviews) {
        view.layer.borderColor = [[UIColor blackColor] CGColor];
    }
    
    if (textField.text.length == self.length && self.didInputCompletionHandler) {
        self.didInputCompletionHandler(textField.text);
    }
}

- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {
    if (range.length == 0) { // Add.
        if (textField.text.length == self.length) {
            return NO;
        }
        NSString *regexp = @"^[A-Za-z0-9]{1}$";
        NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", regexp];
        return [predicate evaluateWithObject:string];
    }
    return YES;
}

#pragma mark - Actions

- (void)textFieldTextDidChangeNotificationAction:(NSNotification *)sender {
    NSString *text = textField.text;
    [self updateInputIndicator];
    if (textField.isEditing && text.length == self.length) {
        // 收起键盘并回调block
        [textField resignFirstResponder];
    }
}

#pragma mark - Private

- (void)setup {
    stackView = [[UIStackView alloc] init];
    stackView.axis = UILayoutConstraintAxisHorizontal;
    stackView.distribution = UIStackViewDistributionFillEqually;
    stackView.spacing = 10.0;
    [self addSubviewToFill:stackView];
    
    textField = [[UITextField alloc] init];
    textField.borderStyle = UITextBorderStyleNone;
    textField.backgroundColor = [UIColor clearColor];
    textField.keyboardType = UIKeyboardTypeNumberPad;
    textField.autocorrectionType = UITextAutocorrectionTypeNo; // 禁用自动联想功能
    textField.tintColor = [UIColor clearColor]; // 清除光标颜色
    textField.textColor = [UIColor clearColor]; // 清除文字颜色
    textField.delegate = self;
    [self addSubviewToFill:textField];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textFieldTextDidChangeNotificationAction:) name:UITextFieldTextDidChangeNotification object:textField];
    
    [self setLength:6];
    self.focusColor = [UIColor colorWithRed:30/255.0 green:144/255.0 blue:255/255.0 alpha:1.0];
}

/// 更新输入指示器状态
- (void)updateInputIndicator {
    NSString *text = textField.text;
    for (NSInteger idx=0; idx<self.length; idx++) {
        UILabel *label = (UILabel *)stackView.arrangedSubviews[idx];
        if (text.length && idx < text.length) {
            label.text = [NSString stringWithFormat:@"%C", [text characterAtIndex:idx]];
        } else {
            label.text = nil;
        }
        if (idx == text.length) {
            label.layer.borderColor = [self.focusColor CGColor];
        } else {
            label.layer.borderColor = [UIColor blackColor].CGColor;
        }
    }
}

- (void)addSubviewToFill:(UIView *)subview {
    [self addSubview:subview];
    subview.translatesAutoresizingMaskIntoConstraints = NO;
    [subview.topAnchor constraintEqualToAnchor:self.topAnchor].active = YES;
    [subview.bottomAnchor constraintEqualToAnchor:self.bottomAnchor].active = YES;
    [subview.leadingAnchor constraintEqualToAnchor:self.leadingAnchor].active = YES;
    [subview.trailingAnchor constraintEqualToAnchor:self.trailingAnchor].active = YES;
}

#pragma mark - setter & getter

- (void)setLength:(NSInteger)length {
    [stackView.arrangedSubviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
    for (NSInteger idx=0; idx<length; idx++) {
        UILabel *label = [[UILabel alloc] init];
        label.textAlignment = NSTextAlignmentCenter;
        label.backgroundColor = [UIColor clearColor];
        label.layer.borderWidth = 1.0;
        label.layer.borderColor = [[UIColor blackColor] CGColor];
        label.layer.cornerRadius = 5.0;
        label.layer.masksToBounds = YES;
        [stackView addArrangedSubview:label];
    }
}

- (NSInteger)length {
    return stackView.arrangedSubviews.count;
}

- (void)setKeyboardType:(UIKeyboardType)keyboardType {
    textField.keyboardType = keyboardType;
}

- (UIKeyboardType)keyboardType {
    return textField.keyboardType;
}

- (NSString *)verifycode {
    return textField.text;
}

@end
```

## 使用示例

```ObjC
VCVerifyCodeInputView *verifyCodeView = [[VCVerifyCodeInputView alloc] init];
verifyCodeView.frame = CGRectMake(20.0, 100.0, 340.0, 40.0);
verifyCodeView.length = 8;
verifyCodeView.focusColor = [UIColor redColor];
[verifyCodeView setDidInputCompletionHandler:^(NSString *verifycode) {
    NSLog(@"输入的验证码是: %@", verifycode);
}];
[self.view addSubview:verifyCodeView];
```
