---
title: 经纬度在 WGS-84/GCJ-02/BD09 坐标系之间的转换
abbrlink: d90d8bd
date: 2023-03-16 10:22:19
tags:
    - Objective-C
    - TypeScript
---

国内各大地图厂商SDK都会内置了一些转换函数以供使用，但是有时候我们的小项目并不需要引入这些庞大的SDK，也没去申请相关授权key（毕竟现在商用都是收费了，费用还不低呢）。

下面整理了一些经纬度在不同的地理坐标系之间相互转换的方法，提供 Objective-C 和 TypeScript 版本。

## Objective-C 版本

苹果自带的地图功能，在中国使用的是由高德地图所提供的数据，在国外则是由 TomTom、OpenStreetMap 等第三方公司提供数据。

使用自带的 `CoreLocation` 和 `MapKit` 框架，需要注意返回的经纬度所采用的坐标系，CoreLocation 返回的是 `WGS-84` 坐标系，而 MapKit 需要的是 `GCJ-02` 坐标系。

```ObjC
#import <Foundation/Foundation.h>
#import <CoreLocation/CoreLocation.h>

@interface XPCoordinateConverter : NSObject

/// 世界标准地理坐标转成中国国测局地理坐标(WGS-84 -> GCJ-02)
+ (CLLocationCoordinate2D)wgs84ToGcj02:(CLLocationCoordinate2D)coordinate;

/// 中国国测局地理坐标转成世界标准地理坐标(GCJ-02 -> WGS-84)
+ (CLLocationCoordinate2D)gcj02ToWgs84:(CLLocationCoordinate2D)coordinate;

/// 世界标准地理坐标转成百度地理坐标(WGS-84 -> BD09)
+ (CLLocationCoordinate2D)wgs84ToBd09:(CLLocationCoordinate2D)coordinate;

/// 百度地理坐标转成世界标准地理坐标(BD09 -> WGS-84)
+ (CLLocationCoordinate2D)bd09ToWgs84:(CLLocationCoordinate2D)coordinate;

/// 中国国测局地理坐标转成百度地理坐标(GCJ-02 -> BD09)
+ (CLLocationCoordinate2D)gcj02ToBd09:(CLLocationCoordinate2D)coordinate;

/// 百度地理坐标转成中国国测局地理坐标(BD09 -> GCJ-02)
+ (CLLocationCoordinate2D)bd09ToGcj02:(CLLocationCoordinate2D)coordinate;

@end
```

