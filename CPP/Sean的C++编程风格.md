# Sean的C++编程风格

> 说明：本文用于逐步总结完善自己风格的C++编程习惯。主要参考Google的C++编程风格（[源项目](https://github.com/google/styleguide); [中文版](https://github.com/zh-google-styleguide/zh-google-styleguide))。

## 1. 版权说明 

```c++
中文版：
// 名称：程序名称
// 版权：仅供学习
// 许可证：（可选）
// 作者：Sean (eppesh@163.com)
// 环境：VS2019;
// 时间：2021/07/11
// 说明：

// 感谢：xxx[相应网站]
```

```c++
英文版：
// Name:
// Copyright 2021 SH Inc.
// License:
// Author: Sean (eppesh@163.com)
// IDE: VS2019 (optional)
// Time: 07/11/2021
// Description: This is ...
```

Every file should contain license boilerplate. Choose the appropriate boilerplate for the license used by the project (for example, Apache 2.0, BSD, LGPL, GPL).    

If you **make significant changes** to a file with an author line, consider **deleting the author line. New files should usually not contain copyright notice or author line.**  

If a .h declares multiple abstractions, the file-level comment should broadly describe the contents of the file, and how the abstractions are related. A **1 or 2 sentence file-level comment may be sufficient**. The detailed documentation about individual abstractions belongs with those abstractions, not at the file level.  

**Do not duplicate comments in both the .h and the .cc**. Duplicated comments diverge.

---

## 2. 头文件

### 2.1 #define保护

所有头文件都应该使用 #define 来防止头文件被多重包含，**命名格式**应该是 <**PROJECT**> _ <**PATH**>_ <**FILE**> _ **H** _ ；  

为保证唯一性, 头文件的命名应该基于所在项目源代码树的全路径. 例如, **项目 pushbox 中的头文件 pushbox/src/game/box.h **可按如下方式保护:

```c++
#ifndef PUSHBOX_GAME_BOX_H_
#define PUSHBOX_GAME_BOX_H_
...
#endif // PUSHBOX_GAME_BOX_H_
```

> Sean注：目前**统一用#ifndef方式**，而不用#pragma once方式（2021/09/10）；

### 2.2 #include名称与顺序

- 在 #include 中**插入空行**以分割：**相关头文件；C 库；C++ 库,；其他库的.h；本项目内的 .h **。  

- **字母顺序**分别对每种类型的头文件进行**二次排序**是不错的主意；  

比如，dir/foo.cpp 或 dir/foo_test.cpp 的主要作用是实现或测试 dir2/foo2.h 的功能，foo.cc 中包含头文件的次序如下:

1. dir2/foo2.h (优先位置, 详情如下)
2. C 系统文件
3. C++ 系统文件
4. 其他库的 .h 文件
5. 本项目内 .h 文件

举例来说, google-awesome-project/src/foo/internal/fooserver.cc 的包含次序如下：

```c++
#include "foo/public/fooserver.h" // 优先位置

#include <sys/types.h>
#include <unistd.h>

#include <hash_map>
#include <vector>

#include "base/basictypes.h"
#include "base/commandlineflags.h"

#include "foo/public/bar.h"
```

---

## 3. 使用namespace

```C++
// .h 文件
namespace mynamespace 
{
// 所有声明都置于命名空间中
// 注意不要使用缩进
class MyClass 
{
public:
    ...
    void Foo();
};
} // namespace mynamespace
--------------------------
// .cpp 文件
namespace mynamespace 
{
// 函数定义都置于命名空间中
void MyClass::Foo() 
{
    ...
}
}
// namespace mynamespace
---------------------------
//更复杂的 .cc 文件包含更多, 更复杂的细节, 比如 gflags 或 using 声明。
#include "a.h"
DEFINE_FLAG(bool, someflag, false, "dummy flag");
namespace a {
...code for a... // 左对齐
} // namespace a
```

---

## 4. 命名规则

### 4.1 文件名 File Names

- 全小写
- 用 _ （下划线）连接
- 成对儿出现（如头文件和源文件）
- 唯一性（Be unique）；（http_server_logs.h 而不是 log.h）

```c++
// For a class called MyUsefulClass
my_useful_class.h
my_useful_class.cpp
myusefulclass.cpp (也可以,但加下划线更直观)
```

### 4.2 类型名 Type Names

- Type: classes, sturcts, type aliases, enums,  and type template parameters; （类型包括：**类class，结构体struct, 别名typedef，枚举enum，以及模板参数**）；
- Type names **start with a capital letter** and **have a capital letter for each new word, with no underscores**: **MyExcitingClass, MyExcitingEnum.** （大写字母开头，其他每个新单词的首字母大写，没有下划线）  

```c++
// classes and structs
class UrlTable { ...
class UrlTableTester { ...
struct UrlTableProperties { ...

// typedefs
typedef hash_map<UrlTableProperties *, std::string> PropertiesMap;

// using aliases
using PropertiesMap = hash_map<UrlTableProperties *, std::string>;

// enums
enum class UrlTableError { ...
```

### 4.3 变量名 Variable Names

The names of **variables** (including **function parameters**) and data members are **all lowercase**, **with underscores between words**. Data members of classes (but not structs) additionally have **trailing underscores**. 

For instance: **a_local_variable, a_struct_data_member, a_class_data_member****_**.

- **一般变量（**以及**函数参数变量）：小写 + 下划线( _ )** ； 

```c++
string table_name;  // OK - lowercase with underscore
string tableName;   // Bad - mixed case.
【string tablename;   // OK - all lowercase. 原版并没有这句；先不用】
```

- **类的数据成员（Class Data Members）：小写 + 下划线( _ ) +** **下划线( _ )结尾**；（**end with _** )

