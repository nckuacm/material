---
tags: 成大高階競技程式設計 2020, ys
---

2020 Week 4: Search
=
這週要介紹的搜尋方法是以**數列**呈現


# 子集生成

顧名思義，是將集合的所有子集挑出來，**所有可能**子集的集合又稱冪集合[^ps-1]
例如 $\{A, B\}$ 的所有子集為 $\{ \emptyset, \{A\}, \{B\}, \{A, B\} \}$

## 遞迴法

```cpp
void powerset(int dep) {
  if(dep == N) {
    for (int i = 0; i < N; i++) printf("%d ", bit[i]);
    putchar('\n');
    return;
  }
  
  bit[dep] = 1;
  powerset(dep+1);
  bit[dep] = 0;
  powerset(dep+1);
}
```
$N = 3$ 將輸出：
```haskell
1 1 1 
1 1 0 
1 0 1 
1 0 0 
0 1 1 
0 1 0 
0 0 1 
0 0 0 
```

## 二進制法
```cpp
for (int i = 0; i < (1<<N); i++) {
  for (int p = 0; p < N; p++) printf("%d ", bool(i&(1<<p)));
  putchar('\n');
}
```
$N = 3$ 將輸出：
```haskell
0 0 0
1 0 0 
0 1 0 
1 1 0 
0 0 1 
1 0 1 
0 1 1 
1 1 1 
```

在許多場合，我們不一定只有 0 和 1 兩種佔位符 (placeholder)，可能有三種甚至更多
所以遞迴法或許比二進制法還要更為泛用。


# 二分搜尋 (binary search)
>許多文章介紹二分搜沒交代好前提，是採用大量案例佐證二分搜的實作很正確。
>我認為這樣不優，這裡會花較多的篇幅把一些約定俗成的事情交代清楚再給出實作

給定一個 $N$ 長的**單調**[^bs-1]數列，找目標值(target)的位置
當目標有 1 個以上，這時有**兩種位置**： upper bound 與 lower bound

考慮數列 $0, 3, 3, 3, 5, 8, 9$，當目標為 $3$：
- 可能的 lower bound **index** 為 `0`, `1`
- 可能的 upper bound **index** 為 `3`,  `4`, `5`, `6`, `7`, ..

當然，通常會希望 upper bound 與 lower bound 越**緊**越好
所以上面拿的 upper bound 要是 `3`、lower bound 要是 `1` 才好？

## 信仰
在繼續進到實作之前，先討論信仰
upper bound 與 lower bound 真的越緊越好嗎？
以上面例子，有沒有可能 lower bound 取 `0`、upper bound 取 `4` 在**應用**中會比較易用？

### 左閉右開區間 $[l, r)$

給定長度 $N$ 的數列 $A_0, A_1, .. , A_{N-1}$

