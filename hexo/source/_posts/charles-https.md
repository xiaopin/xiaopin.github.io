---
title: Charles进行HTTPS抓包
tags:
  - 随记
abbrlink: 3c2aefa
date: 2018-12-27 15:20:41
---

> 当前环境：macOS 10.14.1 + Charles 4.2.7 + iPhone 7 iOS 12.1

## 下载安装Charles

前往 [https://www.charlesproxy.com/download/](https://www.charlesproxy.com/download/) 下载对应平台的 Charles 软件。

## 配置HTTPS证书

1、打开 Charles，选择 Help -- SSL Proxying -- Install Charles Root Certificate 选项

![](/images/201812/charles-01.png)

2、系统会自动打开“钥匙串访问”应用，我们需要双击 "Charles Proxy CA" 证书，然后选择 “始终信任” 该证书

![](/images/201812/charles-02.png)

3、手机安装 HTTPS 证书，选择 Help -- SSL Proxying -- Install Charles Root Certificate on a Mobile Device or Remote Browser 选项即会弹出一个提示框，根据提示信息进行操作

![](/images/201812/charles-03.png)

3.1、设置手机的代理为 192.168.100.101:8888 (这是我电脑的IP，请根据提示信息填写即可)

![](/images/201812/charles-04.png)

3.2、使用 Safari 浏览器打开 `chls.pro/ssl` 页面，会弹窗一个提示框，点击 “允许”，然后根据提示进行安装手机端证书

![](/images/201812/charles-05.png)

![](/images/201812/charles-06.png)

![](/images/201812/charles-07.png)

安装完成后如上图所以。

3.3、对于 iOS 10.3 及以上的系统需要到 “设置--通用--关于本机--证书信任设置” 中对 Charles Proxy 进行授权信任才能正常使用

![](/images/201812/charles-08.png)

## 进行HTTPS抓包

打开 Charles，选择 Proxy -- SSL Proxying Settings 选项，添加你需要抓包的域名(*表示抓包所有域名)

![](/images/201812/charles-09.png)

***

至此，所有配置已设置好，可以愉快地抓包了~~~
