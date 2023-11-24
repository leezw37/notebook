# 目录
[TOC]

# 使用技巧

## MIUI关闭系统广告

关闭系统广告，冰箱冻结智能服务和analytics。



## 刷机

**参考**

[！LineageOS 官方教程——Redmi K30 Pro](https://wiki.lineageos.org/devices/lmi/install/variant2)

[Pixel Experience 安装教程——基于Lineage，原生谷歌生态；貌似更好用，bug更少](https://wiki.pixelexperience.org/devices/lmi/install/)

[网络教程安装MIUI或者lineage](https://www.cnblogs.com/ls1519/p/16088770.html)

[跳过验证谷歌](https://www.hztdst.com/2985.html)

[lineage支持设备列表](https://wiki.lineageos.org/devices/)

[XDA论坛redmi k30 pro 刷机资源收集](https://forum.xda-developers.com/t/poco-f2-pro-redmi-k30-pro-latest-collection-roms-tools-more-miui-14-updated-april-2023.4554851/)

[k30 pro第三方rom汇总对比表](https://docs.google.com/spreadsheets/d/1FdQ9wMOTTmfGB-MiNUpBEB7BC0bVt6uzSQZR_4cSS1g/edit#gid=0)

[derpfest猫头鹰系统](https://sourceforge.net/projects/derpfest-lmi/files/)



**准备**

1. 下载rom包和recovery包！[lineage](https://download.lineageos.org/devices/lmi/builds) ( lineage系统必须使用专用recovery)（或者[microG版本的Lineage系统](https://download.lineage.microg.org/lmi/)）
2. 手机设置——小米账号——云备份——桌面云备份
3. 手机设置——我的设备——备份与恢复——手机备份
4. 连接电脑——复制文件夹MIUI/backup/allbackup
5. 手机设置——更多设置——开发者选项——打开USB调试（我的设备——全部参数——miui版本连续点击）（设备解锁状态，需要提前一周申请）
6. （[下载谷歌套件开源替代——MindTheGapps-13.0.0-arm64-20221025_100653.zip，注意选择安卓版本与arm64](https://wiki.lineageos.org/gapps))（或者[microG版本的Lineage系统](https://download.lineage.microg.org/lmi/)）

（关机时同时长按音量下键和开关机键，可以进入fastboot模式，音量+和电源键则是recovery模式）

**开始刷机**

（如果刷入Gapps谷歌，必选第1、2步！！否则无法上外网就进不去系统，除非有外网热点）

1. 刷miui的Recovery跳过验证谷歌：打开脚本 `H:\3backup\redmi K30 PRO\刷机\TWRP\recovery-twrp一键刷入工具-1-先跳过谷歌验证.bat`，按照指示操作；手机弹窗允许usb调试；重启后自动先后进入fastboot和recovery模式。（更改bat脚本中的包名set rec_img=从而刷入不同的recovery包：刷miui用twrp_3.6.2_LMI_unofficialtwrp.img，刷lineage用lineage_recovery.img）
2. 跳过验证谷歌：打开recovery后，点击高级，点击终端，高通cpu的输入`dd if=/dev/zero of=/dev/block/by-name/frp`，联发科cpu输入dd if=/dev/zero of=/dev/block/bootdevice/by-name/frp，最后重启到Boot Loader（fastboot）。(**注意zero 和of之间有空格。**)
3. 刷入lineage的recovery ( 功能太少 )：打开脚本`H:\3backup\redmi K30 PRO\刷机\TWRP\recovery-twrp一键刷入工具-2-lineage.bat`，按照指示操作后手机自动进入recovery。**(建议使用twrp，以部分防止变砖：1. 清除数据——格式化Data分区，高级清除全选；2. 开启adb侧载——高级，ADB sideload，全选，开始；3. 电脑刷入系统：`adb devices`查看是否已连接设备，如果有，则`adb sideload lineage-xxx.zip`；4. 成功后重启系统)**
4. 刷入lineage系统：Factory Reset——格式化全部三项——返回apply update ——apply from ADB——电脑双击`打开cmd命令行.bat`执行`.\adb.exe sideload lineage-20.0-20230325-nightly-lmi-signed.zip`（或者`adb sideload lineage-20.0-20231109-microG-lmi.zip`）（两个文件和终端路径在同一目录下；Table键可以帮助填充文件名）（刷入microG版本时手机报错鉴权错误时应该继续）
5. 刷入谷歌套件(lineage-microG版不需要)：再次apply update，apply from ADB——执行`.\adb.exe sideload MindTheGapps-13.0.0-arm64-20221025_100653.zip`刷入google——手机提示signature verification failed，点击继续。——手机reboot system now
6. 进入系统初始设置，建议取消google所有勾选，关闭更新lineage恢复。建议关闭系统更新，因为似乎会使magisk失效，冰箱已冻结的app无法启用。（系统app需要重新启用，非系统app需要手动重新安装）

~~4. 刷入miui（出现过lineage无法刷入其他recovery的情况，好像没关系。直接刷其他系统包就行）：清除数据（wipe）——格式化data分区——返回高级清除（advanced wipe）——勾选所有（）——返回重启——recovery。安装（install）——sdcard/twrp(复制lineage安装包到此目录)——重启~~

**问题**

1. lineage 重新刷回 **低版本**安卓的 miui后，如果更新系统会一直卡在recovery，无法进入系统。解决办法是重新刷回lineage，或者最新版本miui（也无法更新系统，但是可以进入系统）。
2. lineage 横屏时音量增减反了；刷入microG套件后发现没有声音。
3. 若无法挂载data无法解密，搜索 twrp k30 pro最近一年，下载最新twrp.img
4. 无法[跳过验证谷歌](https://www.hztdst.com/2985.html)，一直卡在“正在准备你的手机”——进入twrp，点击高级，点击终端，高通cpu的输入dd if=/dev/zero of=/dev/block/by-name/frp，确定（联发科cpu输入dd if=/dev/zero of=/dev/block/bootdevice/by-name/frp）
5. magisk可能干扰微信、中国银行等的人脸验证，在magisk设置中伪装隐藏，但仍然可能无效
6. 微信小程序同程艺龙完全无法使用。支付宝小程序飞猪、12306有时无法选择起始和终点站。（使用微信飞猪，携程一切正常）
7. pubg mobile 开clash能下载安装，但是不能下载资源进入游戏。（5G移动不行，4G和WiFi可以）



**其他**

1. 谷歌相机
   - [教程](https://zhuanlan.zhihu.com/p/538817403?utm_id=0)，lineage相机很差劲，可以搜索自己手机是否有谷歌相机支持。
   - 新建GCam/Configs7文件夹，并将[下载的配置文件](https://www.celsoazevedo.com/files/android/p/f/2021/11/pocof2pro-urnyx05-v2.xml)放入
   - 下载安装[谷歌相机](https://1-dontsharethislink.celsoazevedo.com/file/filesc/GCam_7.3.018_Urnyx05-v2.6.apk)，连击快门按钮周边的区域，选择配置，restore。（[若无谷歌框架需要解决闪退](https://blog.csdn.net/ONE_SIX_MIX/article/details/123444620)，下载安装[谷歌相机伪装](https://github.com/lukaspieper/Gcam-Services-Provider/releases)）
   - 点击上部箭头进入相机设置——照片——使用第三方图库

2. lineage开发者模式，设置——关于手机——版本号连续点击
3. 底部上划进入应用切换页面时，点击应用图标，可以进入应用设置、上下分屏、固定。
4. 搜狗输入法，设置为默认
5. MT文件管理器，设置为默认
6. 开发者选项/无障碍设置，关闭动画
7. 设置，网络和互联网，sim卡，首选网络类型，选NR/LTE/TDSCDMA/开头的，从而打开5G
8. 设置，默认应用，主屏幕应用，隐藏和保护应用
9. 传微信，clash，节点订阅
10. 设置，系统，状态栏，开启网络流量监视
11. magisk可能干扰微信、中国银行等的人脸验证，在magisk设置中伪装隐藏，但仍然可能无效
12. 天气软件weewow完美适配系统日历，闹钟等
13. 

## majisk 获取root

1. 开启lineage调试：进入lineage开发者模式（设置——关于手机——版本号连续点击开启）——返回设置，系统，开发者选项——调试部分打开USB调试——网络部分设置默认USB配置为文件传输。
2. 安装apk：电脑下载[magisk](https://github.com/topjohnwu/Magisk/releases)的安装包，传给手机并安装。然后电脑端修改安装包格式为zip。
3. 将zip刷入系统内：手机关机，长按电源键和音量加键进入recovery模式——apply update ——apply from ADB——电脑`打开cmd命令行.cmd`执行`.\adb.exe sideload Magisk-v25.2.zip`——手机提示验证失败继续（如果卡住电脑cmd回车）——reboot system now
4. 查看magisk 底边栏root可进入，至此magisk获取root成功。



### LSPosed模块生态丰富

1. [下载LSPosed模块zygisk版本](https://hub.fgit.gq/LSPosed/LSPosed/releases)(xposed只支持安卓8及以下)。(riru版本2022停止更新，且可能需要先下载[riru](https://github.com/RikkaApps/Riru/releases) )
2. 打开Magisk – 设置 – 开启 Zygisk
3. 打开Magisk – 模块 – 从本地安装 - 选择LSPosed压缩包 - 重启
4. 重启设备，点开通知栏 lsposed（如果没显示，可以通过拨号键输入 *#*#5776733#*#* 进入LSPosed）
5. 1.创建快捷方式 – 2.状态通知 关闭。若LSPosed 显示“已激活”则成功刷入。



### 冰箱随时冻结、解冻软件

禁用谷歌服务框架，就能直接跳过biubiu加速器广告，不显示广告直接拿到加速时长奖励，加速 PUBG mobile。



### AdHome广告拦截

[下载该模块](https://github.com/410154425/AdGuardHome_magisk)后用magisk添加：

浏览器打开http://127.0.0.1:3000/#filters，新增规则列表。



## kindle用U盘传文件





