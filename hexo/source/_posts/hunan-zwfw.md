---
title: Vue对接湖南省“互联网+政务服务”一体化平台统一身份认证
abbrlink: d07bccbc
date: 2022-07-01 16:49:27
tags:
    - Vue
    - JavaScript
    - TypeScript
---

## 前因

领导给安排了一个任务，给湖南省XX市做一个电子证照的申报模块。

其实这功能分两部分，一部分是嵌入“我的XX” App 中进行使用的，App 提供了相关的 JS SDK，可以获取登录的账号数据，所以账号数据由 App 进行提供，这部分没什么好说的，三下五除二就给他搞完了。

另外一部分就是用于浏览器端的，用户在电脑上通过浏览器访问XX市政务服务官网，点击页面上的“电子证照申报”模块进入我们的站点，然后登录进行申报、查看详情等操作。

一切都看似平淡无奇，毫无波澜，实际上在用户登录这一块就遇到很大阻力，因为一开始 App 端的登录功能我们是直接对接的XX市的服务，但是 Web 端的登录人家不提供啊，因为他们的用户信息也是对接的湖南省政务服务开放平台，所以 Web 端的登录需要我们自己去和省政务平台进行对接。

## 梳理对接流程

先说一下项目的情况吧：

- 在XX市的政务服务官网和“我的XX”App上都会有一个证照申报功能的入口
- 在App上点击进入，则用户信息由App进行登录支持，App端最终也是对接的 `湖南省互联网+政务服务开放平台`
- 在官网上点击进入，市政府则不提供网页端登录支撑，所以由我司和 `湖南省互联网+政务服务开放平台` 直接进行对接
    - 由于我司后端只提供证照申报的后台登录账号和后台操作相关的权限和功能页面(JSP)，所以最终决定直接由我这边前端页面进行对接

别问我为什么不是后端来对接，我也`🤷‍♂️不知道🤷‍♀️`。由于这个是跨部门合作的，我不知道中间发生了什么，大家是怎么沟通的，我只知道领导丢给我一个文档，说让我这边对接一下。

既然任务都给安排了，那就先啃一遍文档呗。

过了一遍文档，发现其实整个对接过程还是蛮轻松的：

