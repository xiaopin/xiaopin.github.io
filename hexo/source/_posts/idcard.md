---
title: 身份证号码那点事
abbrlink: c5db6973
date: 2022-05-27 11:07:07
tags:
    - Other
    - JavaScript
    - Objective-C
---



## 身份证号码的组成格式

直接附上人民日报关于身份证号码格式的说明图片（一图胜过千言万语有木有）：

![idcard.png](/images/2022/idcard.jpg)

## 校验码的计算规则

- 前面17位数分别乘以一个系数并将结果累加(每位数对应的系统为 `7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2`)
- 将累加结果除以11得到余数(取余)
- 余数的结果只能是 `0 1 2 3 4 5 6 7 8 9 10`, 对应身份证最后一位为 `1 0 X 9 8 7 6 5 4 3 2`

因为余数为 `10` 的话, 如果直接用 10 补全则会出现 19 位的长度, 所以用 `X` 进行代替。

假设一个身份证号码为 `451222199802174569`, 则校验码的计算过程如下: 

- 计算和：4×7 + 5×9 + 1×10 + 2×5 + 2×8 + 2×4 + 1×2 + 9×1 + 9×6 + 8×3 + 0×7 + 2×9 + 1×10 + 7×5 + 4×8 + 5×4 + 6×2 = 333
- 取余：333 ÷ 11 = 30...3 (即余数为3)
- 余数 3 对应的校验码是 9

## 15位身份证号码

15位的身份证号码是用在第一代居民身份证上的，2004年1月1日，第二代居民身份证开始换发，第一代居民身份证已经于2013年1月1日正式退出历史舞台。

### 组成格式

组成格式
- 1~6位作为首次登记户口时的地区编码(参考18位身份证号码)
- 7~8位为出生年份(比如34表示1934年)
- 9~10位为出生月份
- 11~12位为出生日期
- 13～14位参考18位身份证, 应该是所在地的派出所代码, 也有说是出生顺序
- 其中第15位用来表示性别

> 说白了15位身份证号码就是18位身份证号码的出生年份去掉前两位和去掉最后一位校验码。

### 转换为18位

既然我们明白了15位和18位的区别，那么我们就能将15位转换升级为18位：
- 出生年份补上`19`
- 计算出最后一位校验码

## 上代码

既然已经了解了身份证号码的格式，那么我们就可以根据规则来做相关的功能了，下面代码提供以下

- 判断身份证是否合法
- 获取性别
- 获取年龄
- 15位转18位

### TypeScript

