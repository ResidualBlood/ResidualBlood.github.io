# 利用SSH跳板机连接内网服务器


## 一、前言
由于本地网络环境实在太差，VSCode Remote 连接编译机无限重试，所以就用ssh跳板抢救一下。
所有命令基于`open-ssh`
## 二、直接连接
```bash
ssh username@目标机器ip -p 22 -o ProxyCommand='ssh -p 22 username@跳板机ip -W %h:%p'
```

例子：

```bash
ssh root@192.168.1.2 -p 22 -o ProxyCommand='ssh -p 22 root@10.1.1.1 -W %h:%p'
```

## 三、添加`config`
每次登录都输入上面命令显然很麻烦，所以可以将配置写在`~/.ssh/config`里，如果没有这个文件新建一个
```bash
nano ~/.ssh/config
```

例子：

```bash
Host jump                    #任意名字，随便使用
    HostName 192.168.1.1        #这个是跳板机的IP，支持域名
    Port 22                     #跳板机端口
    User username_jump       #跳板机用户

Host target                     #同样，任意名字，随便起
    HostName 192.168.1.2        #真正登陆的服务器，不支持域名必须IP地址
    Port 22                     #服务器的端口
    User username               #服务器的用户
    ProxyCommand ssh jump -W %h:%p
```
## 四、使用
直接输入`ssh target`即可，同理连接`跳板`直接输入`ssh jump`

>参考： [ssh 通过跳板机直连跳板机内网服务器](https://outmanzzq.github.io/2018/11/20/ssh-connect-through-springboard/)


