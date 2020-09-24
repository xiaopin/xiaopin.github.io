---
title: iOS地铁图
tags:
  - iOS
abbrlink: 120b73eb
date: 2018-07-06 14:25:51
---

> 使用了高德地图的JS API，文档[戳这里](https://lbs.amap.com/api/subway-api/subway-summary/)

## 原理

通过 WKWebView 加载一个 HTML 字符串，通过注入 JS 在页面加载完毕后加载高德地图的JS库，然后创建地铁地图，同时通过JS库提供的 `getCityList` 接口获取所有的城市信息，然后把城市数据回传给 Objective-C，OC 在拿到数据后在右上角添加一个切换城市的按钮，当选择所要切换的城市后，通过 WKWebView 调用 JS 代码并把城市 id 传递给 JS，JS 拿到数据后通过 [`setAdcode`](https://lbs.amap.com/api/subway-api/reference) 接口来切换地铁数据。

实际上这个 Demo 中可以不考虑 `WKUIDelegate` 和 `WKNavigationDelegate` 这两个协议，但是为了熟悉 WKWebView，我还是遵守了这两个协议。

`XPWKScriptMessageHandlerObject` 这个类的作用只是为了解决 `WKScriptMessageHandler` 协议由于强引用代理对象从而导致内存泄漏的问题，起到一个中间代理人的作用。

## GIF演示效果

![GIF](/images/201807/subway-map.gif)


## 代码

- `.h` 头文件

```ObjC
NS_CLASS_AVAILABLE_IOS(9_0) @interface XPSubwayMapViewController : UIViewController

@end
```

- `.m` 源文件

```ObjC
#import "XPSubwayMapViewController.h"
#import <WebKit/WebKit.h>

#pragma mark -

@interface XPWKScriptMessageHandlerObject : NSObject<WKScriptMessageHandler>

@property (nonatomic, weak) id<WKScriptMessageHandler> scriptMessageHandler;

@end

@implementation XPWKScriptMessageHandlerObject

- (void)dealloc {
#ifdef DEBUG
    NSLog(@"%s", __FUNCTION__);
#endif
}

- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    [self.scriptMessageHandler userContentController:userContentController didReceiveScriptMessage:message];
}

@end

#pragma mark -

@interface XPSubwayMapViewController ()<WKUIDelegate, WKNavigationDelegate, WKScriptMessageHandler>
{
    UIActivityIndicatorView *indicatorView;
    WKWebView *webViwe;
    /// 城市列表数据(二维数组,内层数据中第一个元素为id号,第二个元素为城市名称)
    NSArray<NSArray<NSString*> *> *cities;
    XPWKScriptMessageHandlerObject *scriptMessageHandlerObject;
}
@end

@implementation XPSubwayMapViewController

#pragma mark Lifecycle

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.title = @"北京";
    scriptMessageHandlerObject = [[XPWKScriptMessageHandlerObject alloc] init];
    scriptMessageHandlerObject.scriptMessageHandler = self;
    
    WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
    [config.userContentController addUserScript:[self subwayJavaScript]];
    [config.userContentController addScriptMessageHandler:scriptMessageHandlerObject name:@"showToggleCityButton"];
    
    webViwe = [[WKWebView alloc] initWithFrame:CGRectZero configuration:config];
    webViwe.translatesAutoresizingMaskIntoConstraints = NO;
    webViwe.UIDelegate = self;
    webViwe.navigationDelegate = self;
    [self.view addSubview:webViwe];
    [webViwe.topAnchor constraintEqualToAnchor:self.view.topAnchor].active = YES;
    [webViwe.rightAnchor constraintEqualToAnchor:self.view.rightAnchor].active = YES;
    [webViwe.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor].active = YES;
    [webViwe.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor].active = YES;
    
    indicatorView = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:UIActivityIndicatorViewStyleGray];
    indicatorView.hidesWhenStopped = YES;
    indicatorView.translatesAutoresizingMaskIntoConstraints = NO;
    [self.view addSubview:indicatorView];
    [indicatorView.centerXAnchor constraintEqualToAnchor:self.view.centerXAnchor].active = YES;
    [indicatorView.centerYAnchor constraintEqualToAnchor:self.view.centerYAnchor].active = YES;
    
    [webViwe loadHTMLString:[self html] baseURL:nil];
}

- (void)dealloc {
    [webViwe.configuration.userContentController removeScriptMessageHandlerForName:@"showToggleCityButton"];
#ifdef DEBUG
    NSLog(@"%s", __FUNCTION__);
#endif
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

#pragma mark <WKNavigationDelegate>

/// 在发送请求之前调用,可以决定是否加载网页
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    decisionHandler(WKNavigationActionPolicyAllow);
}

/// 开始加载网页
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation {
    [indicatorView startAnimating];
}

/// 是否接收服务器返回的数据
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler {
    decisionHandler(WKNavigationResponsePolicyAllow);
}

/// 服务器返回内容时调用
- (void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation {
    
}

/// 页面加载完毕
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    [indicatorView stopAnimating];
}

/// 页面加载失败
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation withError:(NSError *)error {
}

/// HTTPS时触发该回调(可以在此验证HTTPS证书)
- (void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential * _Nullable))completionHandler {
    completionHandler(NSURLSessionAuthChallengePerformDefaultHandling, nil);
}

#pragma mark <WKUIDelegate>

/// 处理JavaScript原生的alert函数
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler {
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"" message:message preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *action = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        completionHandler();
    }];
    [alertController addAction:action];
    [self presentViewController:alertController animated:YES completion:nil];
}

/// 处理JavaScript原生的confirm函数
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL))completionHandler {
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"" message:message preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(NO);
    }];
    UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(YES);
    }];
    [alertController addAction:cancelAction];
    [alertController addAction:okAction];
    [self presentViewController:alertController animated:YES completion:nil];
}

/// 处理JavaScript原生的prompt函数
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable))completionHandler {
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"" message:prompt preferredStyle:UIAlertControllerStyleAlert];
    [alertController addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
        [textField setText:defaultText];
        [textField setClearButtonMode:UITextFieldViewModeWhileEditing];
        [textField setReturnKeyType:UIReturnKeyDone];
        //[textField setDelegate:self];
    }];
    [alertController addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(nil);
    }]];
    [alertController addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        NSString *text = [alertController.textFields firstObject].text;
        completionHandler(text);
    }]];
    [self presentViewController:alertController animated:YES completion:nil];
}

#pragma mark <WKScriptMessageHandler>

- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    if ([message.name isEqualToString:@"showToggleCityButton"]) {
        NSArray *cities = message.body;
        if ([cities isKindOfClass:[NSArray class]] && cities.count) {
            UIBarButtonItem *button = [[UIBarButtonItem alloc] initWithTitle:@"切换城市" style:UIBarButtonItemStylePlain target:self action:@selector(toggleCityButtonAction:)];
            self.navigationItem.rightBarButtonItem = button;
            self->cities = cities;
        }
    }
}

#pragma mark Actions

- (void)toggleCityButtonAction:(UIBarButtonItem *)sender {
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:nil message:nil preferredStyle:UIAlertControllerStyleActionSheet];
    [alert addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil]];
    __weak __typeof(self) weakSelf = self;
    for (NSArray<NSString*> *array in cities) {
        NSString *cityId = array.firstObject;
        NSString *cityName = array.lastObject;
        [alert addAction:[UIAlertAction actionWithTitle:cityName style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
            __strong __typeof(weakSelf) strongSelf = weakSelf;
            NSString *js = [NSString stringWithFormat:@"toggleCity(%@)", cityId];
            [strongSelf->webViwe evaluateJavaScript:js completionHandler:^(id _Nullable resp, NSError * _Nullable error) {
#ifdef DEBUG
                NSLog(@"Toggle city fail: %@", error);
#endif
            }];
            strongSelf.title = cityName;
        }]];
    }
    [self presentViewController:alert animated:YES completion:nil];
}

#pragma mark Private

- (NSString *)html {
    return @"<html><head><meta charset=\"UTF-8\"><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0,shrink-to-fit=no\"/></head><body><div id=\"SubwayContainer\"></div></body></html>";
}

- (WKUserScript *)subwayJavaScript {
    NSString *source = @"var script = document.createElement('script');script.setAttribute('src', 'https://webapi.amap.com/subway?v=1.0&key=98cb8314168440c4de1169ffac3a53e7&callback=cbk');document.body.appendChild(script);var mysubway = null;window.cbk = function(){mysubway = subway('SubwayContainer', {easy: 1});mysubway.getCityList(function(resp) {var maps = [];for (var id in resp) {var city = resp[id]['name'];maps.push([id, city]);}window.webkit.messageHandlers.showToggleCityButton.postMessage(maps);});};function toggleCity(id) {mysubway.setAdcode(id);};";
    return [[WKUserScript alloc] initWithSource:source injectionTime:WKUserScriptInjectionTimeAtDocumentEnd forMainFrameOnly:NO];
}

@end
```
