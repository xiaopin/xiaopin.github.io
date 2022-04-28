---
title: WKWebView 与 JavaScript 进行通讯
abbrlink: 3f85097b
date: 2022-01-19 10:21:17
tags:
  - iOS
---

> 2020年4月起App Store将不再接受使用UIWebView的新App上架、2020年12月起将不再接受使用UIWebView的App更新。苹果已经彻底抛弃了UIWebView，所以我们不再讨论关于UIWebView的问题。

苹果从 iOS8.0 为我们带来了全新的 [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview)，WKWebView 为我们和 JavaScript 的交互带来了不一样的体验。

## WKNavigationDelegate 协议拦截

- 首先遵守 [WKNavigationDelegate](https://developer.apple.com/documentation/webkit/wknavigationdelegate/) 协议
- 实现 `webView:decidePolicyForNavigationAction:decisionHandler:` 方法, 匹配对应的URL

示例代码：

index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JavaScript & Objective-C</title>
    <style>
        body { padding-top: 100px; }
    </style>
</head>
<body>
    <button onclick="onLogin()">Login</button>
    <div>OC Response：<span id="resp"><span></div>
</body>
    <script>
        function onLogin() {
            window.location.href = 'login://';
        }
        
        function onLoginResponse(data) {
            document.getElementById('resp').innerHTML = JSON.stringify(data);
            return 'Login Success!!!';
        }
    </script>
</html>
```

```ObjC
@interface ViewController ()<WKNavigationDelegate>

@property (nonatomic, strong) WKWebView *webView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.webView = [[WKWebView alloc] initWithFrame:self.view.bounds];
    self.webView.navigationDelegate = self; // 设置代理
    [self.view addSubview:self.webView];
    
    NSString *path = [[NSBundle mainBundle] pathForResource:@"index.html" ofType:nil];
    NSURL *URL = [NSURL fileURLWithPath:path];
    NSURLRequest *request = [NSURLRequest requestWithURL:URL];
    [self.webView loadRequest:request];
}

- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    NSURL *URL = navigationAction.request.URL;
    // 匹配到相应的协议, 则做相应的事情, 并且取消此次请求即可
    if ([URL.scheme isEqualToString:@"login"]) {
        NSString *user = @"{\"name\":\"xiaoming\",\"age\":18}";
        NSString *jsCode = [NSString stringWithFormat:@"onLoginResponse(%@)", user];
        [self.webView evaluateJavaScript:jsCode completionHandler:^(id _Nullable data, NSError * _Nullable error) {
            NSLog(@"%@", data);
        }];
        // 取消此次请求
        decisionHandler(WKNavigationActionPolicyCancel);
        return;
    }
    decisionHandler(WKNavigationActionPolicyAllow);
}

@end
```

## WKScriptMessageHandler

- 首先通过 [`addScriptMessageHandler:name:`](https://developer.apple.com/documentation/webkit/wkusercontentcontroller/1537172-addscriptmessagehandler/) 注册多个消息处理程序（可以理解为注册多个方法供 JavaScript 进行调用）
- 在 JavaScript 端则通过 `window.webkit.messageHandlers.<name>.postMessage(<messageBody>)` 来调用 Objective-C 方法并传递参数
- 在 Objective-C 通过 `evaluateJavaScript:completionHandler:` 来调用 JavaScript 代码并接收返回值

示例代码：

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JavaScript & Objective-C</title>
    <style>
        body { padding-top: 100px; }
    </style>
</head>
<body>
    <button onclick="onLogin()">Call OC</button>
    <div>OC Response：<span id="resp"><span></div>
</body>
    <script>
        // 调用OC方法
        function onLogin() {
            // postMessage 参数不能省略, 如果不想传递任何数据则使用 null
            // 如果省略了参数, 则 iOS 将接收不到消息的调用
            window.webkit.messageHandlers.Login.postMessage(null);
        }

        // 接收OC返回的数据
        function onLoginResponse(data) {
            document.getElementById('resp').innerHTML = JSON.stringify(data);
            // 这里可以通过 return 返回数据给 iOS
            return 'Login Success!!!';
        }
    </script>
</html>
```

Objective-C 代码

