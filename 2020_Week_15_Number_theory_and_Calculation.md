---
tags: 成大高階競技程式設計 2020, ys
---

2020 Week 15: Number theory & Calculation
=
>數大便是美

本週繼續由[第十四週](https://hackmd.io/@nckuacm/SJiGbcM2U)主題延伸，
也就是說以下內容或多或少需要**最大公因數**以及**模運算**的相關定理。

# 大數乘法
**大數字**的乘法，普通的[直式乘法](https://en.wikipedia.org/wiki/Multiplication_algorithm#Long_multiplication) $O(N^2)$ 不夠快，因此將介紹快速的**乘法**運算
> 這裡 $N$ 為數字的位數

## Gauss's complex multiplication algorithm
對於**複數** $a+bi$ 與 $c+di$ **相乘**
$$
(a+bi) \cdot (c+di) = (ac-bd) + (bc+ad)i
$$
相較於 $4$ 次乘法 $ac, bd, bc, ad$

可只用 $3$ 次乘法 $\begin{cases}k_1 = c(a+b)\\ k_2=b(c+b)\\ k_3=a(d-c)\end{cases}$ 完成複數乘法：

$$
\text{where } \begin{cases}
\begin{split}
(ac - bd) &= ac + \mathbf{bc} - \mathbf{bc} - bd &= \underline{c\cdot (a+b)} - \underline{b\cdot (c+b)} =k_1-k_2\\
(bc + ad) &= \mathbf{ac} + bc + ad -\mathbf{ac} &= \underline{c\cdot (a+b)} + \underline{a\cdot (d-c)} =k_1+k_3
\end{split}
\end{cases}
$$
因此複數相乘
$$
\begin{split}
(a+bi) \cdot (c+di) &= (ac-bd) + (bc+ad)i \\
&= (k_1-k_2) + (k_1+k_3)i
\end{split}
$$

> 乘法操作變少，但加減法操作變多了(？)

## Karatsuba algorithm
對於**整數** $\begin{cases}\begin{split}x &= am+b\\y &= cm+d \end{split}\end{cases}$ **相乘**
> 受到**複數乘法啟發**，將整數 $x$ 分割成兩數 $a, b$

$$
x\cdot y = ac\cdot m^2 + (ad+bc)\cdot m + bd
$$

能用 $3$ 次乘法 $\begin{cases}z_1=ac\\ z_2=bd\\ z_3=(a+b)(c+d)\end{cases}$ 就完成運算：
$$
\begin{split}
(ad+bc) &= \mathbf{ac}+ad+bc+\mathbf{bd}-\mathbf{ac}-\mathbf{bd} \\
&=\underline{(a+b)(c+d)}-\underline{ac}-\underline{bd} \\
&= z_3-z_1-z_2
\end{split}
$$

因此整數相乘 $$x\cdot y=z_1m^2 + (z_3-z_1-z_2)m+z_2$$

#### 範例 $\begin{cases}\begin{split}12345 &= 12 &\cdot 1000 &+345\\6789 &= 6 &\cdot 1000 &+789 \end{split}\end{cases}$相乘：
:::spoiler
首先 $\begin{cases}z_1=12\cdot 6 &= 72\\ z_2=345\cdot 789 &= 272205\\ z_3=(12+345)(6+789) = 357\cdot 795&=283815\end{cases}$

則 $\begin{split}12345 \cdot 6789 &= 72 \cdot 1000^2 + (283815-72-272205)\cdot 1000+272205 \\
&=72000000+11538000+272205 \\ &= 83810205
\end{split}$

> 似乎沒有比較快欸
> 那是因為你**想**的乘法，仍然是 $O(n^2)$ 的[直式乘法](https://en.wikipedia.org/wiki/Multiplication_algorithm#Long_multiplication)
:::


對於上述演算法，凡是遇到乘法運算(求 $z_1, z_2, z_3$)，都使用同樣的演算法
並且對於數字的分割，總是分割成**均等的兩半**
> 例如範例中 $6789$ 應分成 $67$ 和 $89$


```cpp
int k(int x, int y) {
  if(x < 10 || y < 10) return x*y;

  int len = min(log10(x), log10(y)); // 數字位數
  int m = pow(10, len/2 + 1);

  auto [a, b] = div(x, m); // since c++17
  auto [c, d] = div(y, m);

  int z1 = k(a, c);
  int z2 = k(b, d);
  int z3 = k(a+b, c+d);

  return z1*m*m + (z3-z1-z2)*m + z2;
}
```
時間成本 $T(n)$ **約**為
$$
\begin{split}
T(n) &= 3 \cdot T(\lceil n/2 \rceil) + c_n\\
T(1) &= 1 + c_1
\end{split}
$$
此處 $n$ 為整數位數，$c_n$ 為**加法**運算的成本
複雜度為 $O(3^{\log_2n}) = O(n^{\log_23})$

---

# 求大數冪
樸素的求整數 $a$ 的 $n$ 次**冪** $x =a^n$，可以採用連乘：
```cpp
int x = 1;
while (n--) x *= a;
```

## 整數平方求冪 (Exponentiating by squaring)
> 又稱快速冪演算法

基於分治法，
- 若 $n$ 為**偶數**則 $a^n = a^{n\over 2} \times a^{n\over 2}$
- 若 $n$ 為**奇數**則 $a^n = a^{\lfloor {n\over 2} \rfloor} \times a^{\lfloor {n\over 2} \rfloor} \times a$


### Top-down
```cpp
int power(int a, int n) {
  if (n == 1) return a;
  
  int x = power(a, n/2);
  return n&1? x*x*a : x*x;
}
```

### Bottom-up
```cpp
int x = 1;
while (n) {
  if (n&1) x *= a;
  a *= a;
  n >>= 1;
}
```
複雜度從樸素的 $O(n)$ 改進至 $O(\log_2n)$

#### 練習：
[UVa OJ 374 Big Mod](https://uva.onlinejudge.org/external/3/p374.pdf)
[CODEFORCES 615D Multipliers](https://codeforces.com/contest/615/problem/D)


## 矩陣平方求冪
對於**方陣**乘法，也能沿用[快速冪](#整數平方求冪-Exponentiating-by-squaring)的做法
例如求第 $n$ 項的費式數 $f_n$：

$$
\begin{bmatrix}
1 &1 \\
1 &0 \\
\end{bmatrix}^n
\cdot
\begin{bmatrix}
1 \\
0
\end{bmatrix} =
\begin{bmatrix}
f_{n+1} \\
f_n
\end{bmatrix}
$$


### Bottom-up
```cpp
int M[][2] = {{1, 1}, {1, 0}};
int f[] = {1, 0};

while (n) {
  if (n&1) {
    int t[] = {f[0], f[1]};
    f[0] = M[0][0] * t[0] + M[0][1] * t[1];
    f[1] = M[1][0] * t[0] + M[1][1] * t[1];
  }

  int t[][2] = {{M[0][0], M[0][1]}, {M[1][0], M[1][1]}};
  M[0][0] = t[0][0] * t[0][0] + t[0][1] * t[1][0];
  M[0][1] = t[0][0] * t[0][1] + t[0][1] * t[1][1];
  M[1][0] = t[1][0] * t[0][0] + t[1][1] * t[1][0];
  M[1][1] = t[1][0] * t[0][1] + t[1][1] * t[1][1];

  n >>= 1;
}
```

其複雜度 $O(\log_2 n)$

#### 練習：
[ZERO JUDGE b525 先別管這個了，你聽過turtlebee嗎？](https://zerojudge.tw/ShowProblem?problemid=b525)

---

# 質數判斷
若數 $n > 1$ 能被 $1$ 與 $n$ 以外的數整除，則 $n$ 為合數

## $[2, \sqrt{n}]$ 判斷法
假設 $n$ 是**合數**，則 $n = x\cdot y$ 其中 $1 < x < y$，可證明 $x$ 的大小不超過 $\sqrt{n}$；
所以若有 $x \in [2, \sqrt{n}]$ 滿足 $x\mid n$，則 $n$ 不是質數。
> $x\mid n$ 表示 $x$ 整除 $n$

```cpp
for (int i = 2; i <= sqrt(n); i++)
  if (n%i == 0) return false;

return true;
```

#### 練習：
[CODEFORCES 1033B Square Difference](https://codeforces.com/problemset/problem/1033/B)

## Fermat's Primality Test

根據[費馬小定理](https://hackmd.io/@nckuacm/SJiGbcM2U#%E8%B2%BB%E9%A6%AC%E5%B0%8F%E5%AE%9A%E7%90%86)，
$$
n \text{ is prime } \Rightarrow a^{n-1} \equiv 1 \mod n \text{, where }n \not\mid a
$$

雖然根據**邏輯規則**以下**逆命題不見得成立**
$$
n \text{ is prime } \Leftarrow a^{n-1} \equiv 1 \mod n \text{, where }n \not\mid a
$$
但卻有很**高的機率**[^probability_a]是成立的！

```cpp
int a = max(rand()%n, 2); // a in [2, n-1]
int x = power(a, n-1, n); // 求 $a^{n-1}$ 除以 n 的餘數
return x == 1;
```
> 如果 `power` 函數是[快速冪演算法](#整數平方求冪-Exponentiating-by-squaring)，則複雜度 $O(\log n)$

根據**邏輯規則**，如果回傳 `false`，那麼保證 $n$ 不是質數
> $p \Rightarrow q$ 與 $\neg p \Leftarrow \neg q$ 是等價的

注意這是**測試法**，有時會遇到難**判斷**的數，例如 $n = 561$ 對大部分的 $a$ 回傳 `true`

#### 練習：
[UVa OJ 10006 Carmichael Numbers](https://uva.onlinejudge.org/external/100/10006.pdf)

## Miller Rabin Primality Test
> Deus ex machina

繼續從 [Fermat's Primality Test](#Fermat’s-Primality-Test) 發展：

除 $2$ 以外的質數都為奇數，故**只關注奇數**就好
則 $n-1$ 是個**偶數**，可將其寫成 $n-1 = 2^t \cdot d$

再注意到，若 $i < t$ 有 $a^{2^i \cdot d} \equiv \pm 1 \mod n$ 則 $a^{2^t \cdot d} \equiv 1 \mod n$
> 因為 ${\pm 1}^2$ 等於 $1$ 嘛

並且若 $n$ 是個質數，則
$$
\begin{split}
x^2 \equiv 1 \mod n &\Rightarrow n \mid x^2 - 1 = (x+1)\cdot (x-1) \\
&\Rightarrow n \mid x+1 \lor n \mid x-1 \\
&\Rightarrow x \equiv -1 \mod n \lor x \equiv 1 \mod n
\end{split}
$$
> 如果 $n$ 是合數，第二列 $"\Rightarrow"$ 不一定會成立

有了上述性質，又能使質數測試成功機率大大提升
```cpp
if (n%2 == 0) return n == 2; // 2 為質數，反之偶數非質數

a %= n;
if (a == 0) return true;

int d = n-1, t = 0;
while (d%2 == 0) t++, d /= 2;

int x = power(a, d, n);
while (t--) {
  int nx = power(x, 2, n);
  if (nx == 1 && x != 1 && x != n-1) return false;
  x = nx;
}

return x == 1;
```
> 根據**邏輯規則**，如果回傳 `false`，那麼保證 $n$ 不是質數


經測試若數 $n$ 在下列範圍，皆能**正確判斷**質數：
- $n < 2^{32}$ 代入所有 $a \in \{2, 7, 61 \}$
- $n < 2^{64}$ 代入所有 $a \in \{2, 325, 9375, 28178, 450775, 9780504, 1795265022 \}$ 

這樣就能在 $O(\log n)$ 完美判斷常用質數了！

#### 練習：
[ZERO JUDGE a007 判斷質數](https://zerojudge.tw/ShowProblem?problemid=a007) [^萬惡a007]

---

# 質因數分解
根據唯一分解定理，任何合數都能被分解成一些質數的積
> 算術基本定理

## $[2, \sqrt{n}]$ 試除法
> 與[$[2, \sqrt{n}]$ 判斷法](#2-sqrtn-判斷法0)原理相同

合數 $n = p_1^{t_1} \cdot p_2^{t_2} \cdot \cdots \cdot p_k^{t_k}$ 為多個質數的乘積
不失一般性的有 $p_1 < p_2 < \cdots < p_k < \sqrt{n}$，所以在範圍 $[2, \sqrt{n}]$ **從小至大**找數試除即可。


```cpp
for (int p = 2; p <= sqrt(n); p++) {
  int t = 0;
  while (n%p == 0) n /= p, t++;
  if (t) factors.push_back({p, t});
}
if (n != 1) factors.push_back({n, 1});
```

#### 練習：
[CODEFORCES 1165D Almost All Divisors](https://codeforces.com/contest/1165/problem/D)
[CODEFORCES 1114C Trailing Loves (or L'oeufs?)](https://codeforces.com/contest/1114/problem/C)
[LeetCode 952 Largest Component Size by Common Factor](https://leetcode.com/problems/largest-component-size-by-common-factor/)

<!-- 
## Pollard's rho algorithm

:::info
http://math.mit.edu/~goemans/18310S15/factoring-notes.pdf
:::

#### 練習：
[POJ 1811 Prime Test](http://poj.org/problem?id=1811)
 -->


---

# 質數篩檢
>數質數可以有效安撫緊張的情緒

## Sieve of Eratosthenes
其精神是將質數的**倍數**都設為**非質數**(合數)

![](https://i.imgur.com/aws94bk.gif)

```cpp
vector<bool> is_p(maxn, true);
is_p[1] = false;

for (int n = 2; n < sqrt(maxn); n++) {
  if (!is_p[n]) continue;
  for (int m = n*n; m < maxn; m+=n) is_p[m] = false;
}
```
`n*n` 為對於所有 $x < n$，在數到 $n$ 之前 $x\cdot n$ 已經被篩過了。
>記得檢查是否 `n*n` 溢位

#### 練習：
[UVa OJ 543 Goldbach’s Conjecture](https://uva.onlinejudge.org/external/5/543.pdf)
[UVa OJ 10140 Prime Distance](https://uva.onlinejudge.org/external/101/10140.pdf)

## 線性篩法
> 線性是指演算法時間 $T(n) = O(n)$

與 Sieve of Eratosthenes 相反：刪所有質數的倍數，就是刪**所有數的質數倍**
而刪質數倍的過程能簡單的判斷是否能及早收手，讓計算效率大幅提升！

```cpp
vector<bool> is_p(maxn, true);
is_p[1] = false;

for (int n = 2; n < maxn; n++) {
  if (is_p[n]) prime.push_back(n);
  
  for (int p: prime) {
    if (p*n >= maxn) break; // 超出篩檢範圍
    
    is_p[p*n] = false;
    if (n%p == 0) break;
  }
}
```

### 存在性
:::info
數到 $n$ 以前，$n$ 就已經被判定過**是否**為質數了。
:::
根據[唯一分解定理](#質因數分解)，$n = p_1 \cdot p_2 \cdot\cdots\cdot p_k$，其中 $p_1\le p_2\le\cdots\le p_k$

令 $x_1 = p_2 \cdot\cdots\cdot p_k$，由於 $x_1<n$，所以必然有 $p_1$ 與 $x_1$ 相乘得到 $n$
> $p_1$ 就是透過第二層迴圈得來的
> $x_1$ 是在第一層迴圈數到 $n$ 以前曾數到的數字

就算 $p_1$ 整除 $x_1$， `if (n%p == 0) break;` 是之後才執行的，一定能湊得 $n$

### 唯一性
:::info
任意 $n$ 只會被判定過**恰好一次**
:::

令 $x_i = {n \over p_i}$， $i \in \{1, 2,\cdots, k\}$

$n$ **只**會被數到 $x_1$ 時給**判定**，同樣用 $p_1$ 湊得 $n = p_1\cdot x_1$
對於 $x_i \not= x_1$， 在數到 $p_1$ 時 $x_i$ 就被 $p_1$ 給整除了，因而不會湊得 $p_i \cdot x_i$
> $p_i$ 根據假設有 $p_i > p_1$

[^probability_a]: 此機率是根據測試的次數計算，每次測試會有不同的 $a$
[^萬惡a007]: 這根本不是基礎題= =