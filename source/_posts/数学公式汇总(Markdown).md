---
title: 数学公式汇总（Markdown）
tags: Markdown
categories: 学习记录
cover: false
date: 2021-03-11 00:00:00
keywords: Markdown
---

# 1.快捷键以及设置

在行内镶嵌公式的话需要设置![屏幕截图 2021-03-11 103010](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210311103029.png)

使用方法：先按$，再按“esc”；

你好$\theta_1$

公式块的快捷键：按两下$,再按“Enter”。
$$
h_\theta(x)=\theta_0+\theta_1x
$$

# 2.常用公式

## 2.1上下标、正负无穷

![v2-9e56df605e51b7aa0cf7a45d0b5bfde1_r](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210311103411.png)

## 2.2**加减乘，分式，根号，省略号**

![v2-417aefe2addf8328b4865d037864ec4e_r](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210311103520.png)

## 2.3三角函数

![v2-2527327da18ba3cd4d9cfa9483bcbe1f_1440w](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210311103612.png)

## 2.4**矢量，累加累乘，极限**

![v2-701158788db26a5936516dc93d34b378_1440w](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210311103705.png)

## 2.5希腊字母

![v2-ec3ad9e52d4b26648d73c64c43bc217e_1440w](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210311103739.png)

## 2.6**关系运算符**

![v2-9088cec7cffbc94c5daef26147278062_1440w](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210311103806.png)

# 3.矩阵

## 3.1简单矩阵

规则：使用`\begin{matrix}…\end{matrix}`生成， 每一行以`\\`结尾表示换行，元素间以`&`间隔。

```text
 $$
\begin{matrix}
 1 & 2 & 3 \\
 4 & 5 & 6 \\
 7 & 8 & 9 
\end{matrix} \tag{1}
$$
```

$$
\begin{matrix}
 1 & 2 & 3 \\
 4 & 5 & 6 \\
 7 & 8 & 9 
\end{matrix} \tag{1}
$$

## 3.2带括号的矩阵

- 大括号

```text
$$
 \left\{
 \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right\} \tag{2}
$$
OR
$$
 \begin{Bmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{Bmatrix} \tag{6}
$$
```

$$
\left\{
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{matrix}
\right\}
$$




- 中括号

```text
$$
 \left[
 \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right] \tag{3}
$$
OR
$$
 \begin{bmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{bmatrix} \tag{6}
$$
```

$$
\left[
 \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right] \tag{3}
$$

- 小括号

```text
$$
 \left(
 \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right) \tag{4}
$$
```

$$
\left(
 \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right) \tag{4}
$$

- 包含希腊字母与省略号

```text
$$
 \left\{
 \begin{matrix}
 1      & 2        & \cdots & 5        \\
 6      & 7        & \cdots & 10       \\
 \vdots & \vdots   & \ddots & \vdots   \\
 \alpha & \alpha+1 & \cdots & \alpha+4 
 \end{matrix}
 \right\}
$$
```

$$
\left\{
 \begin{matrix}
 1      & 2        & \cdots & 5        \\
 6      & 7        & \cdots & 10       \\
 \vdots & \vdots   & \ddots & \vdots   \\
 \alpha & \alpha+1 & \cdots & \alpha+4 
 \end{matrix}
 \right\}
$$

# 4.表格

```text
$$
\begin{array}{|c|c|c|}
	\hline 2&9&4\\
	\hline 7&5&3\\
	\hline 6&1&8\\
	\hline
\end{array}
$$
```

$$
\begin{array}{|c|c|c|}
	\hline 2&9&4\\
	\hline 7&5&3\\
	\hline 6&1&8\\
	\hline
\end{array}
$$

**规则：**

- 开头 `\begin{array}` ，结尾 `\end{array}`
- `{|c|c|c|}`，其中`c` `l` `r` 分别代表居中、左对齐及右对齐。其中的 `l`表示是否有竖直分割线。
- 在下一行输入前插入 `\hline`，以下图真值表为例。

```text
$$
\begin{array}{cc|c}
	       A&B&F\\
	\hline 0&0&0\\
	       0&1&1\\
	       1&0&1\\
	       1&1&1\\
\end{array}
$$
```

$$
\begin{array}{cc|c}
	       A&B&F\\
	\hline 0&0&0\\
	       0&1&1\\
	       1&0&1\\
	       1&1&1\\
\end{array}
$$

# 5.多行等式对其

```text
$$
\begin{aligned}
a &= b + c \\
  &= d + e + f
\end{aligned}
$$
```

$$
\begin{aligned}
a &= b + c \\
  &= d + e + f
\end{aligned}
$$

## 5.1方程组、条件表达式

```text
$$
\begin{cases}
3x + 5y +  z \\
7x - 2y + 4z \\
-6x + 3y + 2z
\end{cases}
$$
```

$$
\begin{cases}
3x + 5y +  z \\
7x - 2y + 4z \\
-6x + 3y + 2z
\end{cases}
$$

```text
$$
f(n) =
\begin{cases} 
n/2,  & \text{if }n\text{ is even} \\
3n+1, & \text{if }n\text{ is odd}
\end{cases}
$$
```

$$
f(n) =
\begin{cases} 
n/2,  & \text{if }n\text{ is even} \\
3n+1, & \text{if }n\text{ is odd}
\end{cases}
$$

# 6**间隔 (大小空格、紧贴)**

```text
$$
a\!b + ab + a\,b + a\;b + a\ b + a\quad b + a\qquad b
$$
```

$$
a\!b + ab + a\,b + a\;b + a\ b + a\quad b + a\qquad b
$$

# 7通过Python生成LaTeX表达式

## 7.1安装**latexify-py**模块（**只能用pip安装，conda找不到**）

```text
pip install latexify-py
//如果安装了anaconda，需要安装到anaconda包下
pip install latexify-py --target=目标路径
```

## 7.2**基本语法**

```python
import math             //引入数学模块(有些运算的函数需要)
import latexify         //引入latexify模块

@latexify.with_latex    //特定语法，表示之后定义的函数可以转化为LaTeX代码
def f(x,y,z):           //包含的参数
    pass               //此处填写可能需要的数学表达式
    return result       //也可以直接体现数学关系

print(f)               //直接print(函数名)
```

## 7.3实例

- 加减乘除
  $$
  \mathrm{f}(x)\triangleq \frac{2x + 6}{5}
  $$
  ![image-20210311164635367](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210311192616.png)

- 分段函数
  $$
  \mathrm{f}(x)\triangleq \left\{ \begin{array}{ll} x, & \mathrm{if} \ x>0 \\ 0, & \mathrm{otherwise} \end{array} \right.
  $$

  ```text
  @latexify.with_latex
  def f(x):
      if (x > 0):
          return x
      else:
          return 0
  
  print(f)
  ```

- 根号、幂

  ```
  @latexify.with_latex
  def f(a,b,c):
      return math.sqrt(b**2-4*a*c)
  
  print(f)
  ```

- 三角函数

  ```text
  @latexify.with_latex
  def f(x,y):
      return math.sin(x+y)
  
  print(f)
  ```

- 绝对值

  ```text
  @latexify.with_latex
  def f(x):
      return abs(x)
  
  print(f)
  ```

- 对数

  ```text
  @latexify.with_latex
  def f(x,y):
      return math.log2(x+y)
  
  print(f)
  ```

# 参考

https://zhuanlan.zhihu.com/p/261750408

