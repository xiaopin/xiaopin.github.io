---
title: 基于阿里云 OSS 搭建 hexo 博客
tags:
  - Other
abbrlink: 62f4282a
date: 2019-03-26 14:25:28
---

时间过的真快，转眼`弹性Web托管`服务也即将到期了，但是每年 RMB199 的费用确实有点高了，所以今年不打算续费了，转战 OSS。

OSS 的计费方式分为 [按量付费](https://help.aliyun.com/document_detail/48266.html?spm=5176.208357.1107607.11.1a66390fBar6bt) 和 [包年包月](https://help.aliyun.com/document_detail/48272.html?spm=a2c4g.11186623.6.552.ca985cdacROGX2) 两种方式，由于本站访问量实在是低，流量费用简直可以说是忽略不计，所以我采用了按量付费的计价方式。

#### 开通 OSS 服务 & 新建 Bucket

- 新建 Bucket

在控制台中打开 `对象存储 OSS` 面板，新建一个 `Bucket`，注意读写权限需要设置为 `公共读`

![png](/images/201903/create-bucket.png)

新建后会在左侧面板出现你新建的 Bucket，点击进入该 Bucket 的详情。

- 配置静态网页托管

这是[文档说明](https://help.aliyun.com/document_detail/31872.html)，有兴趣你可以自行去查阅。不过说了那么多，就是在 `基础设置--静态页面` 中设置首页和404页面的文件。

![png](/images/201903/static-page-setting.png)

- 绑定自定义域名

文档戳[这里](https://help.aliyun.com/document_detail/31836.html?spm=a2c4g.11186623.4.3.71043cccPsN2Hk)

新建了 Bucket 后，会提供一个默认的对外访问域名，格式：Bucket名称 + Endpoint，如 `bucket-name.oss-cn-shenzhen.aliyuncs.com`。但是为了便于访问，我们可以将自己的域名绑定到 Bucket，直接通过自己的域名来访问资源。

![png](/images/201903/bind-domain.png)

如上图所示，注意*主域名和 www 域名需要分别绑定*。

#### 上传文件

控制台提供了上传文件的选项，可以直接通过网页上传文件，但是不建议使用该功能，因为不好用😂。如果有需要你也可以下载 SDK 自行上传文件，这里推荐下载阿里云提供的图形化工具来管理文件。

- 下载 [oss-browser](https://help.aliyun.com/document_detail/61872.html?spm=a2c4g.11186623.2.10.8bb6744e9b6EQN)

- 获取 AccessKeyId 和 AccessKeySecret

点击头像打开下拉菜单，选择 accesskeys 选项，进入安全信息管理面板，点击 `创建AccessKey` 按钮，

![png](/images/201903/dropmenu.png)

![png](/images/201903/create-access-key.png)

系统会自动为我们创建 AccessKeyId 和 AccessKeySecret，点“保存AK信息”，可将 `.csv` 结尾的文件下载到电脑，打开该文件即可看见我们的 AccessKeyId 和 AccessKeySecret，如下图：

![png](/images/201903/saveak.jpg)

通过 AccessKeyId 和 AccessKeySecret 我们就可以用 OSS-Browser 工具登录到 OSS，然后可以进行上传、删除等等操作。

上面的例子我是直接使用了阿里云账号来创建 AccessKey，这不是推荐的做法，阿里云推荐的是采用RAM子账户创建 AccessKey 来进行访问，更多关于 RAM 的使用说明请自行查阅。

关于 OSS-Browser 工具的登录教程请[点击这里](https://www.aliyunbaike.com/oss/2883/)。


#### 修改 hexo URL 规则

由于 OSS 的资源 URL 都是以绝对路径的方式进行访问的，而 hexo 生成的文章链接都是伪静态的，从而导致不管点击任何链接都会显示首页内容，比如之前的一篇文章地址为：http://www.0daybug.com/article/ ，则现在必须要加上 index.html 才能访问到该文章：http://www.0daybug.com/article/index.html 。

因此需要修改 hexo 生成文章 URL 的规则，部分可以直接在 `_config.yml` 配置文件上直接进行配置，而有些则只能通过修改 hexo 代码来完成了。

- *hexo/_config.yml* 中的 `permalink` 配置项需要添加 index.html 结尾
```bash
permalink: :year/:month/:day/:title/index.html
```

- hexo V5.2.0修改

  - 修改标签的链接：node_modules/hexo/lib/plugins/helper/tagcloud.js 在70行代码中拼接上 index.html

  - 修改文章详情内的标签链接：/node_modules/hexo/lib/plugins/helper/list_tags.js 分别在55/71行代码中拼接上 index.html

  - 修改首页底部的分页链接：/node_modules/hexo/lib/plugins/helper/paginator.js 修改 `createLink` 函数，在第8行代码中拼接 index.html

- hexo V3.8.0修改

  - 修改标签的链接：node_modules/hexo/lib/plugins/helper/tagcloud.js 在227行代码中拼接上 index.html

  - 修改文章详情内的标签链接：/node_modules/hexo/lib/plugins/helper/list_tags.js 分别在40/56行代码中拼接上 index.html

  - 修改首页底部的分页链接：/node_modules/hexo/lib/plugins/helper/paginator.js 修改 `link` 函数，在28行代码中拼接 index.html

> 针对修改 hexo 源码的问题，由于使用不同的主题会有所不同，请根据实际情况进行修改即可。上面的修改针对默认主题有效。

#### 永久链接

上面中通过指定 `permalink` 为 `:year/:month/:day/:title/index.html` 已经修改了永久链接的生成格式，但是这种格式目录太深了，并且标题是中文的话URL中也会出现中文，所以需要改进一下生成的最终链接格式。

- 安装 hexo-abbrlink 插件
```
npm install hexo-abbrlink --save
```

- 修改配置文件 _config.yml
```
permalink: posts/:abbrlink/index.html
# hexo-abbrlink 插件配置(用于生成唯一的永久链接)
abbrlink:
    alg: crc32 # 算法：crc16(default) and crc32
    rep: hex # 进制：dec(default) and hex
```

- 重新打包
```
hexo clean
hexo g
```

打包生成后你会发现原来的文章md文件已经添加了 `abbrlink` 字段，值为对应的唯一ID，所以能够确保我们修改标题、文章内容等不会改变最终的文章链接。

#### HTTPS

OSS 支持 HTTPS 访问，如果你的域名是在万网上面购买的，则可以申请阿里云的免费 HTTPS 证书。

打开 `域名` 控制台，找到你的域名，点击后面的 `管理` 按钮进入域名的基本信息页面，页面上有个 `开启免费SSL证书` 的链接，点击即可进入申请免费 SSL 证书的界面，根据你的需求请自行申请 SSL 证书。

打开 `SSL 证书（应用安全）` 控制台，可以看到你的 SSL 证书的状态，找到已签发的证书，将其下载到电脑备用（内含公钥、私钥）。

打开 `对象存储 OSS` 控制台，找你域名管理（上面的绑定自定义域名有提到），点击证书托管，将 SSL 证书的公钥、私钥填好后提交保存，即可开启 HTTPS 功能。


参考：
- [https://help.aliyun.com/document_detail/67323.html?spm=a2c4g.11186623.4.1.71043ccc1h11YJ](https://help.aliyun.com/document_detail/67323.html?spm=a2c4g.11186623.4.1.71043ccc1h11YJ)
- [https://help.aliyun.com/document_detail/31872.html?spm=a2c4g.11186623.6.646.71043cccMNtliW](https://help.aliyun.com/document_detail/31872.html?spm=a2c4g.11186623.6.646.71043cccMNtliW)
- [https://help.aliyun.com/document_detail/31899.html?spm=a2c4g.11186623.2.13.72df3ccc5fa6sJ#concept-ars-bhz-5db](https://help.aliyun.com/document_detail/31899.html?spm=a2c4g.11186623.2.13.72df3ccc5fa6sJ#concept-ars-bhz-5db)
- [https://www.aliyunbaike.com/oss/2614/]()
