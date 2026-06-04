---
title: stringstream字符流-竞赛中的字符串利器
date: 2026-05-27 10:00:00
tags: [算法, 字符串, stringstream, 上机考试]
---

## 题目描述

> 题目链接：[N诺/DreamJudge 1267](https://noobdream.com/DreamJudge/Issue/page/1267/)

读入一行以 `.` 结尾的文字，统计单词个数并输出每个单词的长度。一个或多个空格隔开的部分为一个单词。

示例：

```
输入: hello world this is a test.
输出: 5 5 4 2 1 4
```

<!--more-->

## 核心思路

读完一行 → 去掉末尾的 `.` → 丢进 `stringstream` → 用 `>>` 逐个读出单词。

```cpp
string s;
getline(cin, s);
if (s.back() == '.') s.pop_back();  // 去掉结尾的句号

stringstream ss(s);
string word;
while (ss >> word) {
    cout << word.size() << " ";
}
```

跟用 `find` + `substr` 手动切分比起来，`stringstream` 一行顶十行。

---

## stringstream 是什么

`stringstream` 定义在 `<sstream>` 头文件中（`bits/stdc++.h` 已包含），本质上是一块**内存中的缓冲区**，既可以往里写数据（像 `cout`），也可以从里读数据（像 `cin`）。

```
┌─────────────────────────────────┐
│         stringstream             │
│                                 │
│   "hello 123 3.14"              │
│          ↓                      │
│   ss >> s  →  "hello"          │
│   ss >> n  →  123              │
│   ss >> d  →  3.14             │
│          ↑                      │
│   ss << "world" << 42;          │
└─────────────────────────────────┘
```

把字符串变成"像 cin 一样可读的流"，或者把数据组装成"像 cout 一样的字符串"——这就是 stringstream 的核心价值。

### 三个变体

| 类 | 方向 | 用途 |
|---|------|------|
| `stringstream` | 读写 | 最常用，既能读也能写 |
| `istringstream` | 只读 | 从字符串中读取数据 |
| `ostringstream` | 只写 | 把数据写入字符串 |

竞赛中 99% 的场景用 `stringstream` 就够了。

---

## 用法一：字符串分割（单词切分）

这是竞赛中最高频的用法——把空格分隔的字符串逐个提取。

### 基本模式

```cpp
string s = "hello world this is a test";
stringstream ss(s);
string word;
while (ss >> word) {
    cout << word << endl;
}
// 输出:
// hello
// world
// this
// is
// a
// test
```

`>>` 遇到空格、换行、Tab 自动停止，所以天然适合按空白字符切分。

### 和手动 find + substr 的对比

```cpp
// ❌ 手动切分：七八行，容易写错
string s = "hello world this is a test";
size_t pos = 0;
while ((pos = s.find(' ')) != string::npos) {
    cout << s.substr(0, pos) << endl;
    s = s.substr(pos + 1);
}
cout << s << endl;  // 还要处理最后一段

// ✅ stringstream：三行
stringstream ss(s);
string word;
while (ss >> word) cout << word << endl;
```

### 统计单词数

```cpp
stringstream ss(s);
string word;
int cnt = 0;
while (ss >> word) cnt++;
cout << cnt << endl;
```

---

## 用法二：字符串 → 数字（类型转换）

`>>` 会自动根据目标变量的类型做转换，效果等同于 `stoi` / `stod`，但不需要为每个字段单独调用函数。

### 基本模式

```cpp
string s = "2026 5 27";
stringstream ss(s);
int year, month, day;
ss >> year >> month >> day;
// year=2026, month=5, day=27
```

一行拆出三个 int，比 `stoi(s.substr(...))` 简洁得多。

### 混合类型提取

```cpp
string s = "Alice 95 3.8";
stringstream ss(s);
string name;
int score;
double gpa;
ss >> name >> score >> gpa;
// name="Alice", score=95, gpa=3.8
```

`>>` 会根据目标变量类型自动解析，string、int、double 混在一起也行。

### 和 stoi 系列函数的对比

```cpp
string s = "100 200 300";

// ❌ stoi 方式：需要先 split 再逐个转
stringstream splitter(s);
string token;
vector<int> nums;
while (splitter >> token) {
    nums.push_back(stoi(token));  // 每次都要 stoi
}

// ✅ stringstream 直接读入 int
stringstream ss(s);
int num;
while (ss >> num) {
    nums.push_back(num);  // 直接转，无需 stoi
}
```

### 读取多行输入中的不定量数据

这是 `stringstream` 最不可替代的场景——每行数据量不固定时：

```cpp
// 输入示例:
// 3
// 1 2 3
// 4 5
// 6 7 8 9

int n;
cin >> n;
cin.ignore();  // ⚠️ 吃掉换行符

for (int i = 0; i < n; i++) {
    string line;
    getline(cin, line);
    stringstream ss(line);
    int x;
    while (ss >> x) {
        // 处理每个数，不管这行有几个
    }
}
```

这是手动 `substr` + `find` 很难优雅处理的场景。

---

## 用法三：数字 → 字符串（反向拼接）

用 `<<` 把数据写入 stringstream，再 `.str()` 取出字符串。

### 基本模式

```cpp
stringstream ss;
ss << "result: " << 42 << ", " << 3.14;
string result = ss.str();
// result = "result: 42, 3.14"
```

### 和 to_string 的对比

```cpp
// ❌ to_string：多个值拼接很啰嗦
string s = to_string(year) + "-" + to_string(month) + "-" + to_string(day);

// ✅ stringstream：自然拼接
stringstream ss;
ss << year << "-" << month << "-" << day;
string s = ss.str();
```

`to_string` 适合单个值转换，`stringstream` 适合多个值拼接——写法更自然。

### 格式化输出

```cpp
stringstream ss;
ss << setw(2) << setfill('0') << month << "-"
   << setw(2) << setfill('0') << day;
string date = ss.str();  // "05-27"
```

和 `cout` 完全相同的格式化操作符（`setw`、`setfill`、`hex`、`fixed`、`setprecision` 等），在 stringstream 上全都有效。

---

## 用法四：逐行读取 + 行内再切分

```cpp
string line;
while (getline(cin, line)) {
    stringstream ss(line);
    string word;
    while (ss >> word) {
        // 处理这一行的每个单词
    }
}
```

这是竞赛中处理"每行不定量数据"的标准模板。

---

## 用法五：用 getline 按指定分隔符切分

`>>` 只能按空格/Tab 切分。如果要按**逗号**、**冒号**等自定义分隔符切分，用 `getline(ss, token, 分隔符)`。

### 例题：逗号分隔的数字排序

```
输入: 5,3,8,1
输出: 1 3 5 8
```

### ❌ 丑陋做法：先 replace 再 `>>`

```cpp
string s;
getline(cin, s);
while (s.find(',') != -1) {
    s.replace(s.find(','), 1, " ");  // 把所有逗号替换成空格
}
stringstream ss(s);
int x;
while (ss >> x) { ... }
```

能跑，但修改了原始字符串，而且多次 `find` + `replace` 效率低。

### ✅ 正确做法：`getline(ss, token, ',')`

```cpp
string s;
getline(cin, s);

stringstream ss(s);
string token;
vector<int> A;

while (getline(ss, token, ',')) {  // 第三个参数指定分隔符
    A.push_back(stoi(token));
}
```

### `getline` 的第三个参数

```cpp
getline(cin, str);          // 默认按 '\n' 切分
getline(ss, token, ',');    // 按 ',' 切分
getline(ss, token, ':');    // 按 ':' 切分
getline(ss, token, '|');    // 按 '|' 切分
```

`getline` 读到分隔符就停止，**分隔符被丢弃**，返回的 token 不包含分隔符。流耗尽时返回 `false`，所以 `while (getline(...))` 自然终止。

### 和 `cin` 的 `getline` 是同一个函数

```cpp
getline(cin, line);           // 从标准输入读一行（按 '\n' 切）
getline(ss, token, ',');      // 从 stringstream 读一段（按 ',' 切）
```

同一个 `getline`，第一个参数可以是 `cin` 也可以是 `stringstream`，行为完全一致。

### 完整代码

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string s;
    getline(cin, s);

    stringstream ss(s);
    string token;
    vector<int> A;

    while (getline(ss, token, ',')) {
        A.push_back(stoi(token));
    }

    sort(A.begin(), A.end());
    for (int i = 0; i < A.size(); i++) {
        if ((i + 1) % 4 != 0) {
            cout << A[i] << " ";
        } else {
            cout << A[i] << endl;
        }
    }
}
```

### 后续小节编号调整

原来的"用法五~七"顺延到"用法六~八"，以下用新编号。

---

## 用法六：清空和重用

同一个 stringstream 处理多组数据时，需要清空内容并重置状态：

```cpp
stringstream ss;

