---
title: cmp函数-最后一行return false的重要性
date: 2026-06-05 10:00:00
tags: [算法, 排序, cmp, 上机考试]
---

## 场景

对 `vector<pair<string, int>>` 排序，cmp 按多个字段逐级比较：

```cpp
bool mycmp(const pair<string, int>& a, const pair<string, int>& b) {
    // 把 first 按 '/' 拆成三个字段 A[0] A[1] A[2]
    // ...

    if (a.second != b.second) return a.second > b.second;
    if (A[0] != B[0]) return A[0] < B[0];
    if (A[1] != B[1]) return A[1] < B[1];
    if (A[2] != B[2]) return A[2] < B[2];
    return false;  // ← 没有这行，程序不输出任何结果
}
```

<!--more-->

## 问题现象

**没有 `return false`，程序不报错、不崩溃、不警告，但就是没有任何输出。** debug 了很长时间才发现是 cmp 漏了一行。

## 为什么必须 return false

### cmp 的约定

`sort` 要求 cmp 满足**严格弱序**：`cmp(a, b)` 返回 `true` 表示 a 排在 b 前面。如果 `cmp(a, b)` 和 `cmp(b, a)` 都返回 `false`，a 和 b 等价。

### 所有字段都相等时

```cpp
// 假设有四条数据，其中两条完全相同：
A[0] = "a", A[1] = "b", A[2] = "c", a.second = 100
B[0] = "a", B[1] = "b", B[2] = "c", b.second = 100

// 进入 mycmp(A, B):
a.second != b.second → false（都是 100），跳过
A[0] != B[0] → false（都是 "a"），跳过
A[1] != B[1] → false（都是 "b"），跳过
A[2] != B[2] → false（都是 "c"），跳过
// ⚠️ 走到函数末尾，没有 return 语句！
```

C++ 中**有返回值的函数走到末尾却没有 return，是未定义行为**。不同编译器表现不同：

| 情况 | 现象 |
|------|------|
| 返回垃圾值（恰好 true） | a<b 且 b<a 都 true → 破坏严格弱序 → sort 死循环 |
| 编译优化 | 程序直接卡死，无输出 |
| 返回垃圾值（恰好 false） | 碰巧正常（但不能依赖） |

我遇到的正是**程序完全不输出**——cmp 的未定义行为破坏了 sort 内部状态，后续的 `cout` 永远执行不到。

### 正确写法

```cpp
if (a.f1 != b.f1) return a.f1 < b.f1;
if (a.f2 != b.f2) return a.f2 < b.f2;
// ...
return false;  // ⚠️ 必须：所有字段相等时，a 不排在 b 前面
```

`return false` 的含义：所有字段都相等，a 和 b 完全等价，不需要交换。这是 cmp 函数的**标准结尾**，不是可选的。

---

## ⚠️ 关键坑点

### 1. 不 return false 的后果：沉默，无报错

- 不输出任何结果
- 排序结果乱
- 程序崩溃
- 本地能跑、OJ 上炸（不同编译器行为不同）

**无报错、无提示。** 这是最坑的地方——debug 时完全不会往 cmp 漏了一行上面想。

### 2. 不是"数据有重复才需要"

即使数据看起来不可能重复，也**必须写**。这是 cmp 函数的形式要求。

### 3. `return false` 不能写成 `return true`

```cpp
return false;  // ✅ "a 不排在 b 前面" → a 和 b 等价
return true;   // ❌ "a 排在 b 前面" → 交换参数后 b 也排在 a 前面 → 矛盾
```

所有字段相等时返回 `true`，等于说 a < b **且** b < a，直接破坏严格弱序，sort 内部必然出错。

---

## 总结

| 要点 | 说明 |
|------|------|
| 现象 | 无 return false → 程序不输出，无报错 |
| 原因 | 走到函数末尾无 return = 未定义行为 |
| 正确写法 | 所有 `!=` 判断之后，以 `return false;` 结尾 |
| 含义 | 所有字段相等时，a 不排在 b 前面 |
| 记忆 | **cmp 的最后一行永远是 `return false;`** |
