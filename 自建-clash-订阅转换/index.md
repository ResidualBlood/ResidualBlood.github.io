# 自建 Clash 订阅转换


> 本文主要说明Clash订阅转换：Docker搭建后端处理subconverter，以及前端Sub-Web，并利用Nginx反代

> 2020年12月23日更新：利用`Docker-compose`一键部署
## 一、准备工作
首先，你需要两个域名，一个给前端，一个给后端使用，本文举例后端：`suc.test.com`，前端：`sub.test.com`
## 二、搭建后端subconverter
> 原始项目地址：[https://github.com/tindy2013/subconverter](https://github.com/tindy2013/subconverter)  

> 修改项目地址：[https://github.com/stilleshan/subconverter](https://github.com/stilleshan/subconverter)

后者相对于原版修改了以下部分：
- 去除`自动选择 url-test`以解决连接数爆涨问题.
- `全球拦截`增加`节点选择`,以解决`Google Analytics`的访问需求.
- 修改时区 镜像默认时区为`Asia/Shanghai`
### Docker启动
Clone至本地

```bash
git clone https://github.com/stilleshan/subconverter.git
```

修改 `pref.ini`

```bash
api_access_token=123456                     #随意设置
managed_config_prefix=https://suc.test.com  #设置成后端域名
listen=127.0.0.1                            #这里改成 127.0.0.1 进行反代
```

重新构建、启动

```bash
docker build -t stilleshan/subconverter:latest .
docker run  -d --name=subconverter --restart=always -p 25500:25500 stilleshan/subconverter:latest
```
### Nginx反代
将以下配置写入你的网站`conf`中，本文例子为：`suc.test.com.conf`，`Oneinstack`默认网站配置路径为`/usr/local/nginx/conf/vhost`
```
location / {
			proxy_pass       http://0.0.0.0:25500;
			proxy_set_header Host      $host;
			proxy_set_header X-Real-IP $remote_addr;
		}
```
然后重载Nginx：`service nginx reload`
最后打开测试：`https://suc.test.com/sub?target=clash&url=sub_url`
实际上转换只使用后端就足够了，但为了偷懒以及方便选择不同配置还是继续搭建前端

## 三、搭建前端Sub-Web
> 项目地址：[https://github.com/CareyWang/sub-web](https://github.com/CareyWang/sub-web)  

这里主要说一下如何添加`ACL4SSR`配置、自定义配置，简要说明安装过程
### 下载
Clone至本地
```bash
git clone https://github.com/CareyWang/sub-web.git
```
### 修改相关配置
修改`/sub-web/src/views/Subconverter.vue`文件
找到大约在257行 `backendOptions:`，替换后面的 `http://127.0.0.1:25500/sub?`为`https://sub.test.com/sub?`
找到下面`RemoteConfig:[`，这是自定义规则地址的地方其结构为：
```
          {
            label: "第一个label", 
            options: [
              {
                label: "第二个label",
                value:
                  "第二个label的具体地址"
              }
            ]
          },
```
对应下图：
![](https://dig4.lwnlh.com/images/2020/12/22/2020-12-22_20-27.png)
这里添加`ACL4SSR`的规则配置，只需要将以下配置粘贴过来即可，也可以参照上面结构加入自定义配置
```
        {
            label: "ACL4SSR",
            options: [
              {
                label: "ACL4SSR_Online 默认版 分组比较全 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online.ini"
              },
              {
                label: "ACL4SSR_Online_AdblockPlus 更多去广告 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_AdblockPlus.ini"
              },
              {
                label: "ACL4SSR_Online_NoAuto 无自动测速 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Online_NoReject 无广告拦截规则 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoReject.ini"
              },
              {
                label: "ACL4SSR_Online_Mini 精简版 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini.ini"
              },
              {
                label: "ACL4SSR_Online_Mini_AdblockPlus.ini 精简版 更多去广告 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_AdblockPlus.ini"
              },
              {
                label: "ACL4SSR_Online_Mini_NoAuto.ini 精简版 不带自动测速 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Online_Mini_Fallback.ini 精简版 带故障转移 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_Fallback.ini"
              },
              {
                label: "ACL4SSR_Online_Mini_MultiMode.ini 精简版 自动测速、故障转移、负载均衡 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiMode.ini"
              },
              {
                label: "ACL4SSR_Online_Full 全分组 重度用户使用 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini"
              },
              {
                label: "ACL4SSR_Online_Full_NoAuto.ini 全分组 无自动测速 重度用户使用 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Online_Full_AdblockPlus 全分组 重度用户使用 更多去广告 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_AdblockPlus.ini"
              },
              {
                label: "ACL4SSR_Online_Full_Netflix 全分组 重度用户使用 奈飞全量 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_Netflix.ini"
              },
              {
                label: "ACL4SSR 本地 默认版 分组比较全",
                value: "config/ACL4SSR.ini"
              },
              {
                label: "ACL4SSR_Mini 本地 精简版",
                value: "config/ACL4SSR_Mini.ini"
              },
              {
                label: "ACL4SSR_Mini_NoAuto.ini 本地 精简版+无自动测速",
                value: "config/ACL4SSR_Mini_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Mini_Fallback.ini 本地 精简版+fallback",
                value: "config/ACL4SSR_Mini_Fallback.ini"
              },
              {
                label: "ACL4SSR_BackCN 本地 回国",
                value: "config/ACL4SSR_BackCN.ini"
              },
              {
                label: "ACL4SSR_NoApple 本地 无苹果分流",
                value: "config/ACL4SSR_NoApple.ini"
              },
              {
                label: "ACL4SSR_NoAuto 本地 无自动测速 ",
                value: "config/ACL4SSR_NoAuto.ini"
              },
              {
                label: "ACL4SSR_NoAuto_NoApple 本地 无自动测速&无苹果分流",
                value: "config/ACL4SSR_NoAuto_NoApple.ini"
              },
              {
                label: "ACL4SSR_NoMicrosoft 本地 无微软分流",
                value: "config/ACL4SSR_NoMicrosoft.ini"
              },
              {
                label: "ACL4SSR_WithGFW 本地 GFW列表",
                value: "config/ACL4SSR_WithGFW.ini"
              }
            ]
          },
```
### 安装依赖与打包发布
这里需要安装 `Node` 与 `Yarn`
你可以通过以下命令查看是否安装成功
```
node -v
yarn -v
``` 
安装构建依赖项

```bash
yarn install
```

使用 webpack 运行 Web 客户端以进行本地开发

```bash
yarn serve
```

浏览器访问 [http://服务器ip:8080/](http://xn--ip-fr5c86lx7z:8080/) 应该可以进行前端 sub-web 的预览了。

打包

```bash
yarn build
```

将生成的 `dist` 目录复制到你的网站目录下，修改网站 `conf`的 `root`位置，后加 `/dist`

```bash
root /root/sub-web/dist;
```
## 四、Docker-compose一键部署
> 参考Yohane聚聚博客：[https://yohane.art/development%20notes/2020/12/23/subc.html](https://yohane.art/development%20notes/2020/12/23/subc.html)
### 创建配置文件
新建`/root/nginx/conf.d/server.conf`文件，写入以下配置
```
!/root/nginx/conf.d/server.conf
	
# vhost of subc with proxy
	
server {
        listen 80;
        server_name <domain>;
        location / {
                proxy_pass http://subc:25500;
        }
}

server {
        listen 80;
        server_name <domain>;
        location / {
                proxy_pass http://subweb;
        }        
}
```
新建`/root/subconverter/docker-compose.yml`文件，写入以下配置
```
!/root/subconverter/docker-compose.yml

version: "3"

services:
  subc:
    hostname: subc
    restart: always
    container_name: subc
    image: tindy2013/subconverter:latest
  subweb:
    hostname: subweb
    restart: always
    container_name: subweb
    image: careywong/subweb:latest
  nginx:
    hostname: nginx
    restart: always
    container_name: nginx
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    links:
      - "subc:subc"
      - "subweb:subweb"
    volumes:
      - /root/nginx/conf.d:/etc/nginx/conf.d
```
### 启动
```
docker-compose -d
```
## 五、总结
至此，你的Clash订阅转换就搭建好了，如果觉得`ALC4ALL`默认的配置文件不适合你，也可以在此基础上魔改出适合自己的规则，并时刻与`ALC4ALL`更新的规则同步，这将在后续文章中讲述。
