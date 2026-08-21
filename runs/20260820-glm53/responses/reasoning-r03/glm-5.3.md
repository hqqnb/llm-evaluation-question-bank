# 解答

**记号**：$3^6=729$，$3^7=2187$。

**预备（周期性）**：由二项式展开，$(a+729)^3=a^3+3a^2\cdot 729+\cdots\equiv a^3 \pmod{2187}$（因为 $3\cdot 729=2187$）。故 $a^3 \bmod 2187$ 只依赖 $a\bmod 729$，于是 $a=1,\dots,729$ 恰代表每个剩余类一次。

## 第 1 步：$a^3 \bmod 2187$ 的分布

按 $v_3(a)$ 分类（个数为 $486+162+54+27=729$ ✓）：

**(i) $3\nmid a$（486 个）**：断言 $a\mapsto a^3\bmod 2187$ 把这 486 个数**双射**到
$$H=\{u\in(\mathbb Z/2187)^*:\ u\equiv \pm1 \pmod 9\},\qquad |H|=\tfrac{2}{6}\cdot 1458=486.$$
- 像含于 $H$：模 9 的单元为 $1,2,4,5,7,8$，其立方 $\equiv\pm1\pmod 9$。
- 单射：若 $a\equiv b\pmod 3$（$a\neq b$ 单元），则 $a^2+ab+b^2=3(1+3(\cdots))$ 恰被 3 整除一次，且 $|a-b|<729\Rightarrow v_3(a-b)\le 5$，故 $v_3(a^3-b^3)\le 6<7$；若 $a\not\equiv b\pmod 3$，则两因子均不被 3 整除。故 $a^3\not\equiv b^3$。

**(ii) $v_3(a)=1$**（$a=3t$，$3\nmid t\le 243$，共 162 个）：$a^3\equiv 27\,(t^3\bmod 81)$。$[1,243]$ 中每个模 81 剩余类出现 3 次；$(\mathbb Z/81)^*$ 上立方映射的像为 $A=\{v\bmod 81: v\equiv\pm1\pmod 9\}$（18 个），每像恰 3 个原像。故 18 个剩余 $27v\,(v\in A)$ **各出现 9 次**。

**(iii) $v_3(a)=2$**（$a=9t$，$3\nmid t\le 81$，共 54 个）：$a^3\equiv 729\,(t\bmod 3)$：剩余 $729$、$1458$ **各 27 次**。

**(iv) $27\mid a$（27 个）**：$a^3\equiv 0$。

## 第 2 步：模 9 筛选

单元立方 $\equiv\pm1\pmod 9$，非单元立方均被 27 整除。故 $a^3+b^3+c^3\equiv0\pmod 9$ 要求**单元个数为 0 或 2**（1 个单元时 $\equiv\pm1$；3 个单元时 $\equiv\pm1,\pm3$，均非 0）。

## 第 3 步：情形 A（三个都被 3 整除）

写 $a=3\alpha$ 等，$\alpha,\beta,\gamma\in[1,243]$，条件化为
$$\alpha^3+\beta^3+\gamma^3\equiv 0\pmod{81}.$$
对此再用模 9 分析：$\alpha,\beta,\gamma$ 中单元个数 $\in\{0,2\}$。

**A1：都被 3 整除**：令 $\alpha=3\alpha'$（$\alpha'\le 81$），条件 $\Leftrightarrow \alpha'+\beta'+\gamma'\equiv0\pmod 3$（因 $x^3\equiv x$）。每个模 3 类在 $[1,81]$ 中 27 个数，故计数 $=81^3/3=\mathbf{177\,147}$。

**A2：恰两个单元**（设 $\alpha,\beta$ 单元，$\gamma=3\gamma'$，$\gamma'\le 81$）：需 $\alpha^3+\beta^3\equiv t\pmod{81}$，其中 $t=-27\gamma'^3\bmod 81\in\{0,27,54\}$。由 (ii)，单元 $\alpha\in[1,243]$ 的立方剩余是 $A$ 中每个恰 9 次。分解 $A=A_+\sqcup A_-$，$A_\pm=\{\pm1+9s: s\in\mathbb Z/9\}$。因 $t\equiv0\pmod 9$，须一正一负，此时 $x+y=9(s_1+s_2)\equiv t$ 即 $s_1+s_2\equiv t/9\pmod 9$，有 9 组解；两种符号共 18 个剩余对，每个对应 $9\times9=81$ 个 $(\alpha,\beta)$。故每个 $\gamma'$ 有 $18\times81=1458$ 对。计数 $=3\times81\times1458=\mathbf{354\,294}$。

$$\text{情形 A 合计 } S=177\,147+354\,294=\mathbf{531\,441}\ (=3^{12}).$$

## 第 4 步：情形 B（恰两个单元）

设 $a,b$ 单元，$c=3\gamma\le 729$（243 种）。需 $a^3+b^3\equiv t\pmod{2187}$，其中 $t=-c^3\equiv0\pmod{27}$。由 (i)，单元 $a$ 与 $x=a^3\bmod 2187\in H$ 一一对应，且 $H=H_1\sqcup(-H_1)$，$H_1=\{1+9s: s\in\mathbb Z/243\}$（243 个）。同号两个之和 $\equiv\pm2\pmod 9$ 不行；若 $x=1+9s_1\in H_1$，则 $y=t-x\equiv-1\pmod 9$ 是单元且属 $-H_1$，被唯一确定，反之亦然。故每个 $t\equiv0\pmod 9$ 恰有 $243+243=486$ 个有序对 $(a,b)$。

$$\text{情形 B 计数 }=3\times243\times486=\mathbf{354\,294}\ (=2\cdot 3^{11}).$$

## 第 5 步：汇总

$$N=531\,441+354\,294=885\,735=5\cdot 3^{11}.$$

**独立验证**（模 2187 的加法特征和）：设 $f$ 为立方剩余的重复度函数，则 $\hat f(k)=81\delta_{3\mid k}+81(\eta^k+\eta^{-k})\delta_{9\mid k}+243(\omega^k+\omega^{-k})\delta_{243\mid k}$，代入 $N=\frac1{2187}\sum_k \hat f(k)^3=\frac{1944\cdot81^3+63\cdot243^3}{2187}=885\,735$，与上面一致。（小规模版本如 $a,b,c\le 9$、模 27 的同类问题用此法算得 81，直接枚举亦为 81，方法可靠。）

$$885\,735 \equiv 735 \pmod{1000}.$$

最终答案：735