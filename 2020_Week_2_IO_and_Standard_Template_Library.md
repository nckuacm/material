---
tags: 成大高階競技程式設計 2020, ys
---

2020 Week 2: I/O & Standard Template Library
=

接下來的章節，採用 **C++** 作為主要的程式語言，原因是相對於其他主流的高階語言(e.g. Python, JavaScript...)，C++ 作為編譯語言有相對快的執行速度，且在某些細節操作上更加自由。不過在部分時間或空間限制較寬鬆的題目中採用 Python 解題可以提升解題速度，這部分就交給讀者自行探索。

# Input/Output

在 C++ 中，主要有兩種教派(?)，`std::cin`, `std::cout` 派與 `scanf`, `printf` 派。
在一般情況，兩種輸出入函式可以混用，他們會**同步**使用同一塊緩衝區。
但這樣對於 `cin/cout` 的負擔很大[^sync]，所以<B>==純用==</B> `cin/cout` 的話建議在使用前加入這行：
```cpp
std::ios_base::sync_with_stdio(false);
```

預設中在執行 `cin` 之前 `cout` 會直接 flush (將緩衝區內容輸出到螢幕)，也建議關掉這功能：
> 有大量的 I/O 將會拖慢時間
```cpp
std::cin.tie(NULL);
```

而 C++ 有個換行操作是 `std::endl`，將會強制進行 flush，建議也用 `'\n'` 取代


# 資料型態的範圍

在撰寫程式時要估算資料大小，以選擇適合的資料型別，就算是身經百戰的選手也可能因沒有小心估算範圍而得到 WA 或 RE，以下給出 C++ 中常用型別以及**大約範圍**:

`int x`: $|x| \le 2 \times 10^9$

`long long x`: $|x| \le 9 \times 10^{18}$

`float x`: 共 6 位精確度
- 例如 $123.456789$ 後面的 $789$ 是不準確的
> 純整數若超過 $± 8 \times 10^{6}$ 會不精確

`double x`: 共 15 位精確度
> 純整數若超過 $± 4 \times 10^{15}$ 會不精確

`double` 與 `float` 的精確度取決**整數位數+小數位數**，若同時存過大且對小數精確度要高的數時，可能會出現誤差。

