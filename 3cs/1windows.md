# 目录
[TOC]

# 使用技巧
## 快捷键
* 大写：Shift + 字母 ：（有效避免忘了关大写锁定！）
* 下一个输入框：Table，上一个输入框`shift table`

# 软件
## 添加环境变量
左下角开始（或右击我的电脑，属性）——设置——系统——关于——高级系统设置——高级——环境变量——（系统变量）新建——3次确定

## EXCEL
### 筛选行号

* 空列，填入公式`=MOD(ROW()-2,10)`
第一行表头，每十行选一行（行号对10取模为2的行）
* 全选该列所有单元格（有数据的行）。
按住 `Shift` 键，拉动侧控制条到最后一个单元格
* `Ctrl + D` 填入所有公式
* 筛选该列数据

## Word
### 尾注默认在文档最后，导致很多操作（如将致谢显示到最后）无法做到
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
### 无法组合图片和插入的文本框（希望覆盖图中文字，随意拖动和排版）
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
软件——[Inoreader网页端](https://www.inoreader.com/all_articles)

找订阅的网站——[RSSHUB](https://docs.rsshub.app/)，
[播客](https://getpodcast.xyz/)，
[中文个人博客](https://blogwe.com/)

插件找订阅——
[RSS+油猴插件](https://greasyfork.org/zh-CN/scripts/373252-rss-show-site-all-rss)，
[Inoreader插件——RSS Reader](https://chrome.google.com/webstore/detail/rss-reader-extension-by-i/kfimphpokifbjgmjflanmfeppcjimgah),
[RSSHB插件——RSSHub Radar](https://chrome.google.com/webstore/detail/rsshub-radar/kefjpfngnndepjbopdmoebkipbgkggaa)

- B站up主视频——进入[up主页](https://space.bilibili.com/697166795), RSS+发现up主的动态订阅

- 订阅Telegram频道——（https://rsshub.app/telegram/channel/TestFlightCN ） ， 最后一词替换成频道名字，在频道信息链接中查看，如 https://t.me/TestFlightCN

- [订阅公众号](https://feeddd.org/feeds)——[人工采集](https://fastly.jsdelivr.net/gh/feeddd/feeds/feeds_all_rss.txt)，感恩珍惜……[其他办法](https://docs.rsshub.app/new-media.html#wei-xin)

- 订阅喜马拉雅专辑——替换最后的专辑编号( https://rsshub.app/ximalaya/album/203355 )。与喜马拉雅网址一致（https://www.ximalaya.com/album/203355 ）

- [订阅 Youtube 频道](https://anotherdayu.com/2023/4530/)：`https://www.youtube.com/feeds/videos.xml?channel_id=` + `Channel id`，ID可以跳过复制频道主页的链接后打开这个[网站](https://commentpicker.com/youtube-channel-id.php)获取

## IPTV直播源
使用方法
* Windows：PotPlayer右键——打开——打开链接——输入直播源地址（.m3u）——确定
* Android：VLC。（或者部分设备浏览器（手机）上，可打开.m3u文件，单独播放其中的.m3u8文件）

[直播源]
* [国内，快，IPV6](https://raw.githubusercontent.com/YueChan/IPTV/main/IPTV.m3u)
* [仓库，国人的](https://github.com/imDazui/Tvlist-awesome-m3u-m3u8)
* [仓库，国外](https://github.com/iptv-org/iptv)

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

6. 机场代理的规则如果不合适可以自行修改：Settings——Profiles——parsers
——修改完一定要在Profiles更新订阅。具体可以参考[lbyczf](https://docs.cfw.lbyczf.com/contents/parser.html)。

```yaml
parsers: # array
  - url: 【替换成你需要配置的的机场订阅链接】
    yaml:
      prepend-rules: # 设置规则，如某域名需要直连或者代理
        - DOMAIN-SUFFIX,nist.gov,DIRECT
        - DOMAIN,yuguo.us,GLOBAL

```

# 问题
### wifi 无法连接，不跳转网页验证
手动在浏览器输入`192.168.1.1`，自动跳转当前wifi的验证网址。

### .msi 安装失败

 以管理员身份运行。右键安装包, 或在Directory Opus中开启。

### 打印机无法识别U盘

- 格式化为FAT32格式。分配单元为默认配置大小。（容量最好32G及以下）
- 硬盘格式：
（不同的操作系统有不同的文件系统，Linux 使用 ext4，OSX使用 HFS +，Windows 使用 NTFS，Solaris 和 Unix 使用ZFS。）
>- FAT32：最老，所有操作系统都支持，兼容性最好。但是，它是为32位计算机设计的，文件不能超过 2^32 - 1 个字节，也就是不能超过 4GB，分区不能超过 8TB。
>- NTFS ： Windows 的默认文件系统，用来替换 FAT32。Windows 的系统盘只能使用这个系统，移动硬盘买来装的也是它。
>- exFAT ：可以看作是 FAT32 的64位升级版，（extended，表示"扩展的 FAT32"），功能不如 NTFS，但是解决了文件和分区的大小问题，两者最大都可以到 128PB。由于 Mac 和 Linux 电脑可以读写这种系统，所以移动硬盘的文件系统可以改成它，但是不推荐。__不适合作为硬盘这种用于存放数据场景下的文件系统。__


#
