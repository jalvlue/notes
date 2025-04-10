### 行内嵌入公式
在 `$$` 内填写公式

$a^2=b^2+c^2$

$X(e^{j\omega}) = \int_{-\infty}^\infty x(t)e^{-j\omega t}dt$

$90\degree \sim 180\degree$


### 在行间独占一行
在 `$$ $$` 中填入公式
可用 `\tag{n}` 对公式进行编号( n 为编号)
用 `\quad` 或 `\qquad` 实现公式内的空格

$$
a^2=b^2+c^2\tag{1.1}
$$

$$
X(e^{j\omega}) = \int_{-\infty}^\infty x(t)e^{-j\omega t}\quad dt\tag{1.2}
$$

$$
90\degree \sim 180\degree
$$

### 简单运算
拉丁字母、阿拉伯数字和 `+-*/` 可直接输入
- $\cdot$ `\cdot` 乘法的圆点
- $\neq$ `\neq` 不等号
- $\equiv$ `\equiv` 恒等号
- $\bmod$ `\bmod` 取模

### 上标和下标
`_` 表示下标
`^` 表示上标
上标和下标为一个式子时要用大括号括起来

$a_{ij}^{2}+b_{2}^{3}=x^{t}+y'+x''_{12}$


### 根式和分式
- $\sqrt{x}$ `\sqrt{x}` x 的平方根
- $\sqrt[n]{x}$ `\sqrt[n]{x}` x 的 n 次方根
- $\frac{x}{y}$ `\frac{x}{y}` x/y

$$
\sqrt{x}+\sqrt{x^{2}+\sqrt{y}}=\sqrt[3]{k_{i}}-\frac{x}{m}
$$


### 上下标记
- $\overline{x+y}$ `\overline{x+y}` 上划线
- $\underline{a+b}$ `\underline{a+b}` 下划线
- 空格 shiyong `\qquad` 获得


- `\overbrace{xxx}^{y}` 在表达式 xxx 上方给出大括号并且标记为y
- `\underbrace{xxx}_{y}` 在表达式下方给出大括号

$$
\overbrace{1+2+\cdots+n}^{n个} \qquad \underbrace{a+b+\cdots+z}_{26}
$$

### 向量
- `\vec` 表示简单向量
- `\overrightarrow` 表示箭头向右的向量
- `\overleftarrow` 表示箭头向左的向量

$$
\vec{b}+\vec{A} + \overrightarrow{AB} + \overleftarrow{DE}
$$

### 极限、积分、求和、求积
- `lim` 表示极限
- `int` 表示积分
- `sum` 表示求和
- `prod` 表示求积

$$
\lim_{x \to \infty} x_{22}^2 - \int_{1}^{5} xdx + \sum^{20}_{n=1}=\prod^{3}_{j=1}y_i+\lim_{x \to -2}\frac{x-2}{x}
$$

### 重音符号
- `\hat`
- `\bar`
- `\tilde`

$$
\hat{x}+\bar{y}+\tilde{z}
$$


### 矩阵
- `matrix` 普通矩阵
$$
\begin{matrix*}
a&b \\
c&d \\
\end{matrix*}
$$

- `bmatrix` 中括号矩阵
$$
\begin{bmatrix}
a&b \\ c&d
\end{bmatrix}
$$

- `vmatrix` 行列式
$$
\begin{vmatrix*}
a&b\\
c&d\\
\end{vmatrix*}
$$
- `pmatrix` 小括号矩阵
$$
\begin{pmatrix*}
a&b\\
c&d\\
\end{pmatrix*}
$$

$$
\begin{bmatrix*}
1&2&\cdots\\
67&95&\cdots\\
\vdots&\vdots&\ddots\\
\end{bmatrix*}
$$

### 希腊字母

$$
\alpha^{2}+\beta=\theta
$$ 
![[v2-da3e717cf670582fbfbdddee33073524_720w.webp]]


### 公式组合
通过 `cases` 环境实现公式组合，&分隔公式和条件

$$
D(x)=\begin{cases*}
\lim\limits_{x \to 0}\frac{a^x}{b+c},&x<3\\
\pi,&x=3\\
\int^{3b}_{a}x_{ij}+e^2dx,&x>3\\
\end{cases*}
$$


### 拆分公式
通过 `split` 环境实现公式拆分

$$
\begin{split}
\cos2x
&=\cos^{2}x-\sin^{2}x\\
&=2\cos^{2}x-1
\end{split}
$$

