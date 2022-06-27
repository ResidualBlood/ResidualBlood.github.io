# Hetzner 安装 Proxmox 开小鸡教程 - 安装篇


> 此系列主要记录 Hetzner 安装 Proxmox 开小鸡的流程，主适用于宿主机为单ipv4,多ipv6，想开小鸡为 NAT ipv4 + 独立 ipv6 的情境
> 本文为此系列第一篇 主要记录 Proxmox 的安装，以及宿主机的网络配置

## 一、准备工作
### 1.修改`hostname`
```
hostnamectl set-hostname 自己设
```
然后编辑`hosts`
```
nano /etc/hosts
```
去掉 ipv6 的主机名
![](https://dig4.lwnlh.com/image/2022/05/14/18-1.png)
你的 ipv6 主机名为红色箭头所示，如果你上面修改了主机名，那么 ipv4 对应的主机名也要修改
测试一下，返回宿主机的 ipv4 地址为正常
```
hostname --ip-address
```
### 2.开启端口转发和BBR
编辑`/etc/sysctl.conf`文件
```
nano /etc/sysctl.conf
```
加入
```
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr 
net.ipv4.ip_forward=1
net.ipv6.conf.all.accept_dad = 1
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.all.accept_redirects = 1
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.all.autoconf = 0
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.all.forwarding=1
```
上面两行作用开启 BBR ，下面的为开启转发
保存生效
```
sysctl -p
```
然后重启宿主机
```
reboot
```
> 如果你要编译安装 lnmp 之类的话，在一切开始之前先装完，以防翻车

## 二、安装 Proxmox
添加官方仓库
```
echo "deb [arch=amd64] http://download.proxmox.com/debian/pve buster pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
```
添加 GPG 密钥
```
wget http://download.proxmox.com/debian/proxmox-ve-release-6.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg
```
赋予权限
```
chmod +r /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg
```
安装
```
apt update -y && apt upgrade -y 
```
```
apt -y install proxmox-ve postfix open-iscsi
```
这个过程中可能会弹出一个postfix的配置界面，先 no，然后直接选择 Local 然后回车即可
安装完毕后， **重启！重启！重启！**

## 三、网络配置
网上有很多教程了，但这几天试下来，ipv4 NAT 没有问题，但独立ipv6一直不好用，有说直接 PVE 直接加一块儿 vmbr0 的网卡，然后就被HZ滥用警告了2333，最终沿袭这位老哥的思路[https://www.liujason.com/article/477.html](https://www.liujason.com/article/477.html)，终于搞定了 ipv6 的问题，目前使用一切正常。
下面贴一下母鸡的网络架构以及一些具体配置
### 宿主机网络基本架构
- enp9s0：物理网卡，空
- vmbr0：物理网卡桥接——公网IPv4+公网IPv6
- vmbr1：物理vLAN网卡桥接——内网IPV4+公网IPv6（无gateway）
### 网卡具体配置
编辑`/etc/network/interfaces`
```
### Hetzner Online GmbH installimage

source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback
iface lo inet6 loopback

auto enp9s0
iface enp9s0 inet manual
#  post-up /sbin/ethtool -K enp9s0 tx off rx off
iface enp9s0 inet6 manual

auto vmbr0
iface vmbr0 inet static
  address 母鸡ip
  netmask 子网掩码
  gateway 网关ip
  pointopoint 同上网关ip
  hwaddress ether MAC地址
  bridge_ports enp9s0
  bridge_stp off
  bridge_fd 0
  bridge_maxwait 0
  # post-up /sbin/ethtool -K vmbr0 tx off rx off

iface vmbr0 inet6 static
  address 母鸡v6地址  #例子 2a01:abc:abc:abc::2
  netmask 64                #6 4或者128,看你之前的网络参数
  gateway fe80::1         #网关
  bridge_ports enp9s0
  bridge_stp off
  bridge_fd 0
  up ip -6 route del 2a01:abc:abc:abc::/64 dev vmbr0   #这个是你母鸡ipv6的网段，含义说明下面有写

auto vmbr1
iface vmbr1 inet static
  address 192.168.1.1
  netmask 255.255.255.0
  bridge_ports none
  bridge_stp off
  bridge_fd 0
  post-up echo 1 > /proc/sys/net/ipv4/ip_forward
  post-up iptables -t nat -A POSTROUTING -s '192.168.1.0/24' -o vmbr0 -j MASQUERADE
  post-down iptables -t nat -D POSTROUTING -s '192.168.1.0/24' -o vmbr0 -j MASQUERADE

auto vmbr1
iface vmbr1 inet6 static
  address  另一个ipv6地址 # 例子 2a01:abc:abc:abc::3/64   
  #gateway fe80::1
  bridge_ports none
  bridge_stp off
  bridge_fd 0
  up ip -6 route add 2a01:abc:abc:abc::/64 dev vmbr1
  post-down ip -6 route del 2a01:abc:abc:abc::/64 dev vmbr1
```
解释一下一些语句的含义
- `up ip -6 route add 2a01:abc:abc:abc::/64 dev vmbr1` 是让所有的ipv6流量走vmbr1,才能将ipv6流量转发出去
- `up ip -6 route del 2a01:abc:abc:abc::/64 dev vmbr0` 是将2a01:abc:abc:abc::/64从vmbr0网桥中去掉
- `post-up iptables -t nat -A POSTROUTING -s '192.168.1.0/24' -o vmbr0 -j MASQUERADE` 是使用 NAT
再额外提一下
- `pre-up` 网卡启用前的动作
- `up` 启用时候的动作
- `post-up` 启用后的动作
- `pre-down` 关闭前的动作
- `down` 关闭时动作
- `post-down` 关闭后动作

### 重启网络服务
```
systemctl restart networking.service
```
查看网络状态
```
systemctl status networking.service
```
如果显示`Active`则一切正常，此时可以重启了
如果你在配置桥接网络这块配完了重启网络服务失败但是机器还有网，那么恭喜你还可以继续折腾，也就是你还可以继续修改网卡的配置文件，但如果你想让你新修改的配置生效就得用下面这条命令强制重启 vmbr0（networking restart是没用的）：
```
ifdown --force vmbr0 && ifup --force vmbr0
```
最后：**重启！重启！重启！**

### 验证
输入
```
ip -6 route 
```
回显如下为正常
```
::1 dev lo proto kernel metric 256 pref medium
2a01:abc:abc:abc::/64 dev vmbr1 proto kernel metric 256 linkdown pref medium
2a01:abc:abc:abc::/64 dev vmbr1 metric 1024 linkdown pref medium
fe80::/64 dev vmbr0 proto kernel metric 256 pref medium
fe80::/64 dev vmbr1 proto kernel metric 256 linkdown pref medium
default via fe80::1 dev vmbr0 metric 1024 onlink pref medium
```
你也可以 Ping 一下相关地址.....

至此 Proxmox 的安装，以及相关网络配置完成，开小鸡教程将在下一篇文章记录。

> 参考：
> - 荒岛：[https://lala.im/4821.html](https://lala.im/4821.html)
> - 老赵部落：[https://blog.faaoo.cn/archives/173](https://blog.faaoo.cn/archives/173)
> - LiuJason's Blog： [https://www.liujason.com/article/477.html](https://www.liujason.com/article/477.html)
