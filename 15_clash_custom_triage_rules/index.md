# Clash 自定义分流规则


> 本文主要讲述如何在`ACL4SSR`的Clash规则基础上魔改出适合自己的规则配置文件，并方便前文订阅转换的调用  
`ACL4SSR`项目地址：[https://github.com/ACL4SSR/ACL4SSR/tree/master](https://github.com/ACL4SSR/ACL4SSR/tree/master)
## 目录结构
`ACL4ALL`目录结构如下所示，这里我们主要关注`Clash`目录下的`Ruleset`目录，以及`config`目录，其中：
- `*.list`文件，这是域名/IP列表
- `*.ini`文件，为分组相关的配置文件
```
.
├── Clash
│   ├── config
│   ├── Providers
│   │   └── Ruleset
│   └── Ruleset
└── Tool
    ├── Surge转acl工具
    └── SwitchyOmega
```
## `*.ini`文件结构
打开`config`目录下的`ACL4SSR_Online_Full_AdblockPlus.ini`文件
其大致结构为
```
[custom]
;不要随意改变关键字，否则会导致出错
;acl4SSR规则

;去广告：支持
;自动测速：支持
;微软分流：支持
;苹果分流：支持
;增强中国IP段：支持
;增强国外GFW：支持

;设置规则标志位
surge_ruleset=全球直连,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/LocalAreaNetwork.list
......
surge_ruleset=全球直连,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Download.list
surge_ruleset=全球直连,[]GEOIP,CN
surge_ruleset=漏网之鱼,[]FINAL
;设置规则标志位

;设置分组标志位
custom_proxy_group=节点选择`select`[]自动选择`[]故障转移`[]负载均衡`[]香港节点`[]台湾节点`[]狮城节点`[]日本节点`[]美国节点`[]韩国节点`[]手动切换`[]DIRECT
custom_proxy_group=手动切换`select`.*
custom_proxy_group=自动选择`url-test`.*`http://www.gstatic.com/generate_204`300
custom_proxy_group=故障转移`fallback`.*`http://www.gstatic.com/generate_204`300
custom_proxy_group=负载均衡`load-balance`.*`http://www.gstatic.com/generate_204`300
custom_proxy_group=电报消息`select`[]节点选择`[]自动选择`[]狮城节点`[]香港节点`[]台湾节点`[]日本节点`[]美国节点`[]韩国节点`[]手动切换`[]DIRECT
......
custom_proxy_group=韩国节点`url-test`(KR|Korea|KOR|首尔|韩|韓)`http://www.gstatic.com/generate_204`300
custom_proxy_group=奈飞节点`select`(NF|奈飞|解锁|Netflix|NETFLIX|Media)
;设置分组标志位

enable_rule_generator=true
overwrite_original_rules=true

;clash_rule_base=https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/GeneralClashConfig.yml

;luck
```
其中：
- `;`后为注释
- `surge_ruleset=全球直连,https://xx.list`:`surge_ruleset`声明规则列表；`全球直连`为分组名称；`https://xx.list`为规则列表地址，一般为Github的`RAW`地址，如上格式
- custom_proxy_group=节点选择\`select\`[]自动选择 : `custom_proxy_group`声明自定义分组；`节点选择`为分组名称；\`select为选择模式
- 分组模式：
  - \`select：选择模式，后加分组或者节点
  - \`url-test：自动测速模式,选择速度最快的节点，后加节点
  - \`fallback:故障转移，后加节点
  - \`load-balance，后加节点
- (NF|奈飞)：筛选出包含括号内关键字的节点，例如上面的`奈飞节点`
## 自定义配置
fork`ACL4SSR`的仓库，然后拉取到你本地参照上面`ACL4SSR_Online_Full_AdblockPlus.ini`添加删除修改相关行进行魔改即可，这里说明几点需要注意的地方：
1. Clash匹配规则是从上至下的，因此，需要将你自定义的规则放在`surge_ruleset`的最前面
1. 规则文件中可能会产生相同IP/域名不同规则的情况，但因为匹配规则从上至下的原因，不要认为规则出现错误，重复只是因为方便作者维护
1. 检查分组是否正确，不要出现不存在的分组
1. 使用机场的同学可以直接将`负载均衡`分组删除，因为会造成连接数爆炸，同理，`自动测试`也少用比较好
## 总结
Clash分流规则不同人有不同需求，适合自己才是最好的～
下一篇文章将说明：`ACL4SSR`规则更新后如何同步到自己fork的仓库并且不损失自己的配置，实际上就是版本合并啦～

