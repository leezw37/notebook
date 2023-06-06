# 目录
[TOC]

# 使用技巧

## MIUI关闭系统广告

关闭系统广告，冰箱冻结智能服务和analytics。

## AdGuard home广告拦截模块

https://github.com/410154425/AdGuardHome_magisk（甚至可以跳过biubiu加速器广告直接拿到时长奖励）：

浏览器打开http://127.0.0.1:3000/#filters，新增规则列表。



## 刷机

**准备**

1. rom包和recovery包！[lineage](https://download.lineageos.org/devices/lmi/builds)(lineage系统必须使用专用recovery)
2. 手机设置——小米账号——云备份——桌面云备份
3. 手机设置——我的设备——备份与恢复——手机备份
4. 连接电脑——复制文件夹MIUI/backup/allbackup
5.  手机设置——更多设置——开发者选项——打开USB调试（我的设备——全部参数——miui版本连续点击）（设备解锁状态，需要提前一周申请）

（关机时同时长按音量下键和开关机键，可以进入fastboot模式，音量+和电源键则是recovery模式）

**开始刷机**

1. 刷miui的Recovery：打开脚本 `H:\3backup\redmi K30 PRO\刷机\TWRP\recovery-twrp一键刷入工具-1-先跳过谷歌验证.bat`，按照指示操作，查看手机允许usb调试，重启后自动先后进入fastboot和recovery模式。（更改bat脚本中的包名set rec_img=从而刷入不同的recovery包：刷miui用twrp_3.6.2_LMI_unofficialtwrp.img，刷lineage用lineage_recovery.img）
2. （可选：跳过验证谷歌）：打开recovery后，点击高级，点击终端，高通cpu的输入dd if=/dev/zero of=/dev/block/by-name/frp，联发科cpu输入dd if=/dev/zero of=/dev/block/bootdevice/by-name/frp，最后重启到fastboot。
4. 刷lineage的recovery：打开脚本`H:\3backup\redmi K30 PRO\刷机\TWRP\recovery-twrp一键刷入工具-2-lineage.bat`,进入recovery。
5. 刷入lineage系统：Factory Reset所有——apply update ——apply from ADB——电脑`打开cmd命令行.bat`执行`.\adb.exe sideload lineage-20.0-20230325-nightly-lmi-signed.zip`
6. 刷入谷歌套件：再次apply update，apply from ADB，执行`.\adb.exe sideload MindTheGapps-13.0.0-arm64-20221025_100653.zip`刷入google
(手机提示signature verification failed，点击继续。)——手机reboot system now

~~4. 刷入miui：清除数据（wipe）——格式化data分区——返回高级清除（advanced wipe）——勾选所有（）——返回重启——recovery。安装（install）——sdcard/twrp(复制lineage安装包到此目录)——重启~~

**问题**

1. lineage 重新刷回 低版本安卓 miui后，如果更新系统会一直卡在recovery，无法进入系统。解决办法是重新刷回lineage，或者最新版本miui（也无法更新系统，但是可以进入系统）。
1. lineage 刷 microG 发现没有声音。（）
2. 若无法挂载data无法解密，搜索 twrp k30 pro最近一年，下载最新twrp.img
3. 无法[跳过验证谷歌](https://www.hztdst.com/2985.html)，进入twrp，点击高级，点击终端，联发科cpu输入dd if=/dev/zero of=/dev/block/bootdevice/by-name/frp后点图3右下角的勾勾就可以了，高通cpu的输入dd if=/dev/zero of=/dev/block/by-name/frp
4. lineage开发者模式，设置——关于手机——版本号点需点击

**参考**

[LineageOS 教程](https://wiki.lineageos.org/devices/lmi/install)

[安装MIUI或者lineage](https://www.cnblogs.com/ls1519/p/16088770.html)

[跳过验证谷歌](https://www.hztdst.com/2985.html)

**majisk 面具**
1. [github](https://github.com/topjohnwu/Magisk/releases)下载magisk的apk包，安装到手机。修改安装包格式为zip传给电脑。
2. 打开脚本`H:\3backup\redmi K30 PRO\刷机\TWRP\recovery-twrp一键刷入工具-2-lineage.bat`,进入recovery。
3.  Factory Reset——apply update ——apply from ADB——电脑`打开cmd命令行.cmd`执行`.\adb.exe sideload Magisk-v25.2.zip`——手机reboot system now
4. 查看magisk 底边栏root是否可进入（magisk和root成功，但lsposed以下内容失败，图标不显示也无法呼出，包括riru和zygisk两个版本）
5. [下载LSPosed模块zygisk版本](https://hub.fgit.gq/LSPosed/LSPosed/releases)(xposed只支持安卓8及以下)。(riru版本2022停止更新，且可能需要先下载[riru](https://github.com/RikkaApps/Riru/releases) )
6. 打开Magisk – 设置 – 开启 Zygisk
7. 打开Magisk – 模块 – 从本地安装 - 选择LSPosed压缩包 - 重启
8. 重启设备，点开通知栏 lsposed（如果没显示，可以通过拨号键输入 *#*#5776733#*#* 进入LSPosed）
9. 1.创建快捷方式 – 2.状态通知 关闭。若LSPosed 显示“已激活”则成功刷入。

**谷歌相机**

[教程](https://zhuanlan.zhihu.com/p/538817403?utm_id=0)

1. 新建GCam/Configs7文件夹，并将[下载的配置文件](https://www.celsoazevedo.com/files/android/p/f/2021/11/pocof2pro-urnyx05-v2.xml)放入
2. 下载安装[谷歌相机](https://1-dontsharethislink.celsoazevedo.com/file/filesc/GCam_7.3.018_Urnyx05-v2.6.apk)，连击快门按钮周边的区域，选择配置。（[若无谷歌框架需要解决闪退](https://blog.csdn.net/ONE_SIX_MIX/article/details/123444620)，方法是下载安装[basic版依赖](https://github.com/lukaspieper/Gcam-Services-Provider/releases)）
3. 点击上部箭头进入相机设置——照片——使用第三方图库