```ObjC
@interface ViewController ()<WKScriptMessageHandler>

@property (nonatomic, strong) WKWebView *webView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.webView = [[WKWebView alloc] initWithFrame:self.view.bounds];
    [self.view addSubview:self.webView];
    
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"Login"];
    
    NSString *path = [[NSBundle mainBundle] pathForResource:@"index.html" ofType:nil];
    NSURL *URL = [NSURL fileURLWithPath:path];
    NSURLRequest *request = [NSURLRequest requestWithURL:URL];
    [self.webView loadRequest:request];
}

- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    if ([message.name isEqualToString:@"Login"]) {
        NSString *user = @"{\"name\":\"xiaoming\",\"age\":18}";
        NSString *jsCode = [NSString stringWithFormat:@"onLoginResponse(%@)", user];
        [self.webView evaluateJavaScript:jsCode completionHandler:^(id _Nullable data, NSError * _Nullable error) {
            NSLog(@"%@", data);
        }];
    }
}

- (void)dealloc {
    [self.webView.configuration.userContentController removeAllScriptMessageHandlers];
}

@end
```

- WKScriptMessageHandler循环引用

当我们调用了 `addScriptMessageHandler:name:` 接口后，发现控制器无法销毁了，由于循环引用造成了内存泄漏。这里采用 NSProxy 方案进行解决。

```ObjC
/// WKScriptMessageHandler代理类
/// 解决由于 `addScriptMessageHandler:name:` 接口导致的循环引用问题
///
/// 用法：
/// id<WKScriptMessageHandler> proxy = (id<WKScriptMessageHandler>)[WKScriptMessageHandlerProxy proxyWithTarget:self];
/// [self.webView.configuration.userContentController addScriptMessageHandler:proxy name:@"fn"];
@interface WKScriptMessageHandlerProxy : NSProxy

+ (instancetype)proxyWithTarget:(id)target;

@end

@implementation WKScriptMessageHandlerProxy
{
    __weak id _target;
}

+ (instancetype)proxyWithTarget:(id)target {
    WKScriptMessageHandlerProxy *proxy = [WKScriptMessageHandlerProxy alloc];
    proxy->_target = target;
    return proxy;
}

- (BOOL)respondsToSelector:(SEL)aSelector {
    return [_target respondsToSelector:aSelector];
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
    return [_target methodSignatureForSelector:sel];
}

- (void)forwardInvocation:(NSInvocation *)invocation {
    if ([_target respondsToSelector:invocation.selector]) {
        [invocation invokeWithTarget:_target];
    }
}

@end
```

## WebViewJavascriptBridge

[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge) 算是最受欢迎的OC与JS桥接的框架了，不过该框架已经几年不维护了。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JavaScript & Objective-C</title>
    <style>
        body { padding-top: 100px; }
    </style>
</head>
<body>
    <button onclick="onLogin()">Call OC</button>
    <div>OC Response：<span id="resp"><span></div>
</body>
    <script>
        function setupWebViewJavascriptBridge(callback) {
            if (window.WebViewJavascriptBridge) { return callback(WebViewJavascriptBridge); }
            if (window.WVJBCallbacks) { return window.WVJBCallbacks.push(callback); }
            window.WVJBCallbacks = [callback];
            var WVJBIframe = document.createElement('iframe');
            WVJBIframe.style.display = 'none';
            WVJBIframe.src = 'https://__bridge_loaded__';
            document.documentElement.appendChild(WVJBIframe);
            setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
        }
        
        setupWebViewJavascriptBridge(function(bridge) {
            // 初始化完毕, 在这里注册方法供 OC 调用
            bridge.registerHandler('onResponse', function(data, responseCallback) {});
        });
        
        function onLogin() {
            // 调用 OC 方法, 并直接在回调函数中接收 OC 返回的数据
            WebViewJavascriptBridge.callHandler('Login', {}, function(data) {
                document.getElementById('resp').innerHTML = JSON.stringify(data);
            });
        }
    </script>
</html>
```

```ObjC
@interface ViewController ()<WKScriptMessageHandler>

@property (nonatomic, strong) WKWebView *webView;
@property (nonatomic, strong) WKWebViewJavascriptBridge *bridge;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.webView = [[WKWebView alloc] initWithFrame:self.view.bounds];
    [self.view addSubview:self.webView];
    
    NSString *path = [[NSBundle mainBundle] pathForResource:@"index.html" ofType:nil];
    NSURL *URL = [NSURL fileURLWithPath:path];
    NSURLRequest *request = [NSURLRequest requestWithURL:URL];
    [self.webView loadRequest:request];
    
    self.bridge = [WKWebViewJavascriptBridge bridgeForWebView:self.webView];
    [self.bridge registerHandler:@"Login" handler:^(id data, WVJBResponseCallback responseCallback) {
        NSDictionary *user = @{@"name": @"xiaoming", @"age": @18};
        responseCallback(user);
    }];
}

@end
```

- [Replacing UIWebView in Your App](https://developer.apple.com/documentation/webkit/replacing_uiwebview_in_your_app)