> 對浮點數實作及精確值有興趣的話可翻閱 [IEEE 754](https://zh.wikipedia.org/wiki/IEEE_754) 規範

第 15 週會討論到如何解決因精度不夠而產生的問題

# 演算法的效率
>演算法的設計，得建立在資料結構之上，並評估**時間**與**空間**上的效率

題目給定輸入規模通常很大，2 倍、3 倍、甚至 10 倍的常數倍優化其實不是競賽時考慮的要點。
我們所設計的演算法必須根據輸入規模 $N$ 而定。

* Big $O$ 表示法
$f(N) = O(g(N)) \iff \exists N_0,c>0. \forall N>N_0.|f(N)|\leq c\cdot |g(N)|$

意思是說在 $N$ 足夠大的時候，存在正數 $c$ 使得 $c\cdot |g(N)|$ 大於 $|f(N)|$
或是說 $g(N)$ 趨近無窮的成長速度不比 $f(N)$ 來得慢

假設寫的程式碼長這樣：
```cpp
for(int i = 0; i < n; i++)
  dp[0][i] = dp[i][0] = 0;

for(int i = 1; i <= n; i++)
  for(int j = 1; j <= n; j++)
    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]) + a[i][j];

cout << dp[n][n] << "\n";
```
對應的時間估計函數就是 $f(x)=x^2 + x + 1$
在 $x$ 很大的時候，主要影響整個函數值的大小是平方項，這時可以說 $f(N) = O(N^2)$[^1]


假設輸入規模為 $N$，常見的複雜度有：
$O(1)\leq O(\log_2N)\leq O(N)\leq O(N\log_2N)\leq O(N^k)\leq O(k^N)\leq O(N!)\leq O(N^N)$
> $k$ 為不受輸入規模影響的常數

## 合理的複雜度
> 通常無特別強調，複雜度都暗指**時間**複雜度。但**空間**複雜度的估計也很重要

### 空間複雜度
記憶體空間的用量在各個比賽中規範都不相同
但有些基本的考慮因素，例如 遞迴深度，使用的變數多寡，程式碼長度[^sloc]

### 時間複雜度
普遍上題目都會以秒為單位去做時間限制 (e.g. 3 秒、10 秒)

但通常我們會直接考慮輸入資料的規模與計算出來的複雜度
有個傳統(?)的範圍：$10^7$ 左右
> 這只是大約的估計範圍

假設輸入規模為 $N$，演算法的複雜度為 $O(N^2\log_2 N)$
那麼需要滿足 $N^2\log_2 N \leq 10^7$

具體一點，上面如果 $N=10^5$，那你就得重新設計演算法了。

# 資料結構及 STL 介紹
資料結構 (data structure) 是種對資料的有系統的整理，
資料結構是為了令在其之上運作的演算法能更好的進行操作，或為了提昇演算法的效率，甚至是方便解釋演算法的運作(抽象化)。
每種資料結構通常是針對：**插入**，**刪除**，**修改**，**查詢**[^crud] 的效率及操作上的追求。
> 所以每個容器通常都有**相似**的函式能用，**函式名稱**或許也一樣。

STL 全名 Standard Template Library
由容器 (Containers)、迭代器 (Iterators)、演算法 (Algorithms)、函式 (Functions) 組成。

下面會陸續介紹幾個常用的 STL 裡的容器及函式
> 在使用前請先行測試過，以及了解其複雜度

絕大部分 STL 的東西只要涉及區間的操作，==區間表示一律為左閉右開==[^start_at_zero]

其中 `std::queue`, `std::stack`, `std::list` 先給出簡易實作，再介紹在 STL 裡的用法。

推薦的參考網站： [cplusplus.com](http://www.cplusplus.com/)、[C++ reference](https://en.cppreference.co)


## Iterator
假設容器 `C`，已經裝了一些元素，若想遍歷 `C` 中的所有元素，那要如何做到呢?
有些容器是沒有 index 可以隨機存取的 (例如： `std::list`)，
為處理此問題，STL 為每個容器提供一個成員型別：**迭代器 (Iterator)**

可用"指標"的概念理解迭代器
>實際上，指標算是一種迭代器

假設迭代器 `it`，存取 `it` 所指向的內容，就用 `*it`

迭代器有 3 類：
- 隨機存取迭代器：能夠和整數的加減法，後移 $n$ 項、前移 $n$ 項
- 雙向迭代器：只能做遞增 (`++`)、遞減 (`--`)
- 單向迭代器：只能做遞增 (`++`)

STL 容器有 `begin()` 與 `end()` 成員函式，它倆都會回傳個迭代器。
> 由於左閉右開 `end()` 的迭代器位置落在容器**最尾端迭代器**的後面

## vector

vector 跟一般在使用的陣列很像，最大的特色就是他的儲存空間是動態增減的

### `std::vector`
```cpp
#include <vector>
using std::vector;
```

#### 宣告：
`vector<T> v`: `v` 為空的 vector，且各元素型別為 `T`
`vector<T> v(size_type a)`: `v` 長度為 `a`
`vector<T> v(size_type a, const T& b)`: `v` 會被 `a` 個 `b` 填滿

#### 函式：
`v.size()`: `v` 目前的長度
`v.push_back(T a)`: 在 `v` 的結尾加一個 `a`
`v.pop_back()`: 刪除 `v` 的最末項[^pop_back]
`v.empty()`: 是否 `v` 為空

```cpp
int a;

do {
  cin >> a;
  v.push_back(a);
} while (a);

cout << v.size() << '\n';

v.pop_back();
v.pop_back();

cout << v.size() << '\n';

cout << (v.empty()? "YES" : "NOT EMPTY");
```
```
> 5 2 8 9 1 2 0
< 7
< 5
< NOT EMPTY
```

`v[i]`: `v` 的第 `i` 項
`v.clear()`: 清空 `v`
`v.resize(size_type a)`: 將 `v` 的長度調至 `a`


```cpp
v.resize(4);
v.resize(8, 100);
v.resize(12);

for (int i = 0; i < v.size(); i++) cout << v[i] << ' ';
cout << '\n';


v.clear();
cout << "size = " << v.size();
```
```
< 5 2 8 9 100 100 100 100 0 0 0 0
< size = 0
```

`v.assign(InputIterator l, InputIterator r)`: 將指標 `l` 到 `r` 的內容覆蓋至 `v`
`v.assign(size_type n, const value_type& val)`: 覆蓋 `n` 個 `val ` 至 `v`

```cpp
v.assign(7, 100); // 7 ints with a value of 100
cout << "#1: " << v.size() << '\n';


vector<int>::iterator it = v.begin() + 1;
v.assign(it, v.end()-1);

cout << "#2: " << v.size();
```
```
< #1: 7
< #2: 5
```
## string
字串 (string)，顧名思義是很多個字它們串在一起
例如： "stringisastring"，就是一堆字 's', 't', 'r', 'i', 'n', 'g', 'i', ..., 'g' 串在一起。

字串可像序列一樣表示， 例如長度為 $n$ 的字串 $A$ 為： $A_1, A_2, ... , A_{n-1}, A_n$
字串的問題與序列問題的區別在於字串通常探討的是字元前後**連續**的特性。

### `std::string`
string 的用法很像 `vector<char>`
但多了一些優化，以及功能、且 **`vector<char>` 有的 `string` 都有**


```cpp
#include <string>
using std::string;
```

若 `t` 是 `string` 或 [C style string](http://archive.oreilly.com/oreillyschool/courses/cplusplus1/cplusplus106.html) 則：

#### 運算：
`s = t`: 會將 `t` 複製給 `s`
`s += t`: 在 `s` 的尾端接上 `t`

#### 函式：
`cin >> s`: 輸入字串至 `s`
`cout << s`: 輸出 `s`
`getline(cin, s, char c)`: 輸入字串至 `s`，直到讀到 `c`。`c` 預設為 `'\n'`

```cpp
getline(cin, s, '\n');

t = " and"; // t is a string now
s += t;
s += " carry on";

cout << s;
```

```
> keep calm
< keep calm and carry on
```

`s == t`: 是否 `s` 跟 `t` 相同
`s.c_str()`: 回傳 `s` 的 C style string

```cpp
char cstr[100];
strcpy(cstr, s.c_str());

cstr[8] = 'l';
cstr[9] = '\0';

cout << cstr << '\n';

cout << (cstr == s? "YES" : "NO");

```
```
< keep call
< NO
```

#### 範例 [UVa 101 The Blocks Problem](https://uva.onlinejudge.org/external/1/101.pdf)：

:::spoiler
>挺複雜的題目

將 4 道指令拆解成兩種指令的合成，也就是 `return_above` 以及 `move_to`
- `return_above(a)`: 將特定 block `a` 上方的 blocks 全數返回到原來的位置。
- `move_to(a, b)`: 將 block `a` 及其上所有 blocks 移動到 block `b` 所在的位置最上方。

宣告 `vector<int> pile[maxn];`
以 `pile[p]` 紀錄位置 `p` 上的所有 blocks (從 0 開始由下而上)

```cpp
#include<iostream>
#include<string>
#include<vector>
using namespace std;
const int maxn = 26;

int n, pos[maxn]; // pos := position
vector<int> pile[maxn];

int height(int obj) {
  int i = 0, p = pos[obj];
  for(; pile[p][i] != obj; i++);

  return i + 1;
}

void return_above(int obj) {
  int p = pos[obj], h = height(obj);

  for(int i = h; i < pile[p].size(); i++) {
    int w = pile[p][i]; // w := wood
    pile[w].push_back(w);
    pos[w] = w;
  }

  pile[p].resize(h);
}

void move_to(int a, int b) {
  int ap = pos[a], bp = pos[b];
  int ah = height(a);

  for(int i = ah-1; i < pile[ap].size(); i++) {
    int w = pile[ap][i]; // w := wood
    pile[bp].push_back(w);
    pos[w] = bp;
  }

  pile[ap].resize(ah-1);
}

int main() {
  cin >> n;
  for(int i = 0; i < n; i++) {
    pos[i] = i;
    pile[i].push_back(i);
  }

  int a, b;
  string c1, c2;
  while(cin >> c1 && c1 != "quit") {
    cin >> a >> c2 >> b;

    if(pos[a] == pos[b]) continue;

    if(c1 == "move") return_above(a);
    if(c2 == "onto") return_above(b);
    move_to(a, b);
  }

  // output
  for(int i = 0; i < n; i++) {
    cout << i << ":";
    for(int j = 0; j < pile[i].size(); j++) cout << " " << pile[i][j];
    cout << endl;
  }

  return 0;
}
```
學習重點：
- 長度(或高度) 與最後一位 index 之間的關係 (尤其從 0 開始數)
- 化繁為簡，將重複性高的功能抽象(函式化)出來

:::

#### 練習：
[CODEFORCES 1287A Angry Students](https://codeforces.com/problemset/problem/1287/A)
[CODEFORCES 1301A Three Strings](https://codeforces.com/problemset/problem/1301/A)
[CODEFORCES 1144A Diverse Strings](https://codeforces.com/problemset/problem/1144/A)
[CODEFORCES 1243B1 Character Swap (Easy Version)](https://codeforces.com/problemset/problem/1243/B1)
\*[CODEFORCES 1304B Longest Palindrome](https://codeforces.com/problemset/problem/1304/B)
\*[CODEFORCES 1243B2 Character Swap (Hard Version)](https://codeforces.com/problemset/problem/1243/B2)

## sort
將一堆元素由"*給定規則*"排成一[順序](https://en.wikipedia.org/wiki/Order_theory)，稱為排序 (sort)。
例如：
`5, 6, 9, 8, 2` 這五個元素由小到大排為 `2, 5, 6, 8, 9`
`a, bc, aa, ay` 這四個元素由字典順序排為 `a, aa, ay, bc`

### `std::sort`
STL 的 `sort` 裡有複雜的優化，**預設**將容器元素由==小排至大==

```cpp
#include <algorithm>
using std::sort;
```

假設有如下資料結構：
```cpp
struct T { int val, num; };
vector<T> v;
```

若想依 `num` 對 `v` 做排序，需寫**比較**函數
比較函數是在定義"**小於**"**運算**：
```cpp
bool cmp(const T &a, const T &b)
  { return a.num < b.num; }
```
> 或將小於運算子 (`operator<`) 重載也能達到一樣的效果


將 `cmp` 做為參數：
```cpp
sort(v.begin(), v.end(), cmp);
```
當然也可以直接把匿名函數寫進去：
```cpp
sort(v.begin(), v.end(), [](T a, T b) { return a.num < b.num; });
```

順帶一提，
若把小於行為內部定義為"大於"：
```cpp
[](T a, T b) { return a.num > b.num }
```
則排序為從大至小。


#### 範例 [CODEFORCES 1114B Yet Another Array Partitioning Task](https://codeforces.com/contest/1114/problem/B)：

:::spoiler
> 乍看之下有夠難欸

仔細觀察發現，
$k$ 個切出來的 subarray 中最大的 $m$ 個元素，收集起來恰好就是原 array 中前 $k\times m$ 大的元素
所以將它們加起來就能得到所有 beauty 的總和

```cpp
#include<bits/stdc++.h>
using namespace std;

int const maxn = 2e5 + 100;

int n, m, k, a[maxn];
vector<int> idx;

int main() {
  scanf("%d%d%d", &n, &m, &k);
  for(int i = 0; i < n; i++) {
    scanf("%d", &a[i]);
    idx.push_back(i);
  }

  long long sum = 0;
  sort(idx.begin(), idx.end(), [&](int x, int y) { return a[x] > a[y]; });
  for(int i = 0; i < m*k; i++) sum += a[idx[i]];
  printf("%lld\n", sum);
    :
    .
```

因為要求輸出間隔的 index 位置，所以我們需要原本的 index 位置
將 `idx` 再排序一遍，就能穩穩輸出來了 :+1:

```cpp
    :
    .
  sort(idx.begin(), idx.begin() + m*k);
  for(int i = m-1; i < m*(k-1); i+=m) printf("%d ", idx[i]+1);
  putchar('\n');

  return 0;
}
```
學習重點：
- 凡事先試試 sort，想想有什麼好處，通常狀況只會變好
- `std::sort` 複雜度 $O(N\log_2 N)$，大部份競賽中並不會比 $O(N)$ 壞很多
- `for` 迴圈中大於 $1$ 以上的累加

:::

#### 練習：
[Kick start Round A 2019 A Training](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01/00000000000698d6)
[Kick start Round F 2018 A Common Anagrams](https://code.google.com/codejam/contest/3314486/dashboard#s=p0)
[CODEFORCES 1294B Collecting Packages](https://codeforces.com/problemset/problem/1294/B)
[CODEFORCES 1107C Brutality](https://codeforces.com/contest/1107/problem/C)



## queue
要搭公車之前，都是先*排隊*進入公車站，等到公車到來時，再從公車站出去搭上公車；
由此，公車站可以作為一種資料結構，人可類比為欲儲存之資料

![](https://i.imgur.com/zqr4zuM.png) [^4]
此種資料結構稱作*隊列* (Queue)；擁有**先進先出 (First In, First Out)** 的特性。
例如在郵局要辦事，會先從機器拿號碼紙等叫號，這時候就是用隊列去儲存編號。

下面給出隊列的簡易實作程式碼：

```cpp
int Q[maxn], front_idx = 0, back_idx = 0;

void enqueue(int data)
  { Q[back_idx++] = data; }

int front()
  { return Q[front_idx]; }
  
bool empty()
  { return front_idx == back_idx; }

void dequeue()
  { if(!empty()) front_idx++; }
```

操作多次 `enqueue` 與 `dequeue`，會使得 `front_idx` 最終會碰到陣列邊界，這樣會導致能儲存的資料量變小。

觀察發現，當 `front_idx` 增加，相當於丟掉以前的資料，這時候就有舊的空間空出來了，該如何使用這塊舊空間呢？
直接的，讓 `back_idx` 碰到邊界後回去初始位置就可以了： 這種資料結構稱作*環狀隊列*

還有種變種隊列，叫做*雙端隊列*(**D**ouble **e**nded **que**ue)，他可以從前面或後面 `enqueue` 或 `dequeue`。

### `std::queue`

```cpp
#include <queue>
using std::queue;
```

#### 宣告：
`queue<T> q`: `q` 為空的隊列，且各元素型別為 `T`

#### 函式：
`q.front()`: 第一個進入 `q` 的元素
`q.back()`: 最後一個進入 `q` 的元素
`q.push(T a)`: 將 `a` 加入 `q` 中 (enqueue)
`q.pop()`: 將第一個進入 `q` 的元素移除 (dequeue)
`q.empty(), q.size()`: `q` 當然也有這兩個函式


```cpp
queue<int> q; // []

q.push(1); // [1]
q.push(2); // [1, 2]

cout << q.front() << ' '; // 1 

q.push(3); // [1, 2, 3]

cout << q.front() << ' '; // 1 

q.pop(); // [2, 3]
q.pop(); // [3]

cout << q.front(); // 3

q.pop(); // []
```
```
< 1 1 3
```

#### 練習：
[UVa OJ 10935 Throwing cards away I](https://uva.onlinejudge.org/external/109/10935.pdf)
[UVa OJ 12100 Printer Queue](https://uva.onlinejudge.org/external/121/12100.pdf)
[Zero Judge c700 壞掉的隊列(queue)](https://zerojudge.tw/ShowProblem?problemid=c700) (建議讀完 stack 再來練習本題)



## stack

考慮將鬆餅每次只放一片疊到盤子上，則最後放上去的鬆餅將會是最上層的鬆餅，如果要吃會依序從最上層開始拿；
由此，盤子可以作為一種資料結構，鬆餅可類比為欲儲存之資料

![](https://i.imgur.com/iPdRCnM.png)[^5]

此種鬆餅(資料)的放法，拿法，是種稱做*堆疊* (stack) 的資料結構；有**後進先出 (Last In, First Out)** 的特性，

下面給出堆疊的簡易實作程式碼：
```cpp
int S[maxn], top_idx = -1;

void push(int data)
  { S[++top_idx] = data; }

int top()
  { return S[top_idx]; }
  
bool empty()
  { return top_idx == -1; }

void pop()
  { if(!empty()) top_idx--; }
```

相較於隊列的簡易實作版，堆疊不用擔心**已用過的**空間會永遠用不到
### `std::stack`
```cpp
#include <stack>
using std::stack;
```

#### 宣告：
`stack<T> st`: `st` 為空的堆疊，且各元素型別為 `T`

#### 函式：
`st.top()`: 存取最後一個進入 `st` 的元素
`st.push(T a)`: 將 `a` 加入 `st` 中
`st.pop()`: 將最後一個進入 `st` 的元素移除

```cpp
stack<int> st; // []

st.push(1); // [1]
st.push(2); // [1, 2]
st.push(3); // [1, 2, 3]

cout << st.top() << ' '; // 3

st.pop(); // [1, 2]

cout << st.top() << ' '; // 2

st.pop(); // [1]

cout << st.top(); // 1

st.pop(); // []
```
```
< 3 2 1
```


#### 練習：
[UVa OJ 514 Rails](https://uva.onlinejudge.org/external/5/514.pdf)
[UVa OJ 673 Parentheses Balance](https://uva.onlinejudge.org/external/6/673.pdf)
[UVa OJ 271 Simply Syntax](https://uva.onlinejudge.org/external/2/271.pdf)
[UVa OJ 11234 Expressions](https://uva.onlinejudge.org/external/112/11234.pdf)



## list
玩撲克牌遊戲時，常會有將牌拿起與將牌插到某個位置的動作
支援這兩個行為的資料結構稱為"連結串列 (Linked list)"[^doubly_linked_list]

![](https://i.imgur.com/6VeOzJd.png)[^6]

連結串列比較複雜點，需要造出兩種結構：
1. 定義一個結構叫做"節點(node)"，它可儲存**資料**及下個節點的**位置**
```cpp
struct node {
  int data;
  node *prev, *next;
} *list[maxn];
```
2. 當資料都放進節點後，要將節點們串起來
  其中 `list[0]` 不當一般資料 (`.data`) 使用，它用來**指向**連接串列的頭
```cpp
for (int i = 0; i < N; i++) {
  node *p = new node;

  if(i) { // 0 or others
    list[i-1]->next = list[i] = p;
    list[i]->prev = list[i-1];
  } else list[0] = p;
}
  
list[0]->prev = list[N-1];
list[N-1]->next = list[0];
```
在最後將 `list[0]`(head pointer) 與串列尾部連接起來，是為了下面兩個函式寫法上的方便(?

當擁有這樣的結構，拔除節點和插入節點只需 $O(1)$：
```cpp
void remove(int idx) {
  list[idx]->prev->next = list[idx]->next;
  list[idx]->next->prev = list[idx]->prev;
  list[idx]->next = list[idx]->prev = NULL;
}

void insert(int idx1, int idx2) {
  list[idx2]->next = list[idx1]->next;
  list[idx1]->next = list[idx2];
  list[idx2]->next->prev = list[idx2];
  list[idx2]->prev = list[idx1];
}
```

> 比起 queue 和 stack，從頭刻 linked list 真的挺累的
> 因應題目需求，有時候還是得自己刻


### `std::list`
```cpp
#include <list>
using std::list;
```

#### 宣告：
`list<T> ls`: `ls` 空的連接串列，且各元素型別為 `T`

#### 函式：
`insert`, `erase` 將回傳操作過後的迭代器位置 (最好自行實驗)

`ls.insert(iterator it, T a)`: 在 `it` 位置**前**插入 `a`
`ls.insert(iterator it, size_type n, T a)`: 在 `it` 位置**前**插入 `n` 個 `a`
`ls.erase(iterator it)`: 把 `it` 位置上的元素移除
`ls.erase(iterator l, iterator r)`: 把 $[l, r)$ 位置上的元素移除

```cpp
list<int> ls;

ls.push_back(1);  // 1
ls.push_back(2);  // 1 <-> 2
ls.push_front(3); // 3 <-> 1 <-> 2

cout << ls.front() << ' '; // 3
cout << ls.back();  // 2
```
```
< 3 2
```

```cpp
for (list<int>::iterator i = ls.begin(); i != ls.end(); i++)
  cout << *i << ' ';
```
```
< 3 1 2
```

```cpp
list<int>::iterator it = ls.begin();
++it; ++it; // it points now to position 2 (start from 0)

ls.insert(it, 3, 100); // 3 <-> 1 <-> 100 <-> 100 <-> 100 <-> 2

for (list<int>::iterator i = ls.begin(); i != ls.end(); i++)
  cout << *i << ' ';
```
```
< 3 1 100 100 100 2
```
此時 `it` 將指向第 $5$ 個位置 ($2 + 3$)
其中：
$2$ 是還未 `ls.insert(it, 3, 100)` 前的位置
$+3$ 因為插入了 $3$ 個 `100`

```cpp
--it; --it;
ls.erase(it, ls.end());

for (list<int>::iterator i = ls.begin(); i != ls.end(); i++)
  cout << *i << ' ';
```
```
< 3 1 100
```

#### 練習：
[UVa OJ 11988 Broken Keyboard (a.k.a. Beiju Text)](https://uva.onlinejudge.org/external/119/11988.pdf)
[CODEFORCES 1154E Two Teams](https://codeforces.com/contest/1154/problem/E)
[UVa OJ 245 Uncompress](https://uva.onlinejudge.org/external/2/245.pdf)
[Sprout OJ 21 陸行鳥大賽車](https://neoj.sprout.tw/problem/21/)
[Sprout OJ 20 中國人排隊問題](https://neoj.sprout.tw/problem/20/)


## inserter
介紹 `std::set` 之前，得先介紹一些集合操作
inserter 顧名思義，就是把一些東西插入到某個地方

```cpp
#include <iostream>     // std::cout
#include <iterator>     // std::inserter
#include <list>         // std::list
#include <algorithm>    // std::copy
using namespace std;

int main () {
  list<int> foo, bar;
  for (int i = 1; i <= 5; i++)
    foo.push_back(i), bar.push_back(i*10);

  list<int>::iterator it = foo.begin();
  advance(it, 3);

  copy(bar.begin(), bar.end(), inserter(foo, it));

  for (auto it : foo) cout << it << ' ';

  return 0;
}
```
```
< 1 2 3 10 20 30 40 50 4 5
```

## set_union
union 就是將兩個集合取聯集

```cpp
#include <iostream>     // std::cout
#include <algorithm>    // std::set_union, std::sort
#include <vector>       // std::vector
using namespace std;

int main () {
  int first[] = { 5, 10, 15, 20, 25 };
  int second[] = { 50, 40, 30, 20, 10 };
  vector<int> V(10);                      // 0  0  0  0  0  0  0  0  0  0
  vector<int>::iterator it;

  sort(first, first+5);     //  5 10 15 20 25
  sort(second, second+5);   // 10 20 30 40 50

  it = set_union(first, first+5, second, second+5, V.begin());
                                               // 5 10 15 20 25 30 40 50  0  0
  V.resize(it - V.begin());                    // 5 10 15 20 25 30 40 50

  for (auto it : V) cout << it << ' ';

  return 0;
}
```
```
< 5 10 15 20 25 30 40 50
```

## set_intersection
取交集

```cpp
#include <iostream>     // std::cout
#include <algorithm>    // std::set_intersection, std::sort
#include <vector>       // std::vector
using namespace std;

int main () {
  int first[] = { 5, 10, 15, 20, 25 };
  int second[] = { 50, 40, 30, 20, 10 };
  vector<int> V(10);                      // 0  0  0  0  0  0  0  0  0  0
  vector<int>::iterator it;

  sort(first, first+5);     //  5 10 15 20 25
  sort(second, second+5);   // 10 20 30 40 50

  it = set_intersection(first, first+5, second, second+5, V.begin());
                                               // 10 20 0  0  0  0  0  0  0  0
  V.resize(it - V.begin());                    // 10 20

  for (auto it : V) cout << it << ' ';

  return 0;
}
```
```
< 10 20
```

## set_difference
差集
```cpp
#include <iostream>     // std::cout
#include <algorithm>    // std::set_difference, std::sort
#include <vector>       // std::vector
using namespace std;

int main () {
  int first[] = { 5, 10, 15, 20, 25 };
  int second[] = { 50, 40, 30, 20, 10 };
  vector<int> V(10);                      // 0  0  0  0  0  0  0  0  0  0
  vector<int>::iterator it;

  sort(first, first+5);     //  5 10 15 20 25
  sort(second, second+5);   // 10 20 30 40 50

  it = set_difference(first, first+5, second, second+5, V.begin());
                                               //  5 15 25  0  0  0  0  0  0  0
  V.resize(it - V.begin());                    //  5 15 25

  for (auto it : V) cout << it << ' ';

  return 0;
}
```

```
< 5 15 25
```

## set
集合(set) 是非常基礎的數學結構，對於資料結構也一樣基礎
特別注意：集合中的**元素不會重複**

### `std::set` 
`std::set` 是使用紅黑樹實作的，插入、刪除、查詢都為 $O(\log_2 N)$，其中元素也不會重複。
且元素們在 `set` 容器中保持**排序的**(可比大小)
> 也就是說迭代器位置會優先從元素值小的排到大的

```cpp
#include <set>
using std::set;
```

#### 宣告：

`set<T> S`: 空的集合

#### 函式：
`S.insert(T a)`: 插入元素 `a`
`S.erase(iterator x)`: ，移除 $x$ 位置上的元素
`S.erase(iterator l, iterator r)`: 把 $[l, r)$ 位置上的元素移除
`S.erase(T a)`: 移除元素 `a`
`S.find(T a)`: 回傳指向元素 `a` 的迭代器；若 `a` 不存在，則回傳 `S.end()`
`S.count(T a)`: 元素 `a` 是否存在

> 在 C++11 後，erase 會回傳最後被移除元素下個元素的迭代器

#### 範例 [CODEFORCES 1157A Reachable Numbers](https://codeforces.com/contest/1157/problem/A)：

:::spoiler
持續使用函數 $f$，除以 1 次 $10$ 前最多只會加 9 次 $1$，
於是對於數字 $n$ 最多只會使用 $O(\log n)$ 次 $f$

在這過程中若遇到**已經見過**的數字，表示不會再出現不曾見過的數字了
```cpp
set<int> S;
while(!S.count(n)) {
  S.insert(n);

  n++;
  while(n % 10 == 0) n /= 10;
}

printf("%d\n", S.size());
```
:::

#### 練習：
[CODEFORCES 1238B Kill \`Em All](https://codeforces.com/contest/1238/problem/B)
[CODEFORCES 1253B Silly Mistake](https://codeforces.com/contest/1253/problem/B)

## key-value pairs (KVP)
鍵(key)值(value)對(pair) 是非常實用的資料結構

例如想表達每個人 $p_i$ 的身高 $h_i$ 可以寫：
$(\text{'john'}, 167), (\text{'aria'}, 145), (\text{'bob'}, 170)$

又或是表達有理數 $220\over 440$ 的分子分母：
$(220, 440), (22, 44), (2, 4), (1, 2)$

### `std::map`
`std::map` 的插入、刪除或查詢為 $O(\log_2 N)$
`map` 的每個元素結構為 `std::pair<T1, T2>` 構成的 KVP
且元素們在 `map` 容器中保持**排序的**

```cpp
#include <map>
using std::map;
```

#### 宣告：
`map<T1, T2> M`: 空的 KVP 集合，鍵型態 `T1`，值型態 `T2`

#### 函式：
`M[k] = a`: 修改鍵 `k` 對應的值為 `a`
`M[k]`: 存取鍵 `k` 對應的值
`M.insert(pair<T1, T2> P)`: 插入一個鍵值對 `P`

#### 範例 [CODEFORCES 1133C Balanced Team](https://codeforces.com/contest/1133/problem/C)：
:::spoiler
每次將 skill 為 $a_i$ student 以上相差 $5$ 以內的 skill 都記錄下來就行

例如有 skill $1$、 $6$ 及 $4$ 的 student 存在
那麼首先記錄 $1$：
```
skill: 1 2 3 4 5 6 7 8 9 10 11
count: 1 1 1 1 1 1 0 0 0  0  0
```
接著 $4$
```
skill: 1 2 3 4 5 6 7 8 9 10 11
count: 1 1 1 2 2 2 1 1 1  0  0
```
然後 $6$
```
skill: 1 2 3 4 5 6 7 8 9 10 11
count: 1 1 1 2 2 3 2 2 2  1  1
```


所以對於 skill $a_i$：
```cpp
for(int i = 0; i < n; i++) {
  scanf("%d", &a[i]);
  for(int k = 0; k <= 5; k++) cnt[a[i]+k]++;
}
```

接著找出 $a_i$ **以下**相差為 $5$ 的 skill 數中的最大值：
```cpp
int best = 1;
for(int i = 0; i < n; i++) best = max(best, cnt[a[i]]);
```

且慢！
注意到一個限制： $1 \leq a_i \leq 10^9$
若把 `cnt` 陣列的空間開得這麼大，明顯的會 Runtime error[^CE]

所以把陣列改成 `map<int, int>`，就能避免空間上的龐大需求 :+1: 

:::


#### 範例 [CODEFORCES 1255C League of Leesins](https://codeforces.com/contest/1255/problem/C)：

:::spoiler

觀察題目中的例子 $(1,4,2),(4,2,3),(2,3,5)$
應該可以很自然地推得數列 $1, 4, 2, 3, 5$

原因是當**起頭確定**時，可得知下兩個數字是啥
確定是 $1$，那一定知道後面是 $4, 2$
確定是 $5$，那一定知道後面是 $2, 3$
並且若知道下兩個數字，又能再推得一個數字
$4, 2$ 的話能推得 $3$，因為有 $(4, 2, 3)$
依此類推，能夠把所有數字都算出來

且起頭的數字是統計所有 triple 中只**出現 1 次**的數字，如 $1, 5$
第二個數字則是統計所有 triple 中只**出現 2 次**的數字，如 $4, 3$

```cpp
#include<bits/stdc++.h>
using namespace std;

int const maxn = 1e5 + 10;

int n, x, y, z;

int main()
{
  scanf("%d", &n);

  int cnt[maxn] = {};
  map<pair<int, int>, vector<int>> tp; // tp := triple

  for(int i = 0; i < n-2; i++) {
    scanf("%d%d%d", &x, &y, &z);

    tp[{x, y}].push_back(z);
    tp[{y, x}].push_back(z);
    tp[{y, z}].push_back(x);
    tp[{z, y}].push_back(x);
    tp[{z, x}].push_back(y);
    tp[{x, z}].push_back(y);

    cnt[x]++;
    cnt[y]++;
    cnt[z]++;
  }

  int p1, p2;
  for(int i = 1; i <= n; i++) if(cnt[i] == 1) p1 = i;
  for(int i = 1; i <= n; i++) if(cnt[i] == 2 && tp.count({p1, i})) p2 = i;

  bool vis[maxn] = {};
  for(int i = 0; i < n; i++) {
    printf("%d ", p1);
    vis[p1] = true;

    if(i == n-1) break;

    auto v = tp[{p1, p2}];
    p1 = p2;
    p2 = vis[v[0]]? v[1] : v[0];
  }

  return 0;
}
```
:::

#### 練習：
[UVa OJ 11991 Easy Problem from Rujia Liu?](https://uva.onlinejudge.org/external/119/11991.pdf)
\*[CODEFORCES 1257C Dominated Subarray](https://codeforces.com/problemset/problem/1257/C)
\*\*[CODEFORCES 1230D Marcin and Training Camp](https://codeforces.com/contest/1230/problem/D)


## Priority queue
優先隊列 (priority queue) 是隊列 (queue) 的一個變種
- dequeue 會先選容器中**優先度最大**的元素
- front 也先選容器中**優先度最大**的元素

### `std::priority_queue`
每次進出的時間複雜度都為 $O(\log_2 N)$

> 對於數值型態的元素，數值越大優先度越大

```cpp
#include <queue>
using std::priority_queue;
```

#### 宣告：
`priority_queue<T> pq`: `pq` 為空的優先隊列，且各元素型別為 `T`

#### 函式：
`pq.top()`: `pq` 中優先度最大的元素
`pq.push(T a)`: 將 `a` 加入 `pq` 中 (enqueue)
`pq.pop()`: 將`pq` 中優先度最大的一個元素移除 (dequeue)



對於未定義**排序**關係的元素(型態)，==要先定義**小於**運算子==，例如：
```cpp
struct XXX {
  int code, weight;
  bool operator<(const XXX &lhs) const
    { return weight < lhs.weight; }
};
```

```cpp
priority_queue<XXX> PQ; // []

PQ.push({2, 2147483647}) // [{2, 2147483647}]
PQ.push({3, -1}) // [{2, 2147483647}, {3, -1}]
PQ.push({2, 0}) // [{2, 2147483647}, {2, 0}, {3, -1}]

cout << PQ.top().weight << endl; // {2, 2147483647}

PQ.pop(); // [{2, 0}, {3, -1}]

puts(PQ.empty()? "yes" : "no"); // no
```
```
< 2147483647
< no
```

`priority_queue` 也能定義比較函數，可以自行實驗

#### 範例 [GCJ Kickstart Round E 2018 B Milk Tea](https://code.google.com/codejam/contest/4394486/dashboard#s=p1)：

:::spoiler
簡單的，將所有 binary string (preferences) 都產出，並把 string 對應的抱怨 (complaint) 算出
接著找出最小抱怨且不屬於 forbidden types 的 binary string 就好。

用 `span` 保存將產出的 binary string：
```cpp
priority_queue<pair<int, string> > span;
span.emplace(0, "");
```

以 `seed` 作為基礎，再將 string 加上 `"0"` 或 `"1"`
並每次將當前的抱怨值算出後保存：
```cpp
for (int i = 0; i < P; i++) {
  priority_queue<pair<int, string> > seed;
  swap(seed, span);

  while (!seed.empty()) {
    int cp; // complaints
    string bin; // binary string
    tie(cp, bin) = seed.top(); seed.pop();

    span.emplace(cp+one[i],  bin+"0");
    span.emplace(cp+zero[i], bin+"1");
  }
}
```
其中 `one[i]` 與 `zero[i]` 為 Shakti 的朋友們對於第 $i$ 個 option 的加總 (選項只有 $0$ 和 $1$)
但光是產出所有 binary string 就足夠讓複雜度導致 TLE。

在上述==過程中，若把某些 string 去除掉，就能使效率增加不少==
合理的，答案只要最小抱怨值的 binary string，那在過程中移除抱怨值較大的 string 就行了：
```cpp
while (span.size() > M+1) span.pop();
```
因為共有 $M$ 個 forbidden type，所以至少要保存 $M+1$ 個 binary string

最後把 string 為 forbidden type (`ban`) 的濾掉，留下抱怨值最小的就好：
```cpp
int ans;
while (!span.empty()) {
  tie(cp, bin) = span.top(); span.pop();
  if (!ban.count(bin)) ans = cp;
}
```
:::

#### 練習：
[ICPC 3135 Argus](https://icpcarchive.ecs.baylor.edu/external/31/3135.pdf)
[UVa OJ 11997 K Smallest Sums](https://uva.onlinejudge.org/external/119/11997.pdf)





[^sloc]: 有時會想試試不靠儲存空間，使用大量的判定輸出，此時程式碼長度的估算也得考量

[^1]: 注意這邊的符號($=$)與數學上的用法"[等價](https://en.wikipedia.org/wiki/Equivalence_relation)"意義不同

[^pop_back]: 若 v 為空，會發生無法預期的結果
[^start_at_zero]: [Why numbering should start at zero](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html)

[^crud]: [wikipedia/ 建立、讀取、更新、刪除](https://zh.wikipedia.org/wiki/%E5%BB%BA%E7%AB%8B%E3%80%81%E8%AE%80%E5%8F%96%E3%80%81%E6%9B%B4%E6%96%B0%E3%80%81%E5%88%AA%E9%99%A4)

[^doubly_linked_list]: 這裡介紹的是更為泛用的 doubly linked list 而非單純的 singly linked list

[^4]: [wikipedia/ Representation of a FIFO (first in, first out) queue](https://en.wikipedia.org/wiki/Queue_(abstract_data_type)#/media/File:Data_Queue.svg)
[^5]: [wikipedia/ Simple representation of a stack runtime with push and pop operations](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)#/media/File:Lifo_stack.png)
[^6]: [wikimedia/ DoubleLinkedListHsrw](https://commons.wikimedia.org/wiki/File:DoubleLinkedListHsrw.png)

[^CE]: 不過實驗證明，提交上 Codeforces 會拿到 Compilation error

[^sync]: [GNU/ Interacting with C/ Performance](https://gcc.gnu.org/onlinedocs/libstdc++/manual/io_and_c.html#std.io.c.sync)
