# 目录
[TOC]

# 使用技巧





## win定时任务自动备份重要文件

1. 新建`backup.bat`备份脚本。（rem是注释命令，注意文件夹路径尾部的`\`）

   ```basic
   @echo off
   
   rem 要备份的文件夹
   set sourcePath=C:\0r720\sourcePath\
   rem 要备份到的文件夹
   set targetPath=D:\BackupHistory\
   rem 自动备份生成的文件名
   set folderName=%date:~0,4%-%date:~5,2%-%date:~8,2%
   
   XCOPY /e /c /y "%sourcePath%*" "%targetPath%%folderName%\"
   
   ```

2. 利用系统定时任务执行脚本：控制面板 > 小图标 > 计划任务，最右侧操作菜单，创建基本任务，输入任务名、周期、时刻、启动程序，浏览选择脚本，勾选`完成时打开任务属性`，完成。在安全选项中勾选`不管用户是否登录都要运行`和`使用最高权限运行`，配置选win10。

3. 此时点确定会弹出`参数错误任务无效`，需要设置用户账户。安全选项，更改用户组，高级，立即查找，例如我是`r720`，选中，确定，确定，确定，即可成功。（可以看到主要是加了文件夹前缀，从`r720`变成了`R720\r720`）

4. 点击最左侧`任务计划程序（本地）`和`任务计划程序库`可以看到所有定时任务，日后可以在此修改设置。选中任务，点击右侧所选项的属性弹出修改。（但是好像又会出现参数错误任务无效的bug）



## 笔记本+工位双屏协同

- 原理：左侧显示器显示远程笔记本的第一屏，右侧显示器直连笔记本显示第二屏，笔记本鼠标可以无缝操作双屏，键盘可以直接用台式机操作双屏！（音量调节等部分功能不行。应该是只要笔记本鼠标把光标放在了第二屏，台式键盘只管输入内容，不论是哪个屏幕）
- 好处：便宜、享受、便携。既能在工位享受双屏，又能在下班后直接带走所有工作。（远程软件如todesk提供虚拟双屏？198元/年。如果不购买只能在一个显示器上来回切换）
- 屏幕：笔记本、台式大屏、笔记本拓展大屏从左向右
- 鼠标：以笔记本鼠标为主，台式鼠标放在右上角辅助操作
- 设置：
  - 笔记本鼠标：点击笔记本右下角通知栏，打开投影，设置为拓展，则右侧大屏显示第二屏（没有的话打开折叠，或者右键编辑图标，加入投影）
  - 台式机鼠标：开启todesk、向日葵等远程软件的全屏模式，则左侧大屏控制第一屏。
- 建议常用、显示文本类的应用放在第二屏，直连笔记本更清晰。
- 需要时间适应操作逻辑，不要混淆笔记本和台式，屏幕和屏幕。



## `attrib`命令隐藏文件夹

- cmd管理员运行 `attrib +s +h 文件夹完整路径`，就能隐藏。不需要了可以解，输入`attrib -s -h  文件夹完整路径`。

- 在默认的资源管理器中，即使勾选显示隐藏项目也不会显示，除非直接输入路径。不过对DirectoryOpus等第三方文件管理器无效，一般够用了。

- 更建议继续在隐藏目录下使用`attrib +s +h * /s /d` 和 `attrib -s -h * /s /d`，应用于所有文件夹和文件。

- `attrib /?` 查看命令用法：+/- 设置/清除属性，R 只读属性，A 存档属性，S 系统属性，H 隐藏属性。/S 和 /D 分别对该目录下所有匹配的文件和文件夹执行

## 添加软件到开始屏幕

找到软件安装目录下的`.exe`可执行程序，右键新建快捷方式，复制到`C:\ProgramData\Microsoft\Windows\Start Menu\Programs`，右键固定到开始屏幕。（可能无法点击开始菜单的程序列表中的程序，但是可以操作开始屏幕中的程序）

## 为无后缀文件设置默认打开程序

- 管理员运行cmd，`assoc  .`，查看无拓展的关联类型，应该显示没有找到文件关联.。
- `assoc .=No Extension`，自定义设置无后缀文件为`No Extension`类型
- `ftype "No Extension"="F:\pro\UltraEdit\uedit64.exe""%1"`，设置默认打开程序为ultraedit。
- `ftype`，查看所有后缀格式的默认打开程序，在`no extention`一行确认。
- (outdta有效，indta无效)

## 一键删除文件.bat

新建文本文件，重命名为`0一键删除outdta和rstplt文件.bat`，编辑以下内容。双击即可运行。

```bat
del /f /s /q outdta
del /f /s /q rstplt
```







## 用 diskgenius 将 win 环境迁移到虚拟机

？

## 批量修改文件名后缀

确保当前文件夹内都需要修改后缀——新建txt文档，写入以下内容 `ren *.* *.jpg*`——保存为bat格式——双击运行

## 删除Windows电脑本地PE系统



**方法1: Win + R输入msconfig打开系统配置，打开引导，删除不必要的启动项**

**方法2: Win + S运行搜索，输入cmd，以管理员身份运行——输入bcdedit，找到标识符，如果你在本地有PE系统，那么这里是一串代码，复制——然后bcdedit /delete \[你复制的代码\回车——最后弹出操作成功完成，说明成功了——然后我们在系统盘里面再把PE的文件删除即可**

## windows设置

- DWS移除win10跟踪及系统应用：[DWS下载后打开](https://github.com/Nummer/Destroy-Windows-10-Spying/releases)，设置菜单，专业模式选择所有（metro是预装应用）。工具菜单，删除所有metro应用（包括应用商店），删除One Drive，禁用UAC，禁用Windows update。主界面菜单，点击Destroy-Windows-10-Spying，重启。

- 任务栏：右键底部任务栏。搜索，显示桌面图标。显示桌面图标，关闭。显示任务视图，关闭。

- 关闭磁盘自动优化：右键磁盘，属性，工具，优化驱动器，关闭所有

- 最佳性能：右键我的电脑，属性，高级，性能，设置，调整为最佳性能，确定。

- 电源高性能模式：右键右下角电池图标，选择电源模式为高性能。如果没有该模式，搜索‘’命令提示符‘’，打开输入`powercfg -SETACTIVE 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c`。（适用于Intel CPU且操作系统是Windows 10 20H1或之后版本）

- 激活windows：HEU_KMS_Activator激活工具

- 迁移系统文件夹：文件管理器，桌面、文档等文件夹，右键属性，位置，文件路径前缀改为`D:\sys\`，确定。

- 启动项管理：ctrl + shift +esc打开任务管理器，启动，基本可以禁用所有。

- 卸载系统多余软件：在开始菜单中寻找，不需要的右键直接卸载。也可以右键开始菜单，弹出菜单中点击“应用和功能（F）”卸载应用。

- 关闭defender防火墙：在搜索栏中输入控制面板，打开，系统和安全，安全和维护下方的【更改用户账户控制设置】，Windows defender防火墙，左侧【启用或关闭Windows defender防火墙】，全部关闭，确定；

- 清空桌面系统图标和开始屏幕：右键桌面，个性化，主题，桌面图标设置，取消所有。在开始菜单中删除固定的不需要的应用，找到我的电脑、控制面板、计算机管理、power shell、命令提示符、运行等，拖动到开始屏幕固定。

- 设置底部任务栏图标：右键桌面，个性化，任务栏，打开或关闭系统图标

- 卸载小娜： Windows 10 2004 以上版本！搜索输入 PowerShell 并以管理员身份运行，`Get-AppxPackage -allusers Microsoft.549981C3F5F10 | Remove-AppxPackage`

- 设置，个性化，颜色，透明效果关闭

- 设置，系统，关于，重命名此电脑
- 关闭安全中心通知：控制面板\所有控制面板项\安全和维护，左侧更改安全与维护设置，取消勾选安全消息部分所有。
- 



## 安装&激活office

1. ccleaner卸载office，清理注册表
2. 复制此前安装的文件夹，或者下载运行[Office Tool Plus.exe](https://otp.landian.vip/zh-cn/download.html)。
3. [安装](https://www.coolhub.top/archives/11)：
   - 如果以前安装过，office tools文件夹会存在已下载的版本，软件自动匹配了相关配置，此时安装命令的配置
   - 如果与以前不一致，可能会无法安装。
   - 此时可以：1.执行以前版本的部署命令，弹出准备部署窗口，记住最后的Office版本，是，提示版本不一致，选择否就不会下载更新的版本，而是用本地安装包直接安装。2.重新下载安装。进入部署菜单，安装文件，删除该版本，执行下面部署命令。
   - 右上角命令，输入
     `deploy /addProduct O365ProPlusRetail_zh-cn_Access,Bing,Groove,Lync,OneDrive,OneNote,Outlook,Publisher,Teams /addProduct VisioProRetail_zh-cn_Groove,OneDrive /channel Current /downloadFirst`，（安装 Microsoft 365 企业应用版（Word, PowerPoint, Excel）+ Visio 2016）
   - 把以前的命令备份一下：`deploy /addProduct ProPlus2021Volume_zh-cn_Access,Groove,Lync,OneDrive,OneNote,Outlook,Publisher,Teams /addProduct VisioPro2021Volume__Groove,OneDrive /channel PerpetualVL2021 /downloadFirst`（安装 Office LTSC 专业增强版 2021 批量许可 (仅 Word, PowerPoint, Excel) + Visio LTSC 专业版 2021 批量许可。）
4. [激活](https://www.coolhub.top/archives/14)：
   1. 激活菜单，许可证管理小箭头，清除激活信息。
   2. 工具箱菜单，office工具，重置 Office 设置为默认设置。
   3. 右上角命令，输入`ospp /insLicID MondoVolume /sethst:kms.loli.best /setprt:1688 /act`和 和 和 和和 和 visio2021`ospp /insLicID VisioPro2021Volume /sethst:kms.loli.best /setprt:1688 /act`。
   4. 备份Office LTSC 专业增强版 2021 批量许可激活命令：`ospp /insLicID ProPlus2021Volume /sethst:kms.loli.best /setprt:1688 /act` 和 和 和 和和 和visio2021 `ospp /insLicID VisioPro2021Volume /sethst:kms.loli.best /setprt:1688 /act`。
   5. 激活失败，可以用HEU KMS Activator激活！ 
   6. 出错一般是激活代码更新了，前往激活教程核对。
5. 关闭自动更新
   在“开始屏幕”打开任意office应用，单击左下角“账户”，“更新选项”，禁用更新



## windows镜像/虚拟机vmware



1. [精简版windows](http://bbs.wuyou.net/forum.php?mod=viewthread&tid=429015)，只运行微信、tim，内存可低至1G。
2. [官方windows下载](https://www.microsoft.com/zh-cn/software-download/windows10ISO)
3. 



### win虚拟机扩展硬盘内存容量

- VMware扩容：虚拟机设置——硬件——硬盘——拓展磁盘容量
- win扩容：右键此电脑——管理——左侧存储，磁盘管理——找到未分配空间前一卷——右键拓展卷

### win虚拟机安装vm tools

- 开机状态下，点击vmware上方虚拟机菜单——安装VMware tools
- 如果安装为灰色：尝试在虚拟机设置中——设备——先删除再添加CD/DVD——使用安装时的iso文件——再次查看vmtools是否可点击（否则多次关机开机并重试）——自动安装开始
- 如果没有自动安装：进入文件管理器查看VMware盘符（D或E）——win+R输入D:\setup64.exe手动安装（或者E盘E:\setup64.exe，或者32位系统用D:\setup.exe）——（或许也可以直接在文件管理器内右键管理员运行？）

### win虚拟机无法使用共享文件夹

- 检查虚拟机设置里面是否有启用并添加了宿主机的共享文件夹
- 检查VMware Tools是否已经安装
- 打开文件管理器——此电脑——上方计算机菜单——映射网络驱动器————文件夹框内填写 `\\vmware-host\Shared Folders` 添加——完成后刷新确认左侧此电脑目录出现 `Shared Folder` 文件夹，网络目录出现 `vmware-host\Shared Folder` 文件夹，里面是所有的共享文件夹（可以右键固定到快速访问）

### win虚拟机迁移微信聊天记录

**将旧电脑WeChat Files目录内文件夹全部复制、替换到新电脑好像不行。**此外，手机和电脑可以相互迁移备份，但是虚拟机可能导致网络不符要求。

[查知乎解决了：路径要相同](https://www.zhihu.com/question/56343206/answer/477481955)

- 先保证版本都是最新的
- 如果设置过微信聊天记录文件位置，比如D:\Documents\Wechat Files，在新电脑选择D:\Documents重启微信。**不要新建Wechat Files 文件夹，会自动生成。**
- 关闭微信，删除新电脑D:\Documents\WeChat Files所有文件夹，复制旧电脑上的D:\Documents\WeChat Files 里面**All Users、wxid_xxxx两个文件夹**到新电脑。
- 可能需要编辑WeChat Files文件夹权限为everyone——完全控制。

> 原文
>
> 微信电脑版聊天记录的迁移的正确方法，微信电脑版重装系统还原聊天记录的办法。
> 百度知乎了很多，大部分人没说清楚原理。QQ聊天记录迁移大家都会，直接拷贝QQ聊天文件夹名称到新的电脑对应文件夹就好。**而微信PC版比较奇葩的地方在于，一定要保证原来的路径目录，而且一定要同时拷贝All Users目录**。原因是All Users目录里保存一些配置信息及微信账号头像，如果不拷贝这个目录，只拷贝你的微信ID文件夹，导致的结果就是登录后会重新更新聊天记录文件，最终导致微信聊天记录迁移失败。**如果你是电脑小白，在操作前，请千万记得备份新旧电脑上的整个Wechat Files文件夹，以免导致聊天记录丢失。如果你没有信心完全通过步骤自己搞定，建议先备份，否则私信找我，我也没有办法恢复。备份操作很简单，直接复制一份出来放到有足够剩余容量的地方即可，可以是当前目录，或者其他盘，或者是移动硬盘。**
>
> 
>
> 下面我来说一下正确的操作步骤。
>
> 情况一：没有设置过任何微信PC版文件存档设置。比如你原来电脑用户名是ABC，更换了一台电脑用户名不一样了叫CBA。**在正常情况下，没有主动变更过微信聊天记录保存位置，或者变更过我的文档路径，那么安装微信后，Windows Vista以上版本系统默认保存的位置就是C:\Users\CBA\Documents\WeChat Files 目录**里面。这个时候要迁移聊天记录，怎么办呢？关键点就在这里了，在新电脑里退出微信，在新电脑的C盘，Users（中文版操作系统可能显示为"用户"）目录下新建一个文件夹，叫ABC，在里面新建一个文件夹，叫Documents 。然后,登录微信PC版，在电脑版微信界面左下角有三个横线一样的标识，点击设置，然后点击左侧通用设置，文件管理，点击更改，然后选择**C:\Users\ABC\Documents**，点击确定，然后会提示重启微信。很多网友失败在这一部，因为Wechat Files 这个文件夹会自动生成，不需要再新建一个Wechat Files 文件夹。这个时候，你把微信关闭，把新电脑C:\Users\ABC\Documents\WeChat Files目录下所有文件夹删除（建议删除前做好备份），把原来电脑上的C:\Users\ABC\Documents\WeChat Files 里面All Users以及你的微信ID文件夹一起拷贝到新电脑目录 C:\Users\ABC\Documents\WeChat Files 里面。拷贝完毕，你会发现，之前旧电脑的所有聊天记录已经恢复。
>
> 
>
> 情况二：旧电脑有设置过微信PC版文件存档设置如果是设置过微信聊天记录文件位置的，比如是设置在D:\Documents\Wechat Files文件夹里的，那么迁移到新电脑，该如何处理呢？在新的电脑里，在D盘新建一个文件夹，叫Documents。然后登录微信PC版，在电脑版微信界面左下角有三个横线一样的标识，点击设置，然后点击左侧通用设置，文件管理，点击更改，然后选择**D:\Documents**，点击确定，然后会提示重启微信。很多网友失败在这一部，因为Wechat Files 这个文件夹会自动生成，不需要再新建一个Wechat Files 文件夹。这个时候，你把微信关闭，把新电脑D:\Documents\WeChat Files目录下所有文件夹删除（建议删除前做好备份），把原来电脑上的D:\Documents\WeChat Files 里面All Users以及你的微信ID文件夹一起拷贝到新电脑目录D:\Documents\WeChat Files 里面。拷贝完毕，你会发现，之前旧电脑的所有聊天记录已经恢复。
>
> 
>
> 情况三：重装系统重装系统情况比较复杂，如果要恢复聊天记录，得满足两个条件**第一，重装系统前知道微信聊天记录存档位置；****第二，重装系统之前有完整备份过整个Wechat Files 文件夹。**重装系统后，在正常情况下，没有主动变更过微信聊天记录保存位置，或者变更过我的文档路径，那么安装微信后，Windows Vista以上版本系统默认保存的位置就是C:\Users\CBA\Documents\WeChat Files （CBA为当前系统用户名）目录里面。而且默认只有重装系统后的聊天记录。这个时候要迁移聊天记录，怎么办呢？如果知道重装系统之前的位置，比如是位于D:\2018\Wechat\Wechat Files里面，那么接下来就在在D盘新建一个文件夹，叫2018，然后在2018里面建一个Wechat。然后登录微信PC版，在电脑版微信界面左下角有三个横线一样的标识，点击设置，然后点击左侧通用设置，文件管理，点击更改，然后选择**D:\2018\Wechat\**，点击确定，然后会提示重启微信。很多网友失败在这一部，因为Wechat Files 这个文件夹会自动生成，不需要再新建一个Wechat Files 文件夹。这个时候，你把微信关闭，把电脑D:\2018\Wechat\Wechat Files目录下所有文件夹删除（建议删除前做好备份），把备份的整个WeChat Files 里面All Users以及你的微信ID文件夹一起拷贝到目录D:\2018\Wechat\Wechat Files 里面。拷贝完毕，你会发现，重装系统之前电脑的所有聊天记录已经恢复。但是重装系统后的聊天记录就不会存在，可以通过登录微信PC版时勾选“登录后同步最近的聊天记录到Windows"来恢复最近两天的聊天记录。不同系统之间迁移，亲测有效，不需要ghost或者其他方法。如果帮助到你请点赞，如果不明白**请在操作前私信我，评论一般不回复，会被封禁**。
>
> 
>
> **2018-11-16 更新**经知友反馈，如果新电脑为更新版本的微信，可能导致在聊天记录管理器中，查找历史文件或者图片时只显示空白结果。解决方案为：1、把老电脑上的老版本微信安装目录直接拷贝至新电脑微信安装目录，覆盖所有文件。（已验证）2、在迁移之前，先把老电脑上的微信版本升级至新版本，与新电脑上的微信版本一致。升级成功后登录微信再退出，然后再迁移文件。
>
> **2018-11-23 更新**经知友反馈，当聊天记录同步好了，登录时会提示“文件默认保存位置无法使用，将不能正常使用微信，请更改位置”。 出现这个情况是权限不够引起的。 解决方案一，在使用微信的时候，不是直接双击打开，而是点右键，选择以管理员身份运行**（缺点：每次登录前都需要进行这个操作）**； 解决方案二，找到Wechat Files文件夹，点右键，属性，安全，编辑Everyone权限的权限为完全控制，然后点确定。
>
> **2019-07-01 更新**经知友反馈，在新电脑里建立同样的目录路径（尤其是C盘原始路径的），并且把微信位置设定为原来目录，重启后再登录时会提示“文件默认保存位置无法使用，将不能正常使用微信，请更改位置”。 出现这个情况仍然是权限不够引起的。 解决方案：关闭微信后，**找到Wechat Files文件夹，点右键，属性，安全，编辑Everyone权限的权限为完全控制，然后点确定。**
>
> [发布于 2018-08-25 11:18 ，编辑于 2020-02-25 18:40](https://www.zhihu.com/question/56343206/answer/477481955)



## 快捷键

* 大写：Shift + 字母 ：（有效避免忘了关大写锁定！）
* 下一个输入框：Table，上一个输入框`shift table`

## 添加环境变量

左下角开始（或右击我的电脑，属性）——设置——系统——关于——高级系统设置——高级——环境变量——（系统变量）新建——3次确定

## 固态/机械/移动硬盘速度对比

读/写（顺序/随机 + 8/1队列）

|                 | 固态 | 机械 | 固态硬盘盒USB3.0 | 机械移动 |
| --------------- | ---- | ---- | ---------------- | -------- |
| 顺序，读，8队列 | 2500 | 140  | 440              | 110      |
| 顺序，读，1     | 1500 | 130  | 340              | 110      |
| 随机，读，8     | 400  | 2    | 180              | 1        |
| 随机，读，1     | 40   | 1    | 23               | 1        |
| 顺序，写，8     | 1000 | 135  | 460              | 100      |
| 顺序，写，1     | 550  | 130  | 390              | 100      |
| 随机，写，8     | 300  | 1.5  | 190              | 9        |
| 随机，写，1     | 130  | 1..5 | 40               | 9        |

## windows永不更新

- 设置，时间与语言，关闭自动设置时间， 手动设置日期为30年后
- 设置，更新与安全，高级选项，暂停更新



# 软件

## BBDown下载B站视频

1. [下载BBDown](https://github.com/nilaoda/BBDown/releases)（windows下win-x64.zip，linux下linux-x64.zip），并解压到BBDown文件夹下，应该只有`BBDown.exe`或者`BBDwon`。

1. [下载ffmpeg](https://www.gyan.dev/ffmpeg/builds/)（ffmpeg-release-full.7z）实现视频音频合并能力，复制`\bin\ffmpeg.exe`到`BBDown.exe`同级文件夹。(ubuntu下载可能是`sudo apt install ffmpeg`)
2. 登录账号：在BBDown文件夹下右键打开cmd命令提示符，输入`BBDown login`，app扫码登陆。
3. 下载：`BBDown 链接或者BV号单集下载`，或者收藏夹等链接批量下载（点击头像进入个人空间，收藏菜单，收藏夹，复制右键或地址栏链接）。（linux先给运行权限`chmod +x BBDown`，然后`./BBDown 链接或BV号`，但是好像多P无法下载，无法合并第一集）
4. 有些字幕有问题无法下载，`BBDown --skip-subtitle 链接或BV号`跳过。

## EXCEL

### 筛选行号

* 空列，填入公式`=MOD(ROW()-2,10)`
第一行表头，每十行选一行（行号对10取模为2的行）
* 全选该列所有单元格（有数据的行）。
按住 `Shift` 键，拉动侧控制条到最后一个单元格
* `Ctrl + D` 填入所有公式
* 筛选该列数据

## Word

### 解决尾注默认在文档最后

尾注默认在文档最后，导致很多操作（如将致谢显示到最后）无法做到

整体思路：1.新增参考文献节、致谢节等从而可以操作这些内容。2.将尾注改到节最后显示从而使参考文献能置于致谢之前。3.通过在整篇文档取消尾注合并所有尾注，此时会在文档最后生成合并版的尾注。4.通过在参考文献节禁用“取消尾注”从而将尾注显示到这一节

1. 新增参考文献节、致谢节：光标放在正文最后——布局——页面设置——分隔符——下一页——输入参考文献四个字——同理继续插入下一页分节符——输入致谢节内容
2. 将尾注改到节最后: 光标放在参考文献四个字后——引用——脚注面板——位置选尾注，节的结尾——应用更改选整篇文档——应用
3. 合并所有尾注到最后一节：光标置于任意位置（或者有尾注的地方）布局——页面设置面板——版式/布局——勾选取消尾注——应用于整篇文档——确定
4. 将致谢节的尾注移动至上一参考文献节：光标置于参考文献节——布局——页面设置面板——布局/版式——取消勾选“取消尾注”——应用于本节——确定

### 复制文档样式
[参考](https://www.sohu.com/a/437851782_257714)
***
>打开**模板**文件，“开始”选项卡→“样式”，“管理样式”→“编辑”选项卡→“导入/ 导出”按钮，打开“管理器”对话框。
>
>单击“样式”选项卡→单击右侧的“关闭文件”，“打开文件”，单击“所有Word 模板”，打开下拉菜单→选择“所有文件”,找到**需要格式化的文件**。
>
>选择需要左侧模板复制的样式（<kbd>Shift</kbd> + <kbd>Ctrl</kbd> + 左键 ）→单击“复制”按钮。（可事先删除右侧样式，以防同名混淆）
### 无法组合图片和插入的文本框

（希望覆盖图中文字，随意拖动和排版）

原因：
>嵌入式的图片无法选中。改为浮与文字上方。
选中插入的文本框需要 Ctrl 且找准框体边缘。找准了会显示一个小加号和虚框。

技巧：
>在图片和文本框下部预留足够空间
图片改为浮动
Ctrl 逐个选中所有文本框，组合，置于顶层
调整位置，组合文本框和图片
组合改为嵌入

### 方程编号
[参考](https://www.cnblogs.com/liyafei/p/10469554.html)
***
>公式编号——插入编号——格式化——高级格式(#C1.#E1)——确定
插入公式——右编号——（复制已有无编号公式）——保存

若章节有误
>单击公式——公式编号——章节——修改分隔符

样式设置
>样式——样式检查器——MTDisplayEquation——修改
>* 段落——间距——段前段后6磅，单倍行距，勾选“相同样式段落无空格”，取消“如果定义文档网格，则对齐到网格”
>* 制表位——（进一步控制公式和编号的位置）——20字符，居中，设置——40字符，右对齐，设置——确定

[方程组](https://blog.csdn.net/qq_17753903/article/details/88320688)
>点击括号栏，选中大括号（2行左一）
点击方块阵列栏，选择合适的布局（2行右2）

### 文本插入模式
***
>将底部状态栏的“改写”改为“插入”。
（右键单击Word底部空白状态栏，在弹出的“自定义状态栏”中，勾选“改写”选项，以在底部显示。）

## Rss订阅

软件——

[Inoreader网页端](https://www.inoreader.com/all_articles)



找订阅的网站——

[RSSHUB](https://docs.rsshub.app/)，

[小宇宙播客排行榜](https://getpodcast.xyz/)，

[中文个人博客](https://blogwe.com/)，

[丑搜博客内容](https://uglysearch.othing.xyz/)



插件找订阅——
[RSS+油猴插件](https://greasyfork.org/zh-CN/scripts/373252-rss-show-site-all-rss)，

[Inoreader插件——RSS Reader](https://chrome.google.com/webstore/detail/rss-reader-extension-by-i/kfimphpokifbjgmjflanmfeppcjimgah),

[RSSHB插件——RSSHub Radar](https://chrome.google.com/webstore/detail/rsshub-radar/kefjpfngnndepjbopdmoebkipbgkggaa)



常用——

- B站up主视频——进入[up主页](https://space.bilibili.com/697166795), RSS+发现up主的动态订阅

- 订阅Telegram频道——（https://rsshub.app/telegram/channel/TestFlightCN ） ， 最后一词替换成频道名字，在频道信息链接中查看，如 https://t.me/TestFlightCN

- [订阅公众号](https://feeddd.org/feeds)——[人工采集](https://fastly.jsdelivr.net/gh/feeddd/feeds/feeds_all_rss.txt)，感恩珍惜……[其他办法](https://docs.rsshub.app/new-media.html#wei-xin)

- 订阅喜马拉雅专辑——替换最后的专辑编号( https://rsshub.app/ximalaya/album/203355 )。与喜马拉雅网址一致（https://www.ximalaya.com/album/203355 ）

- [订阅 Youtube 频道](https://anotherdayu.com/2023/4530/)：`https://www.youtube.com/feeds/videos.xml?channel_id=` + `Channel id`，最后的ID可以跳过复制频道主页的链接后打开这个[网站](https://commentpicker.com/youtube-channel-id.php)获取





