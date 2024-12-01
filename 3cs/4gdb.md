# 目录

[TOC]

# 基本使用

## 开始调试 (矩阵条件数)

```bash
gdb --args ./momentum-dbg -i problems/123-2.i  -pc_type svd -pc_svd_monitor

```

## 断点

```bash
# 设置断点 (行号、函数名), (运行前提示找不到无所谓，y确定）
b /home/lee/projects/moose/modules/fluid_properties/src/userobjects/Water97FluidProperties.C:260
b NonlinearSystem::solve()
b MPI_Abort
# 查看所有断点
info b
# 删除断点
delete 1 2 3
# 禁用断点
disable 1 2 3
# 激活断点
enable 1 2 3

```

## 运行控制


```bash
# 从头开始运行
r
# 继续运行到下一个断点
c
# 下一步
n
# 下一步（深入代码调用）
s
# 运行到循环结束
u
# 被调用函数正常运行到结束，并返回调用函数
finish
# 被调用函数立即结束函数，并返回值
return
# 退出调试
q

```

## 显示上下文

```bash
 l

```

## 变量

```bash
# 设置需要显示的变量parameter (p 和 display)
p _T[_qp]
display _drho_dh[_qp]
# 显示
display
# 禁用变量
disable display 1 2 3
# 删除变量
undisplay 1 2 3
# 查看已设置的变量信息
info display

# 输出某个地址的值
p &0x555555

# 输出某个数组，@length指定某个长度，否则默认只输出第一个元素。
p **array@length

```

## TUI 窗口


```bash
# 打开文本用户交互（TUI）窗口
tui enable
# 调整代码区高度
winheight src -10

```

## 帮助


```bash
# 显示帮助
help 命令

```

# 实践

## 浮点数溢出


```bash
# gdb 调试程序，定位问题
r
# 运行到浮点溢出后，回溯堆栈 backtrace stack
bt
# 找到需要调试的那一行，打断点
b xx/xx/xxx.C:xx


```

