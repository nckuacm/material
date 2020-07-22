---
tags: 成大高階競技程式設計 2020, ys
---

# 2020 Week 14: Number theory
在演算法競賽中，通常涉及的**數論**大部份是**整數**的**代數性質**

# Greatest Common Divisor
中譯 最大公因數，以下簡稱 GCD
給定整數 $a, b$，它倆各自有**自己的因數**[^factor]，取**相同**的因數中**最大**的數，即最大公因數 $\gcd(a, b)$

`#include<algorithm>` 有內建的 `__gcd` 函數

#### 練習：

[CODEFORCES 1155C Alarm Clocks Everywhere](https://codeforces.com/contest/1155/problem/C)
[CODEFORCES 1133D Zero Quantity Maximization](https://codeforces.com/contest/1133/problem/D)
[CODECHEF SnackDown 2019 Round 1A Chef and Periodic Sequence](https://www.codechef.com/problems/PERIODIC)
[CODEFORCES 1107D Compression](https://codeforces.com/contest/1107/problem/D)
### GCD 性質
- $\gcd(a, b) = \gcd(b, a)$
- $\gcd(a, 0) = |a|$
- $\gcd(a, b) = \gcd(a, b \% a)$
> $\%$ 是 C++ 中的取餘數運算，表示存在 $k$ 滿足 $0 \leq b\% a = b - k\cdot a < a$

## Euclidean algorithm
> 輾轉相除法

:::info
給定整數 $a, b$ 求出 $\gcd(a, b)$
:::

令 $a_0=a, b_0=b$，根據 [GCD 性質](#GCD-性質)：
$$
\begin{split}
\gcd(a_0, b_0) &= \gcd(b_0 \% a_0 = \underline{b_1}, a_0) \\
\gcd(\underline{b_1}, a_0) &= \gcd(a_0 \% b_1 = \underline{a_1}, b_1) \\
\gcd(\underline{a_1}, b_1) &= \gcd(b_1 \% a_1 = b_2, a_1) \\
&\vdots \\
\gcd(a_i, b_i) &= \gcd(b_i \% a_i = a_{i+1}, b_i) \\
&\vdots \\
\gcd(\mathbf{0}, b_n) &= |b_n|
\end{split}
$$

#### 練習：
:::warning
證明上述過程 $\gcd$ 中**左**參數比**右**參數要先變為 $0$
:::

![](https://i.imgur.com/HtQU22S.gif) [^euclidean]

綜合上面過程，實作程式碼：
```cpp
int gcd(int a, int b)
  { return a? gcd(b%a, a) : b; }
```

#### 練習：
[UVa OJ 10407 Simple division](https://uva.onlinejudge.org/external/104/10407.pdf)

## Bezout’s Theorem
對於所有整數 $a, b$，存在**整數** $x, y$ 使得 $ax + by = \gcd(a, b)$


## Extended Euclidean algorithm

:::info
給定整數 $a, b$ 求出整數 $x, y$ 滿足 $ax + by = \gcd(a, b)$
:::


令 $g = \gcd(0, g) = \gcd(a, b)$，配合 [Bezout’s Thm](#Bezout’s-Theorem) 得到：
$$ 0\cdot x_n + g\cdot y_n = g$$
明顯的，$x_n \in \mathbb{Z}, y_n = 1$
> 即 $x_n$ 為任意整數

而根據 [GCD 性質](#GCD-性質) $\gcd(a_i, b_i) = \gcd(b_i\%a_i, a_i)$
以及 $b_i \% a_i = b_i - \lfloor{b_i\over a_i}\rfloor \cdot a_i$
得到
$$
\begin{split}
g &=   x_j\cdot (b_i \% a_i) &+ y_j &\cdot a_i \\
&=  x_j\cdot (b_i - \lfloor{b_i\over a_i}\rfloor \cdot a_i)  &+ y_j &\cdot a_i \\
&= x_j \cdot b_i &+ (y_j - \lfloor{b_i\over a_i}\rfloor \cdot x_j) &\cdot a_i
\end{split}
$$

也就是說
$$
g = a_i \cdot x_{j-1} + b_i \cdot y_{j-1} \text{ for }\begin{cases}
x_{j-1} = y_j - \lfloor{b_i\over a_i}\rfloor  \cdot x_j \\
y_{j-1} = x_j
\end{cases}
$$

綜合上述過程：
```cpp
int gcd(int a, int b, int &x, int &y) {
  if(!a) {
    x = 0, y = 1; // x 為任意整數
    return b;
  }

  int g = gcd(b%a, a, x, y);
  int xp = y - b/a * x, yp = x; // p := previous
  x = xp, y = yp;

  return g;
}
```

#### 練習：
[UVa OJ 10104 Euclid Problem](https://uva.onlinejudge.org/external/101/10104.pdf)
[UVa OJ 10090 Marbles](https://uva.onlinejudge.org/external/100/10090.pdf)
[UVa OJ 10413 Crazy Savages](https://uva.onlinejudge.org/external/104/10413.pdf)



# Modular arithmetic (同餘運算)

在競賽中若計算結果超出了數值範圍外，則通常會要求結果對某個數字**取餘數**
所以得熟悉取完餘數後的數字們之間的**關係**及**運算**。

#### 練習：
[CODEFORCES 1165A Remainder](https://codeforces.com/contest/1165/problem/A)

若**數字們**都對 $n$ **取餘數**後，它們就處於"**模 $n$**"的狀態下了
將 $a$ 除以 $n$ 的餘數記做 $a \space (\text{mod } n)$
> 記得檢查若 a 為**負數**的情況

若 $a  \space (\text{mod } n)$ 與 $b \space (\text{mod } n)$ 相等，則記 $a \equiv b \space (\text{mod } n)$，稱 $a, b$ 有**同餘關係**
也就是說 $\forall k \in \mathbb{Z}\centerdot a \equiv a + k\cdot n \space (\text{mod } n)$
> 可將括號內看作數字們處於一個**特定狀態**下，而模 $n$ 通常會省略括號

並且若
$$a \equiv b \mod n \\c \equiv d \mod n$$
則
$$a+c \equiv b+d \mod n\\a\times c \equiv b\times d \mod n$$


#### 練習：
[CODEFORCES 1178C Tiles](https://codeforces.com/contest/1178/problem/C)
[CODEFORCES 1151C Problem for Nazar](https://codeforces.com/contest/1151/problem/C)
[CODEFORCES 1165E Two Arrays and Sum of Functions](https://codeforces.com/contest/1165/problem/E)[^sort]

## 歐拉定理
如果 $a, n$ **互質**[^coprime]，那麼 
$$a^{\phi(n)} \equiv 1 \mod n$$
這裡 $\phi(n)$ 是與 $n$ 互質且小於 $n$ 的**正整數**的個數
> 例如 $1, 5$ 和 $6$ 互質， $\phi(6) = 2$

## 費馬小定理
如果 $a, p$ 互質，且 $p$ 為**質數**，那麼
$$a^{p-1} \equiv 1 \mod p$$
> 是歐拉定理的特例


# 模反元素

數字們模 $n$ 後，對於加法、減法、乘法，其性質都與以往差不多，但對數字做**除法**卻不盡人意
> 注意，在同餘運算中只會有**整數**，有理數無理數等其他數不會出現


**反元素**指的是元素與其**運算**後為**單位元素**的元素
例如：
- 整數 $a$ 的**加**法反元素為 $-a$
- 整數 $a$ 的**乘**法反元素為 $1\over a$

而元素 $a$ 的**模 $n$ 反元素**為 $a^{-1}$ 滿足
$$ a \cdot a^{-1} \equiv 1 \mod n$$

根據 [Bezout's Thm](#Bezout’s-Theorem)
- $ax + ny = \gcd(a, n) = 1$ 在模 $n$ 有 $ax \equiv 1 \mod n$
- $ax + ny = \gcd(a, n) \not= 1$ 在模 $n$ 有 $ax \not\equiv 1 \mod n$

也就是說 $\gcd(a, n) = 1$ (互質)，$a$ 才擁有模 $n$ 反元素

## 求出模 $n$ 反元素
- 當 $n$ 為**質數**時：

根據[費馬小定理](#費馬小定理)有
$$a \cdot a^{n-2} \equiv 1 \mod n$$
所以 $a^{n-2}$ 是 $a$ 的模反元素
> 下週教的**快速求冪演算法**能在 $O(\log_2 n)$ 得到 $a^{n-2}$

- 當 $n$ **非質數**時：

根據 [Bezout's Thm](#Bezout’s-Theorem) 有
$$
\begin{split}
&a\cdot x + n\cdot y \space &= 1 \\
\Rightarrow \space &a \cdot x &\equiv 1 \mod n
\end{split}
$$
可用[擴展歐幾里得演算法](#Extended-Euclidean-algorithm)找到 $x = a^{-1}$

#### 練習：
[ZERO JUDGE a289 Modular Multiplicative Inverse](https://zerojudge.tw/ShowProblem?problemid=a289)
[CODEFORCES 300C Beautiful Numbers](https://codeforces.com/contest/300/problem/C)
[NCKU OJ 22 爬樓梯 k](https://oj.leo900807.tw/problems/22)
[CODEFORCES 935D Fafa and Ancient Alphabet](https://codeforces.com/contest/935/problem/D)

# 孫子定理
:::info
給定 $a_1, a_2, \cdots ,a_n$ 以及 $m_1, m_2, \cdots, m_n$ ，其中任意 $m_i, m_j$ 有 $\gcd(m_i, m_j) = 1$

求滿足下式的 $x$ 值為何？
$$
x \equiv a_1 \mod m_1 \\
x \equiv a_2 \mod m_2 \\
\vdots \\
x \equiv a_n \mod m_n
$$
:::

設 $x = a_1\cdot y_1 + a_2\cdot y_2 + \cdots + a_n\cdot y_n$，且在模 $m_i$ 時為 $x = a_1\cdot 0 + \cdots + a_i\cdot 1 + \cdots + a_n\cdot 0 \mod m_i$
> 要仔細思考，怎樣的 $y_i$ 才會符合問題中的方程式定義？

從**小規模觀察**，假設 $n = 2$，$x$ 要在
- 模 $m_1$ 的時候 $(y_1, y_2) \equiv (1, 0)$
- 模 $m_2$ 的時候 $(y_1, y_2) \equiv (0, 1)$

所以模數應當是**對應的 $y_i$ 因數**，即 $y_1 = m_2\cdot t_1$ 且 $y_2 = m_1\cdot t_2$，這樣
- 模 $m_2$ 的時候 $y_1 \equiv 0$
- 模 $m_1$ 的時候 $y_2 \equiv 0$

接著思考 $y_1 \equiv 1$ 與 $y_2 \equiv 1$ 的狀況
- $y_1 = m_2\cdot t_1 \equiv 1 \mod m_1$，可推出 $t_1$ 是 $m_2$ 的模 $m_1$ 反元素
- $y_2 = m_1\cdot t_2 \equiv 1 \mod m_2$，可推出 $t_2$ 是 $m_1$ 的模 $m_2$ 反元素

最後整理上述有
$$
x = (m_2\cdot t_1) a_1 + (m_1\cdot t_2) a_2 + k\cdot m_1 \cdot m_2
$$
其中 $k$ 為任意整數
> 加法最後項 $k\cdot m_1 \cdot m_2$ 表示 $x$ 有多種解，是根據[同餘關係](#Modular-arithmetic-同餘運算)有 $a \equiv a + k\cdot m \mod m$

根據上述的提示，可以嘗試推廣當 $n > 2$ 時 $x$ 的解
如法泡製的將 $m_i$ 作為**對應的 $y_j$ 因數**，即 $\forall j \not= i\centerdot y_j \equiv 0 \mod m_i$
所以定義 $M_i = {{\prod\limits_{k=1}^n m_k}\over m_i}$[^prod]，該 $y_i$ 可推廣為 $y_i = M_i \cdot t_i$，且 $t_i$ 為 $M_i$ 的模 $m_i$ 反元素

於是推出原問題方程組的解為
$$
\begin{split}
x &= (M_1\cdot t_1) a_1 + (M_2\cdot t_2) a_2 +  \cdots + (M_n\cdot t_n) a_n + k\cdot M \\
&= \sum\limits_{k=1}^n (M_i\cdot t_i) a_i + k\cdot M
\end{split}
$$
其中 $k$ 為任意整數，且 $M = \prod\limits_{k=1}^n m_k$

#### 練習：
[ZERO JUDGE d791 00756 - Biorhythms](https://zerojudge.tw/ShowProblem?problemid=d791)
[CODEFORCES 687B Remainders Game](https://codeforces.com/problemset/problem/687/B)
[Kattis generalchineseremainder Chinese Remainder Theorem (non-relatively prime moduli)](https://open.kattis.com/problems/generalchineseremainder)
[TIOJ 1459 B.完全子圖](https://tioj.ck.tp.edu.tw/problems/1459)



[^factor]: 整數 $n$ 的**因數**為可**整除** $n$ 的數
[^euclidean]: [Wikipedia/ 輾轉相除法的演示動畫](https://zh.wikipedia.org/zh-tw/%E8%BC%BE%E8%BD%89%E7%9B%B8%E9%99%A4%E6%B3%95#/media/File:Euclidean_algorithm_252_105_animation_flipped.gif)
[^sort]: 請留意 mod 後大小關係會改變QQ
[^coprime]: 最大公因數為 $1$
[^prod]: $\prod$ 是**連乘**符號，也就是說 $\prod\limits_{k=1}^n m_k = m_1\times m_2\times \cdots \times m_n$