---
title: 真机调试在线安装 ipa 包
abbrlink: 5be2f1a3
date: 2022-10-02 23:04:20
tags:
    - iOS
    - macOS
    - Objective-C
---

当我们需要真机调试时，最简单快捷的方式还是用 USB 线连接 iPhone/iPad 等设备，然后通过 Xcode 直接编译安装，如果只是一个人在开发时调试倒也没什么问题，但是如果到了测试阶段，需要分发给 N 个测试人员，这个时候还是通过 Xcode 一台一台地安装 App 就显得过于繁琐了，这种情况下我们就可以考虑在线安装的方式了，直接打开安装页面，点击一下则自动安装 App。

## ipa 安装包

首先我们需要通过 Xcode 打包出 ipa 格式的应用包，假设就叫 `myapp.ipa`，把 ipa 包上传到服务器备用。

## myapp.plist

然后我们需要准备一个应用的 `plist` 描述文件，内容如下(需要自行填写 ipa下载地址/bundle id/应用名称 的内容)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string><!-- ipa包的下载地址(实测http地址也能下载安装, 不一定非要是https) -->https://xxx.com/myapp.ipa</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>full-size-image</string>
					<key>needs-shine</key>
					<false/>
					<key>url</key>
					<string></string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>needs-shine</key>
					<false/>
					<key>url</key>
					<string></string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string><!-- 这里填 bundle id --></string>
				<key>bundel-version</key>
				<string>1.0</string>
				<key>kind</key>
				<string>software</string>
				<key>subtitle</key>
				<string></string>
				<key>title</key>
				<string><!-- 这里填App的名称 --></string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```

## install.html

有了 `.ipa` 安装包和 `.plist` 描述文件，我们还缺一个 H5 的安装页面，页面只需要提供一个超链接跳转到 `itms-services://?action=download-manifest&url=https://xxx.com/myapp.plist` 这个地址即可(url后面拼接的是 plist 文件的访问地址，改成真实地址即可)。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App下载</title>
</head>
<body>
    <div style="padding: 20px;text-align:center;font-size:30px;">我的应用名称</div>
    <div style="padding: 2px;text-align:center;font-size:15px;">版本号：1.0.0</div>
    <div align="center"  class="outFrame" style="padding: 15px;">
        <!-- 注意这里的plist文件链接必须是https  -->
        <a href="itms-services://?action=download-manifest&url=https://xxx.com/myapp.plist" style= "font-size:20px">点击此处安装</a>
    </div>
</body>
</html>
```

## 安装App

用自带的 Safari 浏览器访问 `https://xxx.com/install.html`，点击页面上的安装按钮，会弹出一个 `“xxx.com”想要安装“xxx”` 的提示弹窗，点击安装即可，如果一切顺利，稍等片刻就会把 App 自动安装到设备上了。

当然你也可以制作一个链接二维码，微信扫码打开链接，然后再通过微信提供的“在默认浏览器中打开”功能跳转到 Safari 进行下载。

> 最后提示一下，最好确保提供 `https` 的网络环境，如果你部署在 http 环境下发现不能正常安装的话，不妨试试 https 环境。
