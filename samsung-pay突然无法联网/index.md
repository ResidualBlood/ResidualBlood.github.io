# Samsung Pay突然无法联网


## 状况
  Samsung Pay之前一直用的好好的，但是这几天发现交通卡充值，银行卡二维码一直提示：网络错误，无法连接至网络；网络明明正常啊.jpg
## 解决
  尝试百般无果后，清除应用数据走起，然后，就发现了，Samsung Pay居然还需要`Samsung Push Service`启动才能联网
## 因由
  之前我是把`Samsung Push Service`手动`Disable`的，因为这货在Play Store里的描述是这样子的：
> The Samsung push service provides the notification service only for Samsung services (Samsung Apps, Samsung Link, Samsung Wallet, Samsung Pay, etc.) on Samsung devices.
If you delete the Samsung push service, you may not receive the new notification messages.

意为： 
> 三星推送服务仅为三星设备上的三星服务（三星应用程序，三星链接，三星钱包，三星支付等）提供通知服务。
>如果删除三星推送服务，则可能不会收到新的通知消息。

所以我一直以为，这货是推广告的，没想到还需要他`Enable`才能使Samsung Pay联网，真的是.....
