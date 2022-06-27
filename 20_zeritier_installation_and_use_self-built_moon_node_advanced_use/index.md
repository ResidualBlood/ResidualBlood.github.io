# ZeriTier 安装使用、自建 MOON 节点、进阶使用


> 本文主要介绍内网穿透工具**ZeroTier**使用方法，全文总分为三大部分：第一部分为基本安装使用、第二部分为自建`MOON`节点，第三部分为进阶使用--访问`LAN`下的资源。主要面向已经知道`ZeroTier`是什么，并已经准备使用的人群。
## 一、基本安装及使用
### 1. 创建帐号
打开[ZeroTier控制中心](https://my.zerotier.com/network)，注册一个帐号，之后会给你一个`NETWORK ID`，如果没有点上面`Creat A Network`创建一个
![](https://dig4.lwnlh.com/image/2022/05/14/19-1.png)
### 2. 安装客户端
官网链接: [https://www.zerotier.com/download/](https://www.zerotier.com/download/)
- Windows: 下一步下一步下一步......
- Linux (Debain/CentOS): 
```
curl -s https://install.zerotier.com | sudo bash
```
- Linux (Arch): 
```
sudo pacman -S zerotier-one
yay -S zerotier-gui-git # GUI 可选
```
GUI 长这样
![](https://dig4.lwnlh.com/image/2022/05/14/19-2.png)
- 群晖: [http://download.zerotier.com/RELEASES/](http://download.zerotier.com/RELEASES/), 展开到 `/1.8.3/dist/synology/` 下载对应版本然后套件中心手动安装。(看似版本挺新的实际上是1.4.0)
### 3. 使用
- Linux:

启动服务并允许开机启动
```
systemctl start zerotier-one.service
systemctl enable zerotier-one.service
```
加入网络
```
zerotier-cli join 你的NETWORK ID
```
- 群晖: 

套件里启用后 ssh 运行
```
zerotier-cli join 你的NETWORK ID
```
问就是GUI不好使
- Windowns: 直接在GUI上加就行
- OpenWrt: 基本和 Linux 一致，不过一般编译好的固件有 luci，直接在那里把 `NETWORK ID` 加上就行
### 4. 测试
在 Menbers 选项里，把对应的客户端`Auth?`勾上，后面就是这个客户端对应的内网IP，此时你Ping一下，应该就通了，延迟一般就是你两地直连延迟，ZeroTier 服务器实际上是起到一个牵线搭桥的作用，你的两个客户端则是 P2P 打洞，如果你的 Ping 超过 2.300ms 了，说明走的是 ZeroTier 的服务器中转了，此时建议自建一个国内节点的 MOON 服务器，见下文第二部分。
![](https://dig4.lwnlh.com/image/2022/05/14/19-3.png)

## 二、自建 MOON 节点
推荐国内服务器，或者有 公网IP 的
至于为啥叫 MOON 节点，ZeroTier 就是么叫的(:
### 1. 加入网络
参照第一部分，先把网络加入了
### 2. 生成配置文件
进入到 ZeroTier 安装目录
```
cd /var/lib/zerotier-one
```
生成
```
sudo zerotier-idtool initmoon identity.public > moon.json
```
> Windows: C:\ProgramData\ZeroTier\One

> Macintosh: /Library/Application Support/ZeroTier/One (在 Terminal 中应为 /Library/Application\ Support/ZeroTier/One)

> Linux: /var/lib/zerotier-one

> FreeBSD/OpenBSD: /var/db/zerotier-one
### 3. 修改配置文件
```
nano moon.json
```
把 `stableEndpoints` 修改为你的 公网IP
```
"stableEndpoints": [ "1.1.1.1/9993" ] #修改为VPS公网IP/9993
```
### 4. 生成MOON文件
```
sudo zerotier-idtool genmoon moon.json
```
之后会生成一个`000000xxxx.moon`的文件，`xxxxx`就是你 MOON节点 的 ID，记下来留用，文件也可以拷贝出来备份

重启下服务
```
sudo systemctl restart zerotier-one.service
```
此时你的服务器就成为了 MOON节点
### 5. 其余客户端加入 MOON节点
```
sudo zerotier-cli orbit xxxxx xxxxx
```
其中,`xxxxxx`就是你上面记下的 MOON节点ID，没错，要写两遍，返回`200,xx，OK`则加入成功，如果一直加入失败，把上面那个`00000xxx.moon`文件拷贝到安装目录下的`moons.d`,再重启下服务
### 5. 检查是否加入 MOON节点
运行
```
sudo zerotier-cli listpeers
```
检查你的节点列表里是否有 MOON节点
![](https://dig4.lwnlh.com/image/2022/05/14/19-4.png)
如果一直没有，重启下服务再看

## 三、进阶: 访问LAN下资源
一般来讲，穿透都是客户端对客户端，但要是 ZeroTier 挂在路由器下，那么如何访问路由器下的 NAS 等其他资源呢？很简单，在 ZeroTier 后台配置一下就行。
![](https://dig4.lwnlh.com/image/2022/05/14/19-5.png)
在`Advanced`界面，`ADD Routes`下,
- `Destination`为你路由器网段，一般家用为`192.168.1.1`那么这里就填`192.168.1.0/24`
- `(Via)`则填写 ZeroTier 给你分配的客户端的 内网IP 地址![](https://dig4.lwnlh.com/image/2022/05/14/19-6.png)
Ping 一下，看看是不是就通了

高版本的`ZeroTier`已经帮你配置好了防火墙，基本不用管其他问题，直接用就行

> 参考：  
> - ZeroTier: [https://www.zerotier.com/download/](https://www.zerotier.com/download/)  
> - ZqinKing: [https://post.smzdm.com/p/adwrepgk/](https://post.smzdm.com/p/adwrepgk/)