大部份習慣數數從 $1$ 開始數，因為應用在數個數時，當數到最後一數，剛好就是要求的個數。
但根據[皮亞諾公設](https://en.wikipedia.org/wiki/Natural_number#Peano_axioms)對自然數的定義，**數**應該要以 $0$ 為起點；甚至在許多程式語言實作中，陣列的 index 是從 `0` 開始[^bs-2]。
- 總之，對於這個數列，可以用整數子集 $[0, N)$ 寫下他的 index 範圍
這會比符號 $[0, N-1]$ 來的簡潔一些。

- 而且若是問 $[l, r)$ 的長度為何？ 只需計算 $r-l$
但問 $[l, r]$，就得計算 $r-l+1$

- 對所有自然數除以 $P$[^p_in] 的餘數收集起來，會有 $\{[0], [1], .., [P-1]\}$，表達這件事只要寫 $[0, P)$

- 有時會需要用到**空**區間這個狀態，這時可以 $[l, l)$

- 先前提及的，C++的 STL 中，**迭代器**是以左閉右開區間來實作的。

講這麼多左閉右開區間的好(?)，回到[信仰](#信仰)話題，
普遍實作中會認為 lower bound 為 `1`、upper bound 為 `4` 會比較好。
==(以後 lower bound & upper bound 若沒特別設定，都依此慣例)==

## 關於輸出的約定

若要搜尋的數不在數列中，那應該輸出哪個 index？
普遍實作會輸出當此數插入到這個數列中它**最適合**的位置
何謂最適合？就是要保持著數列仍然**單調**[^monotone]。

對於數列的 index 區間 $[l, r)$， lower & upper bound 都會落在 $[l, r)$ 嗎？
欲搜尋的數如果大於整個數列，它最適合的位置就是 $r$ (落在區間外囉)

而對於所有可能輸出的 bound，可以用空區間表示：
也就是 $[l, l), [l+1, l+1), .., [r, r)$[^bs-3]
雖然意義上都是一個空區間，但看符號還能看出 **表達的 index** 是啥

## linear search 找 lower bound
> 先簡單思考，用最直覺的枚舉方式

來實作找 lower bound 的演算法吧：
假設 $[k, k)$ 就是 lower bound，那麼對於原區間 $[l, r)$，就是設法把 $[l, r)$ 壓縮至 $[k, k)$

簡單的作法就是，一直**遞增左界**：
$[l, r), [l+1, r), ..$ 直到碰到 $[k, r)$
* 最後發現 $A_k$ 就是目標，這樣就能求得 $[k, k)$
* 或著發現 $A_k$ 大於目標，$[k, k)$ 還是解 ([最適合原則](#關於輸出的約定))
```cpp
while (l != r) {
  int m = l;
  if (A[m] >= target) r = l;
  else l = m + 1;
}

return l;
```

另一個從**右界遞減**的演算法：
```cpp
while (l != r) {
  int m = r - 1;
  if (A[m] >= target) r = m;
  else l = r;
}

return l;
```
兩個演算法都使 `l`, `r` 相等且得到 $l = r = k$ 與 $[k, k)$


## binary search 找 lower bound

回到小節標題，二分搜？這名字就是演算法的**動機**，將數列**切成兩份**以做到搜尋：
將上述演算法合併起來會得到
```cpp
/* random lower bound search*/
while (l != r) {
  int m = l + rand()%(r-l); // 切成兩份
  if (A[m] >= target) r = m;
  else l = m + 1;
}

return l;
```
因為不知道 `m` 該遵從哪個演算法，就改成在區間中隨機挑了
這一步非常重要，先思考這樣寫==真的正確嗎？==


而 lower bound 普遍的二分搜實作，就僅把 `m` 改成**均等的**兩份：
```cpp
int m = (l+r)/2;
```
>`m` 其實就是 middle 的縮寫哦


而 upper bound 的二分搜實作也是類似的：
```cpp
/* upper bound */
while (l != r) {
  int m = (l+r)/2;
  if (A[m] <= target) l = m + 1;
  else r = m;
}

return l;
```
二分搜複雜度為 $O(\log_2{(r-l)})$, 其中 $l,r$ 為初始左界右界。

同學就跟著 lower bound 的發明過程，試實作 upper bound 二分搜！
光只會使用 STL 中的`std::lower_bound`, `std::upper_bound` 函數還不夠對付所有問題，因應不同場合常得親自設計 (e.g. 三分搜、隱式數列)


#### 範例 [GCJ Kickstart Round G 2018 A Product Triplets](https://code.google.com/codejam/contest/5374486/dashboard#s=p0)：

:::spoiler
在[第三週教材](https://hackmd.io/@nckuacm/r1ZEy4ar8#%E7%B7%B4%E7%BF%92%EF%BC%9A)的練習中，這題的 small dataset 很輕易的就能用枚舉做出來，但對於 large dataset，$O(N^3)$ 就不夠快了。

仔細想想，雖然對數列排序後不會使 $O(N^3)$ 枚舉算出的答案不同
但排序後，當不考慮乘積為 $0$ 的情況下，兩數積 $A_i \times A_j$ 保證會落在 $j$ 後面，其中 $i < j$
這樣想，二分搜就有武用之地了！

乘積 $0$ 的情況比較特殊一點，但也不難處理：
```cpp
sort(A, A+N);

long long cnt = 0; // cnt := counter
for (int i = 0; i < N-1; i++) {
  for (int j = i+1; j < N; j++) {
    long long t = A[i]*A[j]; // t := target
    if (t || !A[j]) cnt += upper_bound(A+j+1, A+N, t) - lower_bound(A+j+1, A+N, t);
    else cnt += upper_bound(A+i+1, A+j, 0) - lower_bound(A+i+1, A+j, 0);
  }
}
```
此題其實還能讓複雜度從 $O(N^2\log_2{N})$ 降到 $O(N^2)$，不過寫起來會稍微麻煩點。
:::


#### 範例 [TIOJ 1432 骨牌遊戲](https://tioj.ck.tp.edu.tw/problems/1432)：

:::spoiler
```cpp
int const maxn = 1001 + 10;
```

簡單的，採用枚舉的方式去找出哪個"**最大傷害強度**"是可行的
把**可行**的"最大傷害強度"找出來，接著在其之中把最小值輸出就行了

首先做出一個可判定此"最大傷害強度"是否可行的函數：
```cpp
bool check(int strength) {
  int cost = 0, cnt = 0;

  for (int i = 0; i < n; i++) {
    cost += s[i];
    if (cost > strength) cost = s[i], cnt++;
  }

  return cnt <= w;
}
```

接著枚舉一下：
```cpp
int l = *max_element(s, s+n), r = maxn*maxn;
for (int i = l; i <= r; i++)
  if (check(i)) return i;
```
可以發現到，第一個遇到可行的"最大傷害強度" (`strength`) 就是題目要求的最終答案。
但可惜的是，這樣 $O(N^3)$ 複雜度明顯會 TLE
> 其中 $N^2$ 來自於 `r = maxn*maxn`

研究一下 `check` 函數可知，因為每次把 `strength` 條大，會造成 `cnt` 越來越小，所以返回的 `bool` 值形成一個**單調數列**，
於是，可以使用二分搜去改進原本枚舉的做法：
```cpp
while (l != r) {
  int m = (l+r)/2;
  if (check(m)) r = m;
  else l = m+1;
}

return l;
```
複雜度改進成 $O(N\log{N^2}) \approx O(N\log N)$
:::

#### 範例 Longest Increasing Subsequence (LIS)：

:::info
在給定 $N$ 長度序列 $a$，找到一個子序列，為**嚴格遞增**且長度**最長**。
:::
> $a = (\textbf{1}, 4, \textbf{2}, 3, 8, \textbf{3}, \textbf{4}, 1, \textbf{9})$ 則 LIS 為 $(1, 2, 3, 4, 9)$ 或 $(1, 2, 3, 8, 9)$
> 建議複習[第三週教材](https://hackmd.io/@nckuacm/r1ZEy4ar8#%E7%AF%84%E4%BE%8B-Longest-Increasing-Subsequence-LIS%EF%BC%9A)的 LIS 做法


:::spoiler



貪心的看，**當前** LIS 末項數字越小，那麼**越有可能**使得 LIS 繼續變長
> 例如 $(1, 4, 2)$ 有 LIS $(1, 4), (1, 2)$，但之後欲接 $3$，則只有 $(1, 2)$ 能接得 $(1, 2, 3)$

則定義狀態 $S(i, l)$ 表示前綴 $(a_1, a_2, .., a_i)$ 中長度為 $l$ 的 LIS **最小**末項
> 例如 $a$ 的 $i = 5$ 前綴為 $(\textbf{1}, 4, \textbf{2}, 3, 8)$，則 $S(5, 2) = 2$

求解 $S(i, l)$ 需要哪些狀態？
$i$ 前綴相對於 $i-1$ 前綴多了 $a_i$，則
$$
 S(i, l) = 
  \begin{cases} 
   S(i-1, l) &\text{if } a_i \ge S(i-1, l) \\
   a_i &\text{if } a_i < S(i-1, l)
  \end{cases}
$$
而考慮讓 LIS 增長的話，$l$ 相對於 $l-1$ 有
$$
S(i, l) = a_i \text{ if } a_i > S(i-1, l-1) 
$$

綜合上述，狀態轉移方程：
$$
 S(i, l) = 
  \begin{cases} 
   \min(S(i-1, l), a_i) &\text{if } a_i > S(i-1, l-1) \\
   S(i-1, l) &\text{otherwise}
  \end{cases}
$$
> $S(i, l)$ 有可能**無解**， 或許找不到長度為 $l$ 的 LIS

邊界：
$$S(1, 1) = a_1$$

```cpp
S[1][1] = a[1];
int L = 1; // LIS 當前長度，由於現在只解出 i = 1 時的狀態
  
for(int i = 2; i <= N; i++) {
  for(int l = 1; l <= L; l++) {
    if(a[i] > S[i-1][l-1]) S[i][l] = min(S[i-1][l], a[i]);
    else S[i][l] = S[i-1][l];
  }

  if(a[i] > S[i-1][L]) L++, S[i][L] = a[i]; // 考慮讓 LIS 增長
}
```
有了 $S$ 的所有解，從 LIS 長度 $L$，就能推得 LIS 為
$$S(n-L+1, 1), S(n-L+2, 2), \cdots, S(n-1, L-1), S(n, L)$$

```cpp
for(int l = 1; l <= L; l++) printf("%d ", S[n-L+l][l]);
```

---

由於狀態的解是 $i$ 從小到大遞推得到，可用**滾動陣列**壓縮一個維度
```cpp
S[1] = a[1];
int L = 1;

for(int i = 1, j; i <= N; i++) { // j 紀錄哪個 S[l] 被解
  for(int l = 1; l <= L; l++)
    if(a[i] > S[l-1] && a[i] <= S[l]) j = l;

  if(a[i] > S[L]) L++, j = L; // 考慮讓 LIS 增長

  // 紀錄 LIS 的樣子
  p[j] = i;
  if(j > 1) f[i] = p[j-1];
  else f[i] = i;

  S[j] = a[i];
}
```
但因為把前綴 $i$ 這項資訊移除，所以要利用 `f` 紀錄 LIS
其中 `p[j]` 為 `S[j]` 在原本 $a$ 序列中的位置

如第三週所教的操作，可將 LIS 利用 `f` 印出
```cpp
void print_LIS(int i) {
  if(i == f[i]) {
    printf("%d ", a[i]);
    return;
  }

  print_LIS(f[i]);
  printf("%d ", a[i]);
}
```
等一等...
仔細觀察 $S$ 的定義，會發現 $S$ 是**單調**的！
於是每次要用 `a[i]` 解 `s[j]` 時，就使用二分搜！
```cpp
for(int i = 1; i <= N; i++) {
  int j = lower_bound(S+1, S+1+L, a[i]) - S;
  if(j > L) L++;

  p[j] = i;
  if(j > 1) f[i] = p[j-1];
  else f[i] = i;

  S[j] = a[i];
}
```

複雜度為 $O(N \log_2 N)$

#### LIS 練習：
[UVa OJ 10131 Is Bigger Smarter?](https://uva.onlinejudge.org/external/101/10131.pdf)
[UVa OJ 10534 Wavio Sequence](https://uva.onlinejudge.org/external/105/10534.pdf)
[UVa OJ 437 The Tower of Babylon](https://uva.onlinejudge.org/external/4/437.pdf)
[CODEFORCES 1257E The Contest](https://codeforces.com/contest/1257/problem/E)

:::

#### 練習：
[Sprout OJ 48 二元搜尋樹](https://neoj.sprout.tw/problem/48/)
[AIZU Online Judge 0524 星座探し](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=0524)
[CODEFORCES 1118D Coffee and Coursework](https://codeforces.com/contest/1118/problem/D2)
[CODEFORCES 1111C Creative Snap](https://codeforces.com/contest/1111/problem/C)
[GCJ Kickstart Round G 2018 B Combining Classes](https://code.google.com/codejam/contest/5374486/dashboard#s=p1): Small dataset
[CODEFORCES 1263C Everyone is a Winner!](https://codeforces.com/contest/1263/problem/C)
[CODEFORCES 1077D Cutting Out](https://codeforces.com/contest/1077/problem/D)




[^ps-1]: [Wikipedia/ Power set](https://en.wikipedia.org/wiki/Power_set)
[^bs-1]: 單調：可以是升序或降序的數列
[^bs-2]: [Why numbering should start at zero (Dijkstra, Edsger Wybe (May 2, 2008))](http://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html)

[^bs-3]: 為什麼不用左閉右閉區間？其實在這邊兩者各有優點，同學自行思考
[^p_in]: 當然 $P$ 也屬於自然數
[^monotone]: 遞增或遞減