1、获取授权码，跳转到[湖南省政务服务平台](https://auth.zwfw.hunan.gov.cn/oauth2/authorize?client_id=sXK6HBx3QwuJqaMXqmx2fQ&code=90000&response_type=gov&redirect_uri=http://zwfw-new.hunan.gov.cn:80/oauth2-login?backUrl=http://zwfw-new.hunan.gov.cn:80/hnvirtualhall/index.jsp&flag=false)进行统一登录，按照要求设置好 `client_id`、`response_type`、`redirect_uri` 这三个参数，`scope` 参数固定传 `scope`，登录成功后会重定向到回调地址中，并且携带 `code` 授权码参数（参见文档的 6.1.1 章节内容）
2、根据授权码获取用户凭证(access_token)（参见文档的 6.2.2 章节内容）
3、根据用户凭证获取用户信息（参见文档的 6.2.3 章节内容）

以上三步就是我们整个登录功能的逻辑处理流程。

## 对接登录，获取用户信息

- 在 `.env` 文件中定义以下环境变量：

```shell
# 湖南省政务服务开放平台的登录回调地址(本项目部署后的访问地址)
VUE_APP_CLIENT_REDIRECT_URI=http://localhost:8081
# 湖南省政务服务开放平台的登录/登出地址
VUE_APP_CLIENT_LOGIN_URL=https://auth.zwfw.hunan.gov.cn
# 湖南省政务服务开放平台的接口地址
VUE_APP_CLIENT_API_URL=https://apis.zwfw.hunan.gov.cn
# 湖南省政务服务开放平台的客户端id
VUE_APP_CLIENT_ID=xxxxxxxxxx
# 湖南省政务服务开放平台的客户端KEY
VUE_APP_CLIENT_SECRET=xxxxxxxxxx
```

- 封装登录/登出功能（`src/utils/helper.ts`）

```ts
export default class Helper {
    /** 打开登录页面(跳转到湖南省政务服务网站进行统一登录) */
    static gotoLoginPage(): void {
        const cliendId = process.env.VUE_APP_CLIENT_ID
        const redirectUri = encodeURIComponent(process.env.VUE_APP_CLIENT_REDIRECT_URI)
        const loginUrl = `${process.env.VUE_APP_CLIENT_LOGIN_URL}/oauth2/authorize?client_id=${cliendId}&response_type=gov&scope=user.read&redirect_uri=${redirectUri}`
        window.location.href = loginUrl
    }

    /** 跳转到湖南省政务服务网站进行统一登出操作 */
    static gotoLogoutPage(): void {
        CacheManager.clear() // 清除本地缓存数据
        setTimeout(() => {
            const cliendId = process.env.VUE_APP_CLIENT_ID
            const redirectUri = encodeURIComponent(process.env.VUE_APP_CLIENT_REDIRECT_URI)
            const loginUrl = `${process.env.VUE_APP_CLIENT_LOGIN_URL}/oauth2/logout?client_id=${cliendId}&response_type=code&redirect_uri=${redirectUri}`
            window.location.href = loginUrl
        }, 200)
    }
}
```

- 接口封装（主要看请求方式、接口地址、请求参数就行，这里不列举更多代码了）

```ts
export default class Api {
    /** 通过授权码获取用户授权令牌(AccessToken) */
    static getAccessToken<T = ApiResp.HunanPlatformAccessTokenModel>(code: string): HttpResponse<T> {
        const parameters = {
            code,
            client_id: process.env.VUE_APP_CLIENT_ID,
            client_secret: process.env.VUE_APP_CLIENT_SECRET,
            grant_type: 'authorization_code'
        }
        return Http.request<T>('GET', '/oauth2/access_token', undefined, parameters, {
            baseURL: process.env.VUE_APP_CLIENT_API_URL
        })
    }

    /** 通过授权令牌获取用户信息 */
    static getUserInfo<T = ApiResp.HunanPlatformUserInfoModel>(access_token: string): HttpResponse<T> {
        const parameters = { access_token: access_token }
        return Http.request<T>('GET', '/oauth2/user_info', undefined, parameters, {
            baseURL: process.env.VUE_APP_CLIENT_API_URL
        })
    }
}
```

- 登录跳转

```html
<el-button type="primary" @click="onHandleLogin" v-else>登录&nbsp;/&nbsp;注册</el-button>
```

```ts
import { Component, Prop, Vue } from 'vue-property-decorator'
import Helper from '@/utils/helper'

export default class AppLayout extends Vue {
    onHandleLogin(): void {
        Helper.gotoLoginPage()
    }
}
```

- 处理 `湖南省政务服务平台` 的登录跳转回调, 接收返回的 `code` 参数

```TypeScript
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class App extends Vue {
    mounted() {
        // 通过匹配 url 是否包含 code 来判断是否为登录回调
        const url = window.location.href
        const matchResult = url.match(/[\?\&]code=[\w-_]+/)
        if (matchResult && matchResult.length) {
            // 湖南省政务服务平台登录重定向
            const code = matchResult[0].replace(/[\?\&]code=/, '')
            this.onHandleHunanPlatformLoginCallback(code)
        } else {
            this.onRestoreAccount()
        }
    }

    onRestoreAccount(): void {
        // TODO: 从缓存中恢复数据
    }

    onHandleHunanPlatformLoginCallback(code: string): void {
        // TODO:
        // 1、调用 /oauth2/access_token 接口获取授权令牌
        // 2、调用 /oauth2/user_info 接口获取登录用户信息
        // 3、将授权令牌和登录信息进行本地化存储，便于后续业务操作
        this.$api.getAccessToken(code)
            .then(res => {
                // TODO: 存储授权令牌
                return this.$api.getUserInfo(res.data.access_token)
            })
            .then(res => {
                // TODO: 存储用户信息
                window.location.href = window.location.origin + window.location.pathname
            })
            .catch((e: Error) => {})
    }
}
```

## 登出

直接调用前面封装好的 `gotoLogoutPage` 方法即可，会先跳转到 `湖南省政务服务平台` 进行统一登出操作，然后再重定向回到我们的站点。

## 参考

- [湖南一件事一次办](http://zwfw-new.hunan.gov.cn/)
- [湖南省“互联网+政务服务”应用能力开放平台](http://open.zwfw.hunan.gov.cn/)
- [湖南省“互联网+政务服务”一体化平台统一身份认证接入标准规范1015.docx](/images/2022/湖南省“互联网+政务服务”一体化平台统一身份认证接入标准规范1015.docx)
