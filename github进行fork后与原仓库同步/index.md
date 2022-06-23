# Github进行fork后与原仓库同步


> 本文主要讲述Github进行fork后如何与原仓库同步，即版本合并；其有两种方法，分别为：本地合并、Github合并
> 主要参照：[https://github.com/selfteaching/the-craft-of-selfteaching/issues/67](https://github.com/selfteaching/the-craft-of-selfteaching/issues/67)
## 一、本地合并
适合代码修改较多

### 设置上游代码库(upstream)
> 使用之前fork的`ACL4SSR`项目作为例子

1.进入本地代码库
```
cd ~/ACL4SSR
```
2.执行`git remote -v`命令查看远程仓库路径
若只有以下两行输出，说明你未设置上游代码库(upstream)
```
origin  git@github.com:xxxxx/ACL4SSR.git (fetch)
origin  git@github.com:xxxxx/ACL4SSR.git (push)
```
3.执行`git remote add upstream https://github.com/ACL4SSR/ACL4SSR/tree/master`，此命令运行后并不会回显，所以再次运行`git remote -v`检查是否成功：
```
origin  git@github.com:xxxxx/ACL4SSR.git (fetch)
origin  git@github.com:xxxxx/ACL4SSR.git (push)
upstream        https://github.com/ACL4SSR/ACL4SSR.git (fetch)
upstream        https://github.com/ACL4SSR/ACL4SSR.git (push)
```
4.执行命令`git status`检查本地是否有未提交的修改。如果有，则把你本地的有效修改，先从本地仓库推送到你的github仓库。最后再执行一次`git status`检查本地已无未提交的修改
**第四步每次合并前都要检查一次**
### 版本合并(merge)
5.执行`git fetch upstream`抓取上游更新
```
remote: Enumerating objects: 64, done.
remote: Counting objects: 100% (64/64), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 36 (delta 34), reused 36 (delta 34), pack-reused 0
Unpacking objects: 100% (36/36), done.
From https://github.com/ACL4SSR/ACL4SSR
   8f6feb5..749784c  master     -> upstream/master
```
6.执行命令`git checkout master`切换到`master`分支：
```
Already on 'master'
Your branch is up-to-date with 'origin/master'.
```
7.执行命令`git merge upstream/master`合并远程的`master分支`：
此时会弹出`commit`编辑，不输入或者输入你的更新信息即可
```
Merge made by the 'recursive' strategy.
 Clash/BanEasyList.list                |  31 +++++-
 Clash/BanEasyListChina.list           |   8 +-
 Clash/BanEasyPrivacy.list             |  34 ++++++-
 Clash/ChinaDomain.list                |   2 -
 Clash/GeneralClashConfig.yml          |   2 -
 Clash/Providers/BanEasyList.yaml      | 276 ++++++++++++++++++++++++++++++++++++++--------------
 Clash/Providers/BanEasyListChina.yaml |  42 +++++++-
 Clash/Providers/ChinaDomain.yaml      |   2 -
 Clash/Providers/ProxyLite.yaml        |   1 -
 Clash/Providers/ProxyMedia.yaml       |   5 +-
 Clash/Providers/Ruleset/AmazonIp.yaml |  25 +++--
 Clash/Providers/Ruleset/HBO.yaml      |   3 +-
 Clash/Providers/Ruleset/SteamCN.yaml  |   4 +-
 Clash/ProxyLite.list                  |   1 -
 Clash/ProxyMedia.list                 |   5 +-
 Clash/Ruleset/AmazonIp.list           |  25 +++--
 Clash/Ruleset/HBO.list                |   3 +-
 Clash/Ruleset/Instagram.list          |   3 +-
 Clash/Ruleset/SteamCN.list            |   4 +-
 backcn-banAD.acl                      |   2 -
 banAD.acl                             |   1 -
 easylist-banAD.acl                    |  64 +++++++++++-
 nobanAD.acl                           |   1 -
 23 files changed, 414 insertions(+), 130 deletions(-)
```
8.执行命令`git push`把本地仓库向github仓库（你fork到自己名下的仓库）推送修改
```
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 515 bytes | 0 bytes/s, done.
Total 4 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To git@github.com:xxxxx/ACL4SSR.git
   31d85cf..fea1fa0  master -> master
```
至此，你fork的仓库已经和原始仓库保持最新了，并且，你本地修改的文件也不会丢失
## 二、Github合并
直接在Github上进行合并，适合修改内容较少的情况
### 发起Pull request
在你的仓库单击下图红框的`Pull request`
![](https://dig4.lwnlh.com/image/2022/05/14/15-1.png)
然后将下图红框`1`中的仓库换成你自己的，红框`2`中的仓库换成原始仓库，即将`1`和`2`的内容交换下，然后单击红框`3`即可
![](https://dig4.lwnlh.com/image/2022/05/14/15-2.png)
### 版本合并
同意下自己的的合并请求即可


