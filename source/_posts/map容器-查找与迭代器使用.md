---
title: map容器-查找与迭代器使用
date: 2026-06-05 18:00:00
tags: [算法, map, STL, 上机考试]
---

## 题目描述

> 题目链接：[N诺/DreamJudge 1476](https://noobdream.com/DreamJudge/Issue/page/1476/)

输入 N 个学生信息（学号、姓名、性别、年龄），然后进行 M 次查询。每次输入一个学号，输出对应学生信息；不存在则输出 `No Answer!`。

```
输入:
4
01 李江 男 21
02 刘唐 男 23
03 张军 男 19
04 王娜 女 19
5
02
03
01
04
05

输出:
02 刘唐 男 23
03 张军 男 19
01 李江 男 21
04 王娜 女 19
No Answer!
```

<!--more-->

这题简单，但 `map` 的 `find` 返回的是迭代器——有个坑踩过一次就不会再犯。

---

## 用 map 存结构体

```cpp
struct Node {
    string name;
    string sex;
    int age;
};

map<string, Node> mp;

// 插入
string id; Node x;
cin >> id >> x.name >> x.sex >> x.age;
mp.insert({id, x});    // 或者 mp[id] = x;
```

map 的 key 是学号，value 是学生信息。**map 内部是红黑树，按 key 自动排序**，查找 O(log N)。

---

## find：查找是否存在某个 key

```cpp
string y; cin >> y;

if (mp.find(y) != mp.end()) {
    auto it = mp.find(y);
    cout << it->first << " ";              // key
    cout << it->second.name << " "         // value
         << it->second.sex << " "
         << it->second.age << endl;
} else {
    cout << "No Answer!" << endl;
}
```

`find(key)` 返回**迭代器**：找到了指向 `{key, value}` pair，没找到等于 `mp.end()`。

---

## ⚠️ 核心坑点：`it->first` 不能写成 `it.first`

**`find` 返回的是迭代器（类似指针），不是 pair 对象本身。** 所以必须用 `->` 或先解引用再用 `.`：

```cpp
auto it = mp.find(y);

// ✅ 正确：迭代器用 ->
cout << it->first;           // key
cout << it->second.name;     // value 的成员

// ✅ 也正确：先解引用再访问
cout << (*it).first;         // 等价于 it->first

// ❌ 错误：迭代器没有 .first 成员！
cout << it.first;            // 编译错误！
```

### 为什么容易搞混

```cpp
// 遍历 map 时，范围 for 拿到的是 pair 对象本身 → 用 .
for (auto& p : mp) {
    cout << p.first << endl;   // ✅ p 是 pair，用 .
}

// find 返回的是迭代器（指向 pair 的指针）→ 用 ->
auto it = mp.find(y);
cout << it->first << endl;    // ✅ it 是迭代器，用 ->
```

**对象用 `.`，指针/迭代器用 `->`。** 范围 for 拿到的是对象，`find` 拿到的是迭代器——来源不同，访问方式不同。

---

## count：只判断有还是没有

如果只需要判断 key 是否存在，不需要取值，用 `count` 更简洁：

```cpp
if (mp.count(y)) {
    // 存在
} else {
    // 不存在
}
```

| | find | count |
|------|------|------|
| 返回值 | 迭代器（指向 pair） | 0 或 1 |
| 需要取值时 | ✅ `it->second` | ❌ 还得再 find |
| 只判断存在 | 写 `!= end()` 稍啰嗦 | ✅ `if (mp.count(k))` |
| 复杂度 | O(log N) | O(log N) |

**需要取值用 `find`，只判断有无用 `count`。**

---

## 其他常用操作

```cpp
mp["05"] = {"王五", "男", 20};  // 插入或覆盖（key 已存在则覆盖）
mp.erase("03");                 // 删除
int sz = mp.size();             // 元素个数
mp.clear();                     // 清空

// 遍历（按 key 升序）
for (auto& p : mp) {
    cout << p.first << " " << p.second.name << endl;
}
```

---

## ⚠️ 关键坑点

### 1. `it->first` 不是 `it.first`（最重要）

```cpp
auto it = mp.find(y);
cout << it->first;    // ✅
cout << (*it).first;  // ✅
cout << it.first;     // ❌ 迭代器没有 .first
```

### 2. 找不到时访问迭代器 = 段错误

```cpp
auto it = mp.find(y);
cout << it->first;  // ⚠️ 没判断 != end()，找不到时直接崩溃
```

必须先 `if (mp.find(y) != mp.end())`。

### 3. `mp[key]` 有副作用

```cpp
mp["05"];  // 如果 "05" 不存在，map 自动插入一个空 Node！
```

只判断存在用 `find` 或 `count`，别用 `mp[key]` 来"测试"。

### 4. 不需要排序用 unordered_map

`map` 内部红黑树，查找 O(log N)。`unordered_map` 用哈希表，查找 O(1)。本题意 N ≤ 1000 两种都行。

---

## 完整代码模板

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    string name;
    string sex;
    int age;
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    map<string, Node> mp;
    int n; cin >> n;
    for (int i = 0; i < n; i++) {
        string id; Node x;
        cin >> id >> x.name >> x.sex >> x.age;
        mp.insert({id, x});
    }

    int m; cin >> m;
    for (int i = 0; i < m; i++) {
        string y; cin >> y;
        if (mp.find(y) != mp.end()) {
            auto it = mp.find(y);
            cout << it->first << " "
                 << it->second.name << " "
                 << it->second.sex << " "
                 << it->second.age << endl;
        } else {
            cout << "No Answer!" << endl;
        }
    }
}
```

---

## 总结

| 要点 | 说明 |
|------|------|
| map 适用场景 | key → value 映射，按 key 查找 |
| **核心坑** | find 返回迭代器（指针），用 `->` 不是 `.` |
| 范围 for vs find | 范围 for 给 pair（用 `.`），find 给迭代器（用 `->`） |
| find vs count | 需要取值用 find，只判断有无用 count |
| `mp[key]` 副作用 | key 不存在时自动插入，别用来测试存在 |
