# 余弦定理证明

[toc]

## 余弦定理

对于任意三角形，假设三边为a，b 和 c，c 边对应的角为 $\gamma$ ，则有如下关系：
$$
\begin{align*}
c^2 = a^2 + b^2 - 2abcos\gamma
\end{align*}
$$

## 前置推导

### 三个初等函数无穷级数展开

#### sinx 的无穷级数展开

$$
\begin{align*}
sinx = x - \frac{1}{3!}x^3 + \frac{1}{5!}x^5 - \frac{1}{7!}x^7 + \ldots
\end{align*}
$$

推导如下：
$$
\begin{align*}
设展开式为:\qquad& sinx = a_0 + a_1x+a_2x^2+a_3x^3+\ldots&\qquad(1)\\
设x=0，可求得a_0:\qquad& sinx|_{x=0} = a_0=0\\
对(1)求导，得:\qquad& (sinx)' = cosx = a_1 + 2a_2x + 3a_3x^2 + \ldots&\qquad(2)\\
同样设x=0，则:\qquad& a_1 = cosx|_{x=0} = 1\\
继续对(2)求导得:\qquad& (cosx)'=-sinx=2a_2 + 3 *2 * a_3x + \ldots&\qquad(3)\\
设(3)中x=0，得:\qquad& -sinx|_{x=0} = 2a_2 = 0, a_2 = 0\\
继续对(3)求导得:\qquad& (-sinx)'=-cosx = 3 * 2 * 1 * a_3 + 4 * 3 * 2 * a_4 * x + 5 * 4 * 3 * x^2 + \ldots&\qquad(4)\\
设(4)中x=0，得:\qquad& -cosx|_{x=0} = 3!a_3=-1, a_3 = -\frac{1}{3!}\\
\qquad& 如此循环，继续对(4)求导求得a_4=0,a_5=-\frac{1}{5!}\ldots\\
于是:\qquad& sinx = x - \frac{1}{3!}x^3 + \frac{1}{5!}x^5 - \frac{1}{7!}x^7 + \ldots\\
\end{align*}
$$

#### cosx 的无穷级数展开

同sinx无穷级数展开，cosx的无求级数展开为：
$$
\begin{align*}
cosx = 1 -\frac{1}{2!}x^2 + \frac{1}{4!}x^4 - \frac{1}{6!}x^6 + \ldots\\
\end{align*}
$$


#### $e^x$ 的无穷级数展开

同sinx无穷级数展开，$e^x$ 展开为：

$$
\begin{align*}
e^x=1 + x + \frac{1}{2!}x^2 + \frac{1}{3!}x^3 + \frac{1}{4!}x^4 + \frac{1}{5!}x^5 + \frac{1}{6!}x^6 +\frac{1}{7!}x^7 + \ldots
\end{align*}
$$


### 欧拉公式

$$
\begin{align*}
由:\qquad&e^x=1 + x + \frac{1}{2!}x^2 + \frac{1}{3!}x^3 + \frac{1}{4!}x^4 + \frac{1}{5!}x^5 + \frac{1}{6!}x^6 +\frac{1}{7!}x^7 + \ldots\\
用ix替换x,得:\\
\qquad&e^{ix} = 1 + ix - \frac{1}{2!}x^2 - \frac{1}{3!}ix^3 + \frac{1}{4!}x^4 - \frac{1}{5!}ix^5 - \frac{1}{6!}x^6 -\frac{1}{7!}ix^7 + \ldots\\
即:\\
\qquad&e^{ix} = (1 -\frac{1}{2!}x^2 + \frac{1}{4!}x^4 - \frac{1}{6!}x^6 + \ldots) + i( x - \frac{1}{3!}x^3 + \frac{1}{5!}x^5 - \frac{1}{7!}x^7 + \ldots)\\
无穷级数展开式替换回sinx和cosx得:\qquad& e^{ix} = cosx + isinx
\end{align*}
$$



## 证明余弦定理

假设三角形为：

![](https://wecache.com/algorithm-media/trippleforlawcosines.jpg)
$$
\begin{align*}
如图，作AD\perp BC 于D，\angle BAD=\alpha,\angle DAC = \beta\\
且由题意设:\qquad&\gamma = \alpha + \beta\\
则:\qquad&c = AB = BD + AD = asin\alpha + bsin\beta \\
两边平方得:\qquad& c^2 = a^2sin^2\alpha + 2absin\alpha sin\beta + b^2sin^2\beta\\
&=a^2(1 - cos^2\alpha) + 2absin\alpha sin\beta + b^2(1-cos^2\beta)\\
&=a^2 + b^2 - (a^2cos^2\alpha + b^2cos^2\beta) + 2absin\alpha sin\beta\qquad(5)\\
而\triangle CBD和\triangle CDA中:\qquad& CD = acos\alpha = b cos\beta\\
于是有:\qquad& a^2cos^2\alpha + b^2cos^2\beta = 2CD^2\qquad(6)\\
又有前面的欧拉公式推理知:\qquad& sin\alpha sin\beta = cos\alpha cos\beta - cos(\alpha + \beta)\\
两边乘以2ab且\gamma = \alpha + \beta:\qquad& 2absin\alpha sin\beta = 2abcos\alpha cos\beta - 2abcos(\alpha + \beta)=2CD^2-2abcos\gamma\qquad(7)\\
(6)(7) 代入(5)得:\qquad& c^2= a^2 + b^2 - 2CD^2 + 2CD^2 - 2abcos\gamma =a^2 + b^2 - 2abcos\gamma\\
即:\qquad& c^2 = a^2 + b^2 - 2abcos\gamma
\end{align*}
$$

