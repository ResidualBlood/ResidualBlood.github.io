# 伪-NAS 利用VPS实现备份、下载、串流播放一条龙(中)


> 本文主要讲述Directory Lister、Aria2、Nextcloud的配置问题


<!--more-->


#一、Directory Lister篇 来自逗比大佬
#不显示文件和目录
如果安装 lnmp一键包上传Directory Lister后，Directory Lister不显示文件和目录，那么可能是 PHP函数 `scandir `被禁用了，取消禁用即可。
```
sed -i 's/,scandir//g' /usr/local/php/etc/php.ini
# 取消scandir函数禁用
/etc/init.d/php-fpm restart
# 重启 PHP生效
```
##简介功能说明
使用这个功能，需要打开 `resources\themes\bootstrap\index.php` 文件，找到第五行的：
```
$md_path = explode("com", $md_path_all);
```
把 `com` 改成你的域名后缀(比如 `xxx.cn` 就是改成 `cn` )，当初只是自用，现在一开源，我给忘了。

反正就是每个文件夹下面放一个 `README.html` 文件，这个文件里写着 简介说明内容即可。

为了避免中文乱码，把 `README.html` 文件用 `UTF-8 无BOM 编码 `保存！
##文件修改说明
文件修改说明修改网站中头部导航标题，去这个文件里搜索 `DOUBI Soft` 然后全部替换为自己要改的。
```
\resources\DirectoryLister.php
```
修改网站标签栏的标题，去这个文件里把开头 `<title> `标签中的` DOUBI Soft` 替换为自己要改的。
```
\resources\themes\bootstrap\index.php
```
修改网站顶部公告栏内容，去这个文件里搜索 `顶部公告栏` 或者找到第 72 行。
```
\resources\themes\bootstrap\index.php
```
网站头部公共文件：
```
\resources\themes\bootstrap\default_header.php
```
网站底部公共文件：
```
\resources\themes\bootstrap\default_footer.php
```
如果想要插入流量统计代码，那只需要把代码写到 `default_header.php` 文件内即可。
#二、Aria2配置
这里贴上我的Aria2配置，文件路径`/root/.aria2.conf`
```
## '#'开头为注释内容, 选项都有相应的注释说明, 根据需要修改 ##
## 被注释的选项填写的是默认值, 建议在需要修改时再取消注释  ##

## 文件保存相关 ##
# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=/data/wwwroot/dd.test.com/download
# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
#disk-cache=32M
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持
# NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
# file-allocation=none
# 断点续传
continue=true

## 下载连接相关 ##
# 最大同时下载任务数, 运行时可修改, 默认:5
max-concurrent-downloads=25
# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=5
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M
# 单个任务最大线程数, 添加时可指定, 默认:5
split=20
# 整体下载速度限制, 运行时可修改, 默认:0
#max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
#max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
max-overall-upload-limit=1M
# 单个任务上传速度限制, 默认:0
#max-upload-limit=1000
# 禁用IPv6, 默认:false
disable-ipv6=false

## 进度保存相关 ##
# 从会话文件中读取下载任务
input-file=/root/.aria2/aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=/root/.aria2/aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
save-session-interval=60

## RPC相关设置 ##
# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
#event-poll=select
# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800
# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
rpc-secret=你自己设置一个密码
# 设置的RPC访问用户名, 此选项新版已废弃, 建议改用 --rpc-secret 选项
#rpc-user=<USER>
# 设置的RPC访问密码, 此选项新版已废弃, 建议改用 --rpc-secret 选项
#rpc-passwd=<PASSWD>
# 是否启用 RPC 服务的 SSL/TLS 加密,
# 启用加密后 RPC 服务需要使用 https 或者 wss 协议连接
rpc-secure=true
# 在 RPC 服务中启用 SSL/TLS 加密时的证书文件(.pem/.crt)
rpc-certificate= /usr/local/nginx/conf/ssl/ar.test.com.crt
# 在 RPC 服务中启用 SSL/TLS 加密时的私钥文件(.key)
rpc-private-key= /usr/local/nginx/conf/ssl/ar.test.com.key

## BT/PT下载相关 ##
# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=51413
# 单个种子最大连接数, 默认:55
#bt-max-peers=55
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=true
# 打开IPv6 DHT功能, PT需要禁用
#enable-dht6=false
# DHT网络监听端口, 默认:6881-6999
#dht-listen-port=6881-6999
# 本地节点查找, PT需要禁用, 默认:false
#bt-enable-lpd=true
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=true
# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K
# 客户端伪装, PT需要
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=0.000001
# 强制保存会话, 即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
#force-save=false
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
#bt-save-metadata=true
```
#注意事项
##一、保存路径
保存路径最好放在`Directory Lister`目录下，这样方便直接访问
如果你放在其他目录，`Directory Lister`目录下放软链接可能会发生错误。
##进度保存
记得把进度保存相关设置前的注释取消掉，就像配置文件第38~44行
##RPC相关设置
密码在第58行修改
由于我们使用全站https，所以在配置文件第67、69行需要填写你的网站证书路径，按照我的上一篇文章建站的话，把`ar.test.com`替换成你的域名即可。
##定时更新bt-tracker
如果你使用逗比大佬的脚本并开启了定时更新bt-tracker，那么会有更新过后Aria2没有启动问题，所以解决方法就是添加一个定时任务：
```
crontab -e
在0 3 * * 1 /bin/bash /root/aria2.sh update-bt-tracker下添加
2 3 * * 1 /bin/bash service aria2 start
```
保存即可。
#三、Nextcloud排错




