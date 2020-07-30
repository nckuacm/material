---
tags: 成大高階競技程式設計 2020, ys
---

2020 Week 5: Sorting
=

這週介紹一些基本的排序演算法

學習這些排序演算法，並**不一定**只為了解決排序問題
例如 merge sort 能應用在逆序數對問題，quick sort 的 partition 能快速找出數列中第 $k$ 大的數字，counting sort 或 radix sort 能解決**更複雜**的排序問題

> 而且這些演算法的實現想法，非常值得模仿與吸收



# 排序

將一堆元素由"*給定規則*"排成一[順序](https://en.wikipedia.org/wiki/Order_theory)，稱為排序 (sort)。
例如：
`5, 6, 9, 8, 2` 這五個元素由小到大排為 `2, 5, 6, 8, 9`
`a, bc, aa, ay` 這四個元素由字典順序排為 `a, aa, ay, bc`

如果兩個一樣的元素，在排序後會發生位置互相調換，則此排序法為**不穩定排序** (unstable sort)。

# $O(N^2)$ 排序法
如第一週所講的，演算法的優劣不僅僅只看複雜度，要**看場合**。

## Bubble sort
中文譯作泡泡排序
概念是陣列中每次有個泡泡(?)會從左到右將當前遇到的**最大值**帶至**最右**端

![](https://upload.wikimedia.org/wikipedia/commons/c/c8/Bubble-sort-example-300px.gif)[^bubble_sort_gif]

```cpp
for (int i = n-1; i >= 1; i--)
  for (int j = 0; j < i; j++)
    if (a[j] > a[j+1]) swap(a[j], a[j+1]);
```
其中 `i` 為每次的最右端，
由於這次已將最大值放至最右端，下次的泡泡不需顧及這次的最右端

重覆行為至多 $n-1$ 次，就能達到排序的效果

## Insertion sort
中文譯作插入排序
其精神為每次將選到的元素插入適當的位置
> 大部分的人玩撲克牌應該都這樣排的吧

![](https://upload.wikimedia.org/wikipedia/commons/0/0f/Insertion-sort-example-300px.gif) [^insertion_sort_gif]

```cpp
for (int i = 1; i < n; i++)
  for (int j = i-1; j >= 0 && a[j+1] < a[j]; j--)
    swap(a[j+1], a[j]);
```
`i` 每次選一個數字，接著藉由比較的方式將其往左依序交換至適當的位置

> 由於在輸入規模小的時候有不錯的時間效率，插入排序也作為 STL sort 與 stdlib qsort 的擴充

#### 練習：
[GCJ Qualification Round 2018 Trouble Sort
](https://codejam.withgoogle.com/2018/challenges/00000000000000cb/dashboard/00000000000079cb)

---

# $O(N\log_2N)$ 排序法
下面介紹的兩個是利用分而治之 (D&C) 精神所實作的演算法
> 通常透過 D&C 實作的演算法，其複雜度都看得見 $\log_2$

## Merge sort
中文譯作合併排序，根據以下實作方法有不少意想不到的應用

![](https://i.imgur.com/h39N4SM.png)


```cpp
int a[maxn], b[maxn]; // a := 欲排序陣列， b := 暫存兩子問題
int n; // 欲排序陣列長度

void mergesort(int l, int r) { // 左閉右開區間 [l, r)
  if (r-l <= 1) return; // 當子問題足夠小，不再分割

  // 分割並求解子問題
  int m = (l+r)/2;
  mergesort(l, m);
  mergesort(m, r);

  // 合併子問題
  int p = l, q = m, i = l; // p := 左半邊的 index, q := 右半邊的 index
  copy(a+l, a+r, b+l); // 複製當前問題範圍的陣列
  
  while (i < r) {
    if (q == r  // 右邊取完了
        || (p != m && b[p] <= b[q])) // 左邊還有且左邊比較小
      a[i++] = b[p++];
    else
      a[i++] = b[q++];
  }
}

int main() {
    :
    .
  mergesort(0, n);
   
  return 0;
}
```
![](https://i.imgur.com/MxfXHyI.gif) [^1]
上圖流程跟程式碼的描述是有點出入的 ~~(要怪筆者懶得做圖)~~，不過足以幫助理解過程。

#### 練習：
[TIOJ 1080 A.逆序數對](https://tioj.ck.tp.edu.tw/problems/1080)

## Quick sort
中文譯作快速排序，是十分優越的演算法。

若隨機挑一元素，然後將陣列中比此元素還大的元素們放到它右邊，小於等於的放到左邊
此元素在這樣的操作後，是否能被擺到**最適合 (排序後)** 的位置？這樣的操作稱為 **partition**，並且稱挑的數為 pivot：
```
8, 7, 2, 1, 4, 5, 5, '3', 9, 0
隨機挑 p = 3, index 值為 7

進行 partition：
1, 0, 2, '3', 4, 5, 5, 9, 8, 7
```
p 的位置被換了，那麼來驗證一下這樣的猜想：
```
進行 sort：
0, 1, 2, '3', 4, 5, 5, 7, 8, 9
```
明顯的，p 進行 partition 後，他的位置到了排序後的位置
>這挺直觀的，從小到大排序的概念就正是大數字往右邊，小數字往左邊
```cpp
int a[maxn];

int partition(int l, int r) {
  int p = a[r], ls = l; // p := pivot, ls := less equal
  for (int i = l; i < r; i++)
    if (a[i] <= p) swap(a[i], a[ls++]);
  swap(a[r], a[ls]);

  return ls;
}
```
![](https://i.imgur.com/4MQKD9J.gif) [^2]

那麼對每一個數字做這樣的操作，是不是整個陣列就排序完了(i.e. 選擇的數字都被放到最適合的位置)？

基於這樣的想法，新的演算法就誕生了：
```cpp
void quicksort(int l, int r) { // 左閉右閉區間 [l, r]
  if (l >= r) return;
  int s = partition(l, r);
  quicksort(l, s-1);
  quicksort(s+1, r);
}

int main() {
    :
    .
  quicksort(0, n-1);
  
  return 0;
}
```
![](https://i.imgur.com/oW2QlrB.gif) [^3]

上面介紹的 partition 是取最右端的元素當 pivot。
若每次 partition 將當前其他元素都只到 pivot 的右邊，會使得複雜度從 $O(N\log_2N)$ 退化至 $O(N^2)$。

若改為隨機取一元素當 pivot 呢？複雜度將幾乎不可能退至 $O(N^2)$，隨著資料規模越來越大，在平均來看，不太可能每次都使得上述構造的壞測資發生，有興趣可自行證明期望的複雜度。

---
> 事實上，只要在排序中用到"*比較大小*"的概念，複雜度不可能低於 $O(N\log_2N)$

mergesort 與 quicksort 的程式碼，在每次呼叫函式時區間表示方式不同，因為：
- mergesort 用到"*區間長*"的需求大
- quicksort 中 partition 每次挑最右邊數字當 pivot，需要該 index 值。


#### 練習：
[ZEROJUDGE a233 排序法~~~ 挑戰極限](https://zerojudge.tw/ShowProblem?problemid=a233)

# Counting sort
counting sort，**不經由比較**元素大小就將元素排序好的演算法。

>哦？不經由比較大小呢 Σ(ﾟдﾟ)

陣列中給定的元素將會有個明確的上界，且只適用於整數的排序
```cpp
int n, count[maxn], sorted[maxn]; //maxn 為數的上界, 

for (int i = 0; i < N; i++) cin >> n, count[n]++;

for (int num = 0, i = 0; num < maxn; num++) 
  while (count[num]--) sorted[i++] = num;
```
複雜度是線性 $O($`maxn`$)$
>若每次給的 $N$ 都很小，那 `maxn` 定的太大就顯得慘烈
>所以該演算法還是要看準場合去使用。

#### 練習：
[NCKU OJ 7 藍的競程之旅--魔法藥](https://oj.leo900807.tw/problems/7)
[Sky 62 我最好的朋友](https://pc2.tfcis.org/sky/index.php/problem/view/62)[^4]
[UVa OJ 11462 Age Sort](https://uva.onlinejudge.org/external/114/11462.pdf)[^age_sort]

# Topological sort (拓樸排序)
> 建議將同樣[第五週的 Graph Structures](/@nckuacm/BJ6l2P2UL) 讀過再讀此章，不然會違反拓樸排序規則(?)

有別於一般對數字或是文字做排序，這裡排序的是圖中點與邊的關係

對於排序後點的數列 $v_0,v_1,...,v_n$
有 $\forall (v_i, v_j) \in E. i < j$ 從 $v_j$ 到 $v_i$ **沒有[路徑](/@nckuacm/BJ6l2P2UL#Graph)**
或是說，$\forall (u, v) \in E. u$ 在排序中要在 $v$ 之前
> $(u, v) \in E$ 代表 $u$ 到 $v$ 有條有向邊，但 $v$ 到 $u$ 不一定有邊

![](https://i.imgur.com/ALZIkFx.png)

根據上圖以及拓樸排序規則，有好幾種排序後的數列：
`5, 7, 3, 11, 8, 2, 9, 10` (從左到右，從上到下看)
`3, 5, 7, 8, 11, 2, 9, 10` (優先看小的數字)
`5, 7, 3, 8, 11, 10, 9, 2` (優先看最少的邊)
`7, 5, 11, 3, 10, 8, 9, 2` (優先看大的數字)
`5, 7, 11, 2, 3, 8, 9, 10` (從上到下，從左到右看)
`3, 7, 8, 5, 11, 10, 2, 9` (隨便選)

而==圖是有向無環圖若且唯若圖有拓樸排序==
> 有向無環圖英文為 Directed Acyclic Graph (DAG)

也就是說想找圖中的拓樸排序，前提圖必須是個 DAG，若*發現環則違反條件*

## Kahn's algorithm
根據規則，入度為 $0$ 的點必定可放進**拓樸排序數列**中的**最前端**
> 入度 $0$ 的點不會有邊連向他，所以不會有點比他排更前面 (除其他入度 $0$ 的點)

```cpp
queue<int> q;
for(int i = 0; i < n; i++)
  if(deg[v[i]] == 0) q.push(v[i]), topo.push_back(v[i]);
```

當這些點排好後，接著是被他們所連向的入度為 $1$ 的點，可依序排進拓樸排序數列
> 因為這些入度為 $1$ 的點也沒有其他點能排到他們之前了

```cpp
while(q.size()) {
  int u = q.front(); q.pop();
  
  for(int i = 0; i < n; i++) {
    if(!E[u][v[i]]) continue; // u 到 v[i] 沒有向邊
    if(deg[v[i]] == 1) topo.push_back(v[i]);
  }
}
```

看看上述讓入度為 $1$ 的點加入拓樸排序數列的動機
所以對於還不能排進拓樸排序數列的點 $v$，有若干個點 $u$ 連向 $v$
但是一旦所有 $u$ 都已進入拓樸排序，$v$ 就可以進入拓樸排序數列中
> 怎樣知道目前有幾個 $u$ 已進入拓樸排序？

每次一個連向 $v$ 的 $u$ 進入拓樸排序，$v$ 的入度就可以減 $1$

```cpp
while(q.size()) {
  int u = q.front(); q.pop();
  topo.push_back(u);
  
  for(int i = 0; i < n; i++) {
    if(!E[u][v[i]]) continue;
    
    deg[v[i]]--;
    if(deg[v[i]] == 0) q.push(v[i]);
  }
}
```



## Depth-first search postorder
反過來想，若是某點沒有任何**後繼點**，那麼可以直接把它加入拓樸排序數列的**最尾端**

依循此法，可以逆著把拓樸排序給造出來
也就是當某點的所有後繼點都已在拓樸排序中，那它就能加入拓樸排序數列
> 這些後繼點的其他子孫後繼點都得在拓樸排序數列中

```cpp
void dfs(int u) {
  for(int i = 0; i < n; i++) {
    if(!E[u][v[i]] || vis[v[i]]) continue; // 沒邊或已拜訪過
    vis[v[i]] = true;
    
    dfs(v[i]);
  }
  
  topo.push_back(u);
}
```

因為此排序是逆著造出的，所以要將順序倒過來：
```cpp
reverse(topo.begin(), topo.end());
```

<!-- 
```cpp
bool dfs(int u) {
  stat[u] = true;

  for(int i = 0; i < n; i++) {
    if(!E[u][v[i]]) continue;
    if(stat[v]) return false;

    if(vis[v]) continue;
    vis[v] = true;

    if(!dfs(v)) return false;
    topo.push_back(v);
  }

  stat[u] = false;
  return true;
}
```
其中當 `stat[i]` 為 `true` 時表示此節點的所有子孫還在拜訪中。
因此若 `i` 的子孫都還在拜訪中時，又拜訪到了 `i`，這代表此圖有環，返回 `false`。
要印出拓樸排序應倒著印出最終的 `topo` 陣列。
 -->
 
#### 練習：
:::warning
給定圖，判斷其是否有環
:::
[UVa OJ 124 Following Orders](https://uva.onlinejudge.org/external/1/124.pdf)
[CODEFORCES 510C Fox And Names](http://codeforces.com/problemset/problem/510/C)
[TIOJ 1092 A.跳格子遊戲](https://tioj.ck.tp.edu.tw/problems/1092)

[^1]: [wikipedia/ an example of merge sort](https://en.wikipedia.org/wiki/Merge_sort#/media/File:Merge-sort-example-300px.gif)
[^2]: [wikipedia/ Animated visualization of the quickselect algorithm](https://en.wikipedia.org/wiki/Quickselect#/media/File:Selecting_quickselect_frames.gif)
[^3]: [wikipedia/ Animated visualization of the quicksort algorithm](https://en.wikipedia.org/wiki/Quicksort#/media/File:Sorting_quicksort_anim.gif)
[^4]: 看得出來大家都先嘗試一波 MLE 才開始意識到此題不單純
[^bubble_sort_gif]: [wikipedia/ An example of bubble sort](https://en.wikipedia.org/wiki/Bubble_sort#/media/File:Bubble-sort-example-300px.gif)

[^insertion_sort_gif]: [wikipedia/ A graphical example of insertion sort](https://en.wikipedia.org/wiki/Insertion_sort#/media/File:Insertion-sort-example-300px.gif)

[^age_sort]: UVa 測資頗弱, 陣列開滿用 STL sort 也能過