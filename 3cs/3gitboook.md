# 目录
[TOC]


# 加密文档
[日记](https://l2w.top/9%E4%B8%AA%E4%BA%BA/%E6%97%A5%E8%AE%B0.html)

工具——
[技术实现](https://github.com/robinmoisson/staticrypt)

[类似的许多工具](https://mprimi.github.io/portable-secret/#prior-art)
# GitBook

[官方文档](https://docs.gitbook.com/)

[打造完美写作系统：Gitbook+Github Pages+Github Actions](https://blog.51cto.com/phyger/5276526#Gitbook_28)

[GitBook 简介安装配置](https://gitbook.curiouser.top//origin/gitbook-简介安装配置.html)

[GitHub Pages 搭建教程](https://sspai.com/post/54608)

## 安装 (node、npm、nvm、gitbook)
[ubuntu 参考](https://blog.csdn.net/PY0312/article/details/105903320)


```bash

# 在/usr/local目录下载nodejs安装包
cd /usr/local
# 不要下最新版本,问题很多 https://nodejs.org/en/download/
get https://nodejs.org/dist/v12.18.1/node-v12.18.1-linux-x64.tar.xz
tar xvf node-v12.18.1-linux-x64.tar.xz
mv node-v12.18.1-linux-x64 nodejs

# 配置，需要创建软链接，以便全局使用：(否则node -v 报错：-bash: /usr/bin/node: 没有那个文件或目录）
ln -s /usr/local/nodejs/bin/node /usr/bin/node
ln -s /usr/local/nodejs/bin/npm /usr/bin/npm

# 解决[/usr/local/bin/npm: 没有那个文件或目录](https://stackoverflow.com/questions/35018661/bash-usr-local-bin-npm-no-such-file-or-directory)
sudo mkdir -p /usr/local/bin
sudo ln -s /usr/bin/npm /usr/local/bin/npm

# 解决npm ERR! code EACCES
sudo chown -R `whoami` ~/.npm
sudo chown -R `whoami` /usr/local/lib/node_modules
sudo chown -R `whoami` /usr/local/bin
# 完成检验：
node -v && npm -v

# 升级 nodejs 和 npm 版本；
sudo npm install n -g
# 不需要最新版的nodejs
# sudo n stable
sudo npm i -g npm
# 配置 npm 镜像源为淘宝源；
npm config set registry http://registry.npm.taobao.org/

# 二、安装 gitbook
# 先安装 gitbook-cli，执行以下命令
sudo npm install -g gitbook-cli
# 查看版本并安装 gitbook
# 输入命令gitbook -V会自动帮我们安装 gitbook，需要注意这一步会很慢，执行 Installing GitBook 3.2.3 大概需要20分钟左右，请耐心等待；
gitbook -V

# 报错 TypeError: cb.apply is not a function
# 62行起的三行都注释掉：fs.stat = statFix(fs.stat), fs.fstat = statFix(fs.fstat), fs.lstat = statFix(fs.lstat)
sudo vim /usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js

# 验证是否安装成功
# 安装成功后，输入gitbook -V再次查看版本看是否安装成功。仅显示CLI、GitBook version即表示安装成功；
gitbook -V

# 报错[TypeError [ERR_INVALID_ARG_TYPE]]
# node 版本过高，安装node.js的版本管理工具nvm。用nvm再安装一个低版本，可切换。
bash -c "$(curl -fsSL https://gitee.com/RubyKids/nvm-cn/raw/master/install.sh)"
# 重启
nvm install v12.18.1
nvm use v12.18.1
npm install gitbook-cli

# 进入自己的gitbook目录
mkdir ~/Desktop/NoteBook/gitbook
cd ~/Desktop/NoteBook/gitbook
gitbook init

# 安装完毕。编辑SUMMARY.md和其他文件后发布
# 插件可自动生成目录
npm install -g gitbook-summary
cd ~/Desktop/Notebook
book sm

gitbook init
gitbook build

# 启动服务，可用于本地浏览器浏览
gitbook serve &
# 用浏览器查看我们创作的内容
http://localhost:4000/

```

[linux 卸载node重新安装](https://www.jianshu.com/p/908527186a68)
[/usr/local/bin/npm: 没有那个文件或目录](https://stackoverflow.com/questions/35018661/bash-usr-local-bin-npm-no-such-file-or-directory)
[npm ERR! code EACCES](https://stackoverflow.com/questions/25290986/how-to-fix-eacces-issues-with-npm-install)
[TypeError: cb.apply is not a function](https://github.com/GitbookIO/gitbook-cli/issues/110#issuecomment-669640662)
[各种错误](https://segmentfault.com/a/1190000040796206#item-4)


## 使用

[打造完美写作系统：Gitbook+Github Pages+Github Actions](https://blog.51cto.com/phyger/5276526#Gitbook_28)
### 本地浏览器
启动服务，可用于本地浏览器浏览 `gitbook serve`
用浏览器查看我们创作的内容 `http://localhost:4000/`

### 插件
编写 book.json

```json

{
    "title": "lzw笔记",
    "outputfile": "SUMMARY.md",
    "catalog": "all",
    "ignores": [],
    "unchanged": [],
    "sortedBy": "-",
    "disableTitleFormatting": true,
    "plugins": [
        "favicon-absolute",
        "katex",
        "-lunr",
        "-search",
        "search-pro",
        "splitter",
        "page-treeview",
        "edit-link",
        "anchor-navigation-ex",
        "expandable-chapters",
        "copy-code-button"
    ],
    "pluginsConfig": {
        "page-treeview": {
            "copyright": "Copyright &#169; aleen42",
            "minHeaderCount": "2",
            "minHeaderDeep": "2"
        },
        "edit-link": {
            "base": "//github.com/xxx/notebook/edit/master",
            "label": "编辑此页面"
        },
        "favicon-absolute": {
            "favicon": "/favicon.ico"
        }
    }
}
```

#### 使用
> 1. book.json内添加插件`"plugins": ["插件名"]`,新插件放前面,install完了就中断。 gitbook install
> 2. 也可以分别直接用淘宝镜像下载npm包，更快


```bash

nvm use v18

npm install gitbook-plugin-favicon-absolute -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-katex -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-search-pro -g --registry=https://registry.npm.taobao.org/
npm install gitbook-summary -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-expandable-chapters --save -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-copy-code-button --save -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-anchor-navigation-ex --save -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-splitter -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-page-treeview -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-edit-link -g --registry=https://registry.npm.taobao.org/
npm install mathjax@2.7.6 -g --registry=https://registry.npm.taobao.org/
npm install gitbook-plugin-mathjax-pro -g --registry=https://registry.npm.taobao.org/
npm install staticrypt -g --registry=https://registry.npm.taobao.org/

```
* [自动生成目录](https://github.com/imfly/gitbook-summary)`gitbook-summary` , 根目录执行 book sm
* 网页目录可折叠`"expandable-chapters"`
* [复制代码按钮](https://github.com/WebEngage/gitbook-plugin-copy-code-button)`"copy-code-button"`
* [标题导航](https://github.com/zq99299/gitbook-plugin-anchor-navigation-ex)`"anchor-navigation-ex"`
* 侧边栏宽度可调节 `"splitter"`
* 页面顶部显示目录 `"page-treeview"`
* 在线编辑文件 `"edit-link"`
* LaTex插件 `"katex"` > `"mathjax-pro"`
* 导出整本电子书 `calibre`

```bash

# 安装 calibre
sudo -v && wget -nv -O- https://download.calibre-ebook.com/linux-installer.sh | sudo sh /dev/stdin
# 可以导出为 pdf、epub 等
gitbook pdf ./ ./mybook.pdf
gitbook epub ./ ./mybook.epub
gitbook mobi ./ ./mybook.mobi

```

#### 禁用
>自带的：在插件名前<kbd>-</kbd>；引入的：json中删除
>
```json
>"plugins":[ "-search"]
>
```

## 启用GitPage
[打造完美写作系统：Gitbook+Github Pages+Github Actions](https://blog.51cto.com/phyger/5276526#Gitbook_28)

[GitBook 简介安装配置](https://gitbook.curiouser.top//origin/gitbook-简介安装配置.html)

[gitbooks-devops-roadmap](https://github.com/Curiouserw/gitbooks-devops-roadmap)

[GitHub Pages 搭建教程](https://sspai.com/post/54608)

* [选网页主题，进github仓库，fork后改名xxx.github.io](http://jekyllthemes.org/)
* settings——pages——custom domain———填入`xxx.github.io`或者其他花名

## gitbook上传到gitpage网页
### 初次部署
vscode左下新建gitbook发布分支gh-pages。
>main分支只有源代码和配置文件；
gh-pages分支生成gitbook，删除其他所有不必要的内容，上传到仓库后会被同步到gitpage网站。

* vscode左下切换到发布分支gh-pages。

```bash
git checkout gh-pages

```
* 创建gitbook，生成_book文件夹

```bash
cd _book
gitbook init
gitbook build

```
* 本地提取_book文件夹内文件，删除父目录内其他所有文件。
* 同步。
* settings——pages——branch——gh-pages——save
* 查看 gitbook 是否生效 https://xxx.github.io/NoteBook/
* 切换回 main 分支。
### 日后部署
>使用gh-pages插件将本地静态页面HTML文件推送到远处仓库gh-pages分支

```bash

npm install gh-pages -g
gh-pages -d _book

```

## PicGo 图床管理

[项目地址](https://github.com/Molunerfinn/PicGo)

[使用手册](https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A)

[jsDelivrCDN](https://www.jsdelivr.com/?docs=gh)自定义域名加速——`https://cdn.jsdelivr.net/gh/xxx/notebook-img`

picgo设置——设置快捷键——禁用快速上传快捷键

## 配置自定义域名
1. 选择[阿里云域名购买](https://wanwang.aliyun.com/domain/)，市场份额过半。top和xyz两个新顶级域名 比较便宜。（top不良使用率高，xyz长期费用贵）
2. 配置 DNS 解析

```bash

# dig 命令显示自定义域名指向的IP地址
dig xxx.github.io

# 报错无dig
sudo apt install bind9-dnsutils
# 再报错：未满足的依赖关系：
# bind9-dnsutils : 依赖: bind9-libs (= 1:9.16.1-0ubuntu2) 但是 1:9.16.1-0ubuntu2.10 正要被安装
sudo apt remove bind9-libs
sudo apt install bind9-libs
sudo apt install bind9-dnsutils

dig xxx.github.io
# 将以下应答中的所有 ipv4 地址填入阿里云
;; ANSWER SECTION:
xxx.github.io.	5	IN	A	185.199.111.153
xxx.github.io.	5	IN	A	185.199.110.153
xxx.github.io.	5	IN	A	185.199.109.153
xxx.github.io.	5	IN	A	185.199.108.153


# 在阿里云域名解析设置中添加四条纪录，格式如下：https://dns.console.aliyun.com/#/dns/setting/l2w.top

# 记录类型 A，主机记录 @，解析请求来源 默认，记录值 185.199.108.153 （上面的IP地址）， TTL 1小时

# 再加一条——记录类型 CNAME，主机记录 www，解析请求来源 默认，记录值 xxx.github.io， TTL 1小时

```
3. 在网站仓库主分支新建文件 CNAME，填写域名 l2w.top。（踩坑：直接在仓库-设置-Pages中填域名不行，每次重建网站都会被清空。）

#
