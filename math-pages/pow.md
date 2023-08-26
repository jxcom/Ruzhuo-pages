[Home](https:\\wecache.com)

# $\pi$ 的计算
[toc]
## 指数和对数无穷级数展开
### 指数展开

$$
\begin{align*}
从一个基本等式开始:&\qquad a^0 = 1\\
那么\epsilon\rightarrow o,\epsilon'\rightarrow o时:&\qquad a^\epsilon = 1 + \epsilon'\\
设\epsilon和\epsilon'有关系:&\qquad\epsilon'=K\epsilon,代入上式得:a^\epsilon = 1 + K\epsilon\\
于是:&\qquad a^{n\epsilon} = (1 + K\epsilon)^n\\
对于n\rightarrow\infin,\epsilon\rightarrow o,说明n\epsilon是一个任意值，设为x，即:&\qquad n\epsilon = x\\
也就是:&\qquad a^x = (1 + K\epsilon)^n\\
对(1 + K\epsilon)^n展开为多项式:&\qquad (1 + K\epsilon)^n = 1 + \binom{n}{1}K\epsilon + \binom{n}{2}(K\epsilon)^2 + \binom{n}{3}(K\epsilon)^3 + \binom{n}{4}(K\epsilon)^4 + \ldots\\
由前面设n\epsilon = x,得\epsilon=\frac{x}{n}代入上式:&\qquad (1 + K\epsilon)^n = 1 + \binom{n}{1}K\frac{x}{n} + \binom{n}{2}(K\frac{x}{n})^2 + \binom{n}{3}(K\frac{x}{n})^3 + \binom{n}{4}(K\frac{x}{n})^4 + \ldots\\
当n\rightarrow\infin时:&\qquad \binom{n}{1}\frac{1}{n}=\frac{n}{1}\frac{1}{n}=1,\binom{n}{2}\frac{1}{n^2}=\frac{n(n-1)}{1*2}\frac{1}{n^2}=\frac{n-1}{1*2}\frac{1}{n}=\frac{1-\frac{1}{n}}{1*2}=\frac{1}{2!}\\
&\qquad\binom{n}{3}\frac{1}{n^3}=\frac{n(n-1)(n-2)}{1*2*3}\frac{1}{n^3}=\frac{(n-1)(n-2)}{1*2*3}\frac{1}{n^2}=\frac{(1-\frac{1}{n})(1-\frac{2}{n})}{1*2*3}=\frac{1}{3!}\\
因此，第n项（从0开始）系数为\frac{1}{n!}, 则原式变为:&\qquad a^x = (1 + K\epsilon)^n = 1 + Kx + \frac{1}{2!}Kx^2 + \frac{1}{3!}Kx^3 + \frac{1}{4!}Kx^4 + \ldots\\
即:&\qquad a^x = 1 + Kx + \frac{1}{2!}Kx^2 + \frac{1}{3!}Kx^3 + \frac{1}{4!}Kx^4 + \ldots\\
若设x=1,K=1,则:&\qquad a = 1 + 1 + \frac{1}{2!} + \frac{1}{3!} + \frac{1}{4!} + \ldots = e = 2.718\ldots\\
\end{align*}
$$

### 对数展开

$$
\begin{align*}
对于\ln(1+x),按之前的思想,有:&\qquad \ln(1+x)=\ln(1+\epsilon)^n, 其中,\epsilon\rightarrow o, n\rightarrow\infin\\
对比上面的指数展开,(1+\epsilon)^n相当于 K = 1的情形,此时a=e,即:&\qquad \ln(1+x)=\ln(1+\epsilon)^n=\ln e^{n\epsilon}=n\epsilon\\
即:&\qquad \ln(1+x)=n\epsilon&\qquad(1)\\
而由\ln(1+x)=\ln(1+\epsilon)^n得:&\qquad 1 + x = (1+\epsilon)^n,解:\epsilon=(1+x)^{\frac{1}{n}}-1\,代入(1)得:\\
 &\qquad \ln(1+x)=n[(1+x)^{\frac{1}{n}}-1]=n(1+x)^{\frac{1}{n}}-n&\qquad(2)\\