```JavaScript
export interface IdcardBirthday {
    /** 出生年月日(yyyyMMdd) */
    date: string
    /** 年(yyyy) */
    year: string
    /** 月(MM) */
    month: string
    /** 日(dd) */
    day: string
}

// 前17位的加权因子
const FACTOR_ARRAY = [7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2]
// 除以11后可能产生的11位验证码(10需要改成X)
const CHECK_CODE_ARRAY = ['1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2']

/** 身份证工具类 */
export default class IdcardUtil {
    /**
     * 校验给定的字符串是否为合法的身份证号码
     *
     * @param idcard 待校验的身份证号码
     */
    static isValidIdcard(idcard: string): boolean {
        if (typeof idcard !== 'string' || !/^(\d{17}[\dXx]|\d{15})$/.test(idcard)) return false
        // 校验省份
        if (!/^(1[1-5]|2[1-3]|3[1-7]|4[1-6]|5[0-4]|6[1-5]|71|8[12])/.test(idcard)) return false
        // 校验出生年月日
        const birthday = idcard.length == 15 ? `19${idcard.substring(6, 12)}` : idcard.substring(6, 14)
        const year = birthday.substring(0, 4)
        const month = birthday.substring(4, 6)
        const day = birthday.substring(6, 8)
        if (!/^(19|2)/.test(year) || !/^(0[1-9]|1[012])$/.test(month) || !/^(0[1-9])|([12][0-9])|(30|31)$/.test(day)) return false
        const date = new Date(parseInt(year), parseInt(month) - 1, parseInt(day))
        if (date.getMonth() !== parseInt(month) - 1 || date.getDate() !== parseInt(day)) return false
        // 校验最后一位校验码
        if (idcard.length == 18) {
            // 用来保存前17位各自乖以加权因子后的总和
            let factorSum = 0
            for (let i = 0; i < 17; i++) {
                factorSum += parseInt(idcard.substring(i, i + 1)) * FACTOR_ARRAY[i]
            }
            const checkCodeIndex = factorSum % 11
            const checkCode = CHECK_CODE_ARRAY[checkCodeIndex]
            const idcardLastChar = idcard.substring(17).toUpperCase()
            if (idcardLastChar !== checkCode) return false
        }
        return true
    }

    /**
     * 根据身份证号码获取出生日期信息
     *
     * @param idcard 身份证号码
     * @returns 身份证号码不正确时返回 undefined
     */
    static birthday(idcard: string): IdcardBirthday | undefined {
        if (!this.isValidIdcard(idcard)) return undefined
        const date = idcard.length == 18 ? idcard.substring(6, 14) : `19${idcard.substring(6, 12)}`
        const year = date.substring(0, 4)
        const month = date.substring(4, 6)
        const day = date.substring(6, 8)
        return { date, year, month, day }
    }

    /**
     * 根据身份证号码获取性别
     *
     * @param idcard 身份证号码
     * @returns 0:女 1:男 -1:身份证号码不正确
     */
    static gender(idcard: string): -1 | 0 | 1 {
        // 18位长度身份证根据第17位数判断, 15位长度身份证则根据最后一位判断
        // 奇数(1/3/5/7/9)表示男性, 偶数(0/2/4/6/8)表示女性
        if (!this.isValidIdcard(idcard)) return -1
        const identifier = idcard.length == 18 ? idcard.substring(16, 17) : idcard.substring(14)
        return parseInt(identifier) % 2 == 0 ? 0 : 1
    }

    /**
     * 根据身份证计算实际年龄(精确到天, 必须到了生日当天才算满一岁)
     *
     * @param idcard 身份证号码
     * @param now 需要进行计算年龄的时间(默认为当前时间)
     * @returns 实际年龄, -1表示身份证号码不正确导致计算错误
     */
    static age(idcard: string, now: Date = new Date()): number {
        const INVALID_VALUE = -1
        const birthday = this.birthday(idcard)
        if (birthday === undefined) return INVALID_VALUE
        const { year, month, day } = birthday
        const birthDate = new Date(parseInt(year), parseInt(month) - 1, parseInt(day))
        if (birthDate.getTime() > now.getTime()) return INVALID_VALUE // 出生日期比判断日期都大
        let ret = now.getFullYear() - birthDate.getFullYear()
        if (now.getMonth() == birthDate.getMonth()) {
            if (now.getDate() < birthDate.getDate()) {
                // 月份相同, 但是未到出生那一天, 不满一岁
                ret = Math.max(0, ret - 1)
            }
        } else if (now.getMonth() < birthDate.getMonth()) {
            // 当前月份小于出生日期月份, 不满一岁
            ret = Math.max(0, ret - 1)
        }
        return ret
    }

    /**
     * 将15位长度身份证号码转成18位长度的身份证号码
     *
     * @param idcard 15位长度的身份证号码
     * @returns 转换成功时返回18位长度的新身份证号码, 转换失败时返回 undefined
     */
    static convert15To18(idcard: string): string | undefined {
        if (!/^\d{15}$/.test(idcard)) return undefined
        // 补全年份
        let ret = idcard.substring(0, 6) + '19' + idcard.substring(6)
        // 计算最后一位校验码并补全
        let sum = 0
        for (let i = 0; i < 17; i++) {
            sum += parseInt(ret.substring(i, i + 1)) * FACTOR_ARRAY[i]
        }
        ret += CHECK_CODE_ARRAY[sum % 11]
        return ret
    }
}
```

### JavaScript

参考上面 TypeScript 的代码，把 TS 相关的类型移除即可；强烈推荐采用 TS，代码提示和数据类型校验等功能真的是越用越上头。

### Objective-C

- IdcardUtil.h

```Objective-C
#import <CoreFoundation/CoreFoundation.h>

typedef NS_ENUM(NSInteger, IdcardGender) {
    IdcardGenderUnknown = -1,// 未知, 保密
    IdcardGenderFemale  = 0, // 女
    IdcardGenderMale    = 1, // 男
};

@interface IdcardUtil : NSObject

/// 校验给定的字符串是否为合法的身份证号码
/// @param idcard 待校验的身份证号码
+ (BOOL)isValidIdcard:(NSString *)idcard;

/// 根据身份证计算实际年龄(精确到天, 必须到了生日当天才算满一岁)
/// @param idcard 身份证号码
+ (int)ageWithIdcard:(NSString *)idcard;

/// 根据身份证计算实际年龄(精确到天, 必须到了生日当天才算满一岁)
/// @param idcard 身份证号码
/// @param now 需要进行计算年龄的时间(默认为当前时间)
+ (int)ageWithIdcard:(NSString *)idcard compareDate:(NSDate *)now;

/// 根据身份证号码获取性别
/// @param idcard 身份证号码
+ (IdcardGender)genderWithIdcard:(NSString *)idcard;

/// 获取身份证的出生年月日(yyyyMMdd)
/// @param idcard 身份证号码
+ (NSString *)birthdayWithIdcard:(NSString *)idcard;

/// 将15位长度身份证号码转成18位长度的身份证号码
/// @param idcard 15位长度的身份证号码
+ (NSString *)conver15To18:(NSString *)idcard;

@end
```

- IdcardUtil.m

