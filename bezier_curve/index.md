# Hermite Curve 以及 Bézier Curve 贝塞尔曲线的简要计算

# Hermite Curve 以及 Bézier Curve 贝塞尔曲线的简要计算
## 尊重原创，请勿转载! 
作者：[图林根の烤肠](https://mahong.me)

详细资料可以查询 wiki 的百科网站，以下内容为个人总结，难免出现错误，欢迎指正最后。

要了解Hermite以及Bézier，首先来简要了解这个三次参数曲线（Parametric cubic curves）：

首先是一个空间三位曲线方程式如下：
{{< raw >}}
\[ 
  Q(u) = U \cdot B \cdot G \\
  Q(u) = (u^3b_{11}+u^2b_{21}+ub_{31}+b_{41})\cdot G_1 +
   (u^3b_{12}+u^2b_{22}+ub_{32}+b_{42})\cdot G_2+\\
   (u^3b_{13}+u^2b_{23}+ub_{33}+b_{43})\cdot G_3 + (u^3b_{14}+u^2b_{24}+ub_{34}+b_{44})\cdot G_4
\]
{{< /raw >}}

这里面的 **U** 是scalar，**B** 是 basic function，而 **G** 就是 vector向量了，请注意这个线性方程可以转换为下图矩阵相乘的形式：
{{< raw >}}
\[ 
  Q(u) = U \cdot B \cdot G = \begin{bmatrix}
                    u^3 & u^2 & u &1
                  \end{bmatrix} \cdot

                  \begin{bmatrix}
                    b_{11} & b_{12} & b_{13} & b_{14} \\
                    b_{21} & b_{22} & b_{23} & b_{24} \\
                    b_{31} & b_{32} & b_{33} & b_{34} \\
                    b_{41} & b_{42} & b_{43} & b_{44} \\
                  \end{bmatrix} \cdot

                  \begin{bmatrix}
                    G_1 \\
                    G_2 \\
                    G_3 \\
                    G_4 \\
                  \end{bmatrix}
\]
{{< /raw >}}

在了解了上面这个矩阵就可以开始正文了：

首先是 Hermite 曲线，它是由一些参数决定：
1. 2个首位端点 P1，P4 及其各自的斜率参数 R1 与 R4
2. Hermite 几何矢量（geometry vector）$G_{H}$
3. Hermite 矩阵为:

{{< raw >}}
\[ 
  \begin{bmatrix}
    2 & -2 & 1 & 1 \\
    -3 & 3 & -2 & -1 \\
    0 & 0 & 1 & 0 \\
    1 & 0 & 1 & 0 \\
  \end{bmatrix}
\]
{{< /raw >}}

Hermite 曲线最后的表述方式如前面介绍的三次参数曲线为：
$Q(u) = U \cdot M_{h} \cdot G_{H}$。这里不仅有人要问，这 Hermite 矩阵为什么是这个样子，其中的推导过程在这偷个懒，直接拽张图片过来
  
{{< figure src="b4.png" title="" >}}
上图中的 `Q(0)` 与 `Q(1)` 分别为 `u=0` 与 `u=1` 用特殊法。带入上面的三次曲线方程的结果，然后对其方程式求导，带入`u=0`与`u=1`，分别求出 `Q(0)` 与 `Q(1)` 的斜率。因为 Q(u) = \[P1:P4:R1:P4\](这是一个4行一列的矩阵)，然后会得出一个等式:

{{< raw >}}
\[ 
  \begin{bmatrix}
    P_1 \\
    P_4 \\
    R_1 \\
    R_4 \\
  \end{bmatrix} = \begin{bmatrix}
                    0 & 0 & 0 & 1 \\
                    1 & 1 & 1 & 1 \\
                    0 & 0 & 1 & 0 \\
                    3 & 2 & 1 & 0 \\
                  \end{bmatrix} ∙ M_{H} ∙ G_{H}
\]
{{< /raw >}}
又因为 $G_{H}$ 也等于[P1;P4;R1;P4],
{{< raw >}}
\[ 
  \begin{bmatrix}
    P_1 \\
    P_4 \\
    R_1 \\
    R_4 \\
  \end{bmatrix} = G_{H}
\]
{{< /raw >}}
所以方程两边可以消去[P1;P4;R1;P4]得到等式。
{{< raw >}}
\[ 
  \begin{bmatrix}
    0 & 0 & 0 & 1 \\
    1 & 1 & 1 & 1 \\
    0 & 0 & 1 & 0 \\
    3 & 2 & 1 & 0 \\
  \end{bmatrix} ∙ M_{H} = 1
\]
{{< /raw >}}

再求逆矩阵便可得到 Hermite 矩阵:
{{< raw >}}
\[ 
  M_{H} = 
  \begin{bmatrix}
    0 & 0 & 0 & 1 \\
    1 & 1 & 1 & 1 \\
    0 & 0 & 1 & 0 \\
    3 & 2 & 1 & 0 \\
  \end{bmatrix} ^{-1} = 

\begin{bmatrix}
    2 & -2 & 1 & 1 \\
    -3 & 3 & -2 & -1 \\
    0 & 0 & 1 & 0 \\
    3 & 2 & 1 & 0 \\
  \end{bmatrix} 
\]
{{< /raw >}}
<!-- {{< figure src="b8.png" title="" >}} -->

按照同样的方法，我们可以接下来获得贝塞尔曲线 Bézier Curve。下面这张图清晰直观的了解到 Bézier Curve 是一个三位空间曲线通过转换投影到2D平面的曲线。

{{< figure src="b9.png" title="图片来源：Watt,A: 3D-Computergrafik" >}}

同上面介绍的 Hermite曲线计算一样，Bézier curve 同理表述为

{{< raw >}}
\[ 
Q(u) = \sum_{i=0}^3 B_{i}(u) \cdot P_{i} = \\
\\
= (-u^3+3u²-3u+1)\cdot P_{0} + (3u^3-6u²+3u)\cdot P_{1} + \\
+ (-3u^3 + 3u^2) \cdot P_{2} + u^3 \cdot P_{3} 
\]
{{< /raw >}}

或者以矩阵的形式表示：
{{< raw >}}
\[ 
  Q(u) = [u^3, u^2, u, 1] \cdot
  \begin{bmatrix}
    -1 & 3 & -3 & 1 \\
    3 & -6 & 3 & 0 \\
    -3 & 3 & 0 & 0 \\
    1 & 0 & 0 & 0 \\
  \end{bmatrix} \cdot

  \begin{bmatrix}
    P_{0} \\
    P_{1} \\
    P_{2} \\
    P_{3} \\
  \end{bmatrix}
\]
{{< /raw >}}

<!-- {{< figure src="b10.jpg" title="" >}} -->

其中 $P_{i}$ 为行向量 (row-vector)，里面的推导过程就不介绍了，详情可以查询 [维基百科](http://en.wikipedia.org/wiki/Bernstein_polynomial) 的网页介绍。
另外推荐一篇介绍 [Hermite 曲线的 OpenGL 博客](http://blog.163.com/liang_ce_521@126/blog/static/7092021520104141443158/)，非常值得一看。

# 参考资料：
3D-Repräsentation，Vorlesung Computeranimation WS 2009 © Nowak, Weigel, Kirchner, Schuller