```ObjC
#import "XPCoordinateConverter.h"

#define CC_X_PI     (3.14159265358979324 * 3000.0 / 180.0)
#define CC_PI       (3.1415926535897932384626)
#define CC_A        (6378245.0)
#define CC_EE       (0.00669342162296594323)

double cc_transformlat(double lng, double lat) {
    double ret = -100.0 + 2.0 * lng + 3.0 * lat + 0.2 * lat * lat + 0.1 * lng * lat + 0.2 * sqrt(fabs(lng));
    ret += (20.0 * sin(6.0 * lng * CC_PI) + 20.0 * sin(2.0 * lng * CC_PI)) * 2.0 / 3.0;
    ret += (20.0 * sin(lat * CC_PI) + 40.0 * sin(lat / 3.0 * CC_PI)) * 2.0 / 3.0;
    ret += (160.0 * sin(lat / 12.0 * CC_PI) + 320 * sin(lat * CC_PI / 30.0)) * 2.0 / 3.0;
    return ret;
}

double cc_transformlng(double lng, double lat) {
    double ret = 300.0 + lng + 2.0 * lat + 0.1 * lng * lng + 0.1 * lng * lat + 0.1 * sqrt(fabs(lng));
    ret += (20.0 * sin(6.0 * lng * CC_PI) + 20.0 * sin(2.0 * lng * CC_PI)) * 2.0 / 3.0;
    ret += (20.0 * sin(lng * CC_PI) + 40.0 * sin(lng / 3.0 * CC_PI)) * 2.0 / 3.0;
    ret += (150.0 * sin(lng / 12.0 * CC_PI) + 300.0 * sin(lng / 30.0 * CC_PI)) * 2.0 / 3.0;
    return ret;
}

BOOL cc_location_in_china(CLLocationCoordinate2D coordinate) {
    if (coordinate.longitude < 72.004 ||
        coordinate.longitude > 137.8347 ||
        coordinate.latitude < 0.8293 ||
        coordinate.latitude > 55.8271) {
        return NO;
    }
    return YES;
}

@implementation XPCoordinateConverter

/// 世界标准地理坐标转成中国国测局地理坐标(WGS-84 -> GCJ-02)
+ (CLLocationCoordinate2D)wgs84ToGcj02:(CLLocationCoordinate2D)coordinate {
    double lng = coordinate.longitude;
    double lat = coordinate.latitude;
    double dlat = cc_transformlat(lng - 105.0, lat - 35.0);
    double dlng = cc_transformlng(lng - 105.0, lat - 35.0);
    double radlat = lat / 180.0 * CC_PI;
    double magic = sin(radlat);
    magic = 1 - CC_EE * magic * magic;
    double sqrtmagic = sqrt(magic);
    dlat = (dlat * 180.0) / ((CC_A * (1 - CC_EE)) / (magic * sqrtmagic) * CC_PI);
    dlng = (dlng * 180.0) / (CC_A / sqrtmagic * cos(radlat) * CC_PI);
    return CLLocationCoordinate2DMake(lat + dlat, lng + dlng);
}

/// 中国国测局地理坐标转成世界标准地理坐标(GCJ-02 -> WGS-84)
+ (CLLocationCoordinate2D)gcj02ToWgs84:(CLLocationCoordinate2D)coordinate {
    double lng = coordinate.longitude;
    double lat = coordinate.latitude;
    double dlat = cc_transformlat(lng - 105.0, lat - 35.0);
    double dlng = cc_transformlng(lng - 105.0, lat - 35.0);
    double radlat = lat / 180.0 * CC_PI;
    double magic = sin(radlat);
    magic = 1 - CC_EE * magic * magic;
    double sqrtmagic = sqrt(magic);
    dlat = (dlat * 180.0) / ((CC_A * (1 - CC_EE)) / (magic * sqrtmagic) * CC_PI);
    dlng = (dlng * 180.0) / (CC_A / sqrtmagic * cos(radlat) * CC_PI);
    return CLLocationCoordinate2DMake(lat * 2 - (lat + dlat), lng * 2 - (lng + dlng));
}

/// 世界标准地理坐标转成百度地理坐标(WGS-84 -> BD09)
+ (CLLocationCoordinate2D)wgs84ToBd09:(CLLocationCoordinate2D)coordinate {
    coordinate = [self wgs84ToGcj02:coordinate];
    return [self gcj02ToBd09:coordinate];
}

/// 百度地理坐标转成世界标准地理坐标(BD09 -> WGS-84)
+ (CLLocationCoordinate2D)bd09ToWgs84:(CLLocationCoordinate2D)coordinate {
    coordinate = [self bd09ToGcj02:coordinate];
    return [self gcj02ToWgs84:coordinate];
}

/// 中国国测局地理坐标转成百度地理坐标(GCJ-02 -> BD09)
+ (CLLocationCoordinate2D)gcj02ToBd09:(CLLocationCoordinate2D)coordinate {
    CLLocationCoordinate2D bd09Coordinate;
    double x = coordinate.longitude;
    double y = coordinate.latitude;
    double z = sqrt(x * x + y * y) + 0.00002 * sin(y * CC_X_PI);
    double theta = atan2(y, x) + 0.000003 * cos(x * CC_X_PI);
    bd09Coordinate.longitude = z * cos(theta) + 0.0065;
    bd09Coordinate.latitude = z * sin(theta) + 0.006;
    return bd09Coordinate;
}

/// 百度地理坐标转成中国国测局地理坐标(BD09 -> GCJ-02)
+ (CLLocationCoordinate2D)bd09ToGcj02:(CLLocationCoordinate2D)coordinate {
    double x = coordinate.longitude - 0.0065;
    double y = coordinate.latitude - 0.006;
    double z = sqrt(x * x + y * y) - 0.00002 * sin(y * CC_X_PI);
    double theta = atan2(y, x) - 0.000003 * cos(x * CC_X_PI);
    return CLLocationCoordinate2DMake(z * sin(theta), z * cos(theta));
}

@end
```

## TypeScript 版本

