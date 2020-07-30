---
tags: 成大高階競技程式設計 2020, ys
---

2020 Week 3: Thought method
=

# 設計演算法的思維
本章先以最大連續和問題探討常見的設計演算法切入點：
- 枚舉
- 分治
- 動態規劃
- 貪心

接著會對各個常見思考方式提供一些範例題目

## 考慮最大連續和問題
給定一個長度為 $N$ 的整數數列 $A_1, A_2, ... , A_N$，要求找到 $1 \leq i \leq j \leq N$，使得 $A_i+A_{i+1}+...+A_j$ 儘量大。
> 注意 數列中可能會有負數

## 枚舉
所謂枚舉，通俗點說就是**數出**部份給定的集合中元素。
下面直接給出程式來解決最大連續和問題：


```cpp
int best = A[1]; //與其用無限小，不如這樣初始化更不易出錯

for (int L = 1; L <= N; i++) {
  for (int R = L; R <= N; R++) {
    int sum = 0;
    for (int k = L; k <= R; k++) sum += A[k];
    best = max(best, sum);
  }
}
```

通常枚舉會做為解題的起手式，有了正確性再考量改進方法
枚舉可將困難的**求值**問題化為簡單的**判定**問題
> 雖然計算量可能會變高，但針對問題特性能再改進


仔細觀察上面的演算法，會發現遞增 $k$ 跟遞增 $R$ 其實是同一回事，可改進為：
```cpp
for(int L = 0; L <= N; L++) {
  int sum = 0;
  for(int R = L; R <= N; R++) {
    sum += A[R];
    best = max(best, sum);
  }
}
```