ss << "first";
ss.str("");     // 清空内容
ss.clear();     // 重置状态标志（eof、fail 等）

ss << "second";
cout << ss.str();  // "second"
```

| 操作 | 作用 |
|------|------|
| `ss.str("")` | 清空缓冲区内容 |
| `ss.clear()` | 重置流状态（eofbit、failbit、badbit） |

**两个都要写**，漏掉 `clear()` 是最常见的 bug——循环到第二组时可能因为上轮的 eof 状态导致读写直接失败。

### 清空 + 重用的标准写法

```cpp
stringstream ss;
for (int i = 0; i < n; i++) {
    ss.str("");   // 清空
    ss.clear();   // 重置
    ss << data[i];
    // 处理 ss...
}
```

---

## 用法七：判断读取是否成功

```cpp
stringstream ss("hello 123");
string s;
int n;

if (ss >> s) {
    cout << "读取字符串成功: " << s << endl;
}
if (ss >> n) {
    cout << "读取数字成功: " << n << endl;
}
if (!(ss >> n)) {
    cout << "读取失败，流已耗尽" << endl;
}
```

`>>` 返回的是流对象的引用，在布尔上下文中：
- 读取成功 → `true`
- 读取失败（类型不匹配 / 流耗尽） → `false`

这个特性让 `while (ss >> word)` 能够自然终止循环。

---

## 用法八：初始化 stringstream 的三种方式

```cpp
// 方式 1：构造时初始化
string s = "hello world";
stringstream ss(s);