```TypeScript
const CC_X_PI   = 3.14159265358979324 * 3000.0 / 180.0
const CC_PI     = 3.1415926535897932384626
const CC_A      = 6378245.0
const CC_EE     = 0.00669342162296594323

function cc_transformlat(lng: number, lat: number): number {
    let ret = -100.0 + 2.0 * lng + 3.0 * lat + 0.2 * lat * lat + 0.1 * lng * lat + 0.2 * Math.sqrt(Math.abs(lng))
    ret += (20.0 * Math.sin(6.0 * lng * CC_PI) + 20.0 * Math.sin(2.0 * lng * CC_PI)) * 2.0 / 3.0
    ret += (20.0 * Math.sin(lat * CC_PI) + 40.0 * Math.sin(lat / 3.0 * CC_PI)) * 2.0 / 3.0
    ret += (160.0 * Math.sin(lat / 12.0 * CC_PI) + 320 * Math.sin(lat * CC_PI / 30.0)) * 2.0 / 3.0
    return ret
}

function cc_transformlng(lng: number, lat: number): number {
    let ret = 300.0 + lng + 2.0 * lat + 0.1 * lng * lng + 0.1 * lng * lat + 0.1 * Math.sqrt(Math.abs(lng))
    ret += (20.0 * Math.sin(6.0 * lng * CC_PI) + 20.0 * Math.sin(2.0 * lng * CC_PI)) * 2.0 / 3.0
    ret += (20.0 * Math.sin(lng * CC_PI) + 40.0 * Math.sin(lng / 3.0 * CC_PI)) * 2.0 / 3.0
    ret += (150.0 * Math.sin(lng / 12.0 * CC_PI) + 300.0 * Math.sin(lng / 30.0 * CC_PI)) * 2.0 / 3.0
    return ret
}

export type LocationDegrees = number | string

export interface LocationCoordinate2D {
    longitude: LocationDegrees
    latitude: LocationDegrees
}

export function LocationCoordinate2DMake(longitude: LocationDegrees, latitude: LocationDegrees): LocationCoordinate2D {
    return { longitude, latitude }
}

export function getLocationDegreesNumber(degrees: LocationDegrees): number {
    if (typeof degrees === 'number') return degrees as number
    return parseFloat(degrees)
}

export default class CoordinateConverter {
    
    static wgs84ToGcj02(coordinate: LocationCoordinate2D): LocationCoordinate2D {
        let lng = getLocationDegreesNumber(coordinate.longitude)
        let lat = getLocationDegreesNumber(coordinate.latitude)
        let dlat = cc_transformlat(lng - 105.0, lat - 35.0)
        let dlng = cc_transformlng(lng - 105.0, lat - 35.0)
        let radlat = lat / 180.0 * CC_PI
        let magic = Math.sin(radlat)
        magic = 1 - CC_EE * magic * magic
        let sqrtmagic = Math.sqrt(magic)
        dlat = (dlat * 180.0) / ((CC_A * (1 - CC_EE)) / (magic * sqrtmagic) * CC_PI)
        dlng = (dlng * 180.0) / (CC_A / sqrtmagic * Math.cos(radlat) * CC_PI)
        return LocationCoordinate2DMake(lng + dlng, lat + dlat)
    }

    static gcj02ToWgs84(coordinate: LocationCoordinate2D): LocationCoordinate2D {
        let lng = getLocationDegreesNumber(coordinate.longitude)
        let lat = getLocationDegreesNumber(coordinate.latitude)
        let dlat = cc_transformlat(lng - 105.0, lat - 35.0)
        let dlng = cc_transformlng(lng - 105.0, lat - 35.0)
        let radlat = lat / 180.0 * CC_PI
        let magic = Math.sin(radlat)
        magic = 1 - CC_EE * magic * magic
        let sqrtmagic = Math.sqrt(magic)
        dlat = (dlat * 180.0) / ((CC_A * (1 - CC_EE)) / (magic * sqrtmagic) * CC_PI)
        dlng = (dlng * 180.0) / (CC_A / sqrtmagic * Math.cos(radlat) * CC_PI)
        return LocationCoordinate2DMake(lng * 2 - (lng + dlng), lat * 2 - (lat + dlat))
    }

    static wgs84ToBd09(coordinate: LocationCoordinate2D): LocationCoordinate2D {
        return this.gcj02ToBd09(this.wgs84ToGcj02(coordinate))
    }

    static bd09ToWgs84(coordinate: LocationCoordinate2D): LocationCoordinate2D {
        return this.gcj02ToWgs84(this.bd09ToGcj02(coordinate))
    }

    static gcj02ToBd09(coordinate: LocationCoordinate2D): LocationCoordinate2D {
        const x = getLocationDegreesNumber(coordinate.longitude)
        const y = getLocationDegreesNumber(coordinate.latitude)
        const z = Math.sqrt(x * x + y * y) + 0.00002 * Math.sin(y * CC_X_PI)
        const theta = Math.atan2(y, x) + 0.000003 * Math.cos(x * CC_X_PI)
        const longitude = z * Math.cos(theta) + 0.0065
        const latitude = z * Math.sin(theta) + 0.006
        return LocationCoordinate2DMake(longitude, latitude)
    }

    static bd09ToGcj02(coordinate: LocationCoordinate2D): LocationCoordinate2D {
        const x = getLocationDegreesNumber(coordinate.longitude) - 0.0065
        const y = getLocationDegreesNumber(coordinate.latitude) - 0.006
        const z = Math.sqrt(x * x + y * y) - 0.00002 * Math.sin(y * CC_X_PI)
        const theta = Math.atan2(y, x) - 0.000003 * Math.cos(x * CC_X_PI)
        const longitude = z * Math.cos(theta)
        const latitude = z * Math.sin(theta)
        return LocationCoordinate2DMake(longitude, latitude)
    }
}
```

调用示例：

```TypeScript
<script setup lang="ts">
import { onMounted } from 'vue'
import CoordinateConverter, { LocationCoordinate2DMake } from './CoordinateConverter'

onMounted(() => {
  const wgs84Coordinate = LocationCoordinate2DMake(114.060239, 22.530405)
  console.log('==> gcj02:', CoordinateConverter.wgs84ToGcj02(wgs84Coordinate))
  console.log('==> bd09:', CoordinateConverter.wgs84ToBd09(wgs84Coordinate))
})

</script>
```

## 参考

感谢 [ni1o1/CoordinatesConverter](https://github.com/ni1o1/CoordinatesConverter) 开源项目，该项目的坐标转换结果相对来说还是比较准确的。

- [CoordinatesConverter](https://github.com/ni1o1/CoordinatesConverter)
- [地图坐标系在线转换](https://tool.lu/zh_CN/coordinate/index.html)
