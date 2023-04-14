---
title: 宝塔 Flarum 论坛的搭建与迁移
date: 2023-04-14 21:30:27
tags: [Flarum]
---
:::tips
本文的重点主要记录在宝塔搭建的Flarum论坛的迁移，搭建过程简略带过。论坛的搭建的详细过程可以参考《[基于宝塔快速搭建 Flarum 轻量级论坛网站，附图文安装教程](https://zhuanlan.zhihu.com/p/499677300)》这篇文章
:::
![image.png](https://raw.githubusercontent.com/wrm244/image/main/2023/202304142254997.png)

什么是 `Flarum`？`Flarum` 是一款基于 `PHP Laravel` 框架开发的论坛网站，拥有轻量、美观、响应式、易上手等特点。
# 部署
## 服务器环境
- 宝塔 7.0.3 或更新版本本文用的是 7.9.6
- Linux Server（本文用的是 CentOS 7 迁移到 Debin 11）
- Apache 或者 Nginx（本文用的是 Nginx 1.22.1）
- MySQL 5.6+（本文使用 MySQL 8.0.24）
- PHP 7.1+（本文 PHP-8.0.24）
### 搭建宝塔面板环境
[宝塔官网](https://www.bt.cn/new/download.html)有详细的脚本教程，这里提供万能安装脚本：

```shell
if [ -f /usr/bin/curl ];then curl -sSO https://download.bt.cn/install/install_panel.sh;else wget -O install_panel.sh https://download.bt.cn/install/install_panel.sh;fi;bash install_panel.sh ed8484bec
```
:::success
`shell`脚本命令太长换行输入操作：`\`
:::
### 配置 PHP
#### 开启PHP扩展
进入软件商店找到 PHP 并打开设置，选择`安装扩展`安装 fileinfo、opcache、exif ，其中`fileinfo`主要用来读取用户上传的图片与文件信息，没有扩展上传文件到论坛会失败

#### 调大PHP脚本内存
进入配置修改，把`memory_limit`参数根据实际情况调大

#### 解禁PHP函数
接下来我们需要对 3 个函数进行禁用解除，在 PHP 设置页面选择 ”禁用函数“，删除掉 `putenv`、`pcntl_signal`、`proc_open` 这三个函数
### 安装 Composer
Flarum 使用 Composer 来管理它的目录和扩展，所以在安装 Flarum 之前，您需要安装下载 Composer 在您的主机上。
1. 进入用户家目录
```shell
cd ~
```
2. 将安装脚本下载到当前目录
```php
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
```
3. 运行安装脚本
```php
php composer-setup.php
```
4. 删除安装脚本
```php
php -r "unlink('composer-setup.php');"
```
5. 全局安装 composer（配置系统环境变量）
```shell
mv composer.phar /usr/local/bin/composer
```
6. 将 composer 源改成阿里云的镜像，如果您使用的是国外服务器此步骤可省略！
```shell
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```
7. 如果当然用户是 root 用户，会提示你不要用 root 用户操作 composer，回复 ”yes“ 继续使用 root
## 安装 Flarum
### 下载 Flarum 程序
因为 Flarum 要求安装目录必须是空目录，因此我们还需要删除刚刚新建网站目录里的所有文件。
使用 SSH 工具连接服务器，进入网站目录。注意网站目录每个人都不一样！记得替换xxx！
```shell
cd /www/wwwroot/xxxx
```
解除 .user.ini 的文件锁定，否则该文件无法删除
```shell
chattr -i .user.ini
```

:::tips
chattr 命令，专门用来修改文件或目录的隐藏属性，只有 root 用户可以使用。该命令的基本格式为：
```
[root@localhost ~]# chattr [+-=] [属性] 文件或目录名
```
+ 表示给文件或目录添加属性，- 表示移除文件或目录拥有的某些属性，= 表示给文件或目录设定一些属性。
:::

进入宝塔面板，找到对应的站点并点击根目录，全选并删除根目录下所有的文件，也可以使用 FTP 工具进入并删除。

使用 SSH 工具连接服务器，并进入网站根目录，使用 composer 下载 Flarum 程序（确保在网站根目录执行）。
```shell
composer create-project flarum/flarum .
```
等待安装成功
## 修改网站 nginx 配置
需要对flarum进行伪静态配置。进入宝塔面板，找到网站设置并选择配置文件，按照下面配置进行修改，具体域名地址请按照自己的实际情况进行修改。
```nginx
include /www/wwwroot/path/to/.nginx.conf;
```
## 安装过程中的注意事项
1. 下载好的Flarum部署文件后要对此的文件给予读写执行的权限
2. 迁移过程可由低环境换到高环境，前提是避免环境版本的不兼容
# 迁移
## 备份文件
在宝塔上备份需要迁移的Flarum**网站文件**以及**数据库文件**

将备份好的文件下载到电脑上

## 重新部署Flarum(见本文部署内容)
在新的服务器部署一个新的Flarum

新服务器的环境需要和之前的保持一致或者高版本，否则可能会有错误产生

部署到可以访问到页面以及绑定了新数据库之后

## 恢复数据
部署完新的Flarum后，上传原Flarum网站文件压缩包至新服务器上，解压到一个单独的文件夹里，不要直接解压覆盖新部署的Flarum。
1. 恢复网页文件
将原网站文件里的public文件夹，vender文件夹，composer.json文件，composer.lock文件覆盖至新部署的Flarum根目录

2. 恢复数据库
解压原Flarum数据库文件获取sql文件，上传并导入新的数据库中即可

3. 重新整理插件依赖
```shell
composer install
```
刷新一下网站，Flarum就完美迁移成功了

:::success
本教程为Flarum最新版本迁移方案
迁移老版本只需要将整个站点打包（除了vender文件夹）
到新的地方恢复数据库 编辑 config.php 然后 composer install
让 composer 帮你检查一次运行环境
:::


