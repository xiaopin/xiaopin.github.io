---
title: Swift4之Codable协议
date: 2018-03-26 16:25:32
tags:
	- iOS
	- Swift
---

最近公司新项目采用了Swift开发，在处理JSON数据时，一开始是准备采用阿里开源的[HandyJSON](https://github.com/alibaba/HandyJSON)，后来了解了下，随着Swift4的到来，苹果为我们带来了[`Codable`](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)协议，用于数据的编解码，虽然现在的Codable并不完善，使用上也并没有那么的友好，但是毕竟是官方出品，后期肯定会越来越完善的，所以决定采用Codable而弃用HandyJSON。


## 基础

首先，Codable是`Decodable`协议和`Encodable`协议的组合类型，它们分别定义了`init(from decoder: Decoder) throws`和`func encode(to encoder: Encoder) throws`方法。如果我们只需单向转换，选择其一即可。

```Swift
/// A type that can encode itself to an external representation.
public protocol Encodable {

    /// Encodes this value into the given encoder.
    ///
    /// If the value fails to encode anything, `encoder` will encode an empty
    /// keyed container in its place.
    ///
    /// This function throws an error if any values are invalid for the given
    /// encoder's format.
    ///
    /// - Parameter encoder: The encoder to write data to.
    public func encode(to encoder: Encoder) throws
}

/// A type that can decode itself from an external representation.
public protocol Decodable {

    /// Creates a new instance by decoding from the given decoder.
    ///
    /// This initializer throws an error if reading from the decoder fails, or
    /// if the data read is corrupted or otherwise invalid.
    ///
    /// - Parameter decoder: The decoder to read data from.
    public init(from decoder: Decoder) throws
}

/// A type that can convert itself into and out of an external representation.
public typealias Codable = Decodable & Encodable
```


## JSON转Model

最理想的情况下，当然是服务器返回的JSON数据中key和我们Model中定义的key保持一致，那么我们只需要将Model声明为遵守Codable协议即可。

```Swift
struct Person: Codable {
    var name: String
    var age: Int
}
```

接下来就可以直接对JSON数据进行解码操作了:

```Swift
let data = "{\"name\":\"xiaopin\", \"age\": 18}".data(using: .utf8)!
let model = try? JSONDecoder().decode(Person.self, from: data)
```

看，是不是so easy！代码简直是6到没朋友啊。

## 自定义键值

当然了，大多数情况下我们都会遇到服务器返回的Key与Model属性名称不一致的情况，此时我们就需要自己定义映射关系了。Codable也为我们提供了解决方案，我们只需要定义一个名称为`CodingKeys`的嵌套枚举，关联值类型为String，并遵守CodingKey协议即可。

假设我们需要给Person增加一个学校信息，服务器返回的Key为`school_name`，而我们Model中则为`schoolName`，我们只需简单修改Person的定义即可:

```Swift
struct Person: Codable {
    enum CodingKeys: String, CodingKey {
        case name, age
        case schoolName = "school_name"
    }
    
    var name: String
    var age: Int
    var schoolName: String
}
```

通过CodingKeys这个枚举，我们便可轻易完成JSON Key和Model之间的映射关系。

默认情况下，编译器会为我们自动生成CodingKeys，并提供`init(from decoder: Decoder) throws`和`func encode(to encoder: Encoder) throws`的默认实现。


## 枚举

Codable提供对枚举的支持，只需定义好枚举的关联值类型，并遵守Codable协议即可。

现在我们给Person增加一个性别属性:

```Swift
struct Person: Codable {
    enum CodingKeys: String, CodingKey {
        case name, age, gender
        case schoolName = "school_name"
    }

    enum Gender: String, Codable {
        case male, female
    }
    
    var name: String
    var age: Int
    var gender: Gender
    var schoolName: String
}
```

只需简单修改，Codable就会自动为我们将JSON中的gender字符串转换为对应的Gender枚举值。


## 日期处理

先安利一个网站[www.nsdateformatter.com](http://www.nsdateformatter.com)

这里暂不做介绍，毕竟我也没研究透。


## 自定义解析

理想情况下我们当然不需要自己实现`init(from decoder: Decoder) throws`方法，但是总有一些特殊情况，或者说是给代码增加容错机制，我们不得不实现该方法，自己实现解码操作。

编译器的默认实现中，都是使用`public func decode<T>(_ type: T.Type, forKey key: KeyedDecodingContainer.Key) throws -> T where T : Decodable`方法进行解码的。

***默认实现有一定的局限性***

- 我们在上面中定义的gender属性，如果JSON数据中并不存在gender这个Key时则会抛出下面的异常信息:
```
keyNotFound(ETNavBarTransparentDemo.Person.(CodingKeys in _34A23078E52B5E180CB78F393D171183).gender, Swift.DecodingError.Context(codingPath: [], debugDescription: "No value associated with key gender (\"gender\").", underlyingError: nil))
```

- 如果Model中属性的类型是Int型，但是JSON中却是String类型的数字，那也会抛出异常:
```
typeMismatch(Swift.Int, Swift.DecodingError.Context(codingPath: [ETNavBarTransparentDemo.Person.(CodingKeys in _34A23078E52B5E180CB78F393D171183).age], debugDescription: "Expected to decode Int but found a string/data instead.", underlyingError: nil))
```

- Bool类型，只能处理true/false，不能处理字符串的"true"/"false"以及数字0/1

- Int和Double的区别，当JSON中的value为88.0时，Int和Double均可以解码，但是如果value为88.01时，Int解码失败，抛出typeMismatch异常信息

对于以上情况，如果你们App觉得无所谓，那就没什么了；但是如果你想自己处理这些情况，保证能够正常的解码JSON数据，那你就只能自行实现`init(from decoder: Decoder) throws`了。

自定义解析:

```Swift
struct DataModel: Codable {
    enum CodingKeys: String, CodingKey {
        case id, flag
    }
    
    var id: Int
    var flag: Bool
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        id = (try container.decodeIntIfPresent(.id)) ?? 0
        flag = (try container.decodeBoolIfPresent(.flag)) ?? true
    }
}
```

针对上面的情况，我给KeyedDecodingContainer扩展了几个方法，已可满足目前项目的需求，后期可视情况再新增:

```Swift
// MARK: - 强制解码,如果key不存在则抛出异常`DecodingError.keyNotFound`
extension KeyedDecodingContainer {
    
    /// 解码CGFloat
    func decodeCGFloat(_ key: K) throws -> CGFloat {
        do {
            let d = try decodeDouble(key)
            return CGFloat(d)
        } catch {
            throw error
        }
    }
    
    /// 解码Double
    func decodeDouble(_ key: K) throws -> Double {
        do {
            return try decode(Double.self, forKey: key)
        } catch {
            if let i = try? decode(Int.self, forKey: key) {
                return Double(i)
            }
            if let s = try? decode(String.self, forKey: key), let d = Double(s) {
                return d
            }
            if let b = try? decode(Bool.self, forKey: key) {
                return b ? 1.0 : 0.0
            }
            throw error
        }
    }
    
    /// 解码Int
    func decodeInt(_ key: K) throws -> Int {
        do {
            return try decode(Int.self, forKey: key)
        } catch {
            if let d = try? decode(Double.self, forKey: key) {
                return Int(d)
            }
            if let s = try? decode(String.self, forKey: key), let i = Int(s) {
                return i
            }
            if let b = try? decode(Bool.self, forKey: key) {
                return b ? 1 : 0
            }
            throw error
        }
    }
    
    /// 解码String
    func decodeString(_ key: K) throws -> String {
        do {
            return try decode(String.self, forKey: key)
        } catch {
            if let i = try? decode(Int.self, forKey: key) {
                return String(i)
            }
            if let d = try? decode(Double.self, forKey: key) {
                return String(d)
            }
            if let b = try? decode(Bool.self, forKey: key) {
                return b ? "true" : "false"
            }
            throw error
        }
    }
    
    /// 解码Bool
    func decodeBool(_ key: K) throws -> Bool {
        do {
            return try decode(Bool.self, forKey: key)
        } catch {
            if let s = try? decode(String.self, forKey: key) {
                if s.isEmpty || s == "0" || s.lowercased() == "false" {
                    return false
                }
                return true
            }
            if let i = try? decode(Int.self, forKey: key) {
                return (i == 0) ? false : true
            }
            if let d = try? decode(Double.self, forKey: key) {
                return (d == 0.0) ? false : true
            }
            throw error
        }
    }
    
}

// MARK: - 忽略`DecodingError.keyNotFound`异常信息,当key不存在时返回`nil`
extension KeyedDecodingContainer {
    
    /// 解码CGFloat数据
    func decodeCGFloatIfPresent(_ key: K) throws -> CGFloat? {
        if let d = try decodeDoubleIfPresent(key) {
            return CGFloat(d)
        }
        return nil
    }
    
    /// 解码Double数据
    func decodeDoubleIfPresent(_ key: K) throws -> Double? {
        do {
            return try decodeIfPresent(Double.self, forKey: key)
        } catch {
            // Int -> Double
            do {
                if let i = try decodeIfPresent(Int.self, forKey: key) {
                    return Double(i)
                }
            } catch {}
            // String -> Double
            do {
                if let s = try decodeIfPresent(String.self, forKey: key) {
                    return Double(s)
                }
            } catch {}
            // Bool -> Double
            do {
                if let b = try decodeIfPresent(Bool.self, forKey: key) {
                    return b ? 1.0 : 0.0
                }
            } catch {}
            
            // failure
            throw error
        }
    }
    
    /// 解码Int数据
    func decodeIntIfPresent(_ key: K) throws -> Int? {
        do {
            return try decodeIfPresent(Int.self, forKey: key)
        } catch {
            // Double -> Int
            do {
                if let d = try decodeIfPresent(Double.self, forKey: key) {
                    return Int(d)
                }
            } catch {}
            // String -> Int
            do {
                if let s = try decodeIfPresent(String.self, forKey: key) {
                    return Int(s)
                }
            } catch {}
            // Bool -> Int
            do {
                if let b = try decodeIfPresent(Bool.self, forKey: key) {
                    return b ? 1 : 0
                }
            } catch {}
            // decode failure.
            throw error
        }
    }
    
    /// 解码String数据
    func decodeStringIfPresent(_ key: K) throws -> String? {
        do {
            return try decodeIfPresent(String.self, forKey: key)
        } catch {
            // Int -> String
            do {
                if let i = try decodeIfPresent(Int.self, forKey: key) {
                    return String(i)
                }
            } catch {}
            // Double -> String
            do {
                if let d = try decodeIfPresent(Double.self, forKey: key) {
                    return String(d)
                }
            } catch {}
            // Bool -> String
            do {
                if let b = try decodeIfPresent(Bool.self, forKey: key) {
                    return b ? "true" : "false"
                }
            } catch {}
            // decode failure.
            throw error
        }
    }
    
    /// 解码Bool数据
    func decodeBoolIfPresent(_ key: K) throws -> Bool? {
        do {
            return try decodeIfPresent(Bool.self, forKey: key)
        } catch {
            // String -> Bool
            do {
                if let s = try decodeIfPresent(String.self, forKey: key) {
                    if s.isEmpty || s == "0" || s.lowercased() == "false" {
                        return false
                    }
                    return true
                }
            } catch {}
            // Int -> Bool
            do {
                if let i = try decodeIfPresent(Int.self, forKey: key) {
                    return (i == 0) ? false : true
                }
            } catch {}
            // Double -> Bool
            do {
                if let d = try decodeIfPresent(Double.self, forKey: key) {
                    return (d == 0.0) ? false : true
                }
            } catch {}
            // decode failure.
            throw error
        }
    }
    
    /// 解码CGSize数据
    func decodeCGSizeIfPresent(_ key: K) throws -> CGSize? {
        do {
            return try decodeIfPresent(CGSize.self, forKey: key)
        } catch {
            do {
                if let s = try decodeIfPresent(String.self, forKey: key) {
                    // 例: 宽x高 宽,高 宽X高
                    for seprator in ["x", ",", "X"] {
                        let array = s.components(separatedBy: seprator)
                        if array.count == 2 {
                            if let w = Double(array.first!), let h = Double(array.last!) {
                                return CGSize(width: w, height: h)
                            }
                        }
                    }
                }
            } catch {}
            throw error
        }
    }
    
}
```

有了这些扩展方法，我们就事半功倍了。

更多信息请参考：[Using JSON with Custom Types](https://developer.apple.com/documentation/foundation/archives_and_serialization/using_json_with_custom_types)