#1. PHP 被设置为移除内联块, 这将导致多个核心应用无法访问
问题：准备LNMP环境和上面一样，只是在LNMP 1.4和OneinStack安装时选择Nginx环境即可，其它的组件都一样有选择性地安装。如果安装了Zend OPcache，可能在安装时会提示错误：`“PHP 被设置为移除内联块, 这将导致多个核心应用无法访问。`

解决方法：修改`php.ini`中Opcache的参数进行修改，如果是Oneinstack的话需要在`/usr/local/php/etc/php.d/02-opcache.ini` 中修改。找到此代码并改成：`opcache.save_comments=1` ，因为默认是0，改完重启php-fpm就行。
```
nano /usr/local/php/etc/php.d/02-opcache.ini
opcache.save_comments=1 #将0改成１
```

##2. 出现错误：No input file specified.
一般来说，在添加虚拟主机那一步选择`rewrite`的时候选择`nextcloud`不会出现这个问题，如果还有问题按照以下步骤来解决
编写URL地址重写规则。出现错误：No input file specified.，主要是Nginx还需要自己写重写规则，你可以将以下规则复制粘贴到/usr/local/nginx/conf/vhost/你的网站.conf，具体规则如下：
```
#(可选)添加如下header主要为了安全
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    #(可选)为了支持user_webfinger app
    rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;
 
    #日历和联系人，建议加上
    location = /.well-known/carddav {
    return 301 $scheme://$host/remote.php/dav;
    }
    location = /.well-known/caldav {
    return 301 $scheme://$host/remote.php/dav;
    }
    #设置上传文件的最大大小
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;
    #将所有请求转发到index.php上
    location / {
    rewrite ^ /index.php$uri;
    }
    #安全设置，禁止访问部分敏感内容
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
    deny all;
    }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
    deny all;
    }
 
    #默认有，替换原来的就行
    location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+)\.php(?:$|/) {
    fastcgi_split_path_info ^(.+\.php)(/.*)$;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_param modHeadersAvailable true;
    fastcgi_param front_controller_active true;
    fastcgi_pass unix:/dev/shm/php-cgi.sock; #这边我改过，参照原来的
    fastcgi_intercept_errors on;
    fastcgi_request_buffering off;
    include fastcgi.conf;
    }
 
    #安全设置，禁止访问部分敏感内容
    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
    try_files $uri/ =404;
    index index.php;
    }
 
    location ~ \.(?:css|js|woff|svg|gif)$ {
    try_files $uri /index.php$uri$is_args$args;
    add_header Cache-Control "public, max-age=15778463";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    access_log off;
    }
    location ~ \.(?:png|html|ttf|ico|jpg|jpeg)$ {
    try_files $uri /index.php$uri$is_args$args;
    access_log off;
    }
```
#3. 模块‘fileinfo’缺失
fileinfo\Memcached\Redis等都可以通过OneinStack一键安装。
具体方法见上篇
#4. 没有安装 "smbclient". 无法挂载 "SMB / CIFS"（测试未通过）
安装：
```
apt-get install smbclient
apt-get install php-smbclient
```
> 反正我安装过后问题没解决.....解决问题的朋友们可以在评论区留言


<!--more-->

#5. 内存缓存未配置
```
'memcache.local' => '\OC\Memcache\Redis',
'memcache.locking' => '\OC\Memcache\Redis',
'redis' => array(
     'host' => 'localhost',
     'port' => 6379,
      ),
```
#6. 一些文件没有通过完整性检查
参数解释如下，自行判读，
```
INVALID_HASH 　　#错误的文件，需要使用正确的替换
EXTRA_FILE 　　　#多余的文件，需要删除
EXCEPTION　　　　 #错误信息
```
**在你解压完nextcloud，并把解压后文件移动到网站根目录的时候，记得检查下是否所有文件都移动过去了，因为解压文件里有两个隐藏文件.....**
#7. 三种方式管理后台任务
三种方式管理后台任务，默认为`AJAX`,但是推荐用 `cron`
```
# www用户添加后台任务
crontab -u www -e
# 填写如下字段，注意目录地址
*/15 * * * * php -f /www/wwwroot/cloud.test.com/ cron.php

# 检查是否添加成功
crontab -u www -l
# 输出如下说明成功
*/15 * * * * php -f /www/wwwroot/cloud.test.com/ cron.php
```
#Nextcloud出现“内部服务器错误”
这个问题的出现是由于文件夹权限设置的不到位。
```
chown -R www:www  你的网站目录
```
这一点很重要！！！

> 参考：
> [一个逗比魔改的Directory Lister~](https://github.com/ToyoDAdoubi/DirectoryLister)
> [『原创』BT/种子/磁力链接下载工具 —— Aria2 一键安装管理脚本](https://doub.io/shell-jc4/)
> [手动安装NextCloud教程-免费开源的私有云存储网盘可播放图片音乐](https://wzfou.com/nextcloud-install/)
