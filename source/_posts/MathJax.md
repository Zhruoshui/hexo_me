---
title: MathJax
tags:
  - 科研工具
  - 数学表达式
categories:
  - Math
cover: 'https://image.aruoshui.fun/i/2024/12/31/vskdu5-0.webp'
description: 描述数学和科学公式的开源MathJax
swiper_index: 1 #置顶轮播图顺序，非负整数，数字越大越靠前
abbrlink: 31281
date: 2024-02-15 07:41:55
---

{% note info flat %}参考：[MathJax新手使用教程](https://blog.csdn.net/weixin_46119529/article/details/133130295)
参考：[Mathjax公式教程](https://bobokele.blog.csdn.net/article/details/79577072?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-3-79577072-blog-48900483.235%5Ev43%5Epc_blog_bottom_relevance_base7&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-3-79577072-blog-48900483.235%5Ev43%5Epc_blog_bottom_relevance_base7&utm_relevant_index=6)
{% endnote %}

# 一、行内公式及块内公式
其实个人认为跟代码块和代码行含义一样
## 行公式
{% tabs 分栏 %}

<!-- tab 示例源码 -->
```md
\```md
$\lim_{x \to \infty}f(x)$
\```
```

<!-- endtab -->

<!-- tab 渲染演示 -->
$\lim_{x \to \infty}f(x)$
<!-- endtab -->

{% endtabs %}

行公式，是在代码块的基础上前面加上`$`，后面加上`$`组成的。

## 块公式
{% tabs 分栏 %}

<!-- tab 示例源码 -->
```md
\```md
$$
\lim_{x \to \infty}f(x)
$$
\```
```

<!-- endtab -->

<!-- tab 渲染演示 -->
$$
\lim_{x \to \infty}f(x)
$$
<!-- endtab -->

{% endtabs %}
块公式则是 输入`$$`和`$$`在公式前后。
 


# 二、希腊字母
| 名称 | 大写 | Tex	| 小写 | Tex |
| ----- | --- | ----- | --- | -----|
| alpha	| A   |	A	|α	|\alpha|
|beta	|B|	B|	β	|\beta|
|gamma	|Γ	|\Gamma|	γ	|\gamma|
|delta  |	Δ	|\Delta|	δ	|\delta|
|epsilon	|E	|E	|ϵ	|\epsilon|
|zeta	|Z	|Z	|ζ	|\zeta|
|eta	|H	|H	|η	|\eta|
|theta	|Θ	|\Theta	|θ	|\theta|
|iota	|I	|I	|ι	|\iota|
|kappa	|K	|K	|κ	|\kappa|
|lambda	|Λ	|\Lambda	|λ	|\lambda|
|mu	|M	|M	|μ	|\mu|
|nu	|N	|N	|ν	|\nu|
|xi	|Ξ	|\Xi	|ξ	|\xi|
|omicron	|O	|O	|ο	|\omicron|
|pi	|Π	|\Pi	|π	|\pi|
|rho	|P	|P	|ρ	|\rho|
|sigma	|Σ	|\Sigma	|σ	|\sigma|
|tau	|T	|T	|τ	|\tau|
|upsilon	|Υ	|\Upsilon	|υ	|\upsilon|
|phi	|Φ	|\Phi	|ϕ	|\phi|
|chi	|X	|X	|χ	|\chi|
|psi	|Ψ	|\Psi	|ψ	|\psi|
|omega	|Ω	|\Omega	|ω	|\omega|

# 三、上下标
上标使用`^`，下标使用`_`，例如`C_5^3`,渲染结果$C_5^3$




# 四、括号
## 小括号和方括号
数学怎么表示这就怎么表示
{% tabs 分栏 %}

<!-- tab 示例源码 -->
```md
\```md
$(a+b)[a+b]$
\```
```

<!-- endtab -->

<!-- tab 渲染演示 -->
$(a+b)[a+b]$
<!-- endtab -->

{% endtabs %}

## 大括号
### 正常表示
为了与分组做出区别，使用`\{`和`\}`来表示大括号，其实跟转义字符写法一样。
除此之外也可以使用`\lbrace`和`\rbrace`来表示
{% tabs 分栏 %}

<!-- tab 示例源码 -->
```md
\```md
$ \{ a*b \} + \lbrace a^b \rbrace $
\```
```

<!-- endtab -->

<!-- tab 渲染演示 -->
$ \{ a*b \} + \lbrace a^b \rbrace $
<!-- endtab -->

{% endtabs %}
### 用于分组
默认情况下，上下标符号仅仅对下一个组起作用。一个组即单个字符或者使用{..}包裹起来的内容。
例如如果我要表示`2^10`，但是渲染结果却是$ 2^10 $，而`2^{10}`才是真正的$ 2^ {10} $ 
{% note warning flat %}这里的大括号就不用加转义了{% endnote %}

同时，大括号也能消除二义性，例如`2^2^2`会是一个错误，必须使用大括号来界定`^`，如`{2^2}^2`:${2^2}^2$,当然这么写`2^{2^2}`也没有任何问题

## 尖括号
使用`\langle`和`\rangle`表示左尖括号和右尖括号，如`\langle a \rangle`:$\langle a \rangle$

## 取整符号
### 上取整
使用`\lceil`和`\rceil`表示。 如`\lceil x \rceil`：$\lceil x \rceil$
### 下取整
使用`\lfloor`和`\rfloor`表示。 如`\lfloor x \rfloor`：$\lfloor x \rfloor$



 
# 五、积分和求和
## 积分
`\int`用来表示积分符号，同样地，其上下标表示积分的上下限。如，
{% tabs 分栏 %}

<!-- tab 示例源码 -->
```md
\```md
$ \int_1^\infty $
\```
```

<!-- endtab -->

<!-- tab 渲染演示 -->
$ \int_1^\infty $
<!-- endtab -->

{% endtabs %}
{% note warning flat %}这里的$\infty$符号写做`\infty`{% endnote %}

另外多重积分其实就是多加几个i
`\iint`：$\iint$
`\iiint`：$\iiint$

## 其他符号
`\sum`用来表示求和符号，其下标表示求和下限，上标表示上限。如：`\sum_1^10`：$\sum_1^{10}$
`\prod`：$\prod$
`\bigcup`：$\bigcup$
`\bigcap`：$\bigcap$
 

# 六、分式与根式
## 分式
第一种，使用`\frac ab`，`\frac`作用于其后的两个组`a`，`b`，结果为$\frac ab$如果你的分子或分母不是单个字符，请使用{..}来分组，如$\frac{a-b}{a+b}$

第二种，使用`\over`来分割，如`{a-b \over a+b}`:${a-b \over a+b}$
{% note warning flat %}要分割的部分请采用`{}`来囊括{% endnote %}

## 多重分式
如：
{% tabs 分栏 %}

<!-- tab 示例源码 -->
```md
\```md
$ x=a_0 + \frac {1^2}{a_1 + \frac {2^2}{a_2 + \frac {3^2}{a_3 + \frac {4^2}{a_4 + ...}}}}
 $
\```
```

<!-- endtab -->

<!-- tab 渲染演示 -->
$ x=a_0 + \frac {1^2}{a_1 + \frac {2^2}{a_2 + \frac {3^2}{a_3 + \frac {4^2}{a_4 + ...}}}}
 $
<!-- endtab -->

{% endtabs %}

## 根式
根式使用`\sqrt`来表示。如，`\sqrt[2]{4}`: $\sqrt[2]{4}$

 

# 七、表达式与方程组
## 多行表达式
可使用`\begin{cases}…\end{cases}`。其中，使用`\\`来分类，使用`&`指示需要对齐的位置，`\空格`表示空格 。如：


{% tabs 分栏 %}

<!-- tab 示例源码 -->
```md
\```md
$$
f(n)
\begin{cases}
\cfrac n2, &if\ n\ is\ even\\
3n + 1, &if\  n\ is\ odd
\end{cases}

$$ 
\```
```

<!-- endtab -->

<!-- tab 渲染演示 -->
$$
f(n)=
\begin{cases}
\cfrac n2, &if\ n\ is\ even\\
3n + 1, &if\  n\ is\ odd
\end{cases}
$$ 

<!-- endtab -->

{% endtabs %}

## 方程组

{% tabs 分栏 %}
<!-- tab 示例源码 -->
```md
\```md
$$
\left \{ 
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\ 
a_2x+b_2y+c_2z=d_2 \\ 
a_3x+b_3y+c_3z=d_3
\end{array}
\right .

$$ 
\```
```
<!-- endtab -->

<!-- tab 渲染演示 -->
$$
\left \{ 
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\ 
a_2x+b_2y+c_2z=d_2 \\ 
a_3x+b_3y+c_3z=d_3
\end{array}
\right .

$$ 
<!-- endtab -->
{% endtabs %}

 

# 八、表格
使用`\begin{array}{列样式}…\end{array}`这样的形式来创建表格，列样式可以是`clr`表示居中，左，右对齐，还可以使用`|`表示一条竖线。表格中各行使用`\\`分隔，各列使用`&`分隔。使用`\hline`在本行前加入一条直线。 
{% tabs 分栏 %}
<!-- tab 示例源码 -->
```md
\```md
$$
\begin{array}{c|lcr}
n & \text{Left} & \text{Center} & \text{Right} \\
\hline
1 & 0.24 & 1 & 125 \\
2 & -1 & 189 & -8 \\
3 & -20 & 2000 & 1+10i \\
\end{array}
$$ 
\```
```
<!-- endtab -->

<!-- tab 渲染演示 -->
$$
\begin{array}{c|lcr}
n & \text{Left} & \text{Center} & \text{Right} \\
\hline
1 & 0.24 & 1 & 125 \\
2 & -1 & 189 & -8 \\
3 & -20 & 2000 & 1+10i \\
\end{array}
$$ 
<!-- endtab -->
{% endtabs %}
 

# 九、矩阵
## 基本用法
{% tabs 分栏 %}
<!-- tab 示例源码 -->
```md
\```md
$$
\begin{matrix}
1 & x & x^2 \\
1 & y & y^2 \\
1 & z & z^2 \\
\end{matrix}
$$ 
\```
```
<!-- endtab -->

<!-- tab 渲染演示 -->
$$
\begin{matrix}
1 & x & x^2 \\
1 & y & y^2 \\
1 & z & z^2 \\
\end{matrix}

$$ 
<!-- endtab -->
{% endtabs %}


## 矩阵表示
可以使用特殊的matrix。即替换`\begin{matrix}…\end{matrix}`中的matrix为`pmatrix`，`bmatrix`，`Bmatrix`，`vmatrix`, `Vmatrix`。

{% tabs 分栏 %}
<!-- tab 示例源码 -->
```md
(1) pmatrix:
\```md
$$
\begin{pmatrix}
1 & 2  \\
3 & 4  \\
\end{pmatrix}
$$ 
\```

(2) bmatrix:
\```md
$$
\begin{bmatrix}
1 & 2  \\
3 & 4  \\
\end{bmatrix}
$$ 
\```

(3) Bmatrix:
\```md
$$
\begin{Bmatrix}
1 & 2  \\
3 & 4  \\
\end{Bmatrix}
$$ 
\```

(4) vmatrix:
\```md
$$
\begin{vmatrix}
1 & 2  \\
3 & 4  \\
\end{vmatrix}
$$ 
\```

(5) Vmatrix:
\```md
$$
\begin{Vmatrix}
1 & 2  \\
3 & 4  \\
\end{Vmatrix}
$$ 
\```
```
<!-- endtab -->

<!-- tab 渲染演示 -->

(1) pmatrix:

$$
\begin{pmatrix}
1 & 2  \\
3 & 4  \\
\end{pmatrix}
$$ 


(2) bmatrix:

$$
\begin{bmatrix}
1 & 2  \\
3 & 4  \\
\end{bmatrix}
$$ 


(3) Bmatrix:

$$
\begin{Bmatrix}
1 & 2  \\
3 & 4  \\
\end{Bmatrix}
$$ 


(4) vmatrix:

$$
\begin{vmatrix}
1 & 2  \\
3 & 4  \\
\end{vmatrix}
$$ 


(5) Vmatrix:

$$
\begin{Vmatrix}
1 & 2  \\
3 & 4  \\
\end{Vmatrix}
$$ 


<!-- endtab -->
{% endtabs %}

## 省略元素
可以使用`\cdots`：$\cdots$，`\ddots`： $\ddots$，`\vdots`：$\vdots$来省略矩阵中的元素

{% tabs 分栏 %}
<!-- tab 示例源码 -->
```md
\```md
$$
\begin{pmatrix}
1&a_1&a_1^2&\cdots&a_1^n\\
1&a_2&a_2^2&\cdots&a_2^n\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
1&a_m&a_m^2&\cdots&a_m^n\\
\end{pmatrix}
$$ 
\```
```
<!-- endtab -->

<!-- tab 渲染演示 -->
$$
\begin{pmatrix}
1&a_1&a_1^2&\cdots&a_1^n\\
1&a_2&a_2^2&\cdots&a_2^n\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
1&a_m&a_m^2&\cdots&a_m^n\\
\end{pmatrix}

$$ 
<!-- endtab -->
{% endtabs %}


## 增广矩阵
使用前面的表格中使用到的`\begin{array} ... \end{array}`来实现
{% tabs 分栏 %}
<!-- tab 示例源码 -->
```md
\```md
$$
\left[
\begin{array}{cc|c}
1&2&3\\\\
4&5&6
\end{array}
\right]
$$ 
\```
```
<!-- endtab -->

<!-- tab 渲染演示 -->
$$
\left[
\begin{array}{cc|c}
1&2&3\\
4&5&6
\end{array}
\right]
$$ 
<!-- endtab -->
{% endtabs %}

# 十、常用符号
{% note info flat %}
更多符号请参考：[Mathjax公式教程](https://bobokele.blog.csdn.net/article/details/79577072?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-3-79577072-blog-48900483.235%5Ev43%5Epc_blog_bottom_relevance_base7&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-3-79577072-blog-48900483.235%5Ev43%5Epc_blog_bottom_relevance_base7&utm_relevant_index=6)
{% endnote %}

## 三角函数
`\sinx`：$\sin$
`\arcsin`：$\arcsin$
## 比较运算符
`\lt`：$\lt$
`\gt`：$\gt$
`\le`：$\le$
`\ge`：$\ge$
`\ne`：$\ne$

## 集合关系
并`\cup`：$\cup$
交`\cap`：$\cap$
差`\setminus`：$\setminus$
子集`\subset`：$\subset$
真子集`\subsetneqq`：$\subsetneqq$
属于`\in`：$\in$
不属于`\notin`：$\notin$
空集`\varnothing`：$\varnothing$

## 逻辑运算
`\land`：$\land$
`\lor`：$\lor$
`\lnot`：$\lnot$
`\forall`：$\forall$
`\exists`：$\exists$

## 操作符
`\oplus`：$\oplus$
`\circ`：$\circ$
`\bullet`：$\bullet$
`\approx`：$\approx$
`\sim`：$\sim$
`\cong`：$\cong$
`\pmod`：如`a = b \pmod n`： $a = b \pmod n$

![](https://cdnimg103.lizhi.fm/audio_cover/2017/08/17/2619367354329486855_580x580.jpg)



# 十一、公式的标记和引用
公式经常复用，Mathjax中有一个类似于函数的功能

使用`\tag{yourtag}`来标记公式，如果想在之后引用该公式，则还需要加上`\label{yourlabel}`在`\tag`之后


