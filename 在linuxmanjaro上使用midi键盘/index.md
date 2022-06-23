# 在Linux(Manjaro)上使用MIDI键盘


> 记录下在Manjaro上使用MIDI键盘的操作。主要参考[Arch wiki](https://wiki.archlinux.org/index.php/USB_MIDI_keyboards)的`USB MIDI keyboards`部分。
##准备工作
###确认你的声卡驱动为`ALSA`
安装`alsa-utils`
```
sudo pacman -S alsa-utils
```
输入`aseqdump`,看到以下输出则正常,接着`Ctrl+C`终止即可
```
Waiting for data at port 128:0. Press Ctrl+C to end.
Source_ Event_________________ Ch _Data__
```
###确认你的MIDI键盘与Manjaro正常通信
1. 插上你的MIDI键盘，打开电源，输入`lsusb`应该能看到以下输出，其中`Nektar Impact GX61`即为MIDI键盘
![](https://dig4.lwnlh.com/image/2022/05/14/11-1.png)
1. 接着输入`lsmod | grep usb`应该能看到以下输出，输出中有`snd_usb_audio `，`snd_usb_lib`即可
![](https://dig4.lwnlh.com/image/2022/05/14/11-2.png)
1. 再输入`aconnect -i`则会列出MIDI所有输入端口，如图所示，需要记下`client 20`
![](https://dig4.lwnlh.com/image/2022/05/14/11-3.png)
1. 输入`aseqdump -p 20`，其中`20`为上面记下的`client 20`，然后敲击键盘，看到如下输出则你的MIDI键盘与Manjaro正常通信
![](https://dig4.lwnlh.com/image/2022/05/14/11-4.png)
##弹奏！
###安装软件合成器`QSynth`
```
sudo pacman -S qsynth
```
###下载`SoundFont`
从[http://soundfonts.narod.ru/](http://soundfonts.narod.ru/)下载名为`Фортепиано`的`Piano.zip`解压备用
![](https://dig4.lwnlh.com/image/2022/05/14/11-11.png)
###设置`QSynth`
1. 输入`qsynth -a alsa`启动`QSynth`
![](https://dig4.lwnlh.com/image/2022/05/14/11-10.png)
1. 输入点击`Setuo`设置下上面下载的音源，个人推荐`Collection`,`CP-70`,`Grand`这三个
![](https://dig4.lwnlh.com/image/2022/05/14/11-12.png)
1. 输入`aconnect -i`查看MIDI输入端口，记下`20`
![](https://dig4.lwnlh.com/image/2022/05/14/11-8.png)
1. 输入`aconnect -o`查看MIDI输出端口，找到`FLUID Synth`，如果在添加`QSynth`中添加了多个配置文件，则会如下所示，否则只有一个输出，记下`128`
![](https://dig4.lwnlh.com/image/2022/05/14/11-9.png)
1. 输入`aconnect 20 128`，敲击MIDI键盘，Enjoy~
> 备注：每重启次PC或者`QSync`都需要重复步骤`5`
