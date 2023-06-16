# 目录

[TOC]

参考：

[Linux 命令大全—— 菜鸟教程](https://www.runoob.com/linux/linux-command-manual.html)

[Linux命令大全(手册)](https://www.linuxcool.com/)

[《Linux 101》在线讲义(左上两级目录)](https://101.lug.ustc.edu.cn/)

[DevOps-Guide](https://github.com/Tikam02/DevOps-Guide)

[Ubuntu官方文档](https://help.ubuntu.com/20.04/ubuntu-help/index.html)

[ubuntu社区](https://help.ubuntu.com/community/CommunityHelpWiki)

# 常见命令

命令|功能
:---:|:---:
rm -r -f 目录名               	|递归、强制地删除
whereis xxx             |查看软件安装位置
dpkg -i xxx.deb         |安装deb包
tar -xzvf xxx.gz        |解压缩（.tar、.tar.gz、.tar.bz2）
unzip xxx.zip -d ./xxx  |解压缩至xxx文件夹（防止遍地都是）
CTRL + D                |退出某个环境
history 10		|查看最近10个命令
history \| grep 'xxx'	|关键词查找命令
Ctrl + r，'xxx'		|关键词查找（回车执行，右方向修改）
alias 	|查看所有程序别名（定义alias xx='命令'）

## `apt`软件管理

```bash
# 列出所有可更新的软件清单命令
sudo apt update
# 升级软件包
sudo apt upgrade
# 列出可更新的软件包及版本信息
apt list --upgradeable
# 升级软件包，升级前先删除需要更新软件包
sudo apt full-upgrade
# 安装指定的软件命令
sudo apt install <package_name>
# 安装多个软件包
sudo apt install <package_1> <package_2> <package_3>
# 更新指定的软件命令
sudo apt update <package_name>
# 显示软件包具体信息,例如
版本号，安装大小，依赖关系等等
sudo apt show <package_name>
# 删除软件包命令
sudo apt remove <package_name>
# 清理不再使用的依赖和库文件
sudo apt autoremove
# 移除软件包及配置文件
sudo apt purge <package_name>
# 查找软件包命令
sudo apt search <keyword>
# 列出所有已安装的包
apt list --installed
# 列出所有已安装的包的版本信息
apt list --all-versions

```

## 软件安装卸载


```bash
# 解压
# .zip
unzip xxx.zip
# .deb
dpkg -i xxx.deb
# .tar、.tar.gz、.tar.bz2
tar -xzvf xxx.gz
## APPimage，先添加可执行权限，然后执行
chmod a+x *.AppImage
./*.AppImage



# 卸载
# deb
dpkg -r linuxqq
# apt
# 知道具体名称
sudo apt-get remove --purge 软件名称
sudo apt-get autoremove --purge 软件名称
# 不知道具体名称，查询
dpkg --get-selections | grep '模糊名称字段'

# 清理残留数据
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P


```

## 查看文件内容

```bash
cat  由第一行开始显示文件内容，如： cat ~/Desktop/1.txt
tac  从最后一行开始显示,逐次上一行（可以看出 tac 是 cat 的倒着写！）
nl   显示的时候，顺道输出行号！
more 一页一页的显示文件内容
less 与 more 类似，但是比 more 更好的是，他可以往前翻页！
head 只看头几行
tail 只看尾巴几行
man 命令名 来查看各个命令的使用文档，如 ： man cp

```

## 查看历史命令


```bash
# 查询历史文件位置
echo $HISTFILE
# 更改文件位置（例如，zsh 未设置 HISTFILE 变量，可在shell配置文件中添加保存位置）
vim ~/.zshrc
HISTFILE=~/.zsh_history
# 还可以更改保存行数（一般已存在，修改而非添加）
HISTFILESIZE=5000
HISTSIZE=5000
# 打开文件
vim ~/.bash_history

```





## 删除用户帐号


```bash
# 如果一个用户的账号不再使用，可以从系统中删除。
# 删除用户账号就是要将/etc/passwd等系统文件中的该用户记录删除。
# 必要时还应该删除用户的主目录。
userdel -r sam
# 此命令删除用户sam在系统文件中的记录，同时删除用户的主目录。
# （主要是/etc/passwd, /etc/shadow, /etc/group等）

```

## 用户口令管理

```bash
passwd 选项 用户名
# 可使用的选项：
# -l 锁定口令，即禁用账号。
# -u 口令解锁。
# -d 使账号无口令。
# -f 强迫用户下次登录时修改口令。
# （用户管理的一项重要内容是用户口令的管理。用户账号刚创建时没有口令，但是被系统锁定，无法使用，必须为其指定口令后才可以使用，即使是指定空口令。）

```

# 使用技巧



## Linux主机上qemu+kvm直通显卡

?

如果是linux虚拟机需要双显卡

## linux作为host，qemu/kvm虚拟windows

?

## `base64`加密

```bash
echo "wang@123"|base64
# 加-i选项也可以，输出 d2FuZ0AxMjMK

echo "d2FuZ0AxMjMK"|base64 -d
# 对上面加密后的字符串进行解密操作，成功解密wang@123

```
## `openssl`加密文件

tar等压缩程序也可以实现。

```bash
openssl aes-256-cbc -salt -in gitbook.sh -out secret.sh
# 加密文件，输入密码并确认。
# （默认有salt，相当于随机数干扰)
# (但是是不是就只能本地解密？因为不知道salt是啥?）

openssl aes-256-cbc -d -in secret.sh -out 2gitbook.sh
# 解密文件

cmp gitbook.sh 2gitbook.sh
# 简单对比两个文件，无返回说明文件相同
# 否则返回行号和字节位置，如：byte 12, line 2

echo $?
# 会返回不同数量，0说明文件相同

```


## `rename`批量修改文件名
linux终端输入`rename`命令，可以修改当前文件夹内的所有文件名

此外，ubuntu自带的重命名功能也很强大。

```bash
# 改为全大写
sudo rename 'y/A-Z/a-z/' *

# 改为全小写
sudo rename 'y/a-z/A-Z/' *

```

## `crontab`设置定时任务

```bash
# https://www.cnblogs.com/0201zcr/p/4739207.html
#查看任务列表
crontab -l
# 编辑
crontab -e

# 时程表的格式如下:
# f1 f2 f3 f4 f5 program
#其中 f1 是表示分钟，f2 表示小时，f3 表示一个月份中的第几日，f4 表示月份，f5 表示一个星期中的第几天。program 表示要执行的程式。
* 3 * * * /home/lee/gitbook.sh
*/5 * * * * ./home/lee/gitbook.sh

# 好像没生效……

```

## vmware自动挂载win共享文件夹


```bash
# 打开‘启动应用程序’
gnome-session-properties
# 添加以下命令，取名自动挂载
sudo vmhgfs-fuse -o allow_other .host:/ /mnt/hgfs

```

## bash-it 插件

https://github.com/Bash-it/bash-it

## 自定义shell命令（程序别名）


```bash
vim ~/.bashrc
# 填入
alias 命令=’可执行文件全局路径’
source ~/.bashrc

```
## 命令含特殊字符，如空格

```bash
# 利用反斜杠\做转义符
# 如文件1 copy.i
./phase2eq5-dbg -i problems/1\ copy.i


```

## 终端后台执行命令、执行多条命令


```bash
# 命令后追加 &
/usr/local/VSCode-linux-x64/code &

# \ 反斜杠按序执行
# ; 分号按序执行
cd /home/PyTest/src ; python suning.py

# && 执行成功，才会去执行后续命令
cd /home/PyTest/src && python suning.py

# || 或者 |
# ||是或的意思，只有前面的命令执行失败后才去执行下一条命令，直到执行成功一条命令为止。
cd /home/PyTest/123 || echo "error234"
cd /home/PyTest/123 | echo "error234"

```

## open-vm-tools自由分辨率+文件拖拽

```bash
sudo apt install open-vm-tools
sudo apt install dkms
sudo apt install open-vm-tools-desktop

```
## swap交换分区
https://help.ubuntu.com/community/SwapFaq

```bash

# 查看交换分区大小
swapon -s
# 如果查看的时候有内容显式表示当前是处于启用状态，如果什么内容都没有则是未启用

# 如果当前swapfile是启用状态则需要先使用swapoff关闭交换分区，再进行大小的调整
sudo swapoff /swapfile

# 调整大小
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
# 这里是设置交换分区单位为1M，大小是16384，也就是16G，一般设置为虚拟机设置的内存的两倍，count值自己计算一下。（感觉用不了这么多）
# 17 笔记本 200 MB/s
# 21 工位台式机 100 MB/s

# 创建swapfile
# 相当于根据调整的大小重新建一个swapfile
sudo mkswap /swapfile

# 启用交换分区
sudo swapon /swapfile

```


## imwheel更改鼠标滚轮速度
[直接自定义配置](https://juejin.cn/post/6859092132347510797) | [原版脚本参考](https://io.bikegremlin.com/11541/linux-mouse-scroll-speed/) | [论坛](https://askubuntu.com/questions/285689/increase-mouse-wheel-scroll-speed)

```bash
sudo apt install imwheel
bash <(curl -s http://www.nicknorton.net/mousewheel.sh)
# 出现窗口，设为4左右

# 配置到开机自启动程序  /usr/bin/imwheel
gnome-session-properties
# 重启生效

```

```bash
# 上面只能上下滚动相同。自定义可修改配置.imwheelrc
sudo vim .imwheelrc
# 在Button4和Button5设置数字为3、4大概一次会滚动7、14行。

```

## 切换工作区

键 | 功能
-|-
Ctrl + Alt + 上下方向键 |快速上下切换
Win |可视化所有工作区

问题：只有主显示器有工作区分区

[安装gnome-tweak-tools](https://linuxhint.com/gnome_tweak_installation_ubuntu/)


```bash
sudo apt update
sudo add-apt-repository universe
# sudo apt install gnome-tweaks
# 报错未满足的依赖关系——换成 aptitude 安装
# https://blog.csdn.net/weixin_39145415/article/details/121726723
sudo apt-get install aptitude
sudo aptitude install gnome-tweak-tool
# ！！！提示依赖的包已安装，但版本比需求的要高
# 1. 是否接受该解决方案, no
# 2. 提醒降级软件包，是否接受该解决方案, yes

# 运行软件
gnome-tweaks

```
## 系统自带截图

键|功能
:--|:--
PrtSc | 获取**整个屏幕**的截图并保存到 Pictures 目录。
Shift + PrtSc| 获取屏幕的某个**区域截图**并保存到 Pictures 目录。
Alt + PrtSc | 获取当前**窗口**的截图并保存到 Pictures 目录。
Ctrl +以上| 存放到**剪贴板**

## Alt + F2 输入r 重启gnome桌面


## nvm 管理node 版本


```bash
sudo apt-get install build-essential

# https://github.com/nvm-sh/nvm#install--update-script
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash

nvm install 18.14.2
nvm use 18.14.2
nvm ls

# default version
nvm alias default v18.14.2

```

## .zshrc自动执行命令

```bash
sudo vim ~/.zsdefaulthrc
# 添加以下命令，即可在打开新终端时自动执行。
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null


```

# 软件

[github超赞的-linux-软件仓库](https://github.com/luong-komorebi/Awesome-Linux-Software/blob/master/README_zh-CN.md#%E8%B6%85%E8%B5%9E%E7%9A%84-linux-%E8%BD%AF%E4%BB%B6)

## ImageMagick批量压缩图指定大小

[软件下载](https://imagemagick.org/script/download.php)AppImage格式

[命令选项](https://imagemagick.org/script/convert.php)

ImageMagick是一个用于查看、编辑位图文件以及进行图像格式转换的开放源代码软件套装。它可以读取、编辑超过100种图帧式。主要由大量的命令行程序组成，而不提供图形界面。它还为很多程序语言提供了API库。

```bash
chmod a+x magick 

# 压缩图片略小于指定值，应该是比较判断
./magick convert 春游*.jpg -define jpeg:extent=25MB output.jpg

# reduce the image size before it is written to the PNG format:
magick rose.jpg -resize 50% rose.png

```



## XTerm终端复制粘贴

XTerm 是一个终端介面，最近刚好使用到，但在使用复制功能时就有点不顺了。传统上我们只要使用 ctrl + c 就能复制 ctrl + v 就能贴上，但在 XTerm 上却无法使用。以下我们就来看看如何处理这个问题。

XTerm 内复制贴上
用鼠标选取想复制的内文，按 shift + insert (Ins) 即可在 XTerm 的里面贴上想复制的文字。

XTerm 外复制
如果要把 XTerm 内的文字复制出来就有点麻烦了。

先打开 ~/.Xresources ( 或是建立一个 )，内文输入如下

XTerm*selectToClipboard: true
接着打开 terminal 执行以下指令

xrdb -merge ~/.Xresources
再重新打开 XTerm 即可使用 ctrl + c 复制文字到外面了



**XTerm终端复制粘贴2**

https://www.cnblogs.com/foreverwith/p/3680842.html


在系统自带的终端中一律是SHIFT+CRTL+C(V),但是现在用了XTerm了,这个快捷键就不支持了

找了一下

[让 xterm 与其它程序间复制粘贴更灵活](http://www.linuxsir.org/bbs/thread290883.html)

按照这个说的来就OK啦,还记得上一篇我们说的在主文件夹下建立一个.Xdefaults文件吗,这次我们在terminal下继续输入vim .Xdefaults

将以下代码复制到 `XTerm\*locale: zh\_CN.UTF-8` 的下面就可以啦,这样以后就可以自由的在XTerm中复制或者粘贴(鼠标中键,也就是滚动轮,不太习惯SHIFT+insert)

鼠标框选XTerm中内容后，中键粘贴到任何地方。

```
     \*VT100\*translations: #override \\n\\ 
    Shift <KeyPress> Insert:insert-selection(CLIPBOARD, CUT\_BUFFER1) \\n\\ 
    ~Shift~Ctrl<Btn2Up>: insert-selection(CLIPBOARD, CUT\_BUFFER1) \\n\\ 
    ~Shift<BtnUp>: select-end(CLIPBOARD, CUT\_BUFFER1) 
```





## UltraEdit

```bash
# 复制以下（或者官网下载，信息随便填得到下载网页）
# https://www.ultraedit.com/downloads/uex-thank-you/?mobile=1&email=1%40gmail.com
wget https://downloads.ultraedit.com/main/ue/linux/deb/focal/uex_21.00.0.0_amd64.deb
sudo dpkg -i uex_21.00.0.0_amd64.deb
sudo apt --fix-broken install

# 破解:删除配置文件，始终30天试用
rm -rfd ~/.idm/uex
rm -rf ~/.idm/*.spl
rm -rf /tmp/*.spl


```
##  Visual Studio Code



```bash
# 更新软件包索引，并且安装依赖软件：
sudo apt update
sudo apt install software-properties-common apt-transport-https wget


# 使用 wget 命令插入 Microsoft GPG key ：
wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -


# 启用 Visual Studio Code 源仓库，输入：
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"

#  一旦 apt 软件源被启用，安装 Visual Studio Code 软件包：

sudo apt install code
code


```


## 音乐播放器qmmp


```bash
# https://launchpad.net/~forkotov02/+archive/ubuntu/ppa
sudo add-apt-repository ppa:forkotov02/ppa
sudo apt-get update
sudo apt-get install qmmp qmmp-plugin-pack

```

Name=typora
Exec=/home/lee/software/typora/Typora
Terminal=false
Icon=/home/lee/software/typora/typora_icon.png
Type=Application
Categories=Development

## 安卓虚拟机waydroid

（适配差，问题多）

https://docs.waydro.id/usage/install-on-desktops

```bash
sudo apt install curl ca-certificates -y
curl https://repo.waydro.id | sudo bash
sudo apt install waydroid -y



```


# 问题

##  网络图标消失


```bash
service NetworkManager stop
# 弹出用户认证
sudo rm /var/lib/NetworkManager/NetworkManager.state
service NetworkManager start
# 弹出用户认证

```

## ubuntu 启动很慢,F2显示1m30s

 (swap 分区不自动挂载)

A start job is running for dev-disk-by\...UUID (xx s/ 1min30s)

```bash
# 查看UUID，发现是swap分区，可能是之前分区操作错误
sudo vim /etc/fstab

# 重新分区，记录第二步mkswap 时的UUID
sudo dd if=/dev/zero of=/swapfile bs=1M count=8096
sudo mkswap /swapfile
sudo swapon /swapfile

# 查看内存使用情况确认有 Mem...Swap
free -m

# 实现永久自动挂载——找到 # swap was on ..., 更新UUID
sudo vim /etc/fstab

UUID=4a5a03bc-46eb-4b1d-a12b-fce2a4c28ed1 /swap           swap    defaults        0       0


```

## 域名解析失败（向日葵断开连接）
只有临时方法有效，临时添加 DNS 服务器。https://askubuntu.com/a/91595

```bash
sudo vim ~/.zshrc
# 添加以下命令，即可在打开新终端时自动执行。
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null


```

