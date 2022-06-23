# 伪-NAS 利用VPS实现备份、下载、串流播放一条龙(下) 


> 本片讲述挂载Google Drive扩展你的VPS硬盘并把Aria2下载后的文件自动上传到Google Drive中等玩法。


<!--more-->

##一、安装rclone
```bash
curl https://rclone.org/install.sh | sudo bash
```
##二、初始化配置
```bash
rclone config
```
按照下面选择走：
```bash
Current remotes:

Name                 Type
====                 ====
gdrive               drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> n   这里输入n，新建一个
name> test         输入一个名字，记住之后会用到
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Alias for a existing remote
   \ "alias"
 2 / Amazon Drive
   \ "amazon cloud drive"
 3 / Amazon S3 Compliant Storage Providers (AWS, Ceph, Dreamhost, IBM COS, Minio)
   \ "s3"
 4 / Backblaze B2
   \ "b2"
 5 / Box
   \ "box"
 6 / Cache a remote
   \ "cache"
 7 / Dropbox
   \ "dropbox"
 8 / Encrypt/Decrypt a remote
   \ "crypt"
 9 / FTP Connection
   \ "ftp"
10 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
11 / Google Drive
   \ "drive"
12 / Hubic
   \ "hubic"
13 / JottaCloud
   \ "jottacloud"
14 / Local Disk
   \ "local"
15 / Mega
   \ "mega"
16 / Microsoft Azure Blob Storage
   \ "azureblob"
17 / Microsoft OneDrive
   \ "onedrive"
18 / OpenDrive
   \ "opendrive"
19 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
20 / Pcloud
   \ "pcloud"
21 / QingCloud Object Storage
   \ "qingstor"
22 / SSH/SFTP Connection
   \ "sftp"
23 / Webdav
   \ "webdav"
24 / Yandex Disk
   \ "yandex"
25 / http Connection
   \ "http"
Storage> 11           输入11
Google Application Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id>            直接回车
Google Application Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret>        直接回车
Scope that rclone should use when requesting access from drive.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1           输入1
ID of the root folder
Leave blank normally.
Fill in to access "Computers" folders. (see docs).
Enter a string value. Press Enter for the default ("").
root_folder_id>     直接回车
Service Account Credentials JSON file path 
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Enter a string value. Press Enter for the default ("").
service_account_file>     直接回车
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n                    输入n
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine or Y didn't work
y) Yes
n) No
y/n> n                    输入n
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=xxxxxxxxxxxxxxxxxxxxxxxxxx
这里会给你一个链接，打开后用你的Google Drive授权就会得到下面一串代码填入即可
Log in and authorize rclone for access
Enter verification code> 4/rQDL5M58Dv_Mjwxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Configure this as a team drive?
y) Yes
n) No
y/n> y                    输入y
Fetching team drive list...
No team drives found in your account--------------------
[test]
type = drive
scope = drive
token = {"access_token":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx","token_type":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx","expiry":"2018-12-08T14:45:49.328865767+08:00"}
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y                 输入y
Current remotes:

Name                 Type
====                 ====
gdrive               drive
test                 drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q             输入q退出

```
##三、挂载
###1.手动挂载

```bash
#新建本地文件夹，路径自己定，即下面的LocalFolder
mkdir /root/GoogleDrive
#挂载为磁盘，下面的DriveName、Folder、LocalFolder参数根据说明自行替换
rclone mount DriveName:Folder LocalFolder --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000
```

###2.开机自动挂载
新建 `/lib/systemd/system/rclone.service` 

```bash
nano /lib/systemd/system/rclone.service
```

写入，注意 `gdrive`后最好跟盘内的文件夹

```bash
[Unit]
Description = rclone

[Service]   
User = root
ExecStart = /usr/bin/rclone mount gdrive: /mnt/gdrive --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000 
Restart = on-abort

[Install]
WantedBy = multi-user.target
```

重载daemon

```bash
systemctl daemon-reload
```

启动Rclone

```bash
systemctl start rclone
```

设置开机自启

```bash
systemctl enable rclone
```

##二、Aria2下载完后自动上传
这里用到了Aria2的一个`on-download-complete`功能，这个功能可以做很多事情，这里只用到了自动上传
新建脚本文件`gdupload.sh`，并复制下面代码:
```bash
#!/bin/bash
path="$3" #取原始路径
downloadpath='/data/wwwroot/dd.test.com/download' #下载目录
rclone='/data/wwwroot/dd.test.com/cloud'   #rclone挂载的目录

if [ $2 -eq 0 ] #下载文件为0跳出脚本
        then
                exit 0
fi

while true; do  #提取下载文件根路径
    filepath=$path
    path=${path%/*}; 
    if [ "$path" = "$downloadpath" ] && [ $2 -eq 1 ]
        then
        rclone=${filepath/#$downloadpath/$rclone} #替换路径
        cp -R "${filepath}" "${rclone}"
        exit 0
    elif [ "$path" = "$downloadpath" ]   #文件夹
        then
        cp -R "${filepath}" "${rclone}"/
        exit 0
    fi
done
```
**注意：本脚本上传下载文件后并没有删除原文件，因为rclone在上传多文件时，可能会丢文件，所以这个脚本并没有删除原文件**
如果想要自动删除的话，可以改成下面这样子：
```bash
.....................前面不变
     rclone=${filepath/#$downloadpath/$rclone} #替换路径
        mv -f "${filepath}" "${rclone}"
        exit 0
    elif [ "$path" = "$downloadpath" ] #文件夹
        then
        mv -f "${filepath}" "${rclone}"/
        rm -rf "${filepath}"
        exit 0
    fi
done
```

###修改Aria2配置文件
首先授权`chmod +x gdupload.sh`然后再到Aria2配置文件中加上一行`on-download-complete=/root/gdupload.sh`即可，后面为脚本的路径。
##三、Nextcloud添加外置存储
由于我们已经把Google Drive挂载到本地了，所以我们直接添加外置存储即可
首先在Nextcloud中安装`外部存储`插件
选择`应用`-`External storage support`-`启用`即可
接着`设置`-`外部存储`按图所示添加即可
![](https://ws1.sinaimg.cn/large/006uw7syly1fxzb3awhgkj31mz053zkn.jpg)

> 参考
> [使用Aira2下载文件后自动上传到Google Drive网盘](https://www.moerats.com/archives/482/)

