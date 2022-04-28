---
title: macOS中搭建PHP+MySQL环境
tags:
  - macOS
abbrlink: 7083edfd
date: 2019-09-17 10:16:35
---

> 请先安装 [HomeBrew](https://brew.sh) 工具

## 安装PHP

macOS自带了PHP，不过自带的PHP版本都是比较老的，我们可以通过brew工具来安装PHP并且不会与系统自带的PHP产生冲突。

- 安装brew提供的最新版PHP(未必是PHP最新版本，只能是brew提供的最新版本)
```
brew install php@7.4
```

- 替换默认版本

```
echo 'export PATH="/usr/local/opt/php@7.4/bin:$PATH"' >> ~/.zshrc
echo 'export PATH="/usr/local/opt/php@7.4/sbin:$PATH"' >> ~/.zshrc

source ~/.zshrc
```

如果你用的是 bash, 请修改 `.bash_profile`

- 修改 apache 配置文件

```
sudo vi /etc/apache2/httpd.conf
```

添加以下内容

```
LoadModule php7_module /usr/local/opt/php@7.4/lib/httpd/modules/libphp7.so

<FilesMatch .php$>
    SetHandler application/x-httpd-php
</FilesMatch>

<IfModule php7_module>
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps
    <IfModule dir_module>
        DirectoryIndex index.html index.php
    </IfModule>
</IfModule>
```

`#LoadModule php7_module libexec/apache2/libphp7.so` 这行保持注释, 打开这个配置则加载的是系统自带的 PHP 版本。

- apachectl命令的使用

启动Apache
```
sudo apachectl start
```

重启Apache
```
sudo apachectl restart
```

关闭Apache
```
sudo apachectl stop
```

- 站点根目录

默认位置：/Library/WebServer/Documents

## 安装MySQL

安装(如果省略版本号, 默认安装的是8.x)
```
brew install mysql@5.7
```

启动MySQL
```
mysql.server start
```

重启MySQL
```
mysql.server restart
```

关闭MySQL
```
mysql.server stop
```

修改root密码
```
set password for root@localhost = password('pwd');
flush privileges;
```

## 安装phpMyAdmin(目前下载的是4.9.0.1版本)

- 从这里下载 [phpMyAdmin](https://www.phpmyadmin.net)，解压并重命名为phpmyadmin
- 将phpmyadmin文件夹拷贝到 `/Library/WebServer/Documents`

#### 允许空密码登录

MySQL默认是空密码的，而phpMyAdmin默认是不允许空密码登录的。

方案一：
- 修改 `phpmyadmin/libraries/config.default.php` 文件，将 `$cfg['Servers'][$i]['AllowNoPassword']` 的值改为 `true`

方案二：(推荐)
- 将 phpmyadmin/config.sample.inc.php 复制一份并重命名为 `phpmyadmin/config.inc.php`
- 将 `$cfg['Servers'][$i]['AllowNoPassword']` 的值改为 `true`

#### 登录报错

- mysqli_real_connect(): (HY000/2002): No such file or directory
	* 解决办法是将 `$cfg['Servers'][$i]['host']` 的值改成 `127.0.0.1`;

- mysqli_real_connect(): The server requested authentication method unknown to the client [caching_sha2_password]
	* 命令行登录MySQL并修改用户密码验证方式
	```
	mysql -uroot -p -h 127.0.0.1
	ALTER USER 你的用户名@localhost IDENTIFIED WITH mysql_native_password BY '你的密码';
	FLUSH PRIVILEGES;
	```

MySQL8.0.13 默认的身份验证方式是：`caching_sha2_password`，PHP的mysqli扩展不支持新的 caching_sha2_password 身份验证功能，需要将验证方式更改为 `mysql_native_password`
