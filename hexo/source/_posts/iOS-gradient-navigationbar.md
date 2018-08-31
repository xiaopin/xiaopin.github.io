---
title: iOS 导航栏颜色渐变
date: 2018-08-31 18:04:07
tags:
	- iOS
---

```ObjC
@interface XPGradientNavigationBar : UINavigationBar
{
    CAGradientLayer *_gradientLayer;
}
@end

@implementation XPGradientNavigationBar

#pragma mark - Lifecycle

- (instancetype)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        [self commonInit];
    }
    return self;
}

- (void)awakeFromNib {
    [super awakeFromNib];
    [self commonInit];
}

- (void)layoutSubviews {
    [super layoutSubviews];

    if (!_gradientLayer.superlayer) {
        UIView *barBackgroundView = self.subviews.firstObject;
        if (@available(iOS 10.0, *)) {
            UIView *backgroundEffectView = [barBackgroundView valueForKey:@"_backgroundEffectView"];
            UIView *gradientView = backgroundEffectView.subviews.lastObject;
            [gradientView.layer addSublayer:_gradientLayer];
        } else {
            for (UIView *subview in barBackgroundView.subviews) {
                if ([subview isKindOfClass:NSClassFromString(@"_UIBackdropView")]) {
                    [subview.layer addSublayer:_gradientLayer];
                    break;
                }
            }
        }
    }

    if (_gradientLayer.superlayer) {
        CGRect frame = (CGRect){CGPointZero, self.frame.size};
        frame.size.height += self.frame.origin.y;
        _gradientLayer.frame = frame;
    }
}

#pragma mark - Private

- (void)commonInit {
    self.translucent = YES;
    _gradientLayer = [CAGradientLayer layer];
    _gradientLayer.colors = @[
                              (__bridge id)UIColor.purpleColor.CGColor,
                              (__bridge id)UIColor.orangeColor.CGColor
                              ];
    _gradientLayer.locations = @[@0.0, @1.0];
    _gradientLayer.startPoint = CGPointMake(0.0, 0.5);
    _gradientLayer.endPoint = CGPointMake(1.0, 0.5);
}

#pragma mark - setter & getter

- (void)setTranslucent:(BOOL)translucent {
    [super setTranslucent:YES];
}

@end
```