其中\,(1+x)^{\frac{1}{n}}&=1+\binom{\frac{1}{n}}{1}x+\binom{\frac{1}{n}}{2}x^2+\binom{\frac{1}{n}}{3}x^3+\ldots\\
&=1+\frac{\frac{1}{n}}{1}x+\frac{\frac{1}{n}(\frac{1}{n}-1)}{1*2}x^2+\frac{\frac{1}{n}(\frac{1}{n}-1)(\frac{1}{n}-2)}{1*2*3}x^3+\ldots\\
则\,n(1+x)^{\frac{1}{n}}&=n+x+\frac{(\frac{1}{n}-1)}{1*2}x^2+\frac{(\frac{1}{n}-1)(\frac{1}{n}-2)}{1*2*3}x^3+\ldots\\
则\,n(1+x)^{\frac{1}{n}}-n&=x+\frac{(\frac{1}{n}-1)}{1*2}x^2+\frac{(\frac{1}{n}-1)(\frac{1}{n}-2)}{1*2*3}x^3+\ldots&\qquad(3)\\
n\rightarrow\infin时,(3)为:&\qquad x-\frac{1}{2}x^2+\frac{1}{3}x^3-\frac{1}{4}x^4+\ldots\\
即(2)可表示为:&\qquad \ln(1+x)=x-\frac{1}{2}x^2+\frac{1}{3}x^3-\frac{1}{4}x^4+\ldots\\
用-x代替得:&\qquad \ln(1-x)=-x-\frac{1}{2}x^2-\frac{1}{3}x^3-\frac{1}{4}x^4-\ldots\\
得到进一步结论:&\qquad \ln\frac{1+x}{1-x}=\ln(1+x)-\ln(1-x)=2(x+\frac{1}{3}x^3+\frac{1}{5}x^5+\frac{1}{7}x^7+\ldots)&\qquad(4))

\end{align*}
$$

## 计算 $\pi$

我们的目的是计算出 $\pi$  ，因此需要对 x 进行无穷级数展开，已知欧拉公式：
$$
\begin{align*}
e^{ix} = \cos x + i\sin x
\end{align*}
$$
使用这个公式解出 x，其推算如下：
$$
\begin{align*}
首先,根据欧拉公式有:&\qquad e^{ix} = \cos x + i\sin x\,和\,e^{-ix} = \cos x - i\sin x\\
两式先左右取\ln 再相减得:&\qquad \ln e^{ix} = ln(\cos x + i\sin x)\qquad\ln e^{-ix} = ln(\cos x - i\sin x)\\
&\qquad ix =  ln(\cos x + i\sin x)\qquad -ix = ln(\cos x - i\sin x)\\
&\qquad 2ix = ln(\cos x + i\sin x) - ln(\cos x - i\sin x)\\
解x:&\qquad x = \frac{1}{2i}(\ln\frac{\cos x + i\sin x}{\cos x - i\sin x})\\
&\qquad = \frac{1}{2i}(\ln\frac{1 + i\tan x}{1 - i\tan x}) &\qquad(5)\\
由前述(4)结论,所以(5)式为: &\qquad(5)= \frac{1}{2i}*2(i\tan x - \frac{1}{3}i\tan^3x + \frac{1}{5}i\tan^5x-\frac{1}{7}i\tan^7x+\ldots)\\
&\qquad = \tan x - \frac{1}{3}\tan^3x + \frac{1}{5}\tan^5x-\frac{1}{7}\tan^7x+\ldots\\
即:&\qquad x = \tan x - \frac{1}{3}\tan^3x + \frac{1}{5}\tan^5x-\frac{1}{7}\tan^7x+\ldots\\
转为反三角函数表示:&\qquad \arctan x = x - \frac{1}{3}x^3 + \frac{1}{5}x^5-\frac{1}{7}x^7+\ldots\\
已知 \tan\frac{\pi}{6}=\frac{\sqrt{3}}{3},即:&\qquad \frac{\pi}{6} = \frac{\sqrt{3}}{3} - \frac{1}{3}(\frac{\sqrt{3}}{3})^3 + \frac{1}{5}(\frac{\sqrt{3}}{3})^5-\frac{1}{7}(\frac{\sqrt{3}}{3})^7+\ldots\\
也就是:&\qquad \pi= 6(\frac{\sqrt{3}}{3} - \frac{1}{3}(\frac{\sqrt{3}}{3})^3 + \frac{1}{5}(\frac{\sqrt{3}}{3})^5-\frac{1}{7}(\frac{\sqrt{3}}{3})^7+\ldots)\\
&\qquad = 6(\frac{\sqrt{3}}{3} - \frac{1}{3}\frac{\sqrt{3}}{3^2} + \frac{1}{5}\frac{\sqrt{3}}{3^3}-\frac{1}{7}\frac{\sqrt{3}}{3^4}+\ldots)\\
&\qquad = 6\sqrt{3}(\frac{1}{3} - \frac{1}{3*{3^2}} + \frac{1}{5*{3^3}}-\frac{1}{7*{3^4}}+\ldots)
\end{align*}
$$

早期计算 $\pi$ 就是利用这个公式计算的，欧拉真是天才！
