# 利用 YouTube-dl 优雅地下载 YouTube 视频


> 下载的视频仅作为交流学习收藏哦，不要作为商用！


----------
“YouTube-dl”是一个不仅可以下载YouTube视频，还可以下载哔哩哔哩等视频的命令行工具。
其项目地址为[https://github.com/rg3/youtube-dl](https://github.com/rg3/youtube-dl)
在这里就说一下如何快速的上手使用～

#安装`ffmpeg`
首先需要安装`ffmpeg`，因为`youtube-dl`下载完成后会自动调用，对视频进行封装。
以下命令基于Ubuntu16.04
```bash
# 添加软件源
$ sudo add-apt-repository ppa:jonathonf/ffmpeg-3

# 更新并安装
$ sudo apt update && sudo apt install ffmpeg libav-tools x264 x265

# 卸载官方源
$ sudo apt autoremove
```
如果出现`add-apt-repository command not found`
运行下：
```bash
apt-get install software-properties-common
```
#安装Youtube-dl
运行
```bash
sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl 
sudo chmod a+rx /usr/local/bin/youtube-dl
```
这样的话`YouTube-dl`就安装完毕了，接下来是基本使用
#直接下载视频
命令格式为`youtube-dl [OPTIONS] URL [URL...]`
不带任何参数，则默认下载画质、音质最好的文案。
例：
```bash
youtube-dl "https://www.youtube.com/watch?v=l4WNrvVjiTw"
```
（似乎4k的视频只会下载1080p，如何选择下载分辨率见下文）
#直接下载播放列表
运行
```bash
youtube-dl "video_url"
```
小心硬盘爆炸，哈哈哈哈
#选择合适的分辨率下载
运行
```bash
youtube-dl -F "video_url"
```
查看可选的视、音频格式
运行后会输出以下例子
```
[youtube] Bey4XXJAqS8: Downloading webpage
[youtube] Bey4XXJAqS8: Downloading video info webpage
[info] Available formats for Bey4XXJAqS8:
format code  extension  resolution note
249          webm       audio only DASH audio   61k , opus @ 50k, 11.46MiB
250          webm       audio only DASH audio   82k , opus @ 70k, 15.43MiB
171          webm       audio only DASH audio  110k , vorbis@128k, 21.97MiB
140          m4a        audio only DASH audio  129k , m4a_dash container, mp4a.40.2@128k, 26.91MiB
251          webm       audio only DASH audio  156k , opus @160k, 29.94MiB
278          webm       256x144    144p  101k , webm container, vp9, 30fps, video only, 16.33MiB
394          mp4        256x144    144p  107k , av01.0.05M.08, 30fps, video only, 14.61MiB
160          mp4        256x144    144p  115k , avc1.4d400c, 30fps, video only, 11.04MiB
242          webm       426x240    240p  233k , vp9, 30fps, video only, 31.57MiB
395          mp4        426x240    240p  250k , av01.0.05M.08, 30fps, video only, 29.34MiB
133          mp4        426x240    240p  252k , avc1.4d4015, 30fps, video only, 22.91MiB
396          mp4        640x360    360p  437k , av01.0.05M.08, 30fps, video only, 54.59MiB
243          webm       640x360    360p  448k , vp9, 30fps, video only, 60.07MiB
134          mp4        640x360    360p  735k , avc1.4d401e, 30fps, video only, 64.87MiB
244          webm       854x480    480p  804k , vp9, 30fps, video only, 109.78MiB
397          mp4        854x480    480p  838k , av01.0.05M.08, 30fps, video only, 100.78MiB
135          mp4        854x480    480p 1356k , avc1.4d401f, 30fps, video only, 131.16MiB
247          webm       1280x720   720p 1755k , vp9, 30fps, video only, 227.80MiB
398          mp4        1280x720   720p 1794k , av01.0.05M.08, 30fps, video only, 209.23MiB
136          mp4        1280x720   720p 2697k , avc1.4d401f, 30fps, video only, 260.78MiB
248          webm       1920x1080  1080p 3236k , vp9, 30fps, video only, 414.56MiB
137          mp4        1920x1080  1080p 4752k , avc1.640028, 30fps, video only, 481.99MiB
271          webm       2560x1440  1440p 9041k , vp9, 30fps, video only, 1.18GiB
313          webm       3840x2160  2160p 18002k , vp9, 30fps, video only, 2.76GiB
17           3gp        176x144    small   70k , mp4v.20.3, mp4a.40.2@ 24k (22050Hz), 14.93MiB
36           3gp        320x180    small  202k , mp4v.20.3, mp4a.40.2 (22050Hz), 42.88MiB
43           webm       640x360    medium , vp8.0, vorbis@128k, 148.66MiB
18           mp4        640x360    medium  552k , avc1.42001E, mp4a.40.2@ 96k (44100Hz), 116.91MiB
22           mp4        1280x720   hd720 1357k , avc1.64001F, mp4a.40.2@192k (44100Hz) (best)
```
在这里我们可以看到，默认`best`选择的是第22个，只有720p的分辨率，所以在这里想下载4k的视频话，在这了需要运行
```bash
youtube-dl -f '313+251' "video_url"
```

 - `'313+251'`：`313`是视频编号，`251`是音频编号
 - `"video_url"`是你的视频地址
这样，就可以下载你想要的分辨率的视频。
好了，如果有什么问题欢迎下面评论～

