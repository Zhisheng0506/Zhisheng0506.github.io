---
title: 二叉树LCA-完全二叉树与二叉搜索树的最近公共祖先
date: 2026-06-14 10:00:00
tags: [算法, 二叉树, LCA, 完全二叉树, 二叉搜索树, 上机考试]
---

## 题目描述

### 例题一：完全二叉树的 LCA

> 题目链接：[N诺/DreamJudge 1337](https://noobdream.com/DreamJudge/Issue/page/1337/)

一棵无限大的完全二叉树，节点按层从上到下、从左到右编号 1, 2, 3, ...。给定两个节点 x 和 y，求它们的最近公共祖先（LCA）。

```
        1
      /   \
     2     3
    / \   / \
   4   5 6   7
  / \ / \ /\ / \
 ...  (无限延伸)

x=5, y=4 → LCA=2
x=6, y=7 → LCA=3
x=5, y=3 → LCA=1
```

`1 ≤ x, y ≤ 2^31-1`。

### 例题二：二叉搜索树的 LCA

给定一棵 BST 和两个节点值，求它们的最近公共祖先。节点值均在 BST 中存在，n ≤ 1000。

<!--more-->

---

## 例题一：完全二叉树的 LCA

### 核心思路

完全二叉树用数组存储时，节点 `i` 的父节点 = `i / 2`。LCA 不用建树——**不断把较大的节点除 2 向上走，直到两个节点相等**。

```
x=5, y=4:

5 > 4 → 5/2=2 → x=2, y=4
2 < 4 → 4/2=2 → x=2, y=2
x == y → LCA = 2
```

### 代码

```cpp
int LCA(int x, int y) {
    while (x != y) {
        if (x > y) x /= 2;
        else       y /= 2;
    }
    return x;
}
```

就这么三行。完全二叉树编号隐含了父子关系，不需要构建任何数据结构。

### 完整执行过程

```
x=10, y=13:

10 != 13:
  10 < 13 → y=13/2=6

10 != 6:
  10 > 6 → x=10/2=5

5 != 6:
  5 < 6 → y=6/2=3

5 != 3:
  5 > 3 → x=5/2=2

2 != 3:
  2 < 3 → y=3/2=1

2 != 1:
  2 > 1 → x=2/2=1

1 == 1 → LCA = 1
```

```
路径:
  x=10: 10→5→2→1
  y=13: 13→6→3→1
  交点 = 1
```

### 为什么除 2 就是父节点

完全二叉树编号规则：

```
节点 i 的左孩子 = 2i
节点 i 的右孩子 = 2i + 1
节点 i 的父节点 = i / 2   （整数除法向下取整）
```

所以 `x /= 2` 就是沿着树向上走一步。

---

## 例题二：二叉搜索树的 LCA

### 核心思路

BST 的 LCA 利用大小关系从根开始一次遍历：

> 如果两个节点都小于根 → LCA 在左子树
> 如果两个节点都大于根 → LCA 在右子树
> 否则（一个小于一个大于，或一个等于根）→ 当前根就是 LCA

```cpp
Node* LCA_BST(Node* root, int x, int y) {
    if (root == nullptr) return nullptr;

    if (x < root->val && y < root->val) {
        return LCA_BST(root->left, x, y);     // 都在左
    } else if (x > root->val && y > root->val) {
        return LCA_BST(root->right, x, y);    // 都在右
    } else {
        return root;                            // 一左一右（或等于根）
    }
}
```

### 执行过程

```
BST:
        6
       / \
      2   8
     / \  / \
    0  4 7  9
      / \
     3  5

找 LCA(3, 5):
  root=6: 3<6, 5<6 → 都小于，去左
  root=2: 3>2, 5>2 → 都大于，去右
  root=4: 3<4, 5>4 → 一左一右 → LCA=4 ✓

找 LCA(0, 5):
  root=6: 0<6, 5<6 → 都小于，去左
  root=2: 0<2, 5>2 → 一左一右 → LCA=2 ✓

找 LCA(7, 9):
  root=6: 7>6, 9>6 → 都大于，去右
  root=8: 7<8, 9>8 → 一左一右 → LCA=8 ✓
```

### 也可以迭代

```cpp
Node* LCA_BST(Node* root, int x, int y) {
    while (root != nullptr) {
        if (x < root->val && y < root->val) {
            root = root->left;
        } else if (x > root->val && y > root->val) {
            root = root->right;
        } else {
            return root;
        }
    }
    return nullptr;
}
```

---

## 两种 LCA 对比

| | 完全二叉树 | 二叉搜索树 |
|------|------|------|
| 结构 | 按层编号 1,2,3... | 左小右大 |
| 建树 | 不需要 | 需要 BST 插入 |
| 算法 | `while(x!=y) 大的除2` | 从根遍历，一左一右时停下 |
| 复杂度 | O(log N) | O(H)，H 为树高 |
| 核心 | 编号隐含父子关系 | 大小关系指导搜索方向 |

---

## ⚠️ 关键坑点

### 1. 完全二叉树：循环条件是 `x != y`

```cpp
while (x != y) { ... }   // ✅ 相等时停下
while (x > 0 && y > 0) { ... } // ❌ 不会正确终止
```

两个节点沿着路径一定会在某个祖先处相遇（最坏在根 1 处相遇）。

### 2. BST 版 LCA：必须同时比较两个值

```cpp
if (x < root->val && y < root->val)       // ✅ 两个都小于
if (x < root->val || y < root->val)       // ❌ 只有一方小于，不能判断方向
```

必须 `&&`——只有当两者都在同一侧时，LCA 才在该侧子树上。

### 3. BST 版：节点可能等于根

```cpp
if (x < root->val && y < root->val) { ... }
else if (x > root->val && y > root->val) { ... }
else { return root; }  // 包括 x==root 或 y==root 的情况
```

如果 `x == root->val`，根就是 LCA（因为根是公共祖先，且没有任何后代同时包含 x 和 y）。

---

## 完整代码模板

### 完全二叉树 LCA

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int x, y;
    while (cin >> x >> y) {
        while (x != y) {
            if (x > y) x /= 2;
            else       y /= 2;
        }
        cout << x << endl;
    }
}
```

### 二叉搜索树 LCA

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int val;
    Node *left, *right;
    Node(int x) : val(x), left(nullptr), right(nullptr) {}
};

void insert(Node* &root, int x) {
    if (root == nullptr) { root = new Node(x); return; }
    if (x < root->val) insert(root->left, x);
    else if (x > root->val) insert(root->right, x);
}

Node* LCA(Node* root, int x, int y) {
    if (root == nullptr) return nullptr;
    if (x < root->val && y < root->val)
        return LCA(root->left, x, y);
    else if (x > root->val && y > root->val)
        return LCA(root->right, x, y);
    else
        return root;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, x, y;
    cin >> n;
    Node* root = nullptr;
    for (int i = 0; i < n; i++) {
        int val; cin >> val;
        insert(root, val);
    }
    cin >> x >> y;
    Node* ans = LCA(root, x, y);
    if (ans) cout << ans->val << endl;
}
```

---

## 总结

| 要点 | 说明 |
|------|------|
| 完全二叉树 LCA | `while(x!=y) 大的/=2`，编号隐含父子关系 |
| BST LCA | 从根遍历，都小去左，都大去右，否则就是 LCA |
| 完全二叉树复杂度 | O(log N) |
| BST 复杂度 | O(H)，H 为树高 |
| 公共点 | 都不需要建完整的图，利用结构特性直接找 |
