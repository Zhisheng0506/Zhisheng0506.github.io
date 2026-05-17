---
title: 进制转换-任意M进制转N进制
date: 2026-05-12 20:00:00
tags: [算法, 进制转换, 上机考试]
mathjax: true
---

## 题目描述（清华大学上机题）

> 题目链接：[N诺/DreamJudge 1422](https://noobdream.com/DreamJudge/Issue/page/1422/)

> 输入的第一行包括两个整数：**M** 和 **N** (2 ≤ M, N ≤ 36)。
> 下面的一行输入一个数 **X**，X 是 M 进制的数，要求你将 M 进制的数 X 转换成 N 进制的数输出。

**输入示例：**
```
16 10
1A
```

**输出示例：**
```
26
```

<!--more-->

## 核心思路

进制转换没有捷径：**以十进制为桥梁**，先把 M 进制转为十进制，再把十进制转为 N 进制。

```
M进制 ──→ 十进制 ──→ N进制
```

---

## 一、M 进制转十进制

### 转换公式

对于一个 M 进制数 $X = d_k d_{k-1} \dots d_1 d_0$（每一位用十进制表示），其十进制值为：

$$X_{10} = d_k \times M^k + d_{k-1} \times M^{k-1} + \dots + d_1 \times M + d_0$$

### 基本写法

逐位读入，每次 `result = result × M + digit`：

```cpp
long long result = 0;
for (int i = 0; i < s.size(); i++) {
    int digit;
    if (s[i] >= 'A' && s[i] <= 'Z') digit = s[i] - 'A' + 10;
    else digit = s[i] - '0';
    result = result * m + digit;
}
```

### ⚠️ 关键坑点：carry 可能会非常大！

上面 `long long` 的写法看起来没问题，但 **M 进制转十进制的过程中，中间值 carry 会爆炸式增长**。当 M 很大、字符串很长时，`long long` 根本装不下！

比如 $M=36$，字符串长度 100，十进制结果的数量级是 $36^{100}$，远超 `long long` 的范围（$2^{63}-1 \approx 9 \times 10^{18}$）。

**所以必须用数组逐位存储十进制结果**，模拟大数运算。核心是每次循环做两件事：**先 `× M`，再加当前位的值**。

```cpp
vector<long long> B(1, 0);  // 十进制结果，B[0]是个位
for (int i = 0; i < s.size(); i++) {
    // 1. 取出当前位的数值
    long long carry = 0;
    if (s[i] >= 'A' && s[i] <= 'Z') {
        carry = s[i] - 'A' + 10;
    } else {
        carry = s[i] - '0';
    }

    // 2. B 的每一位 × M，累加进位
    for (int j = 0; j < B.size(); j++) {
        carry = B[j] * m + carry;
        B[j] = carry % 10;
        carry /= 10;
    }

    // 3. ⚠️ carry 可能会很大！必须用 while 逐位推入
    while (carry) {
        B.push_back(carry % 10);
        carry /= 10;
    }
}
```

上面代码最关键的就是第 3 步的 `while (carry)` 循环。**`× M` 产生的进位可能不止一位**（比如 `999 × 36 = 35964`，carry 积累下来有 4 位），必须用 while 循环逐位拆解推入数组尾部。这是我（Kmon）自己写的时候犯过的错误，直接用 `if (carry) B.push_back(carry)` 只推了一位，结果就错了。

### M 进制转十进制要点总结

| 要点 | 说明 |
| :--- | :--- |
| 字符映射 | `0-9` → 数值 0-9，`A-Z` → 10-35 |
| 转换公式 | `result = result × M + digit` |
| 大数存储 | 用 `vector<long long> B(1, 0)` 逐位存十进制值，B[0] 是个位 |
| 乘法的进位 | `× M` 后 carry 可能有多位，**必须 `while(carry)` 循环拆位**，不能只 `if` |

---

## 二、十进制转 N 进制

### 转换方法：除 N 取余，逆序排列

M 进制转十进制后得到一个大数（存成字符串 `ss`），然后反复对 `ss` 除以 N，余数的**逆序**就是 N 进制结果。

这里同样涉及大数运算：**大数除法** — 从高位到低位模拟竖式除法。

```cpp
// 先把 vector<long long> B 转成字符串 ss（高位在前）
string ss;
for (int i = B.size() - 1; i >= 0; i--) {
    char c = B[i] + '0';
    ss += c;
}

// 除 N 取余
vector<char> A;
while (ss != "0") {
    long long t = 0;  // t 既是当前被除数，也是最终的余数
    for (int i = 0; i < ss.size(); i++) {
        t = t * 10 + ss[i] - '0';
        ss[i] = t / n + '0';  // 商直接写回字符串
        t %= n;               // 余数留给下一位
    }
    // t 就是本轮除法最终的余数
    if (t >= 0 && t <= 9) {
        A.push_back(t + '0');
    } else {
        A.push_back(t - 10 + 'a');
    }
    // 去除前导零
    while (ss.size() > 1 && ss[0] == '0') ss.erase(0, 1);
}

// 逆序输出
for (int i = A.size() - 1; i >= 0; i--) {
    cout << A[i];
}
cout << endl;
```

### 十进制转 N 进制要点总结

| 要点 | 说明 |
| :--- | :--- |
| 核心操作 | 除 N 取余，余数逆序 |
| 大数除法 | 从高位到低位模拟竖式，`t = t * 10 + 当前位`，商写回，余数传递 |
| 余数映射 | 0-9 → `'0'-'9'`，10-35 → `'a'-'z'` |
| 去前导零 | 每轮除完 `erase(0, 1)` 去前导零，否则影响下一轮判断 |
| 终止条件 | 当 `ss` 变为 `"0"` 时停止 |

---

## 三、完整代码模板

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    long long n, m;
    string s;
    cin >> m >> n;
    cin >> s;

    // ========== M 进制 → 十进制大数 ==========
    vector<long long> B(1, 0);  // B[0] 是个位
    for (int i = 0; i < s.size(); i++) {
        long long carry = 0;
        if (s[i] >= 'A' && s[i] <= 'Z') {
            carry = s[i] - 'A' + 10;
        } else {
            carry = s[i] - '0';
        }
        for (int j = 0; j < B.size(); j++) {
            carry = B[j] * m + carry;
            B[j] = carry % 10;
            carry /= 10;
        }
        while (carry) {        // carry 可能会很大！
            B.push_back(carry % 10);
            carry /= 10;
        }
    }

    // 十进制大数 → 字符串（高位在前）
    string ss;
    for (int i = B.size() - 1; i >= 0; i--) {
        char c = B[i] + '0';
        ss += c;
    }

    // ========== 十进制大数 → N 进制 ==========
    vector<char> A;
    while (ss != "0") {
        long long t = 0;
        for (int i = 0; i < ss.size(); i++) {
            t = t * 10 + ss[i] - '0';
            ss[i] = t / n + '0';
            t %= n;
        }
        if (t >= 0 && t <= 9) {
            A.push_back(t + '0');
        } else {
            A.push_back(t - 10 + 'a');
        }
        while (ss.size() > 1 && ss[0] == '0') ss.erase(0, 1);
    }

    // 逆序输出
    for (int i = A.size() - 1; i >= 0; i--) {
        cout << A[i];
    }
    cout << endl;

    return 0;
}
```

---

## 四、总结

| 阶段 | 方法 | 关键坑点 |
| :--- | :--- | :--- |
| M → 10 | 按权展开：`result = result × M + digit` | 中间值可能巨大，**必须用 `vector` 数组模拟大数**；`× M` 后的 carry 可能有多位，**必须 `while(carry)` 循环**，不能只 `if` |
| 10 → N | 除 N 取余，逆序排列 | 大数除法从高位到低位模拟竖式，每次除完记得去前导零 |

一句话记住：**进制转换 = 以十进制为桥梁 + 大数运算保底**。
