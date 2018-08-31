---
title: iOS JSON解析换行符(\r、\n、\r\n)
date: 2018-08-27 16:54:39
tags:
	- iOS
---

项目需要对JSON返回的换行符进行解析，系统提供了解析JSON的方法`[NSJSONSerialization JSONObjectWithData:options:error:]`，
AFNetworking 内部也是采用该方法来解析服务器返回的内容。所以，我只需要 hook 该系统方法，当解析失败的时候进行处理就行了，这样就可以一劳永逸，也无需修改其他地方的代码。
 
服务器采用的是 Windows + C# 组合，通过后台添加的数据回车符为：`\r\n`，通过 Android 应用提交的回车符则是：`\n`，而 iOS 平台则是：`\r`，我统一将回车符替换成 `\r` ，进行转义后再次尝试解析JSON。


```ObjC
#import <objc/message.h>


@interface NSJSONSerialization (SpecialCharacters)

@end


@implementation NSJSONSerialization (SpecialCharacters)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method originalMethod = class_getClassMethod(self, @selector(JSONObjectWithData:options:error:));
        Method swizlledMethod = class_getClassMethod(self, @selector(sc_JSONObjectWithData:options:error:));
        method_exchangeImplementations(originalMethod, swizlledMethod);
    });
}

+ (id)sc_JSONObjectWithData:(NSData *)data options:(NSJSONReadingOptions)opt error:(NSError * _Nullable __autoreleasing *)error {
    if (!data.length) return nil;
    NSError *serializationError = nil;
    id responseObject = [self sc_JSONObjectWithData:data options:opt error:&serializationError];
    if (!responseObject) {
        if (error) {
            *error = serializationError;
        }
        
        if (serializationError && serializationError.code == 3840) {
            NSString *serializationString = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
            if (!serializationString) {
                return nil;
            }
            serializationString = [serializationString stringByReplacingOccurrencesOfString:@"(\\r\\n|\\r|\\n)" withString:@"\\\\r" options:NSRegularExpressionSearch range:NSMakeRange(0, serializationString.length)];
            
            NSData *serializationData = [serializationString dataUsingEncoding:NSUTF8StringEncoding];
            responseObject = [self sc_JSONObjectWithData:serializationData options:opt error:nil];
#ifndef DEBUG
            if (responseObject && error) {
                *error = nil;
            }
#endif
        }
    }
    
    return responseObject;
}

@end
```
