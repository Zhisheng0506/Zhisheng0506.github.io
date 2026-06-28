---
title: 二叉搜索树-判断两序列是否为同一BST
date: 2026-06-15 10:00:00
tags: [算法, 二叉树, 二叉搜索树, 递归, 上机考试]
---

## 题目描述

> 题目链接：[N诺/DreamJudge 1303](https://noobdream.com/DreamJudge/Issue/page/1303/)

判断两个序列是否能构建出同一棵二叉搜索树。

```
输入:
2
567432
543267
576342
0

输出:
YES    ← "567432" 和 "543267" 生成同一棵 BST
NO     ← "567432" 和 "576342" 生成不同的 BST
```

`1 ≤ n ≤ 20`，序列长度 < 10（数字 0~9，无重复）。

<!--more-->

## 核心思路

把两序列分别构建 BST，然后递归比较两棵树是否**结构和值都完全相同**。算法的核心是 `check` 函数。

---

## BST 插入

```cpp
void buildtree(Node* &root, int x) {
    if (root == nullptr) {
        root = new Node(x);
        return;
    }
    if (x > root->val) buildtree(root->left, x);   // 大的去左边
    else if (x < root->val) buildtree(root->right, x); // 小的去右边
}
```

（本题的左右规则和常规 BST 相反——“大的在左、小的在右”不影响比较结果，只要两棵树建树规则一致即可。）

---

## 判断两棵树是否相同（重点）

```cpp
bool check(Node* root1, Node* root2) {
    // ① 都空 → 相同
    if (root1 == nullptr && root2 == nullptr) return true;

    // ② 一个空一个不空 → 不同
    if (root1 == nullptr || root2 == nullptr) return false;

    // ③ 都有值但值不同 → 不同
    if (root1->val != root2->val) return false;

    // ④ 当前节点相同，递归比较左右子树
    return check(root1->left, root2->left)
        && check(root1->right, root2->right);
}
```

### 四个条件的逻辑

```
check(root1, root2):
  │
  ├─ ① 两个都是 nullptr? → true（结构相同，到底了）
  │
  ├─ ② 一个空一个不空? → false（结构不同）
  │
  ├─ ③ 值不相等? → false（值不同）
  │
  └─ ④ 当前节点通过，递归验证左右子树
       └─ 左相同 && 右相同 → true，否则 → false
```

### 执行过程

```
树1:        树2:
    5          5
   / \        / \
  6   4      6   4
 /           /
7           7

check(5, 5):
  都不空 ✓, 值相等 5==5 ✓
  递归左: check(6, 6)
    都不空 ✓, 6==6 ✓
    左: check(7, 7) → 7==7, 都是叶子 → true
    右: check(null, null) → 都空 → true
    左右都 true → true ✓
  递归右: check(4, 4) → 4==4, 都是叶子 → true
  左右都 true → true ✓

结果: YES
```

```
树1: 5-6-7-4      树2: 5-7-6-4

检查 5 和 5 ✓ →
  左: check(6, 7) → 值不同 6≠7 → false!
结果: NO
```

---

## 完整执行流程

```
基准序列: "567432"

建树:
5 → 根=5
6 → 6>5 → 左边
7 → 7>5 → 左边, 7>6 → 6的左边
4 → 4<5 → 右边
3 → 3<5 → 右边, 3<4 → 4的右边
2 → 2<5 → 右边, 2<4 → 4的右边, 2<3 → 3的右边

树:  5
    / \
   6   4
  /     \
 7       3
          \
           2

序列1 "543267":
建同样流程... 如果结果树和基准树完全一样 → YES

序列2 "576342":
建树结构不同 → NO
```

---

## ⚠️ 关键坑点

### 1. 三个判断顺序不能乱

```cpp
// ✅ 正确顺序
if (都空) return true;      // 先判断终止条件
if (一个空) return false;   // 再判断结构不同
if (值不等) return false;   // 最后判断值不同

// ❌ 如果先判断值不同
if (root1->val != root2->val) return false;
// 当 root1 为 null 时，直接段错误！
```

必须**先判空，再比值**。`nullptr->val` 是未定义行为。

### 2. 全空 = 相同

```cpp
if (root1 == nullptr && root2 == nullptr) return true;
```

两棵空树是完全相同的。这是递归的终止条件——叶子节点的左右孩子都是 null，这里判断通过。

### 3. 左右子树都要相同

```cpp
return check(root1->left, root2->left)
    && check(root1->right, root2->right);
```

`&&` 保证左右都相同才返回 true，任意一侧不同就短路返回 false。

---

## 完整代码模板

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int val;
    Node *left, *right;
    Node(int x) : val(x), left(nullptr), right(nullptr) {}
};

void buildtree(Node* &root, int x) {
    if (root == nullptr) { root = new Node(x); return; }
    if (x > root->val) buildtree(root->left, x);
    else if (x < root->val) buildtree(root->right, x);
}

bool check(Node* root1, Node* root2) {
    if (root1 == nullptr && root2 == nullptr) return true;
    if (root1 == nullptr || root2 == nullptr) return false;
    if (root1->val != root2->val) return false;
    return check(root1->left, root2->left)
        && check(root1->right, root2->right);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n;
    while (cin >> n && n) {
        string s; cin >> s;
        Node* root = nullptr;
        for (char c : s) buildtree(root, c);

        for (int i = 0; i < n; i++) {
            string str; cin >> str;
            Node* rootx = nullptr;
            for (char c : str) buildtree(rootx, c);
            cout << (check(root, rootx) ? "YES" : "NO") << endl;
        }
    }
}
```

---

## 总结

| 要点 | 说明 |
|------|------|
| 核心 | 两棵树同时递归，逐节点比较 |
| 都空 | 结构一致到底 → true |
| 一空一非空 | 结构不同 → false |
| 值不同 | 内容不同 → false |
| 值相同 | 递归比较左右子树，必须都相同 |
| 顺序 | 先判空 → 再判结构 → 最后比值 |