## IPTV直播源
使用方法
* Windows：PotPlayer右键——打开——打开链接——输入直播源地址（.m3u）——确定
* Android：VLC。（或者部分设备浏览器（手机）上，可打开.m3u文件，单独播放其中的.m3u8文件）

[直播源]

[随时可能失效，github项目](https://github.com/fanmingming/live)

* 🌏Global直播源	https://live.fanmingming.com/tv/m3u/global.m3u	198个	2023.8.13
* 📺IPTV(IPV6专用)	https://live.fanmingming.com/tv/m3u/ipv6.m3u	120个	2023.8.13
* 📻Radio直播源	https://live.fanmingming.com/radio/m3u/index.m3u	317个	2023.5.3





无法播放？

* 在clash中打开IPV6

## clash 科学上网

1. 下载clash软件并安装。[win/linux——win展开更多，下载Clash.for.Windows.Setup.0.20.21.exe
，ubuntu下载Clash.for.Windows-0.20.21-x64-linux.tar.gz
](https://github.com/Fndroid/clash_for_windows_pkg/releases/download)。[android下载cfa-2.5.12-foss-universal-release.apk](https://github.com/Kr328/ClashForAndroid/releases)。

2. 打开clash——General通用——打开system proxy和Start with windows，从而打开网络代理和自启动。

3. Profiles配置——粘贴机场订阅链接——download——右击机场配置——settings——设置update interval更新间隔为1小时。

4. Proxies代理——选择Rule模式——点击类似WiFi的测速按钮——选择延迟较低的


5. Settings——proxies——Order by——选择Latency，使得代理可以按照延迟排序。

6. 如果无法连接，查看本机ip是否与设置ip一致

   

7. 机场代理的规则如果不合适可以自行修改：Settings——Profiles——parsers
    ——修改完一定要在Profiles更新订阅。具体可以参考[lbyczf](https://docs.cfw.lbyczf.com/contents/parser.html)。

```yaml
parsers: # array
  - url: 【替换成你需要配置的的机场订阅链接】
    yaml:
      prepend-rules: # 设置规则，如某域名需要直连或者代理
        - DOMAIN-SUFFIX,nist.gov,DIRECT
        - DOMAIN,yuguo.us,GLOBAL
```

## 电子书

**看什么书**

1. [豆瓣读书top250](https://book.douban.com/top250?icn=index-book250-all)

2. [2013版豆瓣读书top250](https://www.douban.com/doulist/43621091/)

3. [你所在领域的入门书单是怎样的？ - 知乎](https://www.zhihu.com/question/51265095)

4. [通识千书书单](https://docs.qq.com/sheet/DY2RmcVVMVE9Qd3JV?tab=xwaub9)

   

**从哪找书**

1. [zlibrary](https://lib-c463qd25h2ypw2am7wpqmfy6.1lib.ph)，公网仅有私密域名，与本人账号唯一绑定
2. [读书](https://doosho.com/)——可能是电子书爱好者自制的，少量古籍和现代作品，甚至还有马督工睡前消息的[文字稿](https://doosho.com/cn/44)
3. [安娜书库存档](https://annas-archive.org/)，含zlibrary，libgen等，速度慢？
4. [全国图书馆参考咨询联盟](http://www.ucdrs.superlib.net/)，国家数据库，可查信息，可试看
5. [图书联盟](https://dazzling-bacon-763.notion.site/eecbb4284b434f76a85d99f240349ddb)，付费，网上找书商家应该用的这个
6. [电子书独秀库百度网盘妙传下载](https://freembook.com/)（好像失效了，存不到网盘内）



# 问题





## 电脑蓝牙无法搜到耳机等设备

1. 原因可能是卸载了realtek音频组件等。
2. 在服务中打开`蓝牙支持服务(Bluetooth Support Service)`。
3. 其他可能解决方案：更新蓝牙驱动，打开Bit Locker Drive Encryption Service服务，删除其他所有设备的配对信息或者重置耳机。

## 笔记本无法联网

可能是之前一直是通过虚拟机内弹出的网页验证。

打开虚拟机验证网络。

## VMware的win10蓝屏pfn share count

关闭VMware的3d加速：虚拟机，设置，硬件，显示器，取消加速3D图形。

（问题复现：VMware使用精简版win10的ultraedit的查找功能时，如果在第一个/最后一个查找结果后在查找上一个/下一个，直接蓝屏死机： 错误代码 PFN SHARE COUNT，位于 vm3dmp.sys。）



## VMware虚拟机自动挂起

右键左下角，按键p打开控制面板，搜索电源选项，打开隐藏计划的高性能模式。更改计划设置，从不关闭显示器和进入睡眠。更改高级电源设置，硬盘设置为从不关闭。

## VMware虚拟机正在被使用无法运行

找到虚拟机位置，将后缀为.lck的文件夹删除，或者备份，或者添加此文件夹后缀为.backup。

## wifi不跳转网页验证

手动在浏览器输入`192.168.1.1`，自动跳转当前wifi的验证网址。

## .msi 安装失败

 以管理员身份运行。右键安装包, 或在Directory Opus中开启。



## 无法识别U盘、移动固态硬盘盒

- 打印机无法识别U盘，学校电脑无法识别移动固态硬盘盒。
- U盘格式化为FAT32格式，兼容性最好。分配单元为默认配置大小。（对于打印机，U盘容量最好32G及以下)
- 但是移动固态硬盘盒的固态还是不建议格式化，一个是容量太大兼容性还是不行，另外是浪费性能，讲解个PPT保险起见还是用u盘吧。（超过32G无法使用系统自带的格式化，只能使用Diskgenius等第三方软件，且风险未知。）
- 硬盘格式：
（不同的操作系统有不同的文件系统，Linux 使用 ext4，苹果OSX使用 HFS +，Windows 使用 NTFS，Solaris 和 Unix 使用ZFS。）
>- FAT32：最老，所有操作系统都支持，兼容性最好。但是，它是为32位计算机设计的，单文件不能超过 2^32 - 1 个字节，也就是不能超过 4GB，分区不能超过 8TB。
>- NTFS ： Windows 的默认文件系统，用来替换 FAT32。Windows 的系统盘只能使用这个系统，移动硬盘买来装的也是它。在非Windows系统可能需要手动安装驱动程序，例如在Mac系统中NTFS仅支持读取操作，安装相关驱动程序后才能支持读写，但是读写速度会有所降低。
>  - NTFS内置日志功能可以记录尚未实际写入的磁盘数据变化，这有助于在系统崩溃的情况下恢复，而exFAT文件系统缺少该功能。
>  - 此外，NTFS文件系统是默认加密的，exFAT文件系统中的加密需要手动处理。
>- exFAT ：可以看作是 FAT32 的64位升级版，（extended，表示"扩展的 FAT32"），功能不如 NTFS，但是解决了文件和分区的大小问题，两者最大都可以到 128PB。由于 Mac 和 Linux 电脑可以读写这种系统，所以移动硬盘的文件系统可以改成它，但是不推荐。__不适合作为硬盘这种用于存放数据场景下的文件系统。__

# 0
