# 目录

[TOC]

# 参考

[vscode插件Markdown Preview Enhanced——可转电子书格式](https://shd101wyy.github.io/markdown-preview-enhanced/#/)

[Mermaid 图表](https://mermaid.js.org/intro/)

[html-2-markdown](https://www.convertsimple.com/convert-html-to-markdown/)

# 标题、注释、页内跳转

## 换行

连续空格+换行符;

 空行

## 标题分级

等于号1级、减号2级（要换行），连续#号1~6级

## 注释
以下注释不可见

```
[//]: # "首选兼容性好"
[//]: # "首选兼容性好"
[comment]: <> "我是注释看不见我"
[//]: <> "这也是注释"

<!--单行
多行都可以
 -->

<?单行多行都可以?>

```
## 页内跳转
[点击跳转](#jump)


```
[点击跳转](#jump)

定义一个锚(id)：<span id="jump">跳转到的地方</span>

```

定义一个锚(id)：<span id="jump">跳转到的地方</span>

# 强调
*斜体*  _italic_
**加粗**  __bold__
转义符 \\ , \#
内联 `inline code` 代码
键位<kbd>Ctrl</kbd>+<kbd>Esc</kbd>
~~删除线~~

```
*斜体*  _italic_
**加粗**  __bold__
转义符 \\ , \#
内联 `inline code` 代码
键位<kbd>Ctrl</kbd>+<kbd>Esc</kbd>
~~删除线~~

```
## 引用
>引用
>>内嵌的引用
> 无法脱离内嵌
>
> 成功脱离内嵌
## 3种水平分割线
>星号
***
>下划线
___
>连字符-（内容后要加空行，单独用不需要加）
---

```
>引用
>>内嵌的引用
> 无法脱离内嵌
>
> 成功脱离内嵌
## 3种水平分割线
>星号
***
>下划线
___
>连字符-（内容后要加空行，单独用不需要加）
---

```
# 表
## 列表
无序列表
- 加、减、乘号
- Item 2
   + item 3a
   * item 3b
   or

- [ ] 空心方框
- [x] 实心方块

```
- 加、减、乘号
- Item 2
   + item 3a
   * item 3b

- [ ] 空心方框
- [x] 实心方块

```

有序列表
1. Item 1
2. Item 2
    a. item 3a
    b. item 3b
    c. mmm

```
1. Item 1
2. Item 2
    a. item 3a
    b. item 3b
    c. mmm

```

## 表格

Tips：

Typora预览模式下，回车只能进入下一行的单元格；

需要插入行，ctrl+enter；

单元格内分行，shift+enter。

| Left column | Center column | Right column |
| :---------- | :-----------: | -----------: |
| Cell 1      |   Centered    |        $1600 |
|             |               |              |
| Cell 2      |    Cell 3     |          $12 |

```

| Left column | Center column | Right column |
|:------------|:-------------:|-------------:|
| Cell 1      |   Centered    |        $1600 |
| Cell 2      |    Cell 3     |          $12 |


```
简化写法 (gitbook 无法渲染)
Left column | Center column | Right column
-|-|-
Cell 1   |   Centered    |    $1600
Cell 2   |    Cell 3     |     $12

```

Left column | Center column | Right column
-|-|-
Cell 1   |   Centered    |    $1600
Cell 2   |    Cell 3     |     $12


```

[**表格转换工具**](https://tableconvert.com/)

# 公式

[详情请见 LaTeX ](#LaTeX)


- 内联公式
f(x)\=x∗e2piiξxf(x) = x \* e^{2 pi i \\xi x}

```
# 先换行！！！
# $ 没用！！！
f(x)\=x∗e2piiξxf(x) = x \* e^{2 pi i \\xi x}

```
- 独立公式
$$f(x) = sin(x) + \frac{1}{2} x + 1 $$

```
$$f(x) = sin(x) + \frac{1}{2} x + 1 $$

```

- 矩阵
$$\begin{Bmatrix}
   a & b \\
   c & d
\end{Bmatrix}$$


```
$$\begin{Bmatrix}
   a & b \\
   c & d
\end{Bmatrix}$$

```
- 多行公式
$$D(x) = \begin{cases}
\lim\limits_{x \to 0} \frac{a^x}{b+c}, & x<3 \\
\pi, & x=3 \\
\int_a^{3b}x_{ij}+e^2 \mathrm{d}x, & x>3 \\
\end{cases}$$

```
$$\begin{Bmatrix}
   a & b \\
   c & d
\end{Bmatrix}$$

```

# 代码
~~~javascript
console.log("This is a block code")
~~~
~~~css
.button { border: none; }
~~~
    4个空格缩进形成一个代码块
内联代码 `Inline code` 用反勾包围

# 链接
[谷歌](http://google.com)   |[必应][某个地方]   |<http://google.com>

[某个地方]: http://google.com
## 网址
![RUNOOB 图标(可选：无法加载图片时，显示中括号内的文字)](http://static.runoob.com/images/runoob-logo.png "这是 RUNOOB 的图标")
## 插入HTML标签
Markdown 还没有办法指定图片的高度与宽度，如果你需要的话，你可以使用普通的`<img>`标签。
<img src="https://cdn.jsdelivr.net/gh/xxx/notebook-img/67da0a72cfd34229b2438dbbcd3d1125.jpeg" width="50%">

## 粗暴地嵌入网页
<iframe
src="https://cdn.jsdelivr.net/gh/xxx/notebook-img/67da0a72cfd34229b2438dbbcd3d1125.jpeg"
scrolling="no"
border="0"
frameborder="no"
framespacing="0"
allowfullscreen="true"
width=700
height=500>
</iframe>




# LaTeX
<span id="LaTeX"> LaTeX公式</span>

[樱花赞LaTeX公式手册(全网最全)](https://www.cnblogs.com/1024th/p/11623258.html)

[gitbook用KATEX](https://katex.org/docs/supported.html)

[参考](https://www.cnblogs.com/1024th/p/11623258.html#2562307043)


[latex WikiBook PDF文件 700页](https://upload.wikimedia.org/wikipedia/commons/2/2d/LaTeX.pdf)


## 函数
### 指数
$$\rho_l^{\text{ sat}}, \exp_a b = a^b, \exp b = e^b, 10^m$$

```
$$\rho_l^{\text{ sat}}, \exp_a b = a^b, \exp b = e^b, 10^m$$

```
### 对数
$$\log x, \lg y=\log _{10}y, \ln c$$

```
$$\log x, \lg y=\log _{10}y, \ln c$$

```
### 三角函数

$$\sin a,\cos b,\tan c,\cot d,\arcsin a,\arccos b,\arctan c$$


```
$$\sin a,\cos b,\tan c,\cot d,\arcsin a,\arccos b,\arctan c,\\
 \operatorname{sh}k, \operatorname{ch} l, \operatorname{th}m$$

```
### 符号函数，绝对值
$$\left\vert x \right\vert$$

```
$$\left\vert x \right\vert$$

```
### 最大值，最小值
$$\min(x,y), \max(x,y)$$

```
$$\min(x,y), \max(x,y)$$

```

### 极限
$$\lim _{x \to \infty} \frac{1}{n(n+1)}$$

```
$$\min(x,y), \max(x,y)$$

```

### 微分及导数
$$ dt , \mathrm{d}t, \partial t, \nabla\psi ,
dy/dx, \mathrm{d}y/\mathrm{d}x,
 \frac{dy}{dx}, \frac{\mathrm{d}y}{\mathrm{d}x}, \frac{\partial^2}{\partial x_1\partial x_2}y,
\prime, \backprime, f^\prime, f', f'', f^{(3)}, \dot y, \ddot y$$

```
$$ dt , \mathrm{d}t, \partial t, \nabla\psi ,
dy/dx, \mathrm{d}y/\mathrm{d}x,
 \frac{dy}{dx}, \frac{\mathrm{d}y}{\mathrm{d}x}, \frac{\partial^2}{\partial x_1\partial x_2}y,
\prime, \backprime, f^\prime, f', f'', f^{(3)}, \dot y, \ddot y$$

```
### 模运算
$$s_k \equiv 0 \pmod {5}$$

```
$$s_k \equiv 0 \pmod {5}$$

```
### 根号
$$\surd, \sqrt{2}, \sqrt[n]{}, \sqrt[3]{\frac{x^3+y^3}{2}} $$

```
$$\surd, \sqrt{2}, \sqrt[n]{}, \sqrt[3]{\frac{x^3+y^3}{2}} $$

```