// 方式 2：用 str() 设置内容
stringstream ss;
ss.str("hello world");

// 方式 3：用 << 写入
stringstream ss;
ss << "hello " << "world";
```

一般用方式 1（构造时初始化）最简洁。

---

## ⚠️ 关键坑点

### 1. getline 前要吃掉残留换行符

```cpp
int n;
cin >> n;
cin.ignore();  // ⚠️ 必须吃掉 cin>>n 后残留的 '\n'
string line;
getline(cin, line);  // 否则这行读到的是空串
```

`cin >> n` 读完数字后会留下一个换行符，`getline` 读到换行符就停止，结果读到空串。

### 2. 重用前要同时 str("") 和 clear()

只清内容不重置状态 → 第二轮的状态标志还是 eof → 读写全部跳过。

```cpp
ss.str("");   // 清内容
ss.clear();   // 清状态，两个都写
```

### 3. 空字符串从 stream 读会直接失败

```cpp
stringstream ss("");
string word;
if (ss >> word) {  // false，不会进入
    // ...
}
```

### 4. `>>` 不会读取空格

如果原字符串中有连续空格，`>>` 会全部跳过。这是特性不是 bug，但如果需要保留空格信息，用 `getline(ss, word, ' ')` 代替。

### 5. 类型不匹配时静默失败

```cpp
stringstream ss("abc 123");
int n;
ss >> n;  // 失败！n 的值未定义，流进入 fail 状态
```

读到类型不匹配的数据时，流会设置 `failbit`，后续所有读操作都会被跳过。可以加检查：

```cpp
if (!(ss >> n)) {
    ss.clear();  // 清除错误状态后才能继续读
}
```

---

## 常见场景速查

| 场景 | 写法 |
|------|------|
| 按空格切分单词 | `while (ss >> word)` |
| 字符串 → int | `ss >> num;` |
| 字符串 → double | `ss >> d;` |
| 拼接字符串 | `ss << x << y;` 然后 `ss.str()` |
| 逐行读取 | `getline(cin, line)` |
| 行内再切分 | `stringstream ss(line); while (ss >> x)` |
| 按指定分隔符切分 | `while (getline(ss, token, ','))` |
| 清空重用 | `ss.str(""); ss.clear();` |
| 格式化数字 | `ss << setw(2) << setfill('0') << n;` |
| 判断读取是否成功 | `if (ss >> x)` |
| 读取整行到 stream | `stringstream ss(s);` 或 `ss.str(s);` |

---

## 完整代码模板

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s;
    getline(cin, s);
    if (s.back() == '.') {
        s.pop_back();
    }
    stringstream ss(s);
    string word;
    while (ss >> word) {
        cout << word.size() << " ";
    }
    return 0;
}
```

---

## 总结

| 要点 | 说明 |
|------|------|
| 核心作用 | 把字符串变成"可读写的流"，像 cin/cout 操作内存 |
| 单词切分 | `while (ss >> word)`，空格自动分隔 |
| 类型转换 | `ss >> int/double/string`，自动按目标类型解析 |
| 反向拼接 | `ss << x; ss.str()` 取出结果 |
| 行内切分 | `getline + stringstream`，处理不定量数据 |
| 按分隔符切分 | `getline(ss, token, ',')`，第三参数指定分隔符 |
| 重用清空 | `ss.str("")` + `ss.clear()`，两个都不能少 |
| 格式化 | `setw`、`setfill`、`fixed` 等在 stringstream 上全部可用 |
| 头文件 | `<sstream>`，`bits/stdc++.h` 已包含 |
