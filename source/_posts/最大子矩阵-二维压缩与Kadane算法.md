---
title: 最大子矩阵-二维压缩与Kadane算法
date: 2026-05-25 10:00:00
tags: [算法, 矩阵, 前缀和, 动态规划, 上机考试]
---

## 题目描述

> 题目链接：[N诺/DreamJudge 1245](https://noobdream.com/DreamJudge/Issue/page/1245/)

给定 N×N 矩阵（元素范围 [-127, 127]），求元素和最大的非空子矩阵，输出最大和。

和上篇"最小面积子矩阵"思路几乎一样：**枚举行范围 → 列前缀和压缩成一维 → 在一维数组上求解**。区别在于压缩后的问题不同：上篇是"和 ≥ K 的最短子数组"，本篇是"最大子数组和"——用 Kadane 算法解决。

<!--more-->

## 整体流程

```
枚举行的上下边界 → 压缩成一维数组 → Kadane 算法求最大子数组和
      ↑                                      ↑
   O(N²) 种行范围                         O(N) per row pair
```

总复杂度 $O(N^3)$，对于 N≤100 完全够用。

## 第一步：列前缀和 + 压缩（和上篇一样）

```cpp
// 列前缀和
for (int i = 1; i <= n; i++)
    for (int j = 1; j <= n; j++)
        colsum[i][j] = colsum[i - 1][j] + mtx[i][j];

// 枚举行范围，压缩成一维
for (int top = 1; top <= n; top++) {
    for (int bot = top; bot <= n; bot++) {
        vector<int> colsumInrange(n + 1);
        for (int i = 1; i <= n; i++) {
            colsumInrange[i] = colsum[bot][i] - colsum[top - 1][i];
        }
        // 问题变成：求 colsumInrange 的最大子数组和
    }
}
```

---

## 第二步：Kadane 算法（重点）

Kadane 算法求**一维数组的最大子数组和**，$O(N)$ 时间，$O(1)$ 空间。

### 问题

`arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`，找和最大的连续子数组。答案是 `[4, -1, 2, 1]`，和为 6。

### 核心思想

遍历数组，思考一个问题：

> **以 arr[i] 结尾的最大子数组和**是多少？

以每个位置结尾的最大值中，最大的那个就是答案。

```
arr:   -2   1  -3   4  -1   2   1  -5   4

以 i 结尾的最优:
i=0:   -2              ← 只能是自己
i=1:    1              ← -2<0，不接，自己重新开始
i=2:   -2 (1-3)        ← 1>0，接上：1+(-3)=-2
i=3:    4              ← -2<0，丢弃，从4重新开始
i=4:    3 (4-1)        ← 4>0，接上
i=5:    5 (4-1+2)      ← 3>0，接上
i=6:    6 (4-1+2+1)    ← 5>0，接上 ← 最大
i=7:    1 (6-5)        ← 6>0，接上
i=8:    5 (1+4)        ← 1>0，接上
```

规律一句话：**前面积累 > 0 就接上，≤ 0 就重新开始。**

### 代码

```cpp
int cursum = 0;
int best = INT_MIN;  // ⚠️ 元素可能全为负，不能初始化为 0

for (int i = 1; i <= n; i++) {
    if (cursum > 0) {
        cursum += arr[i];       // 前面有贡献，接上
    } else {
        cursum = arr[i];        // 前面是负数，不接，重新开始
    }
    best = max(best, cursum);   // 每个位置都更新答案
}
```

五行代码，经典又简洁。

### 为什么 `cursum > 0` 时接上？

因为 Kadane 关注的是"以当前元素结尾"的最大子数组和：

- 前面的累加和 > 0 → 加上一定比单独取当前值大
- 前面的累加和 ≤ 0 → 加上只会让结果变小，抛掉

### 另一种等价写法

```cpp
cursum = max(cursum + arr[i], arr[i]);
best = max(best, cursum);
```

更短，但"接上 or 重新开始"的逻辑不如 if 版直观。

### ⚠️ 为什么 best 初始化为 INT_MIN

如果数组全是负数，比如 `[-3, -5, -2]`：

```
cursum: -3 → -5 → -2
best:   -3 → -3 → -2   ← 最终答案是 -2
```

如果 `best` 初始化为 0，答案会错误地输出 0。用 `INT_MIN` 保证第一个元素一定能更新 best。

---

## 第三步：组合

```cpp
int mmax = INT_MIN;
for (int top = 1; top <= n; top++) {
    for (int bot = top; bot <= n; bot++) {
        // 压缩
        vector<int> colsumInrange(n + 1);
        for (int i = 1; i <= n; i++) {
            colsumInrange[i] = colsum[bot][i] - colsum[top - 1][i];
        }

        // Kadane
        int cursum = 0;
        int best = INT_MIN;
        for (int i = 1; i <= n; i++) {
            if (cursum > 0) {
                cursum += colsumInrange[i];
            } else {
                cursum = colsumInrange[i];
            }
            best = max(best, cursum);
        }

        mmax = max(mmax, best);
    }
}
cout << mmax << endl;
```

---

## 与上篇文章的对比

| | 最小面积子矩阵 | 最大子矩阵（本篇） |
|------|------|------|
| 压缩方式 | 列前缀和 | 列前缀和（一样） |
| 一维问题 | 和 ≥ K 的最短子数组 | 最大子数组和 |
| 一维算法 | 滑动窗口 | **Kadane 算法** |
| 哨兵初始值 | `INT_MAX` | `INT_MIN` |
| 复杂度 | $O(N^2 M)$ | $O(N^3)$ |

---

## 完整代码模板

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    while (cin >> n) {
        vector<vector<int>> mtx(n + 1, vector<int>(n + 1));
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                cin >> mtx[i][j];
            }
        }

        vector<vector<int>> colsum(n + 1, vector<int>(n + 1));
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                colsum[i][j] = colsum[i - 1][j] + mtx[i][j];
            }
        }

        int mmax = INT_MIN;
        for (int top = 1; top <= n; top++) {
            for (int bot = top; bot <= n; bot++) {
                vector<int> colsumInrange(n + 1);
                for (int i = 1; i <= n; i++) {
                    colsumInrange[i] = colsum[bot][i] - colsum[top - 1][i];
                }

                int cursum = 0;
                int best = INT_MIN;
                for (int i = 1; i <= n; i++) {
                    if (cursum > 0) {
                        cursum += colsumInrange[i];
                    } else {
                        cursum = colsumInrange[i];
                    }
                    best = max(best, cursum);
                }
                mmax = max(mmax, best);
            }
        }
        cout << mmax << endl;
    }
}
```

---

## 总结

| 要点 | 说明 |
|------|------|
| 二维压缩 | 列前缀和 + 枚举行范围，和上篇相同 |
| Kadane 核心 | 前面积累 > 0 就接，≤ 0 就重新开始 |
| Kadane 复杂度 | $O(N)$ 时间，$O(1)$ 空间 |
| 全负数处理 | `best` 初始化为 `INT_MIN`，不能用 0 |
| 整体复杂度 | $O(N^3)$，枚举行 $O(N^2)$ × Kadane $O(N)$ |