Data members of classes, both static and non-static, are named like ordinary nonmember variables, but **with a trailing underscore**.

```c++
class TableInfo {
  ...
 private:
  std::string table_name_;  // OK - underscore at end.
  static Pool<TableInfo>* pool_;  // OK.
};
```

- **结构体数据成员（Struct Data Members）： no trailing _** 

Data members of structs, both static and non-static, are named like ordinary nonmember variables. They do not have the trailing underscores that data members in classes have.

```c++
struct UrlTableProperties {
  std::string name;
  int num_entries;
  static Pool<UrlTableProperties>* pool;
};
```

- **const 常量：k打头 + mixed case**

Variables declared constexpr or const, and whose value is fixed for the duration of the program, are named with a leading "k" followed by mixed case. Underscores can be used as separators in the rare cases where capitalization cannot be used for separation. For example:

```c++
const int kDaysInAWeek = 7; 
const int kAndroid8_0_0 = 24;  // Android 8.0.0
```

All such variables with static storage duration (i.e., statics and globals, see [Storage Duration](http://en.cppreference.com/w/cpp/language/storage_duration#Storage_duration) for details) should be named this way. This convention is optional for variables of other storage classes, e.g., automatic variables, otherwise the usual variable naming rules apply.

- **函数名称（Function Names）：Mixed case** （大写字母开头+每个单词首字母大写）；

一般情况：

```c++
AddTableEntry(); 
DeleteUrl(); 
OpenFileOrDie();
```

特殊的, setters & getters could be named like variables. Accessors and mutators (get and set functions) may be named like variables. These often correspond to actual member variables, but this is not required. 

```c++
int count() const; 
void set_count(int count)
```

- **Namespace Names: 全小写；**
  1. Namespace names are **all lower-case**.
  2. Top-level namespace names are based on the **project name** .
  3. **Avoid collisions** between nested namespaces and well-known top-level namespaces.

- **宏名称（Macro Names）：尽量不用（用const代替）** ；若用，**全大写 + 下划线( _ )**; 

````c++
#define ROUND(x) ... 
#define PI_ROUNDED 3.0
````

- **枚举名称（Enumerator Names）：k + mixed case; 同const变量风格；**

Enumerators (for both scoped and unscoped enums) should be named like [constants](https://google.github.io/styleguide/cppguide.html#Constant_Names), not like [macros](https://google.github.io/styleguide/cppguide.html#Macro_Names). That is, use **kEnumName** not **~~ENUM_NAME~~**.

```c++
enum class UrlTableError {  
    kOk = 0,  
    kOutOfMemory,  
    kMalformedInput
}; 
// 不用下面这种 
enum class AlternateUrlTableError {  
    OK = 0,  
    OUT_OF_MEMORY = 1,  
    MALFORMED_INPUT = 2
};
```

Until January 2009, the style was to name enum values like [macros](https://google.github.io/styleguide/cppguide.html#Macro_Names). This caused problems with name collisions between enum values and macros. Hence, the change to prefer constant-style naming was put in place. New code should use constant-style naming.

- **Exceptions to Naming Rules**

正在使用一个库, 那么可以 follow 它的命名方式.

```c++
sparse_hash_map // STL-like entity; follows STL naming conventions
```

---
## 5. 注释（Comments）

### 5.1 注释风格

- **统一风格**，只用 **//** （ // 更common，就别再用/* */了）
- 注意对齐和美观
- **不要写废话，尽量别BB**

Bad

```c++
// Find the element in the vector.  <-- Bad: obvious!
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
```

Good

```c++
// Process "element" unless it was already processed.
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
```

Best

```c++
if (!IsAlreadyProcessed(element)) {
  Process(element);
}
```

### 5.2 文件注释：见本文最开头；

### 5.3 类注释

- 类的注释要写清楚**类的功能、用法和注意事项**；最好能**给一个小例子**。

```c++
// Iterates over the contents of a GargantuanTable.
// Example:
//    GargantuanTableIterator* iter = table->NewIterator();
//    for (iter->Seek("foo"); !iter->done(); iter->Next()) {
//      process(iter->key(), iter->value());
//    }
//    delete iter;
class GargantuanTableIterator {
  ...
};
```

### 5.4 函数注释

#### 5.4.1 函数声明 Declaration

在函数声明处描述函数的功能，信息包括：

- What the inputs and outputs are.
- For class member functions: whether the object remembers reference arguments beyond the duration of the method call, and whether it will free them or not.
- If the function allocates memory that the caller must free.
- Whether any of the arguments can be a null pointer.
- If there are any performance implications of how a function is used.
- If the function is re-entrant. What are its synchronization assumptions?

```c++
// Returns an iterator for this table.  It is the client's
// responsibility to delete the iterator when it is done with it,
// and it must not use the iterator once the GargantuanTable object
// on which the iterator was created has been deleted.
//
// The iterator is initially positioned at the beginning of the table.
//
// This method is equivalent to:
//    Iterator* iter = table->NewIterator();
//    iter->Seek("");
//    return iter;
// If you are going to immediately seek to another place in the
// returned iterator, it will be faster to use NewIterator()
// and avoid the extra seek.
Iterator* GetIterator() const;
```

#### 5.4.2 函数实现：在函数实现处描述函数的实现细节；

在函数体内, you should have comments in **tricky, non-obvious, interesting, or important parts** of your code.

- **tricks**

```c++
// Divides result by two, taking into account that x
// contains the carry from the add.
for (int i = 0; i < result->size(); i++) {
  x = (x << 8) + (*result)[i];
  (*result)[i] = x >> 1;
  x &= 1;
}
```

- **non-obvious code**

Also, lines that are non-obvious should get a comment at the end of the line. These **end-of-line comments** should be separated from the code by **2 spaces**. Example:

```c++
// If we have enough memory, mmap the data portion too.
mmap_budget = max<int64>(0, mmap_budget - index_->length());
if (mmap_budget >= data_size_ && !MmapData(mmap_chunk_bytes, mlock))
  return;  // Error already logged.
```

#### 5.4.3 函数参数（Parameters）

当函数参数无法很好地 self-explain 时, 可以这样做：

- 参数是一个任意的常量, ok , 起个名字, 变成 const 常量.
- bool 变量用 enum 变量代替, 直接甩一个 true/false 一脸懵逼, 写成 enum 带自解释就好了
- 多个参数, 不如定义为一个class 或者 struct 吧
- 如果是个太长的表达式, 拜托, 还是起个变量名吧
- 实在懒得动了, 那就在调用的时候用注释声明吧

- Bad

```c++
// What are these arguments?
const DecimalNumber product = CalculateProduct(values, 7, false, nullptr);
```

- Good

```c++
ProductOptions options;
options.set_precision_decimals(7);
options.set_use_cache(ProductOptions::kDontUseCache);
const DecimalNumber product =
    CalculateProduct(values, options, /*completion_callback=*/nullptr);
```

### 5.5 变量注释

- **Class Data Members：尽量见名知义**；

In particular, add comments to describe the existence and meaning of sentinel values, such as **nullptr or -1**, when they are not obvious. For example:

```c++
private:
 // Used to bounds-check table accesses. -1 means
 // that we don't yet know how many entries the table has.
 int num_total_entries_;
```

- **Global Variables：**

All global variables should have a comment describing what they are, what they are used for, and (if unclear) why it needs to be global. For example:

```c++
// The total number of test cases that we run through in this regression test.
const int kNumTestCases = 6;
```

### 5.6 TODO注释

Use **TODO** comments for code that is temporary, a short-term solution, or good-enough but not perfect.

**TODOs** should include the string **TODO** in all caps, followed by **the name, e-mail address, bug ID**, or other identifier of the person or issue with the best context about the problem referenced by the TODO. The main purpose is to have a consistent **TODO** that can be **searched** to find out how to get more details upon request. A **TODO** is not a commitment that the person referenced will fix the problem. Thus when you create a **TODO** with a name, it is almost always your name that is given.

```c++
// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
// TODO(Zeke) change this to use relations.
// TODO(bug 12345): remove the "Last visitors" feature

// TODO(bug 12345): Fix by November 2005
// TODO(bug 12345): Remove this code when all clients can handle XML responses
```

If your **TODO** is of the form "At a future date do something" make sure that you either include a very specific date ("Fix by November 2005") or a very specific event ("Remove this code when all clients can handle XML responses.").

---
## 6. 格式（Formatting）

### 6.1 Line Length：每行最长80个char；

如果无法在不伤害易读性的条件下进行断行, 那么**注释行可以超过 80 个字符**，这样可以方便复制粘贴。例如, 带有命令示例或 URL 的行可以超过 80 个字符。

- 包含长路径的 #include 语句可以超出 80 列
- 头文件保护 可以无视该原则