```Objective-C
#import "IdcardUtil.h"

@implementation IdcardUtil

+ (BOOL)isValidIdcard:(NSString *)idcard {
    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES[cd] %@", @"^(\\d{17}[\\dX]|\\d{15})$"];
    if (![predicate evaluateWithObject:idcard]) return NO;
    // 校验省份
    predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", @"^(1[1-5]|2[1-3]|3[1-7]|4[1-6]|5[0-4]|6[1-5]|71|8[12]).*"];
    if (![predicate evaluateWithObject:idcard]) return NO;
    // 校验出生日期
    NSString *birthday = idcard.length == 18
                        ? [idcard substringWithRange:NSMakeRange(6, 8)]
                        : [@"19" stringByAppendingString:[idcard substringWithRange:NSMakeRange(6, 6)]];
    predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", @"^(19\\d{2}|2\\d{3})(0[1-9]|1[012])((0[1-9])|([12][0-9])|(30|31))$"];
    if (![predicate evaluateWithObject:birthday]) return NO;
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    dateFormatter.timeZone = [NSTimeZone timeZoneWithAbbreviation:@"UTC"];
    dateFormatter.dateFormat = @"yyyyMMdd";
    if (nil == [dateFormatter dateFromString:birthday]) return NO;
    // 校验最后一位校验码
    if (idcard.length == 18) {
        int factorArray[] = { 7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2 };
        NSArray<NSString *> *checkCodeArray = @[@"1", @"0", @"X", @"9", @"8", @"7", @"6", @"5", @"4", @"3", @"2"];
        int sum = 0;
        for (int i=0; i<17; i++) {
            NSString *character = [idcard substringWithRange:NSMakeRange(i, 1)];
            sum += [character intValue] * factorArray[i];
        }
        NSString *code = checkCodeArray[sum % 11];
        NSString *idcardLastChar = [[idcard substringFromIndex:idcard.length-1] uppercaseString];
        if (![idcardLastChar isEqualToString:code]) return NO;
    }
    return YES;
}

+ (int)ageWithIdcard:(NSString *)idcard {
    return [self ageWithIdcard:idcard compareDate:nil];
}

+ (int)ageWithIdcard:(NSString *)idcard compareDate:(NSDate *)now {
    if (![self isValidIdcard:idcard]) return -1;
    if (now == nil) now = [NSDate date];
    if (idcard.length == 15) {
        idcard = [self conver15To18:idcard];
    }
    NSString *birthday = [idcard substringWithRange:NSMakeRange(6, 8)];
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    dateFormatter.timeZone = [NSTimeZone timeZoneWithAbbreviation:@"UTC"];
    dateFormatter.dateFormat = @"yyyyMMdd";
    NSDate *date = [dateFormatter dateFromString:birthday];
    if (nil == date) return -1;
    NSCalendar *calendar = [NSCalendar calendarWithIdentifier:NSCalendarIdentifierGregorian];
    NSUInteger unitFlags = NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay;
    NSDateComponents *cmp1 = [calendar components:unitFlags fromDate:now];
    NSDateComponents *cmp2 = [calendar components:unitFlags fromDate:date];
    int age = (int)(cmp1.year - cmp2.year);
    if (age < 0) return -1;
    if (cmp1.month == cmp2.month) {
        if (cmp1.day < cmp2.day) {
            // 月份相同, 但是未到出生那一天, 不满一岁
            age = MAX(age-1 , 0);
        }
    } else if (cmp1.month < cmp2.month) {
        // 当前月份小于出生日期月份, 不满一岁
        age = MAX(age-1 , 0);
    }
    return age;
}

+ (IdcardGender)genderWithIdcard:(NSString *)idcard {
    if (![self isValidIdcard:idcard]) return IdcardGenderUnknown;
    NSString *code = idcard.length == 18 ? [idcard substringWithRange:NSMakeRange(16, 1)] : [idcard substringFromIndex:14];
    int number = [code intValue];
    return number % 2 == 0 ? IdcardGenderFemale : IdcardGenderMale;
}

+ (NSString *)birthdayWithIdcard:(NSString *)idcard {
    if (![self isValidIdcard:idcard]) return nil;
    if (idcard.length == 15) {
        idcard = [self conver15To18:idcard];
    }
    return [idcard substringWithRange:NSMakeRange(6, 8)];
}

+ (NSString *)conver15To18:(NSString *)idcard {
    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", @"^\\d{15}$"];
    if (![predicate evaluateWithObject:idcard]) return nil;
    NSString *ret = [NSString stringWithFormat:@"%@19%@", [idcard substringToIndex:6], [idcard substringFromIndex:6]];
    int factorArray[] = { 7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2 };
    NSArray<NSString *> *checkCodeArray = @[@"1", @"0", @"X", @"9", @"8", @"7", @"6", @"5", @"4", @"3", @"2"];
    int sum = 0;
    for (int i=0; i<ret.length; i++) {
        NSString *character = [ret substringWithRange:NSMakeRange(i, 1)];
        sum += [character intValue] * factorArray[i];
    }
    NSString *code = checkCodeArray[sum % 11];
    return [ret stringByAppendingString:code];
}

@end
```

## 参考

- [【提醒】原来身份证后四位是这意思！很多人都不知道](https://mp.weixin.qq.com/s/8inPcKdWOOZ3J_4lUJQ9Vw)
- [【实用】原来身份证后4位是这个意思，今天终于弄明白了！](https://mp.weixin.qq.com/s/rP9wHGBrzS4UkAfrettaoA)
- [身份证校验码](https://baike.baidu.com/item/身份证校验码/3800388)
- [中国省市区编码对照表 2020版](https://www.b910.cn/tool/1.htm)
