# 


# 使用 SSH隧道 绕过 PLEX 对 Hetzner 的封锁
## 确定使用环境
1. 一台 docker 宿主机, 一台 反代机
2. PLEX 通过 反代机 Nginx 反代, 且 远程访问 已关闭
3. 宿主机能够 SSH 访问 反代机

## 修改配置文件
修改 `docker-compose` 文件, 网络 更改为 `host`, 添加 `environment`, 如下
```
network_mode: host

```
```
environment:
      - ALL_PROXY=socks5://127.0.0.1:62312

```
## 重新启动容器
```
 docker-compose down
 ```
 ```
 docker-compose up -d
 ```
## 检查是否成功
 打开 PLEX 检查, 此时媒体库应该能正常访问, 没有的话 打开侧边栏-更多
 或者 进入到容器内 `curl ipv4.ip.sb` 看看出口IP是不是你的反代机

## 持久化
SSH隧道每次自动重连可用 autossh 之类的来自动创建并持久化. 这里不展开了
