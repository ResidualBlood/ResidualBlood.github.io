# 添加Cloudflare CDN遇到的问题及解决


[Meting]
[Music server="netease" id="34014166" type="song"/]
[/Meting]
给博客迁移到了搬瓦工传家宝上并使用Cloudflare CDN加速中间的一些坑~
#一 后台管理页面会打不开
如果博客到CDN没有启用https，而CDN启用https可能会造成后台管理页面打不开的状况。
解决方法：编辑站点根目录下的`config.inc.php`
加入下面这行配置
```
/** 开启 HTTPS */
define('__TYPECHO_SECURE__',true);
```
这样后台页面就打开了~

#二 待续
