---
title: macOS's dark mode (暗黑模式)
tags:
  - macOS
abbrlink: 3bdea784
date: 2019-01-11 11:56:34
---

> 苹果在 macOS 10.14 中为我们带来了 `dark mode` (暗黑模式) 功能

可以在 `系统偏好设置`--`通用` 面板中开启暗黑模式

#### 判断当前主题是否为暗黑模式

```ObjC
- (BOOL)isDarkMode {
    if (@available(macOS 10.14, *)) {
        NSDictionary *dict = [[NSUserDefaults standardUserDefaults] persistentDomainForName:NSGlobalDomain];
        BOOL isDarkMode = [[dict objectForKey:@"AppleInterfaceStyle"] isEqualToString:@"Dark"];
        return isDarkMode;
    }
    return NO;
}
```

#### 监听主题变化

可以通过注册 `AppleInterfaceThemeChangedNotification` 这个通知事件来获知主题的变更

```ObjC
[NSDistributedNotificationCenter.defaultCenter addObserver:self selector:nil name:@"AppleInterfaceThemeChangedNotification" object: nil];
```

#### 为应用选择一种固定主题

系统默认已经为 NSView 自动适配了暗黑模式

我们可以为单个视图、整个应用指定固定的主题模式，而不跟随系统的主题变化。

- 单个视图

![](https://docs-assets.developer.apple.com/published/f98658e9ac/909bc4b8-02ac-4f1d-84ce-ed62aa25601f.png)

- 为整个应用指定一个主题

在 `-applicationDidFinishLaunching:` 方法中设置
```ObjC
// 默认主题
[NSApp setAppearance:[NSAppearance appearanceNamed:NSAppearanceNameAqua]];
// 暗黑模式
[NSApp setAppearance:[NSAppearance appearanceNamed:NSAppearanceNameDarkAqua]];
```

#### 参考

- [dark mode human interface guide](https://developer.apple.com/design/human-interface-guidelines/macos/visual-design/dark-mode/)
- [Supporting Dark Mode in Your Interface](https://developer.apple.com/documentation/appkit/supporting_dark_mode_in_your_interface?language=objc)
- [Choosing a Specific Appearance for Your App](https://developer.apple.com/documentation/appkit/nsappearancecustomization/choosing_a_specific_appearance_for_your_app?language=objc)
- [Providing Images for Different Appearances](https://developer.apple.com/documentation/appkit/images_and_pdf/providing_images_for_different_appearances?language=objc)
- [How can it be detected Dark Mode on macOS 10.14?](https://stackoverflow.com/questions/51672124/how-can-it-be-detected-dark-mode-on-macos-10-14)