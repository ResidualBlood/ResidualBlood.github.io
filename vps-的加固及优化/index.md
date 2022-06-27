# VPS 的加固及优化


#VPS加固
##一、SSH防爆破
顾名思义，防止SSH密码被爆破以及减小VPS压力。
这里我们使用`fail2ban`，各位应该都听说过这个程序，可以根据SSH登录失败记录来屏蔽某个IP，该脚本有以下几个功能：
1. 自助修改SSH端口
1. 自定义SSH尝试连接次数
1. 自定义最高封禁IP的时间（以小时为单位）
1. 一键完成SSH防止暴力破解
###安装
运行命令
```bash
wget --no-check-certificate https://raw.githubusercontent.com/FunctionClub/Fail2ban/master/fail2ban.sh && bash fail2ban.sh 2>&1 | tee fail2ban.log
```
根据提示输入相应信息;
1. 第一步：选择是否修改SSH端口。 
1. 第二步：输入最多尝试输入SSH连接密码的次数 
1. 第三步：输入每个恶意IP的封禁时间（单位：小时） 
###卸载
运行命令
```bash
wget --no-check-certificate https://raw.githubusercontent.com/FunctionClub/Fail2ban/master/uninstall.sh && bash uninstall.sh
```
注意事项：
1. 如果你需要更改SSH端口，请记得在防火墙或者安全组中开放新的SSH端口
1. 安装完成后请会重启SSH服务，请重新连接SSH会话
1. 若出现SSH无法连接的情况，请检查是否修改过SSH端口，请填写写改后的正确端口进行连接
> 反正我是没用这个，因为之前有一次把我自己的IP屏蔽了一天，23333
##二、使用密钥登录VPS
密钥的安全性可谓是远大于密码登录，所以，为什么不使用密钥登录呢？还不用担心自己把自己ban掉......
###生成密钥对
在你的VPS或者本地Linux系统运行命令：
```bash
ssh-keygen -t rsa
```
通过上述方式，密钥生成在`/root/.ssh`或`/你的用户根目录/.ssh`下，分别为私钥和公钥文件，公钥文件`id_rsa.pub`就是在接下来需要放置在要VPS的文件，而`id_rsa`就是要保存到本地的私钥文件。
###将密钥保存到VPS
我们需要将`id_rsa.pub`公钥文件放置到VPS用户主目录`/root/.ssh`下，可以使用`scp`命令，例：`scp 本地文件 用户名@VPS地址：远程文件路径`，之后我们需要进行一下操作
1. 将`id_rsa.pub`文件，重命名为 `authorized_keys`;
1. 执行`chmod 600 ./authorized_keys` 命令，修改权限；
1. 执行命令 `sudo nano /etc/ssh/sshd_config`进行配置，将`RSAAuthentication` 和 `PubkeyAuthentication` 后面的值都改成`yes` ，将`PasswordAuthentication yes `修改成 `PasswordAuthentication no`，然后保存；
1. 重启sshd服务，Debian/Ubuntu执行`sudo /etc/init.d/ssh restart `；
1. CentOS执行：`sudo /etc/init.d/sshd restart`；
1. 注：centos7重启服务方式与以前不同，请执行`systemctl restart sshd.service`。

最后，你就可以使用密钥登录你的VPS了～
> 这是一般VPS的修改方法，但是类似Google Cloud之类的实在你创建实例之前就让你填写公钥的。

#VPS优化
这里主要讲解开启魔改版BBR，为什么是魔改版呢？从玄学角度讲因为它比普通版更快呀～
##普通VPS开启魔改版BBR
首先，我们要确认一下VPS是否能开启BBR，可以直接使用此命令进行开启
```bash
wget --no-check-certificate -qO 'BBR.sh' 'https://moeclub.org/attachment/LinuxShell/BBR.sh' && chmod a+x BBR.sh && bash BBR.sh -f
```
**注意:执行此命令会自动重启.**
之后我们执行以下命令安装魔改版BBR
```bash
wget --no-check-certificate -qO 'BBR_POWERED.sh' 'https://moeclub.org/attachment/LinuxShell/BBR_POWERED.sh' && chmod a+x BBR_POWERED.sh && bash BBR_POWERED.sh
```
最后执行`lsmod |grep 'bbr_powered'`，如返回结果不为空，则安装成功。

##Google Cloud开启魔改版BBR
Google Cloud使用上面方法开启魔改版BBR会失败，原因以后再说，这里先立个Flag......
对于Google Cloud，我们需要使用以下命令：
```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/FunctionClub/YankeeBBR/master/bbr.sh && bash bbr.sh install
```
安装过程中如果出现这张图片，请选择NO 来删除其他内核(原谅我无耻的盗个图)：![](https://ws1.sinaimg.cn/large/006uw7syly1fxo0ts498sj30st0j2q3r.jpg)
然后根据提示重启系统，之后运行
```bash
bash bbr.sh start
```
即可启用魔改版BBR。
###查看魔改BBR状态
运行命令：
```bash
sysctl net.ipv4.tcp_available_congestion_control
```
如果看到有`tsunami`就表示开启成功！ 
![](https://ws1.sinaimg.cn/large/006uw7syly1fxo0wqrkwsj30mp014t8o.jpg)

> **参考**
> [使用SSH密钥登录让Linux VPS服务器更安全](https://www.iwwenbo.com/linux-ssh-authorized-keys-login/)
> [Debian/Ubuntu TCP BBR 改进版/增强版/魔改版](https://moeclub.org/2017/06/24/278/?v=417)
> [魔改版BBR一键安装脚本 For Debian8 / Ubuntu16 +](https://ylws.me/tech/68.html)
