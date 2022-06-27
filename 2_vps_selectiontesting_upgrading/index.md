# VPS 选择，测试，升级


> 作者言  
扶墙向：本篇讲述的是如何选择VPS，以及对购买完后的VPS进行测试及系统升级......
#VPS的选择
如何选择VPS，第一要素自然是是看延迟咯，所以我们先上 [这个](https://www.msmbps.com/) 网站测试下你到一些常见的VPS供应商机房的延迟，下面给张图：
![](https://ws1.sinaimg.cn/large/006uw7syly1fxnytipiipj30pl0h8tcx.jpg)

测完延迟后就选一款适合你的.....
> 测试网站的开源地址：[https://github.com/msmbps/msmbps](https://github.com/msmbps/msmbps)
#VPS的测试
这里我们使用[秋水逸冰](https://teddysun.com/444.html)大佬的一键测试脚本来进行VPS性能的测试
命令：
```bash
wget -qO- bench.sh | bash
```
或者
```bash
curl -Lso- bench.sh | bash
```
**备注**
`bench.sh` 既是脚本名，同时又是域名。
给个图：
![](https://ws1.sinaimg.cn/large/006uw7syly1fxnz4r6nsaj30g20gy0xe.jpg)
测试完了，感觉还可以就留着，不行的的话就该退款退款，该换就换，千万别凑活着用，给自己找不自在，最后还发工单撕.....
#系统大版本升级
有时候，VPS商家给的系统模板并不是最新的，这时候我们就需要手动升级下系统（本文Ubuntu向）
首先，我们需要升级下软件包
```bash
sudo apt-get update
sudo apt-get upgrade
```
接着安装`update-manager-core`
```bash
sudo apt-get install update-manager-core
```
然后打开文件`/etc/update-manager/release-upgrades`，确认`Prompt`的值设定为`lts`：
```bash
# Default behavior for the release upgrader.

[DEFAULT]
# Default prompting behavior, valid options:
#
#  never  - Never check for a new release.
#  normal - Check to see if a new release is available.  If more than one new
#           release is found, the release upgrader will attempt to upgrade to
#           the release that immediately succeeds the currently-running
#           release.
#  lts    - Check to see if a new LTS release is available.  The upgrader
#           will attempt to upgrade to the first LTS release available after
#           the currently-running one.  Note that this option should not be
#           used if the currently-running release is not itself an LTS
#           release, since in that case the upgrader won't be able to
#           determine if a newer release is available.
Prompt=lts
```
确认完毕后，我们需要处理一下可能阻碍升级的依赖关系，所以运行
```bash
sudo apt-get dist-upgrade
```
最后，一切准备就绪，使用Ubuntu的发行工具执行升级
```bash
sudo do-release-upgrade
```
或
```bash
sudo do-release-upgrade -d
```
至此，静待系统升级完毕，期间需要重启系统，重启完毕后，使用
```
lsb_release -a
```
可以看到你的系统升级后的版本

> **参考**
> [秋水逸冰一键测试脚本bench.sh](https://teddysun.com/444.html)  
> [How To Upgrade to Ubuntu 16.04 LTS](https://www.digitalocean.com/community/tutorials/how-to-upgrade-to-ubuntu-16-04-lts)  
> [Linode VPS从Ubuntu Server 14.04LTS升级到Ubuntu Server 16.04LTS](https://www.daweibro.com/node/154)

