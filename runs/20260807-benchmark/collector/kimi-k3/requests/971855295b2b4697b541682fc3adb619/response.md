记
\[
M=3^7=2187,\qquad P=3^6=729.
\]

由于
\[
(x+P)^3\equiv x^3\pmod M,
\]
所以 \(a^3\bmod M\) 只取决于 \(a\bmod P\)。而 \(1,2,\ldots,P\) 恰好构成模 \(P\) 的一个完全剩余系，其中 \(729\) 对应剩余类 \(0\)。

### 1. 使用单位根计数

令
\[
\omega=e^{2\pi i/M}.
\]
对任意整数 \(s\)，有
\[
\frac1M\sum_{k=0}^{M-1}\omega^{ks}
=
\begin{cases}
1,&M\mid s,\\
0,&M\nmid s.
\end{cases}
\]
定义
\[
S(k)=\sum_{x=0}^{P-1}\omega^{kx^3}.
\]
于是
\[
N=\frac1M\sum_{k=0}^{M-1}S(k)^3.
\]

显然
\[
S(0)=P=3^6.
\]

下面处理 \(k\ne0\)。写成
\[
k=3^t u,\qquad 0\le t\le6,\quad 3\nmid u.
\]

当 \(t\le5\) 时，\(x^3\bmod 3^{7-t}\) 的周期为 \(3^{6-t}\)，故
\[
S(k)=3^tT_{6-t}(u),
\]
其中
\[
T_n(u)=\sum_{x=0}^{3^n-1}e^{2\pi iux^3/3^{n+1}}.
\]

### 2. 计算 \(T_n(u)\)

对 \(n\ge2\)，写成
\[
x=r+3^{n-1}q,\qquad 0\le r<3^{n-1},\quad q=0,1,2.
\]
则
\[
x^3\equiv r^3+3^nr^2q\pmod{3^{n+1}}.
\]
对 \(q\) 求和，仅当 \(3\mid r\) 时不为零。令 \(r=3s\)，得到
\[
T_n(u)=3\sum_{s=0}^{3^{n-2}-1}
e^{2\pi ius^3/3^{n-2}}.
\]

因此
\[
T_2(u)=3,\qquad T_3(u)=0,
\]
而当 \(n\ge4\) 时，
\[
T_n(u)=9T_{n-3}(u).
\]

再令
\[
\eta=e^{2\pi i/9},\qquad
\alpha(u)=1+\eta^u+\eta^{-u}.
\]
则
\[
T_1(u)=\alpha(u),
\]
从而
\[
T_1=\alpha(u),\quad T_2=3,\quad T_3=0,\quad
T_4=9\alpha(u),\quad T_5=27,\quad T_6=0.
\]

另外，当 \(t=6\) 时，只需模 \(3\) 计算，同样得到 \(S(k)=0\)。

于是各非零频率对应的 \(S(k)\) 为
\[
\begin{array}{c|c}
t&S(3^tu)\\ \hline
0&0\\
1&81\\
2&81\alpha(u)\\
3&0\\
4&243\\
5&243\alpha(u)\\
6&0
\end{array}
\]

### 3. 计算 \(\alpha(u)^3\) 的和

模 \(9\) 的单位为
\[
\pm1,\pm2,\pm4.
\]
记
\[
c_j=\cos\frac{2\pi j}{9},\qquad j=1,2,4.
\]
则
\[
\alpha(j)=1+2c_j.
\]
由九次单位根关系可得
\[
c_1+c_2+c_4=0,\qquad
c_1^2+c_2^2+c_4^2=\frac32.
\]
又因为
\[
\cos\left(3\cdot\frac{2\pi j}{9}\right)=-\frac12,
\]
所以
\[
4c_j^3-3c_j=-\frac12,
\]
即
\[
8c_j^3=6c_j-1.
\]
于是
\[
\alpha(j)^3=(1+2c_j)^3
=12(c_j+c_j^2).
\]
因此
\[
\alpha(1)^3+\alpha(2)^3+\alpha(4)^3
=12\left(0+\frac32\right)=18.
\]
由于 \(\alpha(-u)=\alpha(u)\)，故
\[
\sum_{u\in(\mathbb Z/9\mathbb Z)^\times}\alpha(u)^3=36.
\]

对于 \(t=2\)，此时 \(u\) 遍历模 \(3^5=243\) 的单位。每个模 \(9\) 的单位剩余类有 \(27\) 个提升，所以
\[
\sum_{\substack{u\bmod243\\3\nmid u}}\alpha(u)^3
=27\cdot36=972.
\]

### 4. 汇总

满足 \(v_3(k)=1\) 的 \(k\) 有
\[
\varphi(3^6)=486
\]
个；满足 \(v_3(k)=4\) 的 \(k\) 有
\[
\varphi(3^3)=18
\]
个。因此
\[
\begin{aligned}
N
&=\frac1{3^7}\left(
3^{18}
+486\cdot81^3
+972\cdot81^3
+18\cdot243^3
+36\cdot243^3
\right)\\
&=\frac{3^{18}+2\cdot3^{17}+4\cdot3^{17}
+2\cdot3^{17}+4\cdot3^{17}}{3^7}\\
&=\frac{15\cdot3^{17}}{3^7}\\
&=15\cdot3^{10}\\
&=885735.
\end{aligned}
\]

所以
\[
N\equiv735\pmod{1000}.
\]

最终答案：735