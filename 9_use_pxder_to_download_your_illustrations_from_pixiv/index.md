# 利用 Pxder 批量下载 Pixiv 你收藏的插画


> Pxder是一款利用Node.js编写的能够批量下载你在P站收藏，或者喜欢的画师插画的下载器，其开源地址为：[https://github.com/Tsuk1ko/pxder](https://github.com/Tsuk1ko/pxder)

>本篇主要讲解在Ubuntu16.04下使用，Windows下可参考上面的地址


<!--more-->

##安装Node.js环境
```bash
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
```
接着安装Pxder
```bash
npm i -g pxder
```
卸载的话运行
```bash
npm uninstall -g pxder
```
##配置
首先我们需要配置一下账户及下载设置
###登录
```bash
pxder --login
```
接着输入你的账户密码
###登出
```bash
pxder --logout
```
###下载设置
```bash
pxder --setting
```
然后你会看到以下输出
```
Pxder Options

[1] Download path	/pixiv        # 下载目录，必须设置
[2] Download thread	5             # 下载线程数
[3] Download timeout	30            # 下载超时
[4] Auto rename		off           # 自动重命名（文件夹）
[5] Proxy		Disable       # 使用代理
[0] Exit

Press a key [1...5 / 0]: 0
```
输入相关序号进行设置就好，最后输入0退出
##使用
###下载公开收藏
```bash
pxder -b
# 或
pxder --bookmark
```
插画会被保存在`[bookmark] Public`文件夹中

###下载私密收藏
```bash
pxder -B
# 或
pxder --bookmark--private
```
插画会被保存在`[bookmark] Private`文件夹中


###下载公开关注的画师的所有作品
```bash
pxder -f
# 或
pxder --follow
```
###下载私密关注的画师的所有作品
```bash
pxder -F
# 或
pxder --follow--private
```

###下载或更新某画师的所有插画作品
使用`-u`或`--uid`参数，后跟画师的 `UID`，可单个可多个，如果多个则用英文半角逗号隔开
```bash
pxder -u uid1,uid2,uid3,...
例：
pxder -u 5899479,724607,11597411
```
###更新已下载的画师的画作
```bash
pxder -U
# 或
pxder --update
```

###下载指定 PID 插画
```bash
pxder -p pid1,pid2,pid3,...
例：
pxder -p 70593670,70594912,70595516
```
插画会被保存在`PID`文件夹中

有问题欢迎评论留言～
