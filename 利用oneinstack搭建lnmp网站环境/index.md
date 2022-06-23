# 利用OneinStack搭建LNMP网站环境


> 作者言：没啥营养，不看也罢......
#一、安装
这里提供两种方式安装
##图形化安装
这里我们使用 [官网](https://oneinstack.com/auto/) 的一键脚本安装。在下图中选择你所需要的功能，然后复制安装命令到你的VPS上运行即可。
![](https://ws1.sinaimg.cn/large/006uw7syly1fxr44q36e3j30wd0w9q5n.jpg)
##终端安装
首先，我们要安装一下所需要的软件包
```bash
apt-get -y install wget screen curl python
```
接着下载所需要的一键脚本，解压
```bash
wget http://mirrors.linuxeye.com/oneinstack.tar.gz
tar xzf oneinstack.tar.gz
```
然后进入解压后的文件中
执行
```bash
screen -S lnmp
```
防止SSH连接中断导致安装失败,最后运行
```bash
./install.sh
```
接下来
按照按照提示选择你所需要的组件（又见盗图23333）
![](https://ws1.sinaimg.cn/large/006uw7syly1fxr4g2v2ywj318o2m4akq.jpg)
#二、添加虚拟主机
运行
```bash
./vhost.sh
```
然后按照提示配置你的虚拟主机
![](https://ws1.sinaimg.cn/large/006uw7syly1fxr4hgxo0hj318o2707ej.jpg)
#三、删除虚拟主机
运行
```bash
./vhost.sh del
```
然后按照提示输入即可
#四、管理服务
**Nginx/Tengine/OpenResty:**
```
service nginx {start|stop|status|restart|reload|configtest}
```
**MySQL/MariaDB/Percona:**
```
service mysqld {start|stop|restart|reload|status}
```
**PHP:**
```
service php-fpm {start|stop|restart|reload|status}
```
**Apache:**
```
service httpd {start|restart|stop}
```
**HHVM:**
```
service supervisord {start|stop|status|restart|reload}
注：hhvm进程交给supervisord管理，了解更多请访问《Supervisor管理hhvm进程》
```
**Pure-Ftpd:**
```
service pureftpd {start|stop|restart|status}
```
**Redis:**
```
service redis-server {start|stop|status|restart|reload}
```
**Memcached:**
```
service memcached {start|stop|status|restart|reload}
```
#五、大版本更新
运行
```bash
./upgrade.sh
```
按照提示输入即可

> **参考** 
> [lnmp、lamp、lnmpa一键安装包](https://blog.linuxeye.cn/31.html)
> [OneinStack](https://oneinstack.com/download/)
