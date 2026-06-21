---
title: 埃式筛法-非素数个数与vector-bool陷阱
date: 2026-06-08 10:00:00
tags: [算法, 数学, 质数筛, 前缀和, 上机考试]
---

## 题目描述

> 题目链接：[N诺/DreamJudge 1305](https://noobdream.com/DreamJudge/Issue/page/1305/)

求 [a, b] 之间的非素数个数。特别规定：1 算作素数。

```
输入:
1 10
2 5

输出:
5    ← 1~10 中非素数: 4,6,8,9,10 → 5个
2    ← 2~5 中非素数: 4 → 1个? 再加... 2,3,5是素数,4非 → 1个? 2,3,5质,4非 → 1个
```

`a ≤ b ≤ 10^7`，多组测试数据。

<!--more-->

## 核心思路

求区间非素数个数 = 前缀和 O(1) 查询：

```
非素数个数[a, b] = sum[b] - sum[a-1]
```

其中 `sum[i]` = 1~i 中非素数的个数，可以用埃式筛预处理。

---

## 埃式筛：O(N log log N)

```cpp
const int MAXN = 1e7;
vector<char> isPrime(MAXN + 1, true);  // ⚠️ 用 char，不用 bool

void init() {
    isPrime[0] = false;
    isPrime[1] = true;          // 本题特殊：1 算素数

    for (int i = 2; i * i <= MAXN; i++) {
        if (isPrime[i]) {
            for (int j = i * i; j <= MAXN; j += i) {
                isPrime[j] = false;   // i 的倍数全部标记为非素数
            }
        }
    }
}
```

### 原理

```
1. 从 2 开始，遇到素数就把它的所有倍数标记为非素数
2. 下一个没被标记的数就是素数，重复操作
3. 只需筛到 √N：因为任何合数 n 必有 ≤ √n 的质因子

筛 2:  4, 6, 8, 10, 12, ...   标记为非素数
筛 3:  9, 12, 15, 18, ...      12 被重复标记（不影响正确性）
筛 5:  25, 35, 45, ...
筛 7:  49, 63, 77, ...

到 √MAXN 停止：因为 i*i > MAXN 时，j 起始就超范围了
```

### 为什么内层从 `i * i` 开始

```
i = 5:
  i*2=10  已被 2 筛过
  i*3=15  已被 3 筛过
  i*4=20  已被 2 筛过
  i*5=25  ← 第一个没被更小质数筛过的倍数是 i*i
```

比 `i * i` 小的倍数已被更小的质数处理过，从 `i * i` 开始避免重复标记。

### 构建前缀和

```cpp
vector<int> sum(MAXN + 1, 0);
for (int i = 1; i <= MAXN; i++) {
    sum[i] = sum[i - 1] + (isPrime[i] ? 0 : 1);
    //                     素数→加0    非素数→加1
}
```

### 查询

```cpp
int a, b;
while (cin >> a >> b) {
    cout << sum[b] - sum[a - 1] << endl;
}
```

---

## ⚠️ `vector<bool>` 会超时！

这是本题最隐蔽的坑。

```cpp
// ❌ 超时
vector<bool> isPrime(MAXN + 1, true);

// ✅ 通过
vector<char> isPrime(MAXN + 1, true);
```

### 为什么

`vector<bool>` 是 C++ 标准中的一个**特化模板**——为了节省空间，它把每个 bool **压缩到 1 bit**。

```
vector<char>: 每个元素 1 字节，10^7 个 = 10MB
vector<bool>: 每个元素 1 位， 10^7 个 = 1.25MB

空间省了，但代价是:
  - 访问单个 bit 需要位运算（移位 + 按位与/或）
  - 不能返回引用（&），返回的是代理对象
  - 每次 isPrime[i] 读写比 char 慢数倍
```

在 10^7 级别的大量访问下，这个常数差距直接导致超时。

### 三种选择

| 写法 | 是否超时 | 说明 |
|------|------|------|
| `vector<bool>` | ❌ TLE | bit 压缩，访问慢 |
| `vector<char>` | ✅ AC | 1 字节，速度正常 |
| `bool isPrime[MAXN+1]` | ✅ AC | 原生数组，最快 |

竞赛中一律用 `vector<char>` 或原生 `bool[]` 做布尔数组，永远不用 `vector<bool>`。

---

## 完整代码模板

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e7;
vector<char> isPrime(MAXN + 1, true);   // ⚠️ char 不用 bool
vector<int> sum(MAXN + 1, 0);

void init() {
    isPrime[0] = false;
    isPrime[1] = true;                  // 本题 1 算素数
    for (int i = 2; i * i <= MAXN; i++) {
        if (isPrime[i]) {
            for (int j = i * i; j <= MAXN; j += i) {
                isPrime[j] = false;
            }
        }
    }
    for (int i = 1; i <= MAXN; i++) {
        sum[i] = sum[i - 1] + (isPrime[i] ? 0 : 1);
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    init();
    int a, b;
    while (cin >> a >> b) {
        cout << sum[b] - sum[a - 1] << endl;
    }
}
```

---

## 总结

| 要点 | 说明 |
|------|------|
| 埃式筛 | 从 2 到 √N，素数的倍数全标记 |
| 内层从 `i*i` 开始 | 更小的倍数已被筛过 |
| 复杂度 | O(N log log N) |
| 区间查询 | 前缀和 `sum[b] - sum[a-1]` |
| **vector\<bool\> 陷阱** | 位压缩导致访问慢，10^7 时超时 |
| 正确做法 | `vector<char>` 或 `bool[]` |
