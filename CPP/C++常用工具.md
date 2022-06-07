[toc]

# 前言

记录C++使用过程中常用的一些工具。

借助`clang-format, clang-tidy, cpplint`可以写出风格一致、bug较少、性能较好、遵守Google编码规范的项目。

# clang

编译工具。Ubuntu 16 （虚拟机环境）下使用clang。

## 安装

源码目录：[Github - LLVM](https://github.com/llvm/llvm-project); `LLVM`已经从原来的Apache网站[转移](https://www.linuxidc.com/Linux/2019-01/156379.htm)到了`Github`上。

参考：[Ubuntu - 安装clang](https://blog.csdn.net/wjl18270365476/article/details/122660107); 

- 步骤1：安装`llvm`: `sudo apt-get install llvm`
- 步骤2：安装`clang`: `sudo apt-get install clang`

通过`clang -v`查看版本信息，可以发现默认安装的是`clang 3.8`这样的古董版本，需要升级更新。

安装好`clang`后会在`/usr/bin`目录下同时存在`clang`和`clang++`.

## 升级

参考：
- [Ubuntu下安装高版本clang-format 11,12,13](https://www.codeleading.com/article/78045539655/); 
- [Ubuntu下安装高版本clang-format和clang](https://segmentfault.com/a/1190000040827790); 

步骤：
1. 获取证书：
```sh
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
```
输出如下：
```sh
sean@sean-virtual-machine:~$ wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
--2022-05-17 10:15:43--  https://apt.llvm.org/llvm-snapshot.gpg.key
[sudo] sean 的密码： 正在解析主机 apt.llvm.org (apt.llvm.org)... 151.101.26.49, 2a04:4e42:6::561
正在连接 apt.llvm.org (apt.llvm.org)|151.101.26.49|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 3145 (3.1K) [application/octet-stream]
正在保存至: “STDOUT”

-                               100%[====================================================>]   3.07K  --.-KB/s    in 0s      

2022-05-17 10:15:45 (63.8 MB/s) - 已写入至标准输出 [3145/3145]
 
对不起，请重试。
[sudo] sean 的密码： 
OK
```
好多次都是显示“已写入至标准输出”后就没动静了，这次同样又等了一段时间后，就试着**按下回车键**，此时出现了后面的提示，让输入密码，输入后提示“OK”，此时说明成功了。

**注意：**
必须先获取`key`，否则会报类似下面这样的错误：

```sh
W: GPG 错误：https://apt.llvm.org/xenial llvm-toolchain-xenial InRelease: 由于没有公钥，无法验证下列签名： NO_PUBKEY 15CF4D18AF4F7421
E: 仓库 “http://apt.llvm.org/xenial llvm-toolchain-xenial InRelease” 没有数字签名。
N: 无法安全地用该源进行更新，所以默认禁用该源。
N: 参见 apt-secure(8) 手册以了解仓库创建和用户配置方面的细节。
```

2. 在`/etc/apt/sources.list`文件末尾根据不同版本添加下列内容：
我的是`Ubuntu 16.04`就添加下面的内容（修改时需要用`root`权限）：
```sh
deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial main
deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial main
# 11
deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-11 main
deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial-11 main
# 12
deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-12 main
deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial-12 main
```

3. 更新当前已安装的软件包：`sudo apt update`; 
```sh
sean@sean-virtual-machine:~$ sudo apt update
命中:1 http://security.ubuntu.com/ubuntu xenial-security InRelease
命中:2 http://cn.archive.ubuntu.com/ubuntu xenial InRelease                                 
命中:3 http://ppa.launchpad.net/ubuntu-toolchain-r/test/ubuntu xenial InRelease            
命中:4 http://cn.archive.ubuntu.com/ubuntu xenial-updates InRelease
命中:5 http://cn.archive.ubuntu.com/ubuntu xenial-backports InRelease
获取:8 https://esm.ubuntu.com/infra/ubuntu xenial-infra-security InRelease [7,515 B]
命中:8 https://esm.ubuntu.com/infra/ubuntu xenial-infra-security InRelease     
获取:10 https://apt.llvm.org/xenial llvm-toolchain-xenial/main Sources [2,137 B]
获取:11 https://apt.llvm.org/xenial llvm-toolchain-xenial/main amd64 Packages [10.6 kB]
获取:13 https://esm.ubuntu.com/infra/ubuntu xenial-infra-updates InRelease [7,475 B]
命中:13 https://esm.ubuntu.com/infra/ubuntu xenial-infra-updates InRelease
获取:12 https://apt.llvm.org/xenial llvm-toolchain-xenial/main i386 Packages [1,630 B]
获取:14 https://apt.llvm.org/xenial llvm-toolchain-xenial-11/main Sources [1,614 B]
获取:15 https://apt.llvm.org/xenial llvm-toolchain-xenial-11/main amd64 Packages [8,744 B]
获取:16 https://apt.llvm.org/xenial llvm-toolchain-xenial-11/main i386 Packages [8,813 B]
获取:17 https://apt.llvm.org/xenial llvm-toolchain-xenial-12/main Sources [1,634 B]
获取:18 https://apt.llvm.org/xenial llvm-toolchain-xenial-12/main amd64 Packages [8,710 B]
获取:19 https://apt.llvm.org/xenial llvm-toolchain-xenial-12/main i386 Packages [1,629 B]
已下载 45.5 kB，耗时 7秒 (6,399 B/s)
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
有 4 个软件包可以升级。请执行 ‘apt list --upgradable’ 来查看它们。
```
4. 检查所有可用版本：`apt search clang`; 
在列表中会找到关于`clang, clang-format, clang-tidy`等的所有可用的版本（高版本会先出现在前面），例如：
```sh
clang-13/未知 1:13~++20210327080829+e5f2898bc751-1~exp1~20210327192522.3607 amd64
  C, C++ and Objective-C compiler
clang-format-13/未知 1:13~++20210327080829+e5f2898bc751-1~exp1~20210327192522.3607 amd64
  Tool to format C/C++/Obj-C code
clang-tidy-13/未知 1:13~++20210327080829+e5f2898bc751-1~exp1~20210327192522.3607 amd64
  clang-based C++ linter tool
```

5. 安装想要的`clang`版本：`sudo apt install -y clang-13`，该过程较慢。

  使用`clang-13 -v`查看版本，发现已经是`13.0.0`版本，但使用`clang -v`查看，会发现默认的还是`3.8.0`的版本，需要设置默认版本为`clang-13`。

6. 设置`clang`默认版本为`clang 13`:

   ```sh
   sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-13 100
   sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-13 100
   ```

   此时再通过`clang -v`或`clang++ -v`查看版本信息，就都是`13.0.0`版本了。

7. 安装想要的`clang-format`版本并查看版本：

   ```sh
   $ sudo apt install -y clang-format-13
   $ clang-format-13 --version
   Ubuntu clang-format version 13.0.0-++20210327080829+e5f2898bc751-1~exp1~20210327192522.3607
   
   $ sudo apt install -y clang-format
   $ clang-format --version
   Ubuntu clang-format version 13.0.0-++20210327080829+e5f2898bc751-1~exp1~20210327192522.3607
   ```

8. 安装想要的`clang-tidy`版本并查看版本；

   ```sh
   $ sudo apt install -y clang-tidy-13
   $ clang-tidy-13 --version
   LLVM (http://llvm.org/):
     LLVM version 13.0.0
     
     Optimized build.
     Default target: x86_64-pc-linux-gnu
     Host CPU: skylake
     
   $ sudo apt install -y clang-tidy
   $ clang-tidy --version
   LLVM (http://llvm.org/):
     LLVM version 13.0.0
     
     Optimized build.
     Default target: x86_64-pc-linux-gnu
     Host CPU: skylake
   ```

# clang-format

`Clang-Format`是包含在`LLVM`编译器发布包中的一个小工具，用来**格式化代码风格**。

参考：

- 官方文档：[Clang-Format Style Options](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)；【这个最全，且有示例；英文】
- [Clang-Format 使用指南](https://www.swack.cn/wiki/001558681974020669b912b0c994e7090649ac4846e80b2000/00158131059939952d487f8b3a6425fb6d945fa40a9aa7f000); 
- [团队效率工具：代码格式化之clang-format](https://blog.csdn.net/z2066411585/article/details/82290477); 

安装：见 [clang - 升级](#升级) 中的第7步；

## clang-format 选项

`.clang-format`文件具体参数设置：（参考：[Clang-Format Style Options](https://clang.llvm.org/docs/ClangFormatStyleOptions.html); 以及 [clang-format 配置文件翻译](https://blog.csdn.net/a405197809/article/details/117634686) ）

```sh
---
# 语言 ： None, Cpp, Java, JavaScript, ObjC, Proto, TableGen, TextProto
Language:        Cpp
# BasedOnStyle:  LLVM
 
# 访问权限说明符的偏移（public\private等; -4表示顶头没有缩进；感觉“IndentAccessModifiers :false”没有起作用，如果起作用了，那么AccessModifierOffset就无效了）
AccessModifierOffset: -4
 
# 开括号后的对齐（开圆括号、尖括号、方括号），
# 对齐方式有：Align, DontAlign, AlwaysBreak(总是在开括号后换行)
AlignAfterOpenBracket: Align
 
 
##################################################
# 对齐方式之 AlignConsecutiveStyle
# 连续相同操作时的对齐方式
# -- None - 不对齐
# -- Consecutive - 对齐所有连续操作（遇到空行或注释则不再对齐）
# -- AcrossEmptyLines - 在 Consecutive 的基础上，遇到空行时继续对齐，遇到注释时不再对齐
# -- AcrossComments - 在 Consecutive 的基础上，遇到注释时继续对齐，遇到空行时不再对齐
# -- AcrossEmptyLinesAndComments - 在 Consecutive 的基础上，遇到空行或注释时，继续对齐
##################################################
 
# 连续宏定义时的对齐方式 （@ref AlignConsecutiveStyle）
AlignConsecutiveMacros: Consecutive
 
# 连续赋值时的对齐方式（@ref AlignConsecutiveStyle）
AlignConsecutiveAssignments: AcrossEmptyLinesAndComments
 
# 连续位域的对齐方式（@ref AlignConsecutiveStyle）
AlignConsecutiveBitFields: Consecutive
 
# 连续声明时，变量名的对齐方式（@ref AlignConsecutiveStyle）
AlignConsecutiveDeclarations: Consecutive
 
 
# 反斜杆换行的对齐方式
# -- DontAlign - 不进行对齐
# -- Left - 反斜杠靠左对齐
# -- Right - 反斜杠靠右对齐
AlignEscapedNewlines: Left
 
 
# 二元、三元表达式的对齐方式（当表达式需要占用多行时）
# -- DontAlign - 不进行对齐
# -- Align - 从操作符开始对齐
# -- AlignAfterOperator - 从操作数开始对齐
AlignOperands:   Align
 
 
# 是否对齐行尾注释？
# -- true - 启用对齐
# -- false - 无需对齐
AlignTrailingComments: true
 
 
# 当函数调用和参数无法放在同一行时，是否将参数全部放在下一行
# -- true - 全部放在下一行
# -- false - 不全部放在下一行
AllowAllArgumentsOnNextLine: true
 
 
# 当构造函数的成员变量初始化无法放在同一行时，是否将成员变量的初始化放在下一行？
# -- true - 全部放在下一行
# -- false - 不全部放在下一行
# NOTE : 本配置仅在 ConstructorInitializerAllOnOneLineOrOnePerLine 为 true 时有效
AllowAllConstructorInitializersOnNextLine: true
 
 
# 当函数声明和参数无法放在同一行时，是否将所有参数放在下一行？
# -- true - 所有参数放在下一行
# -- false - 不全部放在下一行
AllowAllParametersOfDeclarationOnNextLine: true
 
 
# 是否允许短的枚举放在同一行？
# -- true - 允许放在同一行
# -- false - 不允许放在同一行
AllowShortEnumsOnASingleLine: false
 
 
# 是否允许短的代码块放在同一行？
# -- Never - 不允许放在同一行
# -- Empty - 只有空的代码块允许放在同一行
# -- Always - 允许短的代码块放在同一行
AllowShortBlocksOnASingleLine: Empty
 
 
# 允许短的 case 标签和语句放在同一行？
# -- true - 允许放在同一行
# -- false - 不允许放在同一行
AllowShortCaseLabelsOnASingleLine: false
 
 
# 是否允许短的函数放在同一行？
# -- None - 不把短的函数放在同一行
# -- InlineOnly - 只把类内的内联函数放在同一行，全局的空函数不放在同一行
# -- Empty - 只把空的函数放在同一行
# -- Inline - 把类内的内联函数放在同一行，全局的空函数不放在同一行
# -- All - 都允许放在同一行
AllowShortFunctionsOnASingleLine: InlineOnly
 
 
# 允许短的 lambda 语句放在同一行？
# -- None - 不把短的 lambda 语句放在同一行
# -- Empty - 只把空的lambda 语句放在同一行
# -- Inline - 当短的 lambda 语句作为函数参数时，放在同一行
# -- All - 将短的 lambda 语句放在同一行
AllowShortLambdasOnASingleLine: All
 
 
# 是否允许短的 if 语句放在同一行？
# -- Never - 不把短的if语句放在同一行
# -- WithoutElse - 当没有 else 时，把短的 if 语句放在同一行
# -- OnlyFirstIf - 把短的 if 语句放在同一行，短的 else if 和 else 不放在同一行
# -- AllIfsAndElse - 把短的 if、else if、else 语句放在同一行
AllowShortIfStatementsOnASingleLine: Never
 
 
# 是否允许短的循环放在同一行？
# -- true - 允许放在同一行
# -- false - 不允许放在同一行
AllowShortLoopsOnASingleLine: false
 
 
# 总是在定义函数的返回类型后换行？
# -- None - 由 @ref PenaltyReturnTypeOnItsOwnLine 决定
# -- All - 总是换行
# -- TopLevel - 在顶层函数（类外函数）中总是换行
# Note : 该配置已不推荐使用
AlwaysBreakAfterDefinitionReturnType: None
 
 
# 函数声明、定义时，返回值类型的换行方式
# -- None - 由 @ref PenaltyReturnTypeOnItsOwnLine 决定
# -- All - 总是在返回值类型后换行
# -- TopLevel - 仅在顶层函数（类外函数）的声明、定义中换行
# -- AllDefinitions - 在所有函数的定义中换行
# -- TopLevelDefinitions - 仅在顶层函数（类外函数）的定义中换行
AlwaysBreakAfterReturnType: None
 
 
# 是否总是在多行string前换行？
# -- true - 换行
# -- false - 不换号
AlwaysBreakBeforeMultilineStrings: false
 
 
# 模板声明的换行方式
# -- None - 由 @ref PenaltyBreakTemplateDeclaration 控制
# -- MultiLine - 仅在模板声明需要占用多行时换行
# -- Yes - 总是换行
AlwaysBreakTemplateDeclarations: MultiLine
 
 
# 定义语言或编译器的扩展属性字符串，以免 clang 无法识别
AttributeMacros:
  - __capability
 
  
# 函数调用时，参数的放置规则
# -- false - 参数要么放在同一行，要么每个参数占用一行
# -- true - 不做强制要求
BinPackArguments: false
 
 
# 函数声明、定义时，参数的放置规则
# -- false - 参数要么放在同一行，要么每个参数占用一行
# -- true - 不做强制要求
BinPackParameters: false
 
 
# 大括号放置风格
# -- Attach - 大括号紧随前方内容，放在同一行
# -- Linux - 与 Attach 类似，除了 函数、命名空间、类定义 的大括号放在下一行
# -- Mozilla - 与 Attach 类似，除了枚举、函数、结构（class\struct\union)的大括号放在下一行
# -- Stroustrup - 与 Attach 类似，但函数定义前、catch前方、else前方的 {} 放在单独一行
# -- Allman - 总是换行
# -- Whitesmiths - 类似 Allman，但 {} 和内部的语句对齐到同样位置
# -- GNU - 总是换行，但在控制语句后的{} 总是对齐到下一个位置
# -- WebKit - 与 Attach 类似，但在函数定义前换行 
# -- Custom - 依赖 @ref BraceWrapping
BreakBeforeBraces: Custom
 
 
# 大括号换行规则，只有当 @ref BreakBeforeBraces 设置为 Custom 时才有效
BraceWrapping:
  # 是否在 case 后面换行？ （true\false）
  AfterCaseLabel:  true
  
  # 是否在 class 后换行？（true\false）
  AfterClass:      true
  
  # 控制语句后的换行规则
  # -- Never - 控制语句后不换行
  # -- MultiLine - 当控制语句由多行组成时，继续换行
  # -- Always - 总是换行
  AfterControlStatement: Always
  
  # 是否在枚举定义后换行？（true\false）
  AfterEnum:       true
  
  # 是否在函数后换行？（true\false）
  AfterFunction:   true
  
  # 是否在命名空间后换行？（true\false）
  AfterNamespace:  true
  
  # 是否在 ObjC 定义后换行？（true\false）
  AfterObjCDeclaration: true
  
  # 是否在 struct 定义时换行？（true\false）
  AfterStruct:     true
  
  # 是否在 union 定义后换行？（true\false）
  AfterUnion:      true
  
  # 是否在 extern 后换行？（true\false）
  AfterExternBlock: true
  
  # 是否在 catch 前换行？（true\false）
  BeforeCatch:     true
  
  # 是否在 else 前换行？（true\false）
  BeforeElse:      true
  
  # 是否在 lambda 前换行？（true\false）
  BeforeLambdaBody: true
  
  # 是否在 do-while 的 while 前换行？（true\false）
  BeforeWhile:     false
  
  # 大括号是否参与缩进？
  IndentBraces:    false
  
  # 当空白函数的 {} 和函数名称不需要放在同一行时，是否拆分函数体？
  # -- false - {} 可以放在同一行
  # -- true - {} 分别放在两行
  SplitEmptyFunction: true
  
  # 当空白结构（class\struct\union)的 {} 需要放在单独的行时，是否拆分 {} ?
  # -- false - {} 可以放在同一行
  # -- true - {} 分别放在两行
  SplitEmptyRecord: true
  
  # 当空白的命名空间的 {} 需要放在单独的行时，是否拆分 {} ？
  # -- false - {} 可以放在同一行
  # -- true - {} 分别放在两行
  SplitEmptyNamespace: true
  
  
# 二元表达式的换行风格
# -- None - 在操作符后之后换行，操作符位于上一行尾部
# -- NonAssignment - 除赋值操作符外，其它操作符位于换行后的头部
# -- All - 所有操作符放在换行后的头部
BreakBeforeBinaryOperators: None
 
 
# 是否在 concept 声明前换行？（true\false）
BreakBeforeConceptDeclarations: true
 
 
# 类的继承列表的分割方式
# -- BeforeColon - 在冒号 ':' 前方分割，冒号位于行首，逗号','位于行尾
# -- BeforeComma - 在冒号和逗号前方分割，冒号和逗号都位于行首，并且对齐 
# -- AfterColon - 在冒号和逗号后方分割，冒号和逗号位于行尾
# -- AfterComma - 仅在逗号后方分割，冒号和首个基类位于同一行
BreakInheritanceList: BeforeComma
 
 
# ？？ 未找到介绍
BreakBeforeInheritanceComma: false
 
 
# 当三元表达式不能放在同一行时，是否在三元操作符前方换行
# -- true - 操作符位于新行的首部
# -- false - 操作符位于上一行的尾部
BreakBeforeTernaryOperators: true
 
 
# 构造函数初始化列表分割方式
# -- BeforeColon - 在冒号 ':' 前方分割，冒号位于行首，逗号','位于行尾
# -- BeforeComma - 在冒号和逗号前方分割，冒号和逗号都位于行首，并且对齐 
# -- AfterColon - 在冒号和逗号后方分割，冒号和逗号位于行尾
BreakConstructorInitializers: BeforeComma
 
 
# ？？未找到介绍
BreakConstructorInitializersBeforeComma: false
 
# 是否在每个java注解后方换行？（true\false）
BreakAfterJavaFieldAnnotations: false
 
 
# 是否分割过长的字符串？（true\false）
BreakStringLiterals: true
 
 
# 列宽长度限制
# -- 0 - 0代表没有限制
ColumnLimit:     80
 
 
# 用于匹配注释信息的正则表达式，被匹配的行不会做任何修改
CommentPragmas:  '^ IWYU pragma:'
 
 
# 是否压缩紧接的命名空间？
# -- true - 将紧跟的命名空间放在同一行
# -- false - 每个命名空间位于新的一行
CompactNamespaces: false
 
 
# 是否将构造函数的初始化列表放在同一行或各放一行？
# -- true - 如果可能，初始化列表放在同一行；如果不满足长度选择，则每个单独放一行
# -- false - 初始化列表可以随意放置
ConstructorInitializerAllOnOneLineOrOnePerLine: false
 
 
# 构造函数的初始化列表和基类集成列表的对齐宽度
ConstructorInitializerIndentWidth: 4
 
 
# 延续语句的对齐宽度
ContinuationIndentWidth: 4
 
 
# C++ 11 初始化列表风格
# -- true - 
# -- false - 
Cpp11BracedListStyle: true
 
 
# 是否自动分析行结尾方式？
# -- true - 自动分析文件的行结尾方式，若无法分析，则使用 @ref UseCRLF
# -- false - 不自动分析
DeriveLineEnding: true
 
 
# 是否自动分析指针的对齐方式？
# -- true - 自动分析并使用指针的对齐方式，若无法分析，则使用 @ref PointerAlignment
# -- false - 不自动分析
DerivePointerAlignment: false
 
 
# 是否禁用格式化？（true\false）
DisableFormat:   false
 
 
# 访问权限控制符（public\protected\private等）后方的空行规则
# -- Never - 移除所有空行
# -- Leave - 保留所有空行，受 @ref MaxEmptyLinesToKeep 限制
# -- Always - 若没有空行，则添加一个空行；若已有空行，则保留，并受 @ref MaxEmptyLinesToKeep 限制
#EmptyLineAfterAccessModifier: Never
 
 
# 访问权限控制符（public\protected\private等）前方的空行规则
# -- Never - 移除所有空行
# -- Leave - 保留所有空行
# -- LogicalBlock - 仅在新的逻辑块前方加空行
# -- Always - 除第一个外，都在前方加空行
EmptyLineBeforeAccessModifier: LogicalBlock
 
 
# 实验性功能：是否自动分析 bin pack 类型？
# NOTE : 不建议启用
ExperimentalAutoDetectBinPacking: false
 
 
# 是否自动修正命名空间的结束注释？
# -- true - 在短的命名空间尾部，自动添加或修改错误的命名空间结束注释
# -- false - 不自动修正
FixNamespaceComments: true
 
 
# 应该被解释为 foreach 循环，而不是函数调用的宏定义
ForEachMacros:
  - foreach
  - Q_FOREACH
  - BOOST_FOREACH
  
  
# 需要被忽略的宏定义，这些宏定义类似属性标签 
StatementAttributeLikeMacros:
  - Q_EMIT
 
 
# 多个 include 块（有空行分隔的include）排序时的分组规则
# -- Preserve - 保留原有的块分隔，各自排序
# -- Merge - 将所有的块视为同一个，然后进行排序
# -- Regroup - 将所有的块视为同一个进行排序，然后按照 @ref IncludeCategories 的规则进行分组
IncludeBlocks:   Preserve
 
 
# 用于 include 排序的规则
IncludeCategories:
  - Regex:           '^"(llvm|llvm-c|clang|clang-c)/'
    Priority:        2
    SortPriority:    0
    CaseSensitive:   false
  - Regex:           '^(<|"(gtest|gmock|isl|json)/)'
    Priority:        3
    SortPriority:    0
    CaseSensitive:   false
  - Regex:           '.*'
    Priority:        1
    SortPriority:    0
    CaseSensitive:   false
IncludeIsMainRegex: '(Test)?$'
IncludeIsMainSourceRegex: ''
 
 
# 是否缩进 case 标签？
# -- true - case 不与 switch 对齐
# -- false - case 和 switch 对齐
IndentCaseLabels: false
 
 
# 是否缩进 case 对应的大括号 "{}" ？
# - false - 块语句和case 标签对齐
# - true - 块语句在 case 标签后缩进
IndentCaseBlocks: false
 
 
# 是否缩进 goto 标签？
# -- false - goto 位于最左侧，不参与缩进
# -- true - goto 参与缩进
IndentGotoLabels: false
 
 
# 预处理命令(#if\#ifdef\#endif等)的缩进规则
# -- None - 不进行缩进
# -- AfterHash - 在前导'#'后缩进，'#'放在最左侧，之后的语句参与缩进
# -- BeforeHash - 在前导'#'前进行缩进
IndentPPDirectives: None
 
 
# extern 内容的缩进规则
# -- AfterExternBlock - 依赖 @ref AfterExternBlock 的配置
# -- NoIndent - 不进行缩进
# -- Indent - 进行缩进 
IndentExternBlock: NoIndent
 
 
# 是否缩进模板中的 requires？（true\false）
IndentRequires:  false
 
 
# 缩进宽度
IndentWidth:     4
 
 
# 当函数过长导致换行时，是否进行缩进？（true\false）
IndentWrappedFunctionNames: false
 
 
# 仅用于 JavaScript 的配置，需要配置为 None 
InsertTrailingCommas: None
 
 
# JavaScript 中的字符串引号规则
# -- Leave - 保持原样
# -- Single - 全部使用单引号
# -- Double - 全部使用双引号 
JavaScriptQuotes: Leave
 
 
# 是否在 JavaScript 的 import/export 语句后换行？（true\false）
JavaScriptWrapImports: true
 
 
# 是否在块的起始位置保留空行？（true\false）
# -- true - 保留块起始的空行
# -- false - 删除块起始的空行
KeepEmptyLinesAtTheStartOfBlocks: false
 
 
# 用于识别宏定义型块起始的正则表达式
MacroBlockBegin: ''
 
 
# 用于识别宏定义型块结束的正则表达式
MacroBlockEnd:   ''
 
 
# 允许保留的空行行数
MaxEmptyLinesToKeep: 1
 
 
# 命名空间内部的缩进规则
# -- None - 都不缩进
# -- Inner - 只缩进嵌套的命名空间内容
# -- All - 缩进所有命名空间内容
NamespaceIndentation: None
 
 
# Objective-C 相关配置
ObjCBinPackProtocolList: Auto
ObjCBlockIndentWidth: 2
ObjCBreakBeforeNestedBlockParam: true
ObjCSpaceAfterProperty: false
ObjCSpaceBeforeProtocolList: true
 
 
# ？？惩罚？？
PenaltyBreakAssignment: 2
PenaltyBreakBeforeFirstCallParameter: 19
PenaltyBreakComment: 300
PenaltyBreakFirstLessLess: 120
PenaltyBreakString: 1000
PenaltyBreakTemplateDeclaration: 10
PenaltyExcessCharacter: 1000000
PenaltyReturnTypeOnItsOwnLine: 60
PenaltyIndentedWhitespace: 0
 
 
# 指针（*和&）的对齐规则
# -- Left - * 靠近左侧
# -- Right - * 靠近右侧
# -- Middle - * 放在中间
# NOTE : 在 @ref SpaceAroundPointerQualifiers 为 Default，
# 	     且 @ref DerivePointerAlignment 失效后启用
PointerAlignment: Right
 
 
# 是否重排注释？（true\false）
ReflowComments:  true
 
 
# include 排序规则
# -- Never - 不进行排序
# -- CaseSensitive - 排序时大小写敏感
# -- CaseInsensitive - 排序时大小写不敏感
SortIncludes:    true
 
 
# java 中静态 import 的排序规则
# -- Before - 静态放在非静态前方
# -- After - 静态放在非静态后方
SortJavaStaticImport: Before
 
 
# 是否排序所有 using 声明？（true\false）
SortUsingDeclarations: true
 
 
# 是否在 C 类型的强制转换后加空格？（true\false）
SpaceAfterCStyleCast: false
 
 
# 是否在逻辑取反（!）后加空格？（true\false）
SpaceAfterLogicalNot: false
 
 
# 是否在 template 关键字后加空格？（true\false）
SpaceAfterTemplateKeyword: true
 
 
# 是否在赋值运算符前加空格？（true\false）
SpaceBeforeAssignmentOperators: true
 
 
# 是否在 case 的冒号前添加空格？（true\false）
SpaceBeforeCaseColon: true
 
 
# 是否在 C++11 的初始化列表前加空格？（true\false）
SpaceBeforeCpp11BracedList: false
 
 
# 是否在构造函数的初始化冒号 “:” 前加空格？（true\false）
SpaceBeforeCtorInitializerColon: true
 
 
# 是否在构造函数的继承冒号 “:” 前加空格？（true\false）
SpaceBeforeInheritanceColon: true
 
 
# 小括号“()” 前加空格的规则
# -- Never - 从不加空格
# -- ControlStatements - 只在控制语句(for/if/while...)时加空格
# -- ControlStatementsExceptForEachMacros - 类型 ControlStatements，只是不再 ForEach 后加空格
# -- Always - 总是添加空格
# -- NonEmptyParentheses - 类似 Always，只是不再空白括号前加空格  
SpaceBeforeParens: Always
 
 
# 指针前后的空格规则
# -- Default - 使用 @ref PointerAlignment 控制空格
# -- Before - 确保指针前有空格
# -- After - 确保指针后有空格
# -- Both - 确保前后都有空格
SpaceAroundPointerQualifiers: Default
 
 
# 是否在 for 循环的冒号“:” 前加空格？（true\false）
SpaceBeforeRangeBasedForLoopColon: true
 
 
# 是否在空白的 {} 中添加空格？（true\false）
SpaceInEmptyBlock: false
 
 
# 是否在空白的小括号 () 中添加空格？（true\false）
SpaceInEmptyParentheses: false
 
 
#  注释内容与注释起始符"//" 之间的空格数量
SpacesBeforeTrailingComments: 1
 
 
# 中括号 “<>” 中的空格规格
# -- Never - 移除所有空格
# -- Always - 总是添加空格
# -- Leave - 至多保留1个空格
#SpacesInAngles:  false
 
 
# 是否在条件语句（if/for/switch/while）中添加空格？
SpacesInConditionalStatement: true
 
 
# 是否在容器中添加空格？（true\false）
SpacesInContainerLiterals: true
 
 
# 是否在 C 类型的强制转换的小括号内加空格？（true\false）
SpacesInCStyleCastParentheses: false
 
 
# 是否在小括号中加空格？（true\false）
SpacesInParentheses: true
 
 
# 是否在中括号中加空格？（true\false）
# NOTE：当中括号内没有数据时，不受本规则影响（如空白的lambda 捕获表、不定长度的数组声明）
SpacesInSquareBrackets: false
 
 
# 是否在方括号前方加空格？（true\false）
# NOTE 1: lambda 捕获表不受影响
# NOTE 2: 连续的方括号，仅在第一个方括号前加空格
SpaceBeforeSquareBrackets: false
 
 
# 位域中冒号":" 的添加规则
# -- Both - 在前后都加空格
# -- None - 在前后都不加空格，除非受 @ref AlignConsecutiveBitFields 影响
# -- Before - 只在前方加空格
# -- After -- 只在后方加空格
BitFieldColonSpacing: Both
 
 
# 语言标准
Standard:        Latest
 
 
# 应被视为表达式的宏定义列表
# NOTE : 主要用于宏扩展后自带分包的语句
StatementMacros:
  - Q_UNUSED
  - QT_REQUIRE_VERSION
 
 
# tab 宽度
TabWidth:        4
 
 
# 使用使用 "\r\n" 作为换行？（true\false）
UseCRLF:         false
 
 
# 是否使用 tab
# -- Never - 从不使用
# -- ForIndentation - 仅用于缩进
# -- ForContinuationAndIndentation - 
# -- AlignWithSpaces -
# -- Always - 
UseTab:          Never
 
 
# 对括号敏感的宏定义列表，不允许修改这些宏定义
WhitespaceSensitiveMacros:
  - STRINGIZE
  - PP_STRINGIZE
  - BOOST_PP_STRINGIZE
  - NS_SWIFT_NAME
  - CF_SWIFT_NAME
...
```

或参考下面这个：

```sh
# 语言: None, Cpp, Java, JavaScript, ObjC, Proto, TableGen, TextProto #  
Language:   Cpp  
# 基础格式 #  
BasedOnStyle: Google  
# 访问说明符(public、private等)的偏移 #  
AccessModifierOffset: -4  
  
# 开括号(开圆括号、开尖括号、开方括号)后的对齐: Align, DontAlign, AlwaysBreak(总是在开括号后换行)  
AlignAfterOpenBracket:  Align  
# 连续赋值时，对齐所有等号  
AlignConsecutiveAssignments:    true  
# 连续声明时，对齐所有声明的变量名  
AlignConsecutiveDeclarations:   true  
# 左对齐逃脱换行(使用反斜杠换行)的反斜杠  
AlignEscapedNewlinesLeft:   true  
# 水平对齐二元和三元表达式的操作数  
AlignOperands:  true  
# 对齐连续的尾随的注释  
AlignTrailingComments:  true  
# 允许函数声明的所有参数在放在下一行  
AllowAllParametersOfDeclarationOnNextLine:  true  
# 允许短的块放在同一行  
AllowShortBlocksOnASingleLine:  false  
# 允许短的case标签放在同一行  
AllowShortCaseLabelsOnASingleLine:  false  
# 允许短的函数放在同一行: None, InlineOnly(定义在类中), Empty(空函数), Inline(定义在类中，空函数), All  
AllowShortFunctionsOnASingleLine:   Empty  
# 允许短的if语句保持在同一行  
AllowShortIfStatementsOnASingleLine:    false  
# 允许短的循环保持在同一行  
AllowShortLoopsOnASingleLine:   false  
# 总是在定义返回类型后换行(deprecated)  
AlwaysBreakAfterDefinitionReturnType:   None  
# 总是在返回类型后换行: None, All, TopLevel(顶级函数，不包括在类中的函数),  
#   AllDefinitions(所有的定义，不包括声明), TopLevelDefinitions(所有的顶级函数的定义)  
AlwaysBreakAfterReturnType: None  
# 总是在多行string字面量前换行  
AlwaysBreakBeforeMultilineStrings:  false  
# 总是在template声明后换行  
AlwaysBreakTemplateDeclarations:    false  
# false表示函数实参要么都在同一行，要么都各自一行  
BinPackArguments:   true  
# false表示所有形参要么都在同一行，要么都各自一行  
BinPackParameters:  true  
# 大括号换行，只有当BreakBeforeBraces设置为Custom时才有效  
BraceWrapping:  
	# class定义后面  
	AfterClass: false  
	# 控制语句后面  
	AfterControlStatement:  false  
	# enum定义后面  
	AfterEnum:  false  
	# 函数定义后面  
	AfterFunction:  false  
    # 命名空间定义后面  
    AfterNamespace: false  
    # ObjC定义后面  
    AfterObjCDeclaration:   false  
    # struct定义后面  
    AfterStruct:    false  
    # union定义后面  
    AfterUnion: false  
    # catch之前  
    BeforeCatch:    true  
    # else之前  
    BeforeElse: true  
    # 缩进大括号  
    IndentBraces:   false  
	# 在二元运算符前换行: None(在操作符后换行), NonAssignment(在非赋值的操作符前换行), All(在操作符前换行)  
	BreakBeforeBinaryOperators: NonAssignment  
# 在大括号前换行: Attach(始终将大括号附加到周围的上下文), Linux(除函数、命名空间和类定义，与Attach类似),  
#   Mozilla(除枚举、函数、记录定义，与Attach类似), Stroustrup(除函数定义、catch、else，与Attach类似),  
#   Allman(总是在大括号前换行), GNU(总是在大括号前换行，并对于控制语句的大括号增加额外的缩进), WebKit(在函数前换行), Custom  
#   注：这里认为语句块也属于函数  
BreakBeforeBraces:  Custom  
# 在三元运算符前换行  
BreakBeforeTernaryOperators:    true  
# 在构造函数的初始化列表的逗号前换行  
BreakConstructorInitializersBeforeComma:    false  
# 每行字符的限制，0表示没有限制  
ColumnLimit:    200  
# 描述具有特殊意义的注释的正则表达式，它不应该被分割为多行或以其它方式改变  
CommentPragmas: '^ IWYU pragma:'  
# 构造函数的初始化列表要么都在同一行，要么都各自一行  
ConstructorInitializerAllOnOneLineOrOnePerLine: false  
# 构造函数的初始化列表的缩进宽度  
ConstructorInitializerIndentWidth:  4  
# 延续的行的缩进宽度  
ContinuationIndentWidth:    4  
# 去除C++11的列表初始化的大括号{后和}前的空格  
Cpp11BracedListStyle:   false  
# 继承最常用的指针和引用的对齐方式  
DerivePointerAlignment: false  
# 关闭格式化  
DisableFormat:  false  
# 自动检测函数的调用和定义是否被格式为每行一个参数(Experimental)  
ExperimentalAutoDetectBinPacking:   false  
# 需要被解读为foreach循环而不是函数调用的宏  
ForEachMacros:  [ foreach, Q_FOREACH, BOOST_FOREACH ]  
# 对#include进行排序，匹配了某正则表达式的#include拥有对应的优先级，匹配不到的则默认优先级为INT_MAX(优先级越小排序越靠前)，  
#   可以定义负数优先级从而保证某些#include永远在最前面  
IncludeCategories:  
- Regex:    '^"(llvm|llvm-c|clang|clang-c)/'  
Priority:   2  
- Regex:    '^(<|"(gtest|isl|json)/)'  
Priority:   3  
- Regex:    '.*'  
Priority:   1  
# 缩进case标签  
IndentCaseLabels:   false  
  
# 缩进宽度 #  
IndentWidth: 4  
  
# 函数返回类型换行时，缩进函数声明或函数定义的函数名  
IndentWrappedFunctionNames: false  
# 保留在块开始处的空行  
KeepEmptyLinesAtTheStartOfBlocks:   true  
# 开始一个块的宏的正则表达式  
MacroBlockBegin:    ''  
# 结束一个块的宏的正则表达式  
MacroBlockEnd:  ''  
# 连续空行的最大数量  
MaxEmptyLinesToKeep:    1  
# 命名空间的缩进: None, Inner(缩进嵌套的命名空间中的内容), All  
NamespaceIndentation:   Inner  
# 使用ObjC块时缩进宽度  
ObjCBlockIndentWidth:   4  
# 在ObjC的@property后添加一个空格  
ObjCSpaceAfterProperty: false  
# 在ObjC的protocol列表前添加一个空格  
ObjCSpaceBeforeProtocolList:    true  
# 在call(后对函数调用换行的penalty  
PenaltyBreakBeforeFirstCallParameter:   19  
# 在一个注释中引入换行的penalty  
PenaltyBreakComment:    300  
# 第一次在<<前换行的penalty  
PenaltyBreakFirstLessLess:  120  
# 在一个字符串字面量中引入换行的penalty  
PenaltyBreakString: 1000  
# 对于每个在行字符数限制之外的字符的penalty  
PenaltyExcessCharacter: 1000000  
# 将函数的返回类型放到它自己的行的penalty  
PenaltyReturnTypeOnItsOwnLine:  60  
# 指针和引用的对齐: Left, Right, Middle  
PointerAlignment:   Left  
# 允许重新排版注释  
ReflowComments: true  
# 允许排序#include  
SortIncludes:   true  
# 在C风格类型转换后添加空格  
SpaceAfterCStyleCast:   false  
# 在赋值运算符之前添加空格  
SpaceBeforeAssignmentOperators: true  
# 开圆括号之前添加一个空格: Never, ControlStatements, Always  
SpaceBeforeParens:  ControlStatements  
# 在空的圆括号中添加空格  
SpaceInEmptyParentheses:    false  
# 在尾随的评论前添加的空格数(只适用于//)  
SpacesBeforeTrailingComments:   2  
# 在尖括号的<后和>前添加空格  
SpacesInAngles: true  
# 在容器(ObjC和JavaScript的数组和字典等)字面量中添加空格  
SpacesInContainerLiterals:  true  
# 在C风格类型转换的括号中添加空格  
SpacesInCStyleCastParentheses:  true  
# 在圆括号的(后和)前添加空格  
SpacesInParentheses:    true  
# 在方括号的[后和]前添加空格，lamda表达式和未指明大小的数组的声明不受影响  
SpacesInSquareBrackets: true  
# 标准: Cpp03, Cpp11, Auto  
Standard:   Cpp11  
# tab宽度  
TabWidth:   4  
# 使用tab字符: Never, ForIndentation, ForContinuationAndIndentation, Always  
UseTab: Never  
```

### 自用

后期熟悉`clang-format`的用法及相应参数时，**维护**一个自己常用的编码风格的格式文件`.clang-format`。（下面的`.clang-format`已经运行过，注意不要使用汉字）

```sh
# Sean's own format

Language:   Cpp
BasedOnStyle:   Google
AccessModifierOffset:   -4
#AlignAfterOpenBracket: DontAlign
AlignConsecutiveAssignments: true
AlignConsecutiveBitFields: true 
AlignConsecutiveDeclarations: true
AlignConsecutiveMacros: true 
AlignEscapedNewlines: Right
AlignOperands:   Align
AlignTrailingComments: true
AllowAllArgumentsOnNextLine: true
AllowAllParametersOfDeclarationOnNextLine: true
AllowShortBlocksOnASingleLine: Never
AllowShortCaseLabelsOnASingleLine: false
AllowShortEnumsOnASingleLine: false
AllowShortFunctionsOnASingleLine: false
AllowShortIfStatementsOnASingleLine: Never
AllowShortLambdasOnASingleLine: All
AllowShortLoopsOnASingleLine: false
AlwaysBreakAfterReturnType: None
AlwaysBreakBeforeMultilineStrings: false
AlwaysBreakTemplateDeclarations: Yes
BinPackArguments: true
BinPackParameters: true
BitFieldColonSpacing: Both
BraceWrapping:
  AfterCaseLabel:  true
  AfterClass:      true
  AfterControlStatement: Always
  AfterEnum:       true
  AfterFunction:   true
  AfterNamespace:  true
  AfterObjCDeclaration: true
  AfterStruct:     true
  AfterUnion:      true
  AfterExternBlock: true
  BeforeCatch:     true
  BeforeElse:      true
  BeforeLambdaBody: false
  BeforeWhile:     true
  IndentBraces:    false        # not confirm
  SplitEmptyFunction: true
  SplitEmptyRecord: true
  SplitEmptyNamespace: true
BreakAfterJavaFieldAnnotations: true
BreakBeforeBinaryOperators: None
BreakBeforeBraces: Custom
BreakBeforeConceptDeclarations: true
BreakBeforeTernaryOperators: false
BreakConstructorInitializers: AfterColon
BreakInheritanceList: AfterColon
BreakStringLiterals: true
ColumnLimit:     120
CommentPragmas:  '^ IWYU pragma:'
CompactNamespaces: false
ConstructorInitializerIndentWidth: 4
ContinuationIndentWidth: 4
Cpp11BracedListStyle: true
DeriveLineEnding: true
DerivePointerAlignment: false
DisableFormat:   false
EmptyLineBeforeAccessModifier: Always
ExperimentalAutoDetectBinPacking: false
FixNamespaceComments: true
ForEachMacros:  # not confirm
  - foreach
  - Q_FOREACH
  - BOOST_FOREACH
IncludeBlocks:   Regroup
IncludeCategories:  # not confirm
  - Regex:           '^"(llvm|llvm-c|clang|clang-c)/'
    Priority:        2
    SortPriority:    2
    CaseSensitive:   true
  - Regex:           '^((<|")(gtest|gmock|isl|json)/)'
    Priority:        3
  - Regex:           '<[[:alnum:].]+>'
    Priority:        4
  - Regex:           '.*'
    Priority:        1
    SortPriority:    0
IncludeIsMainRegex: '(Test)?$'  # not confirm
IncludeIsMainSourceRegex: ''  # not confirm
# IndentAccessModifiers: false   # not confirm
IndentCaseBlocks: false
IndentCaseLabels: false
IndentExternBlock: AfterExternBlock
IndentGotoLabels: true
IndentPPDirectives: None
IndentWidth: 4
IndentWrappedFunctionNames: false
KeepEmptyLinesAtTheStartOfBlocks: false
MacroBlockBegin: ''
MacroBlockEnd:   ''
MaxEmptyLinesToKeep: 1
NamespaceIndentation: None
ObjCBinPackProtocolList: Auto
ObjCBlockIndentWidth: 4
ObjCBreakBeforeNestedBlockParam: true
ObjCSpaceAfterProperty: false
ObjCSpaceBeforeProtocolList: true
PenaltyBreakAssignment: 2
PenaltyBreakBeforeFirstCallParameter: 19
PenaltyBreakComment: 300
PenaltyBreakFirstLessLess: 120
PenaltyBreakString: 1000
PenaltyBreakTemplateDeclaration: 10
PenaltyExcessCharacter: 1000000
PenaltyReturnTypeOnItsOwnLine: 60
PointerAlignment: Right
ReflowComments:  true
SortIncludes:    CaseInsensitive
SortUsingDeclarations: true
SpaceAfterCStyleCast: false
SpaceAfterLogicalNot: false
SpaceAfterTemplateKeyword: false
SpaceBeforeAssignmentOperators: true
SpaceBeforeCaseColon: false
SpaceBeforeCpp11BracedList: true
SpaceBeforeCtorInitializerColon: true
SpaceBeforeInheritanceColon: true
SpaceBeforeParens: ControlStatements
SpaceBeforeRangeBasedForLoopColon: true
SpaceBeforeSquareBrackets: false
SpaceInEmptyBlock: false
SpaceInEmptyParentheses: false
SpacesBeforeTrailingComments: 4
SpacesInAngles:  false
SpacesInConditionalStatement: false
SpacesInContainerLiterals: false
SpacesInCStyleCastParentheses: false
SpacesInParentheses: false
SpacesInSquareBrackets: false
Standard:        Latest
StatementMacros:
  - Q_UNUSED
  - QT_REQUIRE_VERSION
TabWidth:        4
UseCRLF:         false
UseTab:          Never
WhitespaceSensitiveMacros:
  - STRINGIZE
  - PP_STRINGIZE
  - BOOST_PP_STRINGIZE
```



## Linux下使用

- 使用**内置**风格：可选项为`LLVM, Google, Chromium, Mozilla, WebKit`; 

  ```sh
  $ clang-format -style=Google -i test.cpp
  ```

  表示：使用`Google`编码风格对`test.cpp`文件进行代码格式化，结果直接写到`test.cpp`中；`-i`表示**将格式化后的内容写入原文件**。

  对指定行格式化：

  ```sh
  $ clang-format -lines=1:2 test.cpp
  ```

- 使用**参数文件**指定风格：即`-style`的值为`file`; 

  ```sh
  $ clang-format -style=file -i test.cpp
  ```

  传入参数`-style-file`时，它会在当前目录寻找`.clang-format`格式文件，找不到就依次向上一层目录寻找，直到找到为止。

  使用自定义风格时，可以按如下步骤操作：

  - Step 1: 首先生成`.clang-format`文件；

    ```sh
    # clang-format -style=风格名 -dump-config > 文件名
    $ clang-format -style=Google -dump-config > .clang-format
    ```

  - Step 2: 修改`.clang-format`文件中的参数：

    根据[clang-format选项](#clang-format选项)中的参数说明，修改参数，以适应自己的要求。

如果当前目录或上层目录中已有了`.clang-format`文件，则可以直接使用：

```sh
$ clang-format -i test.cpp
```

# clang-tidy

`clang-tidy`是`clang`编译器的代码分析工具，既可以查找代码中的静态错误，也可以提示可能会在运行时发生的问题，以及通过代码分析给出可提升程序性能的建议。

安装：

- 见 [clang - 升级](#升级) 中的第8步；
- [clang-tidy 静态代码分析框架](https://inktea.xyz/2022/b47a.html); 
- [clang-tidy静态语义检查，安装、使用、检查项注解](https://blog.csdn.net/Fenplan/article/details/119755111);

使用：

```sh
// 列出所有的check
$ clang-tidy -list-checks

// 找出simple.cc中所有没有用到的using declarations. 后面的`--`表示这个文件不在compilation database里面，可以直接单独编译；
$ clang-tidy -checks="-*,misc-unused-using-decls" path/to/simple.cc --

// 找出simple.cc中所有没有用到的using declarations并自动fix(删除掉)
$ clang-tidy -checks="-*,misc-unused-using-decls" -fix path/to/simple.cc --

// 找出a.c中没有用到的using declarations. 这里需要path/to/project/compile_commands.json存在
$ clang-tidy -checks="-*,misc-unused-using-decls" path/to/project/a.cc
```

常用的方法：

`clang-tidy -checks="-*,xxxx" main.cpp --`;

使用时，位于`main.cpp`所在目录并输入上述命令，其中`xxxx`表示某种检查规则（如果是`-checks="*"`表示对所有项进行检查，可能会出现太多警告），这些规则可以参考[clang-tidy checks](https://clang.llvm.org/extra/clang-tidy/checks/list.html); 以其中的`modernize-use-nullptr`为例：

如果代码中有一个指针是`lru_cache = 0`,在使用clang-tidy后如下所示：

```sh
sean@sean-virtual-machine:~/xxx/cache$ clang-tidy -checks="-*,modernize-use-nullptr" main.cpp --
58 warnings generated.
/home/sean/xxx/cache/main.cpp:26:17: warning: use nullptr [modernize-use-nullptr]
    lru_cache = 0;
                ^
                nullptr
Suppressed 57 warnings (57 in non-user code).
Use -header-filter=.* to display errors from all non-system headers. Use -system-headers to display errors from system headers as well.
```

即提示空指针不能是`NULL`或`0`，而应该是`nullptr`；如果加上`-fix-errors`参数，则可以用来改正该错误:

```sh
sean@sean-virtual-machine:~/xxx/cache$ clang-tidy -checks="-*,modernize-use-nullptr" -fix main.cpp --
58 warnings generated.
/home/sean/xxx/cache/main.cpp:26:17: warning: use nullptr [modernize-use-nullptr]
    lru_cache = 0;
                ^
                nullptr
/home/sean/xxx/cache/main.cpp:26:17: note: FIX-IT applied suggested code changes
clang-tidy applied 1 of 1 suggested fixes.
Suppressed 57 warnings (57 in non-user code).
Use -header-filter=.* to display errors from all non-system headers. Use -system-headers to display errors from system headers as well.

sean@sean-virtual-machine:~/xxx/cache$ clang-tidy -checks="-*,modernize-use-nullptr" -fix-errors main.cpp --
57 warnings generated.
Suppressed 57 warnings (57 in non-user code).
Use -header-filter=.* to display errors from all non-system headers. Use -system-headers to display errors from system headers as well.
```

经测试，修复错误时需要使用这两个命令：(即先`-fix`再`-fix-errors`，两次`-fix-errors`也成功修复了)

```sh
$ clang-tidy -checks="-*,modernize-use-nullptr" -fix main.cpp --
$ clang-tidy -checks="-*,modernize-use-nullptr" -fix-errors main.cpp --
```





# cpplint

`cpplint`是Google开发的**C++代码风格检查工具**，用来检查代码是否符合`Google's C++ Style Guide`；可以当做静态代码分析工具，可以查找代码中错误、违反约定和建议修改的地方。

该工具就是个`python`脚本，可从 [Github - cpplint](https://github.com/google/styleguide/blob/gh-pages/cpplint/cpplint.py) 上下载。注意：`cpplint.py`是基于`python 2`的脚本。

参考：

- [cpplint工具使用过程](https://www.cnblogs.com/happyamyhope/p/11301527.html); 
- [Google代码规范工具Cpplint的使用](https://blog.csdn.net/Csdn_Darry/article/details/72791475);

## Linux下使用

### 安装

方式1：通过 [Github - cpplint](https://github.com/google/styleguide/blob/gh-pages/cpplint/cpplint.py) 下载，得到`cpplint.py`文件；使用时可采用`$ python cpplint.py temp.cpp`的方式；（但这种方式略显麻烦）

方式2：使用`pip`安装(`pip`是`python`包管理工具，该工具提供了对`python`包的查找、下载、安装和卸载功能)；使用时可采用`$ cpplint temp.cpp`方式；使用方式2安装时遇到过的问题：

- 使用`$pip install cpplint`安装时安装失败并提示`pip`版本低（Ubuntu 16.04 虚拟机）：

  ```sh
  You are using pip version 8.1.1, however version 22.1 is available.
  You should consider upgrading via the 'pip install --upgrade pip' command.
  ```

  然后使用它推荐的`$ pip install --upgrade pip`命令依然会出现上述问题。

  解决方法1：使用命令`sudo -H python -m pip install --upgrade pip`；参考：[ubuntu16.04 更新pip](https://blog.csdn.net/Kangyucheng/article/details/79978953); 

  但该方法未解决我的情况，报下面错误：

  ```sh
  $ sudo -H python -m pip install --upgrade pip
  Collecting pip
    Downloading https://files.pythonhosted.org/packages/99/bb/696e256f4f445809f25efd4e4ce42ff99664dc089cafa1e097d5fec7fc33/pip-22.1.tar.gz (2.1MB)
      100% |████████████████████████████████| 2.1MB 45kB/s 
      Complete output from command python setup.py egg_info:
      Traceback (most recent call last):
        File "<string>", line 1, in <module>
        File "/tmp/pip-build-xs6o0g/pip/setup.py", line 7
          def read(rel_path: str) -> str:
                           ^
      SyntaxError: invalid syntax
  ```

  解决方法2：去 [官网下载](https://pypi.org/project/pip/#files) `pip`文件，再手动解压、安装；（参考 [Ubuntu16.04 python pip版本过低，更新一直出错](https://blog.csdn.net/moguchen/article/details/84502708) ）

  但解压（`tar -zvxf pip-xxx.tar.gz`）安装（进入解压后的目录`cd pip-xxx`，再`sudo python setup.py install`）时也会报类似上述的错误：

  ```sh
  sean@sean-virtual-machine:~/download/pip-22.1$ python setup.py install
    File "setup.py", line 7
      def read(rel_path: str) -> str:
                       ^
  SyntaxError: invalid syntax
  ```

  上述语法错误的解决方法：（参考 [pip升级报错：def read(rel_path: str) -＞ str SyntaxError: invalid syntax](https://blog.csdn.net/sunkangke/article/details/123530119) ）

  ```sh
  python -m pip install --user --upgrade pip==20.2.4
  /usr/bin/python -m pip install --upgrade pip
  ```

  结果如下：

  ```sh
  $ python -m pip install --user --upgrade pip==20.2.4
  Collecting pip==20.2.4
    Downloading https://files.pythonhosted.org/packages/cb/28/91f26bd088ce8e22169032100d4260614fc3da435025ff389ef1d396a433/pip-20.2.4-py2.py3-none-any.whl (1.5MB)
      100% |████████████████████████████████| 1.5MB 41kB/s 
  Installing collected packages: pip
  Successfully installed pip-8.1.1
  You are using pip version 8.1.1, however version 22.1 is available.
  You should consider upgrading via the 'pip install --upgrade pip' command.
  sean@sean-virtual-machine:~/workspace/cpp/deletable/SHNet$ /usr/bin/python -m pip install --upgrade pip
  DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
  Defaulting to user installation because normal site-packages is not writeable
  Collecting pip
    Downloading pip-20.3.4-py2.py3-none-any.whl (1.5 MB)
       |████████████████████████████████| 1.5 MB 32 kB/s 
  Installing collected packages: pip
    Attempting uninstall: pip
      Found existing installation: pip 20.2.4
      Uninstalling pip-20.2.4:
        Successfully uninstalled pip-20.2.4
  Successfully installed pip-20.3.4
  ```

  终于把`pip`安装成功了，此时继续用`$pip install cpplint`安装`cpplint`；又报下面的错误：

  ```sh
  $ pip2 install cpplint
  DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
  Defaulting to user installation because normal site-packages is not writeable
  Collecting cpplint
    Using cached cpplint-1.6.0.tar.gz (361 kB)
      ERROR: Command errored out with exit status 1:
       command: /usr/bin/python -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-ppKaie/cpplint/setup.py'"'"'; __file__='"'"'/tmp/pip-install-ppKaie/cpplint/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base /tmp/pip-pip-egg-info-mC0NZr
           cwd: /tmp/pip-install-ppKaie/cpplint/
      Complete output (23 lines):
      Couldn't find index page for 'pytest-runner' (maybe misspelled?)
      No local packages or download links found for pytest-runner==5.2
      Traceback (most recent call last):
        ...
      ----------------------------------------
  ERROR: Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
  ```

  上述问题的核心是：`Couldn't find index page for 'pytest-runner'`，进一步安装`pytest-runne`：

  ```sh
  sean@sean-virtual-machine:~$ pip install pytest-runner
  DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
  Defaulting to user installation because normal site-packages is not writeable
  Collecting pytest-runner
    Downloading pytest_runner-5.2-py2.py3-none-any.whl (6.8 kB)
  Installing collected packages: pytest-runner
  Successfully installed pytest-runner-5.2
  ```

  成功后再安装`cpplint`：

  ```sh
  sean@sean-virtual-machine:~$ pip2 install cpplint
  DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
  Defaulting to user installation because normal site-packages is not writeable
  Collecting cpplint
    Using cached cpplint-1.6.0.tar.gz (361 kB)
  Building wheels for collected packages: cpplint
    Building wheel for cpplint (setup.py) ... done
    Created wheel for cpplint: filename=cpplint-1.6.0-py2-none-any.whl size=78208 sha256=268fd464aee85db577388ac7707a1f3f7fcf79cf857af3e0c216f45e6daf2af0
    Stored in directory: /home/sean/.cache/pip/wheels/33/00/8a/90118bbb8d930b8c1a6f40a9b9f6dfa8ec02d2111c67bdd8b3
  Successfully built cpplint
  Installing collected packages: cpplint
  Successfully installed cpplint-1.6.0
  ```

  可以使用`cpplint --help`查看使用方法。

### 使用

- 检测单个文件：

  temp.cpp内容如下：

  ```c++
  #include <iostream>
  
  void Print()
  {
      std::cout << "Hello world." << std::endl;
  }
  
  int main()
  {
      Print();
      return 0;
  }
  ```

  运行及结果：

  ```sh
  sean@sean-virtual-machine:~/workspace/SHNet$ python cpplint.py temp.cpp 
  temp.cpp:0:  No copyright message found.  You should have a line: "Copyright [year] <Copyright Owner>"  [legal/copyright] [5]
  temp.cpp:4:  { should almost always be at the end of the previous line  [whitespace/braces] [4]
  temp.cpp:9:  { should almost always be at the end of the previous line  [whitespace/braces] [4]
  Done processing temp.cpp
  Total errors found: 3
  ```

  或者:`$ python2 cpplint.py temp.cpp`，即指明是`python2`。

  修改后：

  ```c++
  // Copyright (c) 2022 Sean. All rights reserved.
  void Print(){
      std::cout << "Hello world." << std::endl;
  }
  
  int main(){
      Print();
      return 0;
  }
  ```

  使用`$ cpplint temp.cpp`(把当前目录的`cpplint.py`文件去掉)，结果如下：

  ```sh
  sean@sean-virtual-machine:~/workspace/SHNet$ cpplint temp.cpp
  Done processing temp.cpp
  ```

- 检测批量文件：

  根据要检测的文件编写`shell`脚本，然后运行。示例如下：

  ```sh
  #! /bin/bash
  echo "^@^cpplint code style check through shell====^@^"
  index=0
  config=""
  pwd_path=`pwd`
  cpplint_path="$pwd_path/cpplint.py"
  echo cpplint_path=$cpplint_path
  
  src_path="$pwd_path/src"
  echo src_path=$src_path
  # add file to an array,and check file in array last
  # for file in `find $src_path -name "*.h" -type f`
  for file in `find $src_path -maxdepth 1 -type f | grep -E "\.h$|\.cc$|\.cu$|\.cpp$"`
  do
      echo file=$file
      echo -e "\033[36m ===> [FILE] \033[0m \033[47;31m $file \033[0m"
      check_files[$index]=$file
      index=$(($index+1))
  done
  # accroding to config options make a checking command
  # first check if python2 exists
  check_cmd=""
  is_python2_exists=`ls /usr/bin/ | grep -E '^python2$' | wc -l`
  if [ $is_python2_exists -ge 1 ]
  then
      # read cpplint.ini to decide which option to add
      for file in ${check_files[*]}
      do
          check_cmd="python2 $cpplint_path --linelength=80"
          echo -e "\033[33m =========== check file $file =============\033[0m"
          check_cmd="$check_cmd"$file
          eval $check_cmd
          echo -e "\033[45m ==========================================\033[0m"
      done
  fi
  ```

  运行：`$sudo bash cpplint_shell.sh`;
  
- 自定义使用

  参考：[cpplint中filter参数的每个可选项的含义](https://blog.csdn.net/albertsh/article/details/118076005); [代码风格审查工具Cpplint](https://cloud.tencent.com/developer/article/1494003); 

  使用`cpplint --help`可以得到其详细语法说明：

  ```sh
  Syntax: cpplint.py [--verbose=#] [--output=emacs|eclipse|vs7|junit|sed|gsed]
                     [--filter=-x,+y,...]
                     [--counting=total|toplevel|detailed] [--root=subdir]
                     [--repository=path]
                     [--linelength=digits] [--headers=x,y,...]
                     [--recursive]
                     [--exclude=path]
                     [--extensions=hpp,cpp,...]
                     [--includeorder=default|standardcfirst]
                     [--quiet]
                     [--version]
          <file> [file] ...
  ```

  注意第2行的`[--filter=-x,+y,...]`，可以使用这种命令进行自定义地过滤。

  常见报错原因有：

  1. Tab found; better to use spaces 没有使用四个空格代替缩进
  2. Lines should be <= 80 characters long 存在大于80字符的行
  3. Should have a space between // and comment 应该在//和注释之间有一个空格
  4. An else should appear on the same line as the preceding }
  5. If an else has a brace on one side, it should have it on both [readability/braces] 上两个错误经常一起出现，为大括号的位置不合规范
  6. Extra space for operator ++; ++符号和变量间不能有空格
  7. Redundant blank line at the end of a code block should be deleted. 代码块最后的空行应该被删除
  8. Line contains invalid UTF-8 (or Unicode replacement character) 使用了中文注释报的错
  9. Line ends in whitespace. 代码行最后存在空格
  10. Include the directory when naming .h files  [build/include_subdir] [4]

  以第10种情况为例，可以按要求进行修改（比如，修改前是 `#include "server.h"`，修改后是`#include "./server.h"`，所在文件与`server.h`位于同一个`include`目录下）；或者对该报错进行飞屏蔽，方法可以是：

  - 在报错语句所在行后面添加注释`// NOLINT`；如`#include "server.h"    // NOLINT`; 【注意`NOLINT`需是大写】
  - 输入y`cpplint`命令时，添加`--filter`条件过滤；如：`$ cpplint --filter=-build/include_subdir,+readability/braces test.h` ;其中`-`号表示屏蔽，`+`号表示取消屏蔽，`build/include_subdir`就是出错的标识（中括号中间的内容），多个条件时中间以逗号`,`隔开。

## cpplint 与 clang-tidy 区别

参考：[clang-tidy 静态代码分析框架](https://hokein.github.io/clang-tools-tutorial/clang-tidy.html); 

- cpplint 
  - 是一个`python`脚本，采用**正则表达式匹配**找出违反`Google code style`的代码，故其检测功能受限于正则表达式，无法检测出所有违反`style`的地方，而且还会出现`False positive`和`True positive`。
  - 不需要源文件编译，对文件内容直接正则匹配，运行更快；
  - 除了`MacOS`和`Linux`，还支持`Windows`；
- clang-tidy
  - 基于抽象语法树（AST）对源文件进行分析，分析结果**更准确**，检测到的问题更多；
  - 需要对源代码进行语法分析（编译源文件），故它需要知道源文件的编译命令；对于依赖较多较大的文件，花费时间会较长；
  - 每次只针对一个编译单元（`Translation Unit`，可理解为一个`*.cpp`文件）进行静态分析，因此只能查出一个编译单元里面的代码问题，对于只在跨编译单元出现的问题，则无能为力；
  - 只支持`MacOS`和`Linux`；
