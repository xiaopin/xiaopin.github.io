---
title: React Navigation 6.x 的使用
abbrlink: 3dd5863a
date: 2022-08-10 20:37:37
tags:
    - React Native
---

作为 `react native` 小菜鸡一枚，第一次接触 [react navigation](https://reactnavigation.org/docs/getting-started) 路由框架，对其也是很懵逼，在不断探索学习中；由于平时都是用 `Vue` 进行开发，所以对 [vue-router](https://router.vuejs.org/zh/) 比较熟悉一些，故而想着用 `vue-router` 路由表那一套用法来使用 `react navigation`，当然这个用法不是必须和唯一的，你可以根据你的习惯来开发即可。

Let's go, Just do it!!!

## 当前环境

- react 18.0.0
- react-native 0.69.3
- @react-navigation/native 6.0.11
- @react-navigation/native-stack 6.7.0
- @react-navigation/bottom-tabs 6.3.2
- @react-navigation/drawer 6.4.3

## 安装

- [新建项目](https://www.react-native.cn/docs/environment-setup)
```shell
$ npx react-native init ReactNavigationExample --template react-native-template-typescript
```

- [安装 react-navigation 以及相关依赖库](https://reactnavigation.org/docs/getting-started)
```shell
$ yarn add @react-navigation/native
$ yarn add react-native-screens react-native-safe-area-context
```

- [安装 Stack Navigator](https://reactnavigation.org/docs/native-stack-navigator)
```shell
$ yarn add @react-navigation/native-stack
```

- [安装 Tabs Navigator](https://reactnavigation.org/docs/bottom-tab-navigator)
```shell
$ yarn add @react-navigation/bottom-tabs
```

- [安装 Drawer Navigator（侧边栏菜单）以及相关依赖库](https://reactnavigation.org/docs/drawer-navigator#installation)
```shell
$ yarn add @react-navigation/drawer
$ yarn add react-native-gesture-handler react-native-reanimated
```

> 从 React Native 0.60 及更高版本开始，[Link](https://github.com/react-native-community/cli/blob/main/docs/autolinking.md)是自动的，所以你不需要运行 `react-native link` 命令。

`react-native-screens` 库需要做一些修改才能在 Android 设备上正常运行，编辑 `android/app/src/main/java/<your package name>/MainActivity.java` 文件：

1、导入包
```java
import android.os.Bundle;
```

2、重写 onCreate 方法
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(null);
}
```

对于 iOS 项目，别忘了通过 CocoaPods 安装相关依赖：
```shell
$ cd ios && pod install && cd ..
```

## 使用示例

### 功能目标

App 底部有一个 TabBar 管理着 `首页` 和 `我的` 两个页面，然后还提供了通过 `push` 方式打开 `个人中心`、`设置页面` 这两个页面，对了，还提供了一个模态视图的用例。En..., 大改就像下面这样👇：
```
App
├── TabBar
│   ├── 首页
│   └── 我的
├── 个人中心
├── 设置页面
└── 版本更新的模态视图
```

直接上图看效果：

![GIF](/images/2022/react-navigation-example.gif)

### 目录结构

在根目录下新建一个 `src` 文件夹用于存放我们的代码，新建 `src/routes` 目录用于存放路由表配置，新建 `src/screen` 目录用于存放所有页面（控制器），新建 `src/@types` 目录存放 TypeScript 的 `*.d.ts` 声明文件，此时目录结构如下所示：

```
ReactNavigationExample
├── App.tsx
├── Gemfile
├── android
├── app.json
├── babel.config.js
├── index.js
├── ios
├── metro.config.js
├── package.json
├── src
│   ├── @types
│   │   └── navigation.d.ts
│   ├── routes
│   │   └── index.tsx
│   └── screen
│       ├── HomeScreen.tsx
│       ├── ProfileScreen.tsx
│       ├── SettingScreen.tsx
│       ├── TabScreen.tsx
│       ├── UserCenterScreen.tsx
│       └── UpgradeModalScreen.tsx
├── tsconfig.json
└── yarn.lock
```

根据目录结构把 `src` 下的文件都创建好。

### 源码

#### navigation.d.ts

该文件内容如下：

```ts
/** 个人中心页面跳转时的路由携带参数 */
declare type UserCenterRouteParam = { userid: string | number } | undefined

/**
 * 定义App提供的所有页面路由名称以及参数
 * 其中 key 为路由跳转时的名称, value 为路由跳转时携带的参数
 */
declare type AppParamList = {
    Tab: object | undefined
    Home: object | undefined
    Profile: object | undefined
    Setting: object | undefined
    UserCenter: UserCenterRouteParam
    Upgrade: object | undefined
}
```

该文件主要是用于定义 App 中路由相关的参数，其中 `AppParamList` 定义的是所有页面的路由名称以及携带的参数数据，比如打开页面：
```ts
navigation.push('这里就是 AppParamList 的Key', {/* 这里就是 AppParamList 中 Key 所对应的值类型 */})
// 比如跳转个人中心页面：navigation.push('UserCenter', { userid: 100001 })
```

而我们获取页面传递过来的参数时：
```ts
// 这里的 params 就是 AppParamList 中 Key 所对应的值类型
// 比如个人中心页面, 这里 params 的数据类型就是 UserCenterRouteParam
const params = route.params
```

#### HomeScreen.tsx

```ts
import { NativeStackScreenProps } from '@react-navigation/native-stack'
import React, { useLayoutEffect } from 'react'
import { Button, Text, View } from 'react-native'

const HomeScreen: React.FC<NativeStackScreenProps<AppParamList, 'Home'>> = props => {
    const { navigation, route } = props

    useLayoutEffect(() => {
        navigation.setOptions({
            headerTitle: `首页${Math.ceil(Math.random() * 100)}`,
            headerLeft: () => <Button title="个人中心" onPress={() => props.navigation.push('UserCenter', { userid: 10002 })} />,
            headerRight: () => <Button title="设置" onPress={() => props.navigation.push('Setting')} />
        })
    }, [navigation, route])

    return (
        <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
            <Text>这里是首页</Text>
            <Button title="模态视图" onPress={() => props.navigation.navigate('Upgrade')} />
            <Button title="我的" onPress={() => props.navigation.navigate('Profile')} />
        </View>
    )
}

export default HomeScreen
```

#### ProfileScreen.tsx

```ts
import React from 'react'
import { Text, View } from 'react-native'

const ProfileScreen: React.FC<{}> = props => {
    return (
        <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
            <Text>我的</Text>
        </View>
    )
}

export default ProfileScreen
```

#### TabScreen.tsx

```ts
import React from 'react'
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs'
import HomeScreen from './HomeScreen'
import ProfileScreen from './ProfileScreen'

const Tab = createBottomTabNavigator()

const TabScreen: React.FC<{}> = props => {
    return (
        <Tab.Navigator
            screenOptions={{
                headerStyle: {
                    backgroundColor: 'orange'
                },
                headerTintColor: '#fff',
                headerTitleStyle: {
                    fontWeight: 'bold'
                }
            }}>
            <Tab.Screen name="Home" component={HomeScreen} options={{ title: '首页', tabBarLabel: '首页' }} />
            <Tab.Screen name="Profile" component={ProfileScreen} options={{ title: '我的', tabBarLabel: '我的' }} />
        </Tab.Navigator>
    )
}

export default TabScreen
```

#### UserCenterScreen.tsx

```ts
import { NativeStackScreenProps } from '@react-navigation/native-stack'
import React from 'react'
import { Text, View, Button } from 'react-native'

const UserCenterScreen: React.FC<NativeStackScreenProps<AppParamList, 'UserCenter'>> = props => {
    const userid = props.route.params?.userid
    return (
        <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
            <Text>个人中心页面</Text>
            <Text>传递过来的userid：{userid}</Text>
            <Button title="个人中心" onPress={() => props.navigation.push('UserCenter', { userid: Math.ceil(Math.random() * 10000000) })} />
            <Button title="设置" onPress={() => props.navigation.navigate('Setting')} />
        </View>
    )
}

export default UserCenterScreen
```

#### SettingScreen.tsx

```ts
import React from 'react'
import { Text, View } from 'react-native'

const SettingScreen: React.FC<{}> = props => {
    return (
        <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
            <Text>设置</Text>
        </View>
    )
}

export default SettingScreen
```

#### UpgradeModalScreen.tsx

```ts
import { NativeStackScreenProps } from '@react-navigation/native-stack'
import React from 'react'
import { Text, TouchableOpacity, View } from 'react-native'

const UpgradeModalScreen: React.FC<NativeStackScreenProps<AppParamList, 'Upgrade'>> = props => {
    return (
        <TouchableOpacity style={{ flex: 1, alignItems: 'center', justifyContent: 'center', backgroundColor: '#00000066' }} onPress={() => props.navigation.goBack()}>
            <View style={{ width: 300, height: 200, backgroundColor: '#fff', borderRadius: 5 }}>
                <Text style={{ fontSize: 16, fontWeight: '700' }}>发现新版本</Text>
                <Text style={{ fontSize: 14, color: '#999' }}>发现新版本，快来更新吧！！！</Text>
            </View>
        </TouchableOpacity>
    )
}

export default UpgradeModalScreen
```

#### routes/index.ts

接下来就是最关键的路由表配置信息了，先上代码：

```ts
import React from 'react'
import { RouteConfig, NavigationState, NavigationContainer, EventMapBase } from '@react-navigation/native'
import { createNativeStackNavigator, NativeStackNavigationOptions } from '@react-navigation/native-stack'
import TabScreen from '../screen/TabScreen'
import UserCenterScreen from '../screen/UserCenterScreen'
import SettingScreen from '../screen/SettingScreen'
import HomeScreen from '../screen/HomeScreen'
import ProfileScreen from '../screen/ProfileScreen'
import UpgradeModalScreen from '../screen/UpgradeModalScreen'

export type AppRouteConfig = RouteConfig<AppParamList, keyof AppParamList, NavigationState, NativeStackNavigationOptions, EventMapBase>

const routes: AppRouteConfig[] = [
    {
        name: 'Tab',
        getComponent: () => TabScreen,
        options: {
            headerShown: false
        }
    },
    {
        name: 'UserCenter',
        component: UserCenterScreen,
        options: {
            title: '个人中心'
        }
    },
    {
        name: 'Setting',
        component: SettingScreen
    },
    {
        name: 'Upgrade',
        component: UpgradeModalScreen,
        options: {
            headerShown: false,
            presentation: 'transparentModal',
            animation: 'fade'
        }
    },
    {
        name: 'Home',
        component: HomeScreen
    },
    {
        name: 'Profile',
        component: ProfileScreen
    }
]

const Stack = createNativeStackNavigator()

const AppNavigation = () => {
    return (
        <NavigationContainer>
            <Stack.Navigator
                screenOptions={{
                    headerStyle: {
                        backgroundColor: 'orange'
                    },
                    headerTintColor: '#fff',
                    headerTitleStyle: {
                        fontWeight: 'bold'
                    },
                    headerBackTitle: ''
                }}>
                {routes.map(route =>
                    route.component ? (
                        <Stack.Screen name={route.name} component={route.component} options={route.options} key={route.name} />
                    ) : (
                        <Stack.Screen name={route.name} getComponent={route.getComponent!} options={route.options} key={route.name} />
                    )
                )}
            </Stack.Navigator>
        </NavigationContainer>
    )
}

export default AppNavigation
```

我们定义了 `AppRouteConfig` 用来表示路由配置信息的数据类型，通过定义 `routes` 变量来存储所有路由配置信息，最后通过导出 `AppNavigation` 函数组件，在适当的时机将路由表配置转换成对应的 `Stack.Screen` 屏幕并显示出来。

不管是 `栈（Stack）` 还是 `（模态）Modal` 等样式，都可以通过配置 `options` 中的属性来控制，灵活使用 `options` 可以实现各种各样的效果。

#### App.ts

经过了这么多的配置，不要忘了在 `App.ts` 中调用 `AppNavigation` 来显示我们的页面。

```ts
import React from 'react'
import AppNavigation from './src/routes'

const App: React.FC<{}> = () => {
    return <AppNavigation />
}

export default App
```

## 可能出现的错误

如果你安装了 `react-native-reanimated` 库，并且在运行过程中报如下错误：
```
error: node_modules/react-native-reanimated/src/index.ts: /xxx/ReactNavigationExample/node_modules/react-native-reanimated/src/index.ts: Export namespace should be first transformed by `@babel/plugin-proposal-export-namespace-from`.
  5 | export * from './reanimated1';
  6 | export * from './reanimated2';
> 7 | export * as default from './Animated';
```

需要修改 `babel.config.js` 文件，具体可以参考[官方文档](https://docs.swmansion.com/react-native-reanimated/docs/fundamentals/installation#babel-plugin) 的说明：
```js
module.exports = {
    plugins: [
        'react-native-reanimated/plugin',
    ],
};
```

## 参考

- [React Navigation Docs](https://reactnavigation.org/docs/getting-started)
- [react-native-reanimated](https://docs.swmansion.com/react-native-reanimated/docs/fundamentals/installation)