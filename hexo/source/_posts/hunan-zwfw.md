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

## 对接

先说一下项目的组织结构吧

- 在XX市的政务服务官网和“我的XX”App上都会有一个证照申报功能的入口
- 在App上点击进入，则用户信息由App进行登录支持，App端最终也是对接的 `湖南省互联网+政务服务开放平台`
- 在官网上点击进入，市政府则不提供网页端登录支撑，所以由我司和 `湖南省互联网+政务服务开放平台` 直接进行对接
    - 由于我司后端只提供证照申报的后台登录账号和后台操作相关的权限和功能页面(JSP)，所以最终决定直接由我这边前端页面进行对接

别问我为什么不是后端来对接，我也`🤷‍♂️不知道🤷‍♀️`。由于这个是跨部门合作的，我不知道中间发生了什么，大家是怎么沟通的，我只知道领导丢给我一个文档，说让我这边对接一下。

既然任务都给安排了，那就先啃一遍文档呗。

过了一遍文档，发现其实整个对接过程还是蛮轻松的

- 获取授权码，跳转到[湖南省政务服务平台](https://auth.zwfw.hunan.gov.cn/oauth2/authorize?client_id=sXK6HBx3QwuJqaMXqmx2fQ&code=90000&response_type=gov&redirect_uri=http://zwfw-new.hunan.gov.cn:80/oauth2-login?backUrl=http://zwfw-new.hunan.gov.cn:80/hnvirtualhall/index.jsp&flag=false)进行统一登录，按照要求设置好 `client_id`、`response_type`、`redirect_uri` 这三个参数，`scope` 参数固定传 `scope`，登录成功后会重定向到回调地址中，并且携带 `code` 授权码参数（参见文档的 6.1.1 章节内容）
- 根据授权码获取用户凭证(access_token)（参见文档的 6.2.2 章节内容）
- 根据用户凭证获取用户信息（参见文档的 6.2.3 章节内容）

```shell
# 湖南省政务服务开放平台的登录回调地址(本项目部署后的访问地址)
VUE_APP_CLIENT_REDIRECT_URI=http://220.248.166.113:8081/cdh5
# 湖南省政务服务开放平台的登录地址
VUE_APP_CLIENT_LOGIN_URL=https://auth.zwfw.hunan.gov.cn
# 湖南省政务服务开放平台的接口地址
VUE_APP_CLIENT_API_URL=https://apis.zwfw.hunan.gov.cn
# 湖南省政务服务开放平台的客户端id
VUE_APP_CLIENT_ID=xxxxxxxxxx
# 湖南省政务服务开放平台的客户端KEY
VUE_APP_CLIENT_SECRET=xxxxxxxxxx
```

```TypeScript
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class App extends Vue {
    mounted() {
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
    }
}
```

## 参考

- [湖南一件事一次办](http://zwfw-new.hunan.gov.cn/)
- [湖南省“互联网+政务服务”应用能力开放平台](http://open.zwfw.hunan.gov.cn/)
- [湖南省“互联网+政务服务”一体化平台统一身份认证接入标准规范1015.docx](/images/2022/湖南省“互联网+政务服务”一体化平台统一身份认证接入标准规范1015.docx)
