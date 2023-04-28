# 目录

[TOC]

[菜鸟教程——shell](https://www.runoob.com/linux/linux-shell-variable.html)

# 语法
## 注释
* 单行注释： <kbd>#</kbd>
* 多行注释：

```bash script
<< 'COMMENT'
    your comment 1
    comment 2
    blah
COMMENT

<< EOF
Cmd line 1
Cmd line 2
Cmd line 3
Cmd line 4
EOF

```

## 创建shell脚本

```bash
vim gitbook.sh
i
# 编辑文本。
# 第一行#!/bin/zsh，其他可以同终端输入一样复制过来。
chmod +x ./gitbook.sh
./gitbook.sh

```
例如 gitbook 部署，原本批量运行大量命令行，如今可由以下脚本替代。

```bash
#!/bin/zsh
cd ~/Desktop/NoteBook
git pull
git add --all
git commit -a -m "." --no-verify
book sm
gitbook init
gitbook build
cd _book
git init
git add --all
git branch -M main
git commit -a -m "." --no-verify
git push --force --quiet "https://github.com/xxx/NoteBook.git" main:gh-pages
cd ~/Desktop/NoteBook
git push




```
