# 伪 NAS 利用 VPS 实现备份、下载、串流播放一条龙(上)


> 本文主要讲述使用VPS安装Nextcloud、DirectoryLister、Aria2、AriaNg等服务来实现备份、下载、串流播放等功能......

#新建虚拟主机
首先我们需要新建三个虚拟主机、假设三个虚拟主机网址分别为`cloud.test.com`、`dd.test.com`、`ar.test.com`,
在`lnmp`目录下运行：
```bash
./vhost.sh
```
接着根据提示创建即可
#安装DirectoryLister
LNMP创建的网站根目录为`/data/wwwroot/dd.test.com`
##官网下载安装
[DirectoryLister官网](https://www.directorylister.com/)
##使用魔改版DirectoryLister
这里的魔改版DirectoryLister由[逗比](https://github.com/ToyoDAdoubi/DirectoryLister)大佬提供
网站根目录运行：
```bash
git clone https://github.com/ToyoDAdoubi/DirectoryLister.git
```
##文件结构
```
/data/wwwroot/dd.test.com
├─ resources
│   ├ themes
│   │ └ bootstrap
│   │    └ .....
│   │
│   ├ DirectoryLister.php
│   ├ config.php
│   └ fileTypes.php
│
├ README.html # 文件夹内的 说明简介文件 #
├ index.php
│
├─ 测试文件夹
│   ├ 测试文件.txt
│   └ README.html # 文件夹内的 说明简介文件 #
│
└ 测试文件.txt
  
 绝对路径分别为：
/data/wwwroot/dd.test.com/resources/
/data/wwwroot/dd.test.com/README.html
/data/wwwroot/dd.test.com/index.php
/data/wwwroot/dd.test.com/测试文件夹/
/data/wwwroot/dd.test.com/测试文件.txt
```
> 一些注意事项见中篇
#安装Aria2及AriaNG
##安装依赖包
首先我么需要安装以下依赖包，来使Aria2能够下载图中的文件类型
```bash
sudo apt install wget screen unzip gcc
sudo apt install -y libgnutls-dev nettle-dev libgmp-dev libssh2-1-dev libc-ares-dev libxml2-dev  zlib1g-dev libsqlite3-dev pkg-config
```
![](https://ws1.sinaimg.cn/large/006uw7syly1fxrhlf6lebj30nj05cdfy.jpg)
##安装Aria2
###方式一、手动编译安装
运行以下命令：
```bash
sudo apt install auto-apt checkinstall
cd /root
wget https://github.com/aria2/aria2/releases/download/release-1.33.1/aria2-1.33.1.tar.gz
tar xzvf aria2-1.33.1.tar.gz
cd aria2-1.33.1
auto-apt run ./configure
make
checkinstall
```
这样会生成一个deb包，卸载和重新安装就非常方便了
###方式二、一键脚本安装（建议）
还是推荐一键脚本安装，比手动编译会少很多问题.....
使用逗比大佬的一键脚本：
```bash
wget -N --no-check-certificate https://softs.loan/Bash/aria2.sh && chmod +x aria2.sh && bash aria2.sh

# 如果上面这个脚本无法下载，尝试使用备用下载：
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/aria2.sh && chmod +x aria2.sh && bash aria2.sh
```
根据脚本提示安装即可
> 其中，`aria2.conf`配置文件需要做一些修改，我会另开一篇文章详细说明
#安装AriaNG
到网站根目录，这里是`ar.test.com`运行
```bash
wget https://github.com/mayswind/AriaNg/releases/download/1.0.0/AriaNg-1.0.0.zip
```
之后解压，然后访问你的域名填写Aria2的密钥即可
#安装Nextcloud
##添加MySQL
在安装Nextcloud之前，我们需要在MySQL中添加一下账户及数据库
访问你的IP，会出现OneinStack的界面![](https://ws1.sinaimg.cn/large/006uw7syly1fxri4shvu9j30zc0j2wgq.jpg)
点击`phpMyAdmin`，登陆后，点击账户，新增用户账户，然后按照如图所示操作
![](https://ws1.sinaimg.cn/large/006uw7syly1fxria4fofyj30s90gmwg8.jpg)
最后点击最下面的`执行`
这样，MySQL部分完毕
##安装Nextcloud
首先进入到网站根目录`cloud.test.com`
然后下载Nextcloud
```bash
wget https://download.nextcloud.com/server/releases/nextcloud-14.0.4.zip
```
下载完后解压，输入域名进行接下来的安装即可
至此，Nextcloud安装完毕
> 安装错误等问题见中篇......


<!--more-->


> 参考：  
> [一个逗比魔改的Directory Lister~](https://github.com/ToyoDAdoubi/DirectoryLister)  
> [AriaNg](https://github.com/mayswind/AriaNg)  
> [『原创』BT/种子/磁力链接下载工具 —— Aria2 一键安装管理脚本](https://doub.io/shell-jc4/)
