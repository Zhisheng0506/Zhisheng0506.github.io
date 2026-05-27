---
title: stable_sort稳定性排序-以字符串排序为例
date: 2026-05-27 18:00:00
tags: [算法, 排序, stable_sort, 上机考试]
---

## 题目描述

> 题目链接：[N诺/DreamJudge 1255](https://noobdream.com/DreamJudge/Issue/page/1255/)

对输入字符串中的字符按以下规则排序：

- **规则 1**：英文字母从 A 到 Z 排列，不区分大小写
- **规则 2**：同一个英文字母的大小写同时存在时，按照**输入顺序**排列
- **规则 3**：非英文字母的其它字符**保持原来的位置**

```
输入: BabA
输出: aABb    ← 不区分大小写排序，但大小写保持输入顺序

输入: By?e
输出: Be?y    ← '?' 保持原位，字母重排
```

示例：

```
输入: A Famous Saying: Much Ado About Nothing(2012/8).
输出: A aaAAbc dFgghh : iimM nNn oooos Sttuuuy (2012/8).
```

<!--more-->

## 核心思路

这道题有两个关键点：

1. **非字母不参与排序，保持原位** — 先把所有字母提出来单独排，再塞回原位
2. **不区分大小写排序，但大小写保持输入顺序** — 需要 `stable_sort`，普通 `sort` 做不到

```cpp
while (getline(cin, s)) {
    // 1. 提取所有字母
    vector<char> letter;
    for (auto c : s) {
        if (isalpha(c)) letter.push_back(c);
    }

    // 2. 稳定排序
    stable_sort(letter.begin(), letter.end(), mycmp2);

    // 3. 塞回原位
    int idx = 0;
    for (char& c : s) {
        if (isalpha(c)) c = letter[idx++];
    }
    cout << s << endl;
}
```

---

## stable_sort 详解

### sort 和 stable_sort 的区别

`sort` 和 `stable_sort` 都在 `<algorithm>` 中，签名几乎一样：

```cpp
sort(v.begin(), v.end(), cmp);         // 不保证稳定性
stable_sort(v.begin(), v.end(), cmp);  // 保证稳定性
```

**稳定性的含义**：当两个元素比较结果为"相等"时，排序后它们的相对顺序保持不变。

```
原始: [B, a, b, A]       字母        输入序号

sort (不稳定):  可能得到  a A B b    谁在前不确定
                也可能得到  a b A B   无法预测

stable_sort:    一定得到  B a b A    大小写保持输入顺序
                          ↑ ↑ ↑ ↑
                          B在a前，b在A前 — 和输入一致
```

规则 2 要求"同一个字母的大小写保持输入顺序"，这是稳定性排序的经典场景。**这种题只要遇到，一定是 `stable_sort`，不能用 `sort`。**

### 为什么 sort 不稳定

`sort` 底层是快速排序（introsort），交换时可能打乱等值元素的顺序。`stable_sort` 底层通常是归并排序，天然保证稳定性。

| | sort | stable_sort |
|------|------|------|
| 稳定性 | 不保证 | 保证 |
| 时间复杂度 | $O(N \log N)$ | $O(N \log N)$，内存不足时最高 $O(N \log^2 N)$ |
| 空间 | $O(\log N)$ | $O(N)$（需要额外缓冲区） |
| 使用场景 | 普通排序 | 需要保持等值元素原始顺序 |

竞赛中 99% 的场景 `sort` 够用，但**多关键字排序中，当比较器只比较部分字段、其余字段要保持原始顺序时，必须用 `stable_sort`**。

---

## 两种 cmp 写法对比

### mycmp1 — 只有大小写无关，没有保持顺序

```cpp
bool mycmp1(char a, char b) {
    return tolower(a) < tolower(b);
}
```

逻辑：a 的小写 < b 的小写 → 排前面。

**问题：当 `tolower(a) == tolower(b)`（同一个字母的大小写），cmp 返回 false。**

对于 `sort`：`a < b` 和 `b < a` 都返回 false，sort 认为 a 和 b 等价，**可以任意交换它们**——结果不确定，不符合规则 2。

对于 `stable_sort` 配合 `mycmp1`：返回 false 虽然表示"不需要交换"，但 `mycmp1` 本身没有区分"不同字母要排序"和"同字母不排序"这两个意图。**写法上不够明确，依赖 stable_sort 的稳定性来兜底**——不如 mycmp2 清晰。

### mycmp2 — 正确的稳定排序写法

```cpp
bool mycmp2(char a, char b) {
    char x = tolower(a);
    char y = tolower(b);
    if (x != y) {
        return x < y;    // 不同字母：按字母顺序排
    }
    return false;        // 同一字母：不排，靠 stable_sort 保持原序
}
```

当 `x == y`（同一个字母的大小写），返回 `false`。对于 `stable_sort`，返回 false 意味着"a 不应该排在 b 前面"，结合稳定性——**原本在前面的人继续保持**。

```cpp
// 输入: BabA → 字母: [B, a, b, A]
stable_sort([B, a, b, A], mycmp2);

// 比较 B vs a: tolower→ b vs a, b < a 为 false → a 排在 B 前
// 比较 b vs A: tolower→ b vs a, 同，返回 false → 保持 b 在 A 前
// 比较 B vs b: tolower→ b vs b, 同，返回 false → 保持 B 在 b 前
// 结果: [B, a, b, A] — 同一字母大小写的输入顺序被完美保留
```

### ⚠️ 为什么返回 false 而不是 true

cmp 返回 true 表示"a 应该排在 b 前面"。当两个字符大小写忽略后相同时，不应该再做任何重排，所以统一返回 false。

```cpp
// ✅ 正确：同字母返回 false
if (x != y) return x < y;
return false;

// ❌ 错误：同字母返回 true
if (x != y) return x < y;
return true;   // 会强制 a 排在 b 前面，破坏稳定性
```

### 两种 cmp 的适用场景

| cmp | 行为 | 适用 |
|-----|------|------|
| mycmp1 | 不区分大小写比较，相同靠 stable_sort 兜底 | 简单场景，但不够显式 |
| mycmp2 | 明确分离"不同字母排序"和"同字母保持"两个逻辑 | **推荐**，意图清晰 |

---

## 非字母不参与排序的实现

这种"只排一部分元素"的题目，通用的三步法：

```
原始字符串:  B y ? e

第一步: 提取
  遍历原串，isalpha 的提出来 → letter = [B, y, e]

第二步: 单独排序
  stable_sort(letter) → letter = [B, e, y]

第三步: 塞回原位
  再遍历原串，遇到字母就从 letter 里取下一个塞进去
  非字母跳过

  B y ? e
  ↓ ↓   ↓
  B e ? y    ← 非字母 '?' 原地不动
```

### 代码对应

```cpp
// 第一步：提取
vector<char> letter;
for (auto c : s) {
    if (isalpha(c)) letter.push_back(c);
}

// 第二步：排序
stable_sort(letter.begin(), letter.end(), mycmp2);

// 第三步：塞回
int idx = 0;
for (char& c : s) {
    if (isalpha(c)) {
        c = letter[idx++];
    }
}
```

这个模式在机试中反复出现——日志排序、成绩单排序、以及各种"跳过特定元素"的排序题，用的都是同一套**提取 → 排序 → 塞回**流程。

### isalpha 函数

`isalpha(c)` 定义在 `<cctype>`（`bits/stdc++.h` 已包含），判断字符是否为英文字母（大小写均返回 true）。

| 函数 | 判断 |
|------|------|
| `isalpha(c)` | 是否为字母（A-Z, a-z） |
| `isdigit(c)` | 是否为数字（0-9） |
| `isalnum(c)` | 是否为字母或数字 |
| `isspace(c)` | 是否为空白字符 |
| `tolower(c)` | 转小写 |
| `toupper(c)` | 转大写 |

---

## ⚠️ 关键坑点

### 1. sort 和 stable_sort 不能混用

规则要求保持大小写输入顺序 → 必须 `stable_sort`。用 `sort` 的代码在某些输入下能过样例、但正式数据必然出错。

### 2. cmp 返回 false 不能写成返回 true

同一字母时返回 false 才是"保持顺序"。返回 true 等于告诉算法"必须交换"，反而破坏了稳定性。

### 3. 修改原字符串时用引用

```cpp
for (char& c : s) {  // ⚠️ 必须是 char&，不是 char
    if (isalpha(c)) c = letter[idx++];
}
```

用 `char c` 不会修改原串，结果输出还是原样。必须用 `char& c` 才能原地修改。

### 4. 处理多组输入

```cpp
while (getline(cin, s)) {  // ⚠️ 用 while，不是 if
    // ...
}
```

题目可能有空格和空行，`cin >> s` 会跳过空格和换行，**必须用 `getline`**。

### 5. 空格和标点原样保留

`isalpha` 对空格、标点、数字等都返回 false，它们会被跳过，保留在原位。这恰好满足规则 3。

---

## 完整代码模板

```cpp
#include <bits/stdc++.h>
using namespace std;

bool mycmp2(char a, char b) {
    char x = tolower(a);
    char y = tolower(b);
    if (x != y) {
        return x < y;
    }
    return false;
}

int main() {
    string s;
    while (getline(cin, s)) {
        vector<char> letter;
        for (auto c : s) {
            if (isalpha(c)) {
                letter.push_back(c);
            }
        }
        stable_sort(letter.begin(), letter.end(), mycmp2);
        int idx = 0;
        for (char& c : s) {
            if (isalpha(c)) {
                c = letter[idx++];
            }
        }
        cout << s << endl;
    }
}
```

---

## 总结

| 要点 | 说明 |
|------|------|
| stable_sort vs sort | 前者保持等值元素原始顺序，后者不保证 |
| 何时用 stable_sort | 多关键字排序中，cmp 只比较部分字段、其余字段要保持原序 |
| cmp 返回 false | 同字母返回 false，靠 stable_sort 保持顺序 |
| mycmp1 vs mycmp2 | mycmp2 显式分离两个意图，更推荐 |
| 部分排序模式 | 提取 → 排序 → 塞回，三步走 |
| isalpha | 过滤字母，标点空格数字保留原位 |
| getline | 处理含空格的整行输入 |