#### 練習：
[GCJ Kickstart Round G 2018 A Product Triplets](https://code.google.com/codejam/contest/5374486/dashboard#s=p0): Small dataset


## 分治法
分治 (divide & conquer) 簡稱 D&C，就是將一個大的問題，**分成**幾個互相*獨立*的子問題，然後再將子問題分成子子問題[^2]，一直重複分割的動作，直到最小的問題足夠和別的最小問題**合併求解**出父問題。

將數列切一半，問左半的以及右半的最大連續和多少，以及問包含切開的那道分水嶺的最大連續和為多少，選出三者中最大值，它就是整個數列(原問題)的最大連續和：
```cpp
int maxsum(int l, int r) { // 此為左閉右開區間 [l, r)
  if (r-l == 1) return A[l];

  int m = (r+l)/2, sum, centre = A[m];

  sum = 0;
  for (int i = m; i < r; i++)
    centre = max(centre, sum += A[i]);
  
  sum = centre;
  for (int i = m-1; i >= l; i--)
    centre = max(centre, sum += A[i]);

  return max(centre, max(maxsum(l, m), maxsum(m, r)));
}
```



要驗證分治法的正確性，只需考慮子問題[^3]們解完後(假設已拿到解)，再合併為父問題看是否解完即可，並考慮最小的孫子問題到的邊界是否正確。

## 動態規劃
部份朋友可能知道可以令 $S_i = A_1 + A_2 + ... A_i$
而 $A_i+A_{i+1}+...+A_j = S_j - S_{i-1}$
這樣子有了 $S_i$ 就可將連續和的計算從 $O(N)$ 降為 $O(1)$

構造 $S_i$ 非常的直覺：
```cpp
S[0] = 0;
for (int i = 1; i <= N; i++) S[i] = S[i-1] + A[i];
```
從**邊界**遞推地**記錄**所有問題的解，且一個項用到前一項的**最佳**結果，就是動態規劃的精神。
通常應用動態規劃思考的問題會是**求極值**或是**確定值**的問題。
> $S_i$ 的值就是個確定值的例子

而程式可改為：
```cpp
for (int L = 1; L <= N; L++)
  for (int R = L; R <= N; R++) best = max(best, S[R] - S[L-1]);
```
複雜度為 $O(N^2)$。

[CODEFORCES 327A Flipping Game](https://codeforces.com/contest/327/problem/A)

## 貪心法
籠統的講，每次做一個在**當下看起來**最佳的決策，進而漸漸求出全局最佳解
> 這種短視近利的心態，居然也是個不錯的思維(
> 貪心法是動態規劃的特例，貪心可用動態規劃的角度去看

一直累加**只會增大**的話就繼續累加
而若是途中會減少，但後面還會有正數，所以繼續加
但如果累計值為**負**的話，不如捨棄掉這個累計值

```cpp
int best = A[N], sum = 0;

for (int R = 1; R <= N; R++) {
  sum = max(A[R], sum + A[R]);
  best = max(best, sum);
} 
```

#### 練習：
[ZEROJUDGE d652 貪婪之糊](https://zerojudge.tw/ShowProblem?problemid=d652)


## 枚舉範例

#### 範例 最長回文子字串：
:::info
給定一長度 $N$ 字串 $S$，算出**最長回文子字串**的長度
:::

例如 $\text{aabab}$
有 $\text{a, a, b, a, b, aa, aba, bab}$ 共 6 個**回文子字串**
> 回文為正著看與反著看**一樣**的字

而在其中，$\text{aba, bab}$ 是**最長**的，所以答案為 3

可以採用上面[連續和](^枚舉)的做法，先將每個區間數出來，再判斷其是否為回文
```cpp
int ans = 0;
for(int L = 0; L < N; L++)
  for(int R = L; R < N; R++)
    if(is_palindrome(L, R)) ans = max(ans, R-L+1);
```
但判斷字串為回文 (`is_palindrome(L, R)`) 需 $O(N)$，綜合起來這做法要 $O(N^3)$
> 由於要正著和反著一一比對

仔細觀察回文字串的特性，回文的組成是兩字元**相同**成對的
當有回文 $A$，其 $aAa, baAab, \cdots$ 也是回文
當字串 $B$ 不是回文，$aBa, baBab, \cdots$，不管**兩端同時**擴充什麼字元都不是回文

所以對每個字元往外擴充，或是從字元跟字元的隙縫擴充，看是否為回文

```cpp
int ans = 1;

// 從單個字元擴充
for(int i = 0; i < N; i++)
  for(int l = 1; i-l >= 0 && i+l < N; l++) {
    if(S[i-l] != S[i+l]) break;
    ans = max(ans, l*2 + 1);
  }

// 從字元跟字元間擴充
for(int i = 0; i < N; i++)
  for(int l = 1; i-l >= 0 && i+l-1 < N; l++) {
    if(S[i-l] != S[i+l-1]) break;
    ans = max(ans, l*2);
  }
```
> 也就是說找對**對象**很重要，若不研究題目**性質**而草率枚舉效率也就一般般。

#### 範例 印出九九九九九九九九九九九九九九九乘法表
> 先數一下到底有幾個九，好，共 $15$ 個

若是九九乘法表(九只有兩個)應該能很輕易的寫出來吧？
```cpp
for(int L = 1; L <= 9; L++)
  for(int R = 1; R <= 9; R++)
    printf("%d * %d = %d\n", L, R, L*R);
```

但 $15$ 個的話，寫 $15$ 層迴圈未免也太累了
這狀況就體現了[遞迴](#%E7%AF%84%E4%BE%8B-%E5%8D%B0%E5%87%BA%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%9D%E4%B9%98%E6%B3%95%E8%A1%A8)保存過去的狀態的方便之處：
```cpp
int const depth = 15;

void dfs(int i, long long res) { // res := result
  if(i > depth) {
    for(int j = 1; j <= depth; j++)
      printf("%d %c ", q[j], (j != depth)? '*' : '=');
    printf("%lld\n", res);
    
    return;
  }
  
  for(int j = 1; j <= depth; j++) {
    q[i] = j;
    dfs(i+1, res * j);
  }
}
```
呼叫函式為 `dfs(1, 1)`
> 如果好奇 `dfs` 是甚麼的話，第五週會教 DFS (Depth-First Search)


<!-- 
#### 範例 XOR Equation：

:::info
給定 $N, V$ 及 $a_i$ 其中 $1 ≤ i ≤ N$，
算出有多少組 $1 ≤ x_i ≤ a_i$ 滿足 $x_1 ⊕ x_2 ⊕ \cdots ⊕ x_N = V$
限制 $N ≤ 15, a_i ≤ 9, ∏\limits_{i=1}^N a_i ≤ 40000$
:::

簡單的枚舉在 $N = 3$ 時你可能會這樣寫
```cpp
int cnt = 0;
for(int i = 1; i <= a1; i++)
  for(int j = 1; j <= a2; j++)
    for(int k = 1; k <= a3; k++)
      if(i^j^k == V) cnt++;
```
 -->


## 分治範例

#### 範例 [TIOJ 1080 A.逆序數對](https://tioj.ck.tp.edu.tw/problems/1080)：
:::info
給長度為 $n$ 的數列 $a_0, a_1, \cdots , a_{n-1}$
若 $i < j$ 且 $a_i > a_j$，則 $(a_i, a_j)$ 稱為逆序數對
請計算出該數列中有多少逆序數對
:::

先試試枚舉：
```cpp
int cnt = 0;
for(int i = 0; i < n; i++)
  for(int j = i+1; j < n; j++)
    if(a[i] > a[j]) cnt++;
```
了無新意，來開始研究題目的性質吧
一次要處理太多數字很不知所措，那麼就先**觀察小規模**的問題

$n=2$，有 $4, 3$，則有逆序數對 $(4, 3)$
$n=4$，有 $3, 4, 7, 1$，則有逆序數對 $(3, 1), (4, 1), (7, 1)$
綜觀全局，會發現逆序就是大的數字在左邊，小的數字在右邊
> 這不是廢話嗎？

但小規模的話可能會一對對數字去看，大規模則可以**切成兩區**去比對
這給了一種解法的動機：分而治之

每次切成兩個區塊，區塊內的數對假設已計算好了
接著從分界的兩邊去看，左區若是有比右區還大的數字，就記一筆

假設左區有 $N$ 個數字 $a_i$，右區有 $M$ 個數字 $b_j$，則：
```cpp
int cnt = 0;
for(int i = 0; i < N; i++)
  for(int j = 0; j < M; j++)
    if(a[i] > b[j]) cnt++;
```
整個完整程式碼就變成如下：

```cpp
int count(int l, int r) { // [l, r) 左閉右開區間
  if(r-l == 1) return 0;

  int m = (l+r)/2, cnt = 0;
  cnt += count(l, m);
  cnt += count(m, r);

  for(int i = l; i < m; i++)
    for(int j = m; j < r; j++)
      if(a[i] > a[j]) cnt++;

  return cnt;
}
```
但估算一下，這複雜度不是沒改善嗎？
再回去觀察一下問題，例如 $4, 5, 1, 2, 3, 9$

從中間切開的話則為 $4, 5, 1$ 與 $2, 3, 9$
假設從 $4$ 開始比對完 $(4, 2), (4, 3)$
那麼從 $5$ 比對的話，由於 $5 > 4$，能得知至少也有兩個數 $2, 3$ 與 $5$ 組成逆序

這個給了個契機讓左區的數們都按照**升序**排列
也讓右區的數字**升序**的話，在遇到右區數 $a$ 比左區數 $b$ 大時，由於 $a$ 之後的數都會比 $b$ 大，也能讓左區換下個數。

```cpp
int count(int l, int r) { // [l, r)
  if(r-l == 1) return 0;

  int m = (l+r)/2, cnt = 0;
  cnt += count(l, m);
  cnt += count(m, r);

  vector<int> b; // 保存升序數列
  int j = m;
  for(int i = l; i < m; i++) {
    while(j < r && a[i] > a[j]) b.push_back(a[j++]);
    b.push_back(a[i]);

    cnt += j-m;
  }
  while(j < r) b.push_back(a[j++]);

  copy(b.begin(), b.end(), a+l);

  return cnt;
}
```

<!-- 
#### 範例 [CODEFORCES 1111C Creative Snap](https://codeforces.com/contest/1111/problem/C)：

#### 範例 填 L 型格子
:::info
給 $N \times N$ 二維白色表格，其中有一格維黑色的
要求用 $L$ 型黑色格子填滿的方法
![](https://i.imgur.com/AnBNMNy.png)
(左邊是二維表格、右邊是 $L$ 型格子(可旋轉))
:::
 -->


## 動態規劃範例
> 若這節沒法好好理解，先看個感覺就行，第七週將會有動態規劃的完整介紹

#### 範例 [LeetCode 70 Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)：
:::info
上樓梯每走一次可以走 $1$ 或 $2$ 階，從 $0$ 階(地板)開始走到 $n$ 階的走法有幾種？ 
:::

若 $n = 0$，則答案直接為 $1$
若 $n = 1$，則答案為 $1$
若 $n = 2$，由於可以每次都走 $1$ 階，或直接走 $2$ 階，則答案為 $2$

得知造成答案變動的**決策**就只有每次要走 $1$ 階還是走 $2$ 階的選擇
設**狀態** $dp(i)$ 表示從 $0$ 走到第 $i$ 階的走法數
則從 $i-1$ 走 $1$ 階會到 $i$ 階，從 $i-2$ 走 $2$ 階會到 $i$ 階，除此之外沒有別的走法
得知**狀態轉移** $dp(i) = dp(i-1) + dp(i-2)$

在程式中，$dp(i-1), dp(i-2)$ 會是已經解完的狀態(子問題)
而透過這兩個狀態，能將尚未求解的 $dp(i)$ 得解，進而再去求解其他大問題

```cpp
dp[0] = dp[1] = 1;
for(int i = 2; i <= n; i++) dp[i] = dp[i-1] + dp[i-2];
```

> 這其實就是在求費式數


#### 範例 [LeetCode 64 Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/)：
:::info
給定 $M \times N$ 表格，每格有正整數 $a_{r, c} \mid 1 \le r \le M, 1 \le c \le N$
從左上角走到右下角，每次只往右或下走的路徑**最小總和**為何？
:::

例如 $M = N = 3$ 表格如下：
|1|2|3|
|-|-|-|
|6|5|**4**|
|7|8|**9**|
粗體路徑上面的數加總起來比其他路徑加總的和還要小

根據題目，每次能做的**決策**只有兩個：往右、往下
往右，那麼位置 $(i, j-1)$ 就會變成 $(i, j)$
往下，那麼位置 $(i-1, j)$ 就會變成 $(i, j)$

定義 $dp(i, j)$ 為從起點到 $(i, j)$ 的最小和
直接的，$dp(i, j) = \min(dp(i-1, j), dp(i, j-1)) + a_{i, j}$


```cpp
for(int t = 1; t <= max(M, N); t++) dp[0][t] = dp[t][0] = 0; // 對豎軸或橫軸為 0 初始化

for(int i = 1; i <= M; i++)
  for(int j = 1; j <= N; j++)
    dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + a[i][j];
```



#### 範例 [CODEFORCES 1033C Permutation Game](https://codeforces.com/problemset/problem/1033/C)：

注意到，當 token 為 $n$ 那麼**該局**移動者必輸，因為沒有比 $n$ 更大的數
> 可以從邊界觀察出解法切入點

一局先手贏則後手輸，反之亦然；所以看先手狀態即可
先手輸贏只看**移動**決策得知，即**狀態轉移**只看從左來或右來
所以狀態為 $w(i)$ 表示從 $i$ 開局先手是輸是贏

從 $a_i$ 開局，根據題目設 $j \equiv i \mod a_i$ 且 $a_j > a_i$ 且 $w(j)$ 為輸，則 $w(i)$ 為贏
>因為先手 Alice 只要從 $i$ 移動到 $j$，後手 Bob 就無法移動了

```cpp
for(int i = 1; i <= n; i++) {
  scanf("%d", &a[i]);
  idx[a[i]] = i;
}

memset(w, false, sizeof w); // w[n] 已解先手必輸
for(int ai = n-1; ai >= 1; ai--) {
  for(int j = idx[ai]%ai; j <= n; j+=ai)
    if(a[j] > ai && !w[j]) w[idx[ai]] = true;
}
```

`idx` 是記錄每個數字分別對應的位置
注意**計算順序**從 `n-1` 減至 `1`，因為對於欲解 $w(i)$ 時保證比 $a_i$ 大的所有數 $a_j$，$w(j)$ 都已解

#### 範例 [CODEFORCES 429B Working out](https://codeforces.com/contest/429/problem/B)：

沿用[最小路徑和](#範例-LeetCode-64-Minimum-Path-Sum：)的作法，
可將左上與左下到見面點，以及見面點到右上與右下的最大和都求出來
```cpp
for(int i = 1; i <= n; i++)
  for(int j = 1; j <= m; j++)
    LT_mt[i][j] = a[i][j] + max(LT_mt[i-1][j], LT_mt[i][j-1]);

for(int i = n; i >= 1; i--)
  for(int j = m; j >= 1; j--)
    RB_mt[i][j] = a[i][j] + max(RB_mt[i+1][j], RB_mt[i][j+1]);

for(int i = n; i >= 1; i--)
  for(int j = 1; j <= m; j++)
    LB_mt[i][j] = a[i][j] + max(LB_mt[i+1][j], LB_mt[i][j-1]);

for(int i = 1; i <= n; i++)
  for(int j = m; j >= 1; j--)
    RT_mt[i][j] = a[i][j] + max(RT_mt[i-1][j], RT_mt[i][j+1]);
```

注意題意，若是他們只能見面恰好一次，
那麼對於見面的點 $(x, y)$，一方只能**上下經過**，另一方只得**左右經過**
若一方不這麼做，例如 Iahub 走 $(x-1, y) \to (x, y) \to (x, y+1)$
則 Iahubina 從 $(x, y)$ 往上或右分別會碰到 $(x-1, y), (x, y+1)$，就不是恰好碰面一次

於是，將所有可能的見面點都考慮可得：
```cpp
int best = 0;
for(int i = 2; i < n; i++)
  for(int j = 2; j < m; j++)
    best = max({best, LT_mt[i-1][j]+RB_mt[i+1][j] + LB_mt[i][j-1]+RT_mt[i][j+1],
                      LT_mt[i][j-1]+RB_mt[i][j+1] + LB_mt[i+1][j]+RT_mt[i-1][j]});
```


#### 範例 Guitar Fingering：
:::info
給你一把吉他，和 $N$ 個要彈的音符 $a_i$，以及指法難度 $d$
$d(a, x, b, y)$ 表示 $a$ 音符用第 $x$ 根手指彈奏，緊接著再用 $y$ 手指彈奏 $b$ 音符的難度，其中 $1 \le x, y \le L$
要求彈完所有音符所需的**最低**總指法難度
:::
> 一般人類 $L = 5$，因為只有 $5$ 根手指，但假設非人類存在(e.g. 外星人、機器人)


明顯的，定義狀態 $dp(i)$ 表示從第 $i$ 個音符開始彈奏的最小難度
對於考慮使用 $x$ 彈奏 $a_i$，狀態轉移為 $dp(i) = \min\{dp(i+1)+d(a_i, x, a_{i+1}, y) \mid 1 \le x \le L\}$

但等一下，$y$ 是甚麼？這裡並沒有提出該用哪根手指彈下個音符！
於是需要提供**更多資訊**，好讓狀態能滿足問題所求

$dp(i, x)$ 表示從第 $i$ 個音符開始彈奏的最小難度，並用 $x$ 彈奏 $a_i$
那麼狀態轉移就為 $\forall x. dp(i, x) = \min\{dp(i+1, y)+d(a_i, x, a_{i+1}, y) \mid 1 \le y \le L\}$


```cpp
memset(dp, 0x3f, sizeof dp); // 初始為無限大
for(int x = 1; x <= L; x++) dp[N][x] = 0; // 邊界

for(int i = N-1; i >= 1; i--)
  for(int x = 1; x <= L; x++)
    for(int y = 1; y <= L; y++)
      dp[i][x] = min(dp[i][x], dp[i+1][y] + d(a[i], x, a[i+1], y));
      

int best = 0x3f3f3f3f;
for(int x = 1; x <= L; x++) best = min(best, dp[1][x]);
```

#### 範例 Longest Increasing Subsequence (LIS)：
> Longest Increasing Subsequence 中譯為 最長**遞增**子序列

:::info
在給定 $N$ 長度序列 $a$，找到一個子序列，為**嚴格遞增**且長度**最長**。
:::

例如 $a = (\textbf{1}, 4, \textbf{2}, 3, 8, \textbf{3}, \textbf{4}, 1, \textbf{9})$
則 LIS 為 $(1, 2, 3, 4, 9)$ 或 $(1, 2, 3, 8, 9)$

仔細考慮，若某數字在某遞增子序列**後**出現，
且它比此序列的末項還**大**，那麼加入它就能形成更長的遞增子序列！
不過在那之前，這個遞增序列是如何求得的？

通過上述，定義狀態 $S(n)$ 為以第 $n$ 個數為**結尾**的 LIS **長度**，
> 意思是在 $a_1, a_2, .., a_n$ 之間找個一定要包含 $a_n$ 的 LIS

且狀態轉移方程為 $S(n) = \max\{S(i) + 1 \mid i < n, a_i < a_n\}$
>也就是找出所有在 $a_n$ 之前的遞增子序列，選出最長的

若找不到 $a_n$ 之前的遞增子序列，邊界為 $S(n) = 1$


```cpp
for (int n = 1; n <= N; n++) { // N 為 a 的總長度
  S[n] = 1, f[n] = n;
  for (int i = 1; i <= n; i++)
    if (a[i] < a[n] && S[n] < S[i] + 1) {
      S[n] = S[i] + 1;
      f[n] = i; // 紀錄遞增子序列
    }
}
```
複雜度為 $O(N^2)$

其中 `f[n]` 代表遞增子序列中 `a[f[n]]` 下個接 `a[n]`
例如 $a = (\textbf{1}, 4, \textbf{2}, \textbf{3}, \textbf{8}, 3, 4, 1, \textbf{9})$
得出 $f = (\textbf{1}, 1, \textbf{1}, \textbf{3}, \textbf{4}, 3, 4, 8, \textbf{5})$

所以利用 `f` 就能將其中一個 LIS 輸出！
$a_9 = \textbf{9}$ 為末項的 LIS 為例：
$f_9 = 5 \rightarrow a_5 = \textbf{8}$
$f_5 = 4 \rightarrow a_4 = \textbf{3}$
$f_4 = 3 \rightarrow a_3 = \textbf{2}$
$f_3 = 1 \rightarrow a_1 = \textbf{1}$
$f_1 = 1$
> $f_i = i$ 表示 $a_i$ 為欲輸出的 LIS 首項


<!-- 
#### 範例 [ZEROJUDGE d978 最长回文字串](https://zerojudge.tw/ShowProblem?problemid=d978)：

> $O(N)$ 複雜度 [Manacher 演算法](https://segmentfault.com/a/1190000003914228) -->


[^2]: 子子問題就是指從子問題直接分割出來的更小子問題
[^3]: 子問題而非子子問題也非子子..子問題

