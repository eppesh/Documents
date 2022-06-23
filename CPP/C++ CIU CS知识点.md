[toc]

说明：《C++ Coding Interview University》中的知识点汇总只是罗列和概述，本系列是对其中知识点展开学习时的笔记。为了方便，本系列中`Coding Interview University`用`CIU`代指。

本系列主要分为以下三大部分：

- C++ CIU 编程知识点：记录编程相关的建议、工具、原则等；
- C++ CIU CS知识点：需要在日后学习工作中不断学习和巩固的各项知识点；
- C++ CIU 面试知识点：跟面试相关的经验建议等（一般不包括“CS知识点”中的内容）；

---

注意事项：

- 每个知识点都会有“**先把书读薄，后把书读厚，再把书读薄**”的过程，**Patience & Persistence**;

  > 先把书读薄：略读，对其有整体/大概的印象；
  >
  > 后把书读厚：精读，深入理解其中的具体实现原理；
  >
  > 再把书读薄：归纳总结；

- 英文参考链接（特别是视频）如果一下子无法理解，可搜索其中文关键词了解大概，通过中英文不同资料交替理解；

- 很多知识点会随着自己能力的提升而变得更易理解，因此在某个知识点上卡太久时，不妨跳过，等积累一段时间后再看；

- 不是必须要按**顺序**去学习这些知识点，可结合实际情况选择合适的学习次序；

- 量变才会引起质变，很多情况下，“量”并不够；

- 学习完**要有输出**；笔记、博客、讲解视频、公众号等，如果能**把该知识点教会另一个人**，说明掌握了；

---

# 计算机如何处理程序*

> 对于未完成的知识点，会在`*`号标注，方便查询。

- [ ] **计算机是如何处理一段程序：**
  - [x] [CPU 是如何执行代码（视频）](https://www.youtube.com/watch?v=XM4lGflQFvA)
  - [ ] [计算机如何计算（视频）](https://youtu.be/1I5ZMmrOfnA)
  - [ ] [寄存器和内存（视频）](https://youtu.be/fpnE6UAfbtU)
  - [ ] [中央处理单元（视频）](https://youtu.be/FZGugFqdr60)
  - [ ] [指令和程序（视频）](https://youtu.be/zltgXvg6r3k)

## CPU如何执行代码

参考：

- [B站 Fetch-Decode-Execute Cycle 5min](https://www.bilibili.com/video/BV1Ab4y1Q7bH); （同原YouTube视频）
- [B站 你知道CPU是如何执行你的指令的吗 6min](https://www.bilibili.com/video/BV1VL411x7w5); 
- [CPU执行程序的秘密，藏在了这15张图里](https://cloud.tencent.com/developer/article/1729507);
- [CPU内部组成结构及指令执行过程](https://blog.csdn.net/u010926964/article/details/45693773);
- [寄存器地址和内存地址_通俗易懂和你聊聊寄存器那些事(精美图文)](https://blog.csdn.net/weixin_39719127/article/details/111330968); 
- [CPU基础知识-CPU的组成 运算器、控制器、寄存器](https://www.cnblogs.com/gnivor/p/15679241.html)；
- [CPU内部结构图](https://blog.csdn.net/jiuyueguang/article/details/9350793);



参考图：

![pic](https://img-blog.csdnimg.cn/fd355cb76be444fa873254736a179ff6.png#pic_center)

![CPU内部结构图](https://images.cnitblog.com/blog/170763/201305/06095529-d0a85d069ba94f6faa85860013972f11.png)

部分术语：

CPU主要由运算器、控制器和寄存器组成（可以理解为“运算”、“控制”和“存储”三个模块）。

- 运算器：负责算术/逻辑运算；一般包括1个算术逻辑单元(ALU)和至少3个寄存器；

  - 算术逻辑单元(Arithmetic Logic Unit, **ALU**);

  - 累加寄存器(Accumulator, ACC)，又简称为"AC"，是一个通用寄存器；

    > 累加器的功能是：当运算器的算术逻辑单元ALU执行算术或逻辑运算时，为ALU提供一个工作区，可以为ALU暂时保存一个操作数或运算结果。
    > 显然，运算器中至少要有一个累加寄存器。

  - 乘商寄存器(Multiplier-Quotient,MQ);

  - 操作数寄存器(X);

- 控制器：

  - 控制单元(Control Unit, **CU**): 根据指令所需完成的操作和信号，发出各种微操作命令序列，用以控制所有被控对象，完成指令的执行;

  - 程序计数寄存器(Program Counter Register, **PC**): 又称“指令地址寄存器”，存放下一条要执行指令的内存地址。

    > 在程序执行之前，首先必须将程序的首地址，即程序第一条指令所在主存单元的地址送入PC，因此PC的内容即是从主存提取的第一条指令的地址。
    > 当执行指令时，CPU能自动递增PC的内容，使其始终保存将要执行的下一条指令的主存地址，为取下一条指令做好准备。
    >
    > 若为单字长指令，则(PC)+1àPC，若为双字长指令，则(PC)+2àPC，以此类推。【暂时还不太懂】
    >
    > 但是，当遇到转移指令时，下一条指令的地址将由转移指令的地址码字段来指定，而不是像通常的那样通过顺序递增PC的内容来取得。
    > 因此，程序计数器的结构应当是具有寄存信息和计数两种功能的结构。

  - 指令寄存器(Instruction Register, **IR**): 存放当前正在执行的一条指令，存放的内容来自于数据寄存器(DR)。

    > 当执行一条指令时，首先把该指令从主存读取到数据寄存器中，然后再传送至指令寄存器。
    >
    > 指令包括操作码和地址码两个字段，为了执行指令，必须对操作码进行测试，识别出所要求的操作，指令译码器（Instruction Decoder，ID）就是完成这项工作的。指令译码器对指令寄存器的操作码部分进行译码，以产生指令所要求操作的控制电位，并将其送到微操作控制线路上，在时序部件定时信号的作用下，产生具体的操作控制信号。
    >
    > 指令寄存器中操作码字段的输出就是指令译码器的输入。操作码一经译码，即可向操作控制器发出具体操作的特定信号。

  - 指令译码器(Instruction Decoder, **ID**): 对指令进行解码。

    > 在计算机执行一条指令的时候，必须首先分析这条指令的操作码是什么，以决定操作的性质和方法，然后控制计算机的其他各部件协同完成指令表达的功能；这中间的分析工作就是指令译码器(ID)完成的。

  - 时序产生器(Timing Generator, TG): 给计算机各部分提供工作所需的时间标志。

    > 类似于“时间作息表”，一般利用定时脉冲的顺序和不同的脉冲间隔来实现。

- 寄存器：一种有限存储容量的高速存储部件；可以用来**暂存指令/数据/地址**，既可以传达控制器的命令给运算器，又可以帮运算器记录待处理/已处理完的数据。

  在CPU中至少有以下几类寄存器：

  - 指令寄存器(Instruction Register, **IR**)（见上文）

  - 程序计数寄存器(Program Counter Register, **PC**)（见上文）

  - 累加寄存器(Accumulator, **ACC**)（见上文）

  - 程序状态字寄存器(Program Status Word Register, PSW): 用来表征当前运算的状态及程序的工作方式。

    > 程序状态字寄存器用来保存由算术/逻辑指令运行或测试的结果所建立起来的各种条件码内容，如运算结果进/借位标志（C）、运算结果溢出标志（O）、运算结果为零标志（Z）、运算结果为负标志（N）、运算结果符号标志（S）等，这些标志位通常用1位触发器来保存。
    >
    > 除此之外，程序状态字寄存器还用来保存中断和系统工作状态等信息，以便CPU和系统及时了解机器运行状态和程序运行状态。
    >
    > 因此，程序状态字寄存器是一个保存各种状态条件标志的寄存器。

  - 存储地址寄存器(Memory Address Register, **MAR**)，又简称“**地址寄存器**(**AR**)": 用来保存CPU当前所访问的主存单元的地址。

    This register is used to **access data and instructions from memory** during the **execution phase** of instruction. 

    MAR既用来保存CPU将要取的数据的内存地址，又可保存CPU将要将数据写入的内存地址；

    MAR **holds the address** of a memory block in which read or write.

    > 由于在主存和CPU之间存在操作速度上的差异，所以必须使用地址寄存器来暂时保存主存的地址信息，直到主存的存取操作完成为止。
    >
    > 当CPU和主存进行信息交换，即CPU向主存存入数据/指令或者从主存读出数据/指令时，都要使用地址寄存器和数据寄存器。
    >
    > 如果我们把外围设备与主存单元进行统一编址，那么，当CPU和外围设备交换信息时，我们同样要使用地址寄存器和数据寄存器。

  - 存储缓存寄存器(Memory Buffer Register, MBR) / 存储数据寄存器(Memory Data Register, MDR)，又简称”**数据寄存器(DR) / 缓存寄存器(BR)**": 其主要功能是作为CPU和主存、外设之间信息传输的中转站，用以弥补CPU和主存、外设之间操作速度上的差异。

    It is the [register](https://en.wikipedia.org/wiki/Processor_register) in a computer's [CPU](https://en.wikipedia.org/wiki/Central_processing_unit) that stores the data being transferred to and from the immediate access storage. (Primary storage is also known as Immediate Access Storage and is where data is stored on the main computer memory) 

    > 数据寄存器用来暂时存放由主存储器读出的一条指令或一个数据字；反之，当向主存存入一条指令或一个数据字时，也将它们暂时存放在数据寄存器中。
    > 数据寄存器的作用是 ：
    > （1）作为CPU和主存、外围设备之间信息传送的中转站；
    > （2）弥补CPU和主存、外围设备之间在操作速度上的差异；
    > （3）在单累加器结构的运算器中，数据寄存器还可兼作操作数寄存器。

  > MAR 与 MDR 区别：MDR **holds data** waiting to be written in or data read from the location pointed by the MAR.
  >
  > MAR：存放的是要写的/要读的数据的内存地址，其位数对应存储单元的个数；
  >
  > MDR：存放的是要写的/要读的数据，而这些数据的内存地址存放在MAR中；

- 内存管理单元（Memory Management Unit，MMU）：负责跟内存进行通信（处理CPU的内存访问请求）；

  > 功能包括虚拟地址与物理地址的转换等；

- 地址总线（Address Bus）；

- 数据总线（Data Bus）；

- 控制总线（Control Bus）；

CPU执行程序过程：

- 取指令（Fetch）：把要执行的指令装载到“指令寄存器(IR)”的过程；

  CPU读取“程序计数器(PC)“的值，然后CPU的”控制单元(CU)"操作地址总线指定需要访问的内存地址，接着通知内存设备准备数据；数据准备好后通过数据总线将指令数据传给数据寄存器(DR)，然后再存入到“指令寄存器(IR)”；

  > Fetch: PC -> AR -> Memory -> DR -> IR;

- 译码（Decode）：对指令进行解码；

  CPU分析“指令寄存器(IR)”中的指令，进行解码，确定指令的类型和参数；如果是计算类型的指令，就把指令交给ALU运算，如果是存储类型的指令，则交由CU执行；

- 执行（Execute）：执行指令；

  CPU执行完指令后，“程序计数器(PC)“的值自增（表示指向下一条指令）；

- 数据回写（Store）：CPU将计算结果存回寄存器，或将寄存器的值存入内存；

上述4个过程称为**指令周期（Instruction Cycle）**。

---

# 算法复杂度

算法复杂度 / Big-O / 渐进分析法。

- 并不需要实现
- 这里有很多视频，看到你真正了解它为止。你随时可以回来复习。
- 如果这些课程太过数学的话，你可以去看看最下面离散数学的视频，它能让你更了解这些数学背后的来源以及原理。
- [ ] [Harvard CS50 —— 渐进表示（视频）](https://www.youtube.com/watch?v=iOq5kSKqeR4)
- [ ] [Big O 记号（通用快速教程）（视频）](https://www.youtube.com/watch?v=V6mKVRU1evU)
- [ ] [Big O 记号（以及 Omega 和 Theta）——  最佳数学解释（视频）](https://www.youtube.com/watch?v=ei-A_wy5Yxw&index=2&list=PL1BaGV1cIH4UhkL8a9bJGG356covJ76qN)
- [ ] Skiena 算法：
  - [视频](https://www.youtube.com/watch?v=gSyDMtdPNpU&index=2&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b)
  - [幻灯片](http://www3.cs.stonybrook.edu/~algorith/video-lectures/2007/lecture2.pdf)
- [ ] [对于算法复杂度分析的一次详细介绍](http://discrete.gr/complexity/)
- [ ] [增长阶数（Orders of Growth）（视频）](https://class.coursera.org/algorithmicthink1-004/lecture/59)
- [ ] [渐进性（Asymptotics）（视频）](https://class.coursera.org/algorithmicthink1-004/lecture/61)
- [ ] [UC Berkeley Big O（视频）](https://youtu.be/VIS4YDpuP98)
- [ ] [UC Berkeley Big Omega（视频）](https://youtu.be/ca3e7UVmeUc)
- [ ] [平摊分析法（Amortized Analysis）（视频）](https://www.youtube.com/watch?v=B3SpQZaAZP4&index=10&list=PL1BaGV1cIH4UhkL8a9bJGG356covJ76qN)
- [ ] [举证“Big O”（视频）](https://class.coursera.org/algorithmicthink1-004/lecture/63)
- [ ] TopCoder（包括递归关系和主定理）：
  - [计算性复杂度：第一部](https://www.topcoder.com/community/data-science/data-science-tutorials/computational-complexity-section-1/)
  - [计算性复杂度：第二部](https://www.topcoder.com/community/data-science/data-science-tutorials/computational-complexity-section-2/)
- [ ] [速查表（Cheat sheet）](http://bigocheatsheet.com/)

---

# 数据结构

## 数组（Arrays）

目标：实现一个可自动调整大小的动态数组。（比如：手动实现`std::vector`）

- [x] 介绍：
  - [数组（视频）](https://www.coursera.org/learn/data-structures/lecture/OsBSF/arrays)
  - [UC Berkeley CS61B - 线性数组和多维数组（视频）](https://archive.org/details/ucberkeley_webcast_Wp8oiO_CZZE)（从15分32秒开始）
  - [动态数组（视频）](https://www.coursera.org/learn/data-structures/lecture/EwbnV/dynamic-arrays)
  - [不规则数组（视频）](https://www.youtube.com/watch?v=1jtrQqYpt7g)
- [ ] 实现一个动态数组（可自动调整大小的可变数组）：
  - [ ] 练习使用数组和指针去编码，并且指针是通过计算去跳转而不是使用索引
  - [ ] 通过分配内存来新建一个原生数据型数组
    - 可以使用 int 类型的数组，但不能使用其语法特性
    - 从大小为16或更大的数（使用2的倍数 —— 16、32、64、128）开始编写
  - [ ] size() —— 数组元素的个数
  - [ ] capacity() —— 可容纳元素的个数
  - [ ] is_empty()
  - [ ] at(index) —— 返回对应索引的元素，且若索引越界则愤然报错
  - [ ] push(item)
  - [ ] insert(index, item) —— 在指定索引中插入元素，并把后面的元素依次后移
  - [ ] prepend(item) —— 可以使用上面的 insert 函数，传参 index 为 0
  - [ ] pop() —— 删除在数组末端的元素，并返回其值
  - [ ] delete(index) —— 删除指定索引的元素，并把后面的元素依次前移
  - [ ] remove(item) —— 删除指定值的元素，并返回其索引（即使有多个元素）
  - [ ] find(item) —— 寻找指定值的元素并返回其中第一个出现的元素其索引，若未找到则返回 -1
  - [ ] resize(new_capacity) // 私有函数
    - 若数组的大小到达其容积，则变大一倍
    - 获取元素后，若数组大小为其容积的1/4，则缩小一半
- [ ] 时间复杂度
  - 在数组末端增加/删除、定位、更新元素，只允许占 O(1) 的时间复杂度（平摊（amortized）去分配内存以获取更多空间）
  - 在数组任何地方插入/移除元素，只允许 O(n) 的时间复杂度
- [ ] 空间复杂度
  - 因为在内存中分配的空间邻近，所以有助于提高性能
  - 空间需求 = （大于或等于 n 的数组容积）* 元素的大小。即便空间需求为 2n，其空间复杂度仍然是 O(n)

---

### 一维数组示例

Sieve of Eratosthenes (埃氏质数筛选算法):

参考：[LeetCode - Count Primes](https://leetcode.cn/problems/count-primes/); (Given an integer `n`, return the number of prime numbers that are strictly less than `n`.)

```c++
int countPrimes(int n) 
{
    /* 普通数组
    bool *prime = new bool[n + 1];
    for (int i = 2; i <= n; ++i)
    {
        prime[i] = true;
    }
    */
    std::vector<bool> prime(n+1, true); // 不确定是否是质数时,为true; 确定一定不是质数时,为false
    for(int divisor = 2; divisor*divisor <= n; ++divisor)
    {
        if(prime[divisor])
        {
            for(int i = 2*divisor; i<=n; i+=divisor)
            // for( int i = divisor*divisor; i<=n; i+=divisor)	// 进一步优化
            {
                prime[i] = false;
            }
        }
    }
    int count = 0;
    for(int i=2; i<n; ++i)
    {
        if(prime[i])
        {
            count++;
        }
    }
    return count;
}
```

说明：

1. 埃氏质筛法思路：质数`divisor`的倍数`i*divisor (i>=2)`一定不是质数，即`divisor`的2倍、3倍、4倍、...、`i`倍一定不是质数。先假设所有数都可能是质数，然后过滤掉所有**一定不是质数**的数，剩下的就一定是质数。因此，算法的一个核心目标就是**寻找那些一定不是质数的数**。

2. 如何理解内层循环`for(int i = 2*divisor; i<=n; i+=divisor)`?

   - 如上面所说，**质数`divisor`的2倍、3倍、4倍、...、`i`倍一定不是质数**。当`divisor`是质数时，循环初始值`2*divisor`表示该质数的2倍，下一次循环时`i  = 2*divisor + divisor = 3*divisor`表示该质数的3倍，这样一直循环到超过给定的范围上限为止。

   - 可以发现，对每一个质数`divisor`，我们都是把它的“2倍、3倍、4倍、...、`i`倍”数进行过滤，即：

   | 质数 | 质数的2倍,3倍,4倍, ... i 倍 ( i>=2)                    |
   | ---- | ------------------------------------------------------ |
   | 2    | **4**, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, ... , 2*i |
   | 3    | 6, **9**, 12, 15, 18, 21, 24, 27, ... , 3*i            |
   | 5    | 10, 15, 20, **25**, ... , 5*i                          |
   | 7    | 14, 21, 28, 35, ... , 7*i                              |

   - 在上述表格中会发现存在很多重复，因此，还可以进一步优化：`for( int i = divisor*divisor; i<=n; i+=divisor)`;

     这是因为，比`divisor*divisor`小的合数，在之前的质数倍数中已经被过滤掉了，无须重复过滤。

3. 为何外层循环要使用`divisor*divisor <= n`而不是`divisor <= n`?

   - 如果一个数`divisor`一定不是质数，那么必有其对应的`n/divisor`也一定不是质数，它们俩互为数字`n`的两个因子。显然这两个因子，只需要判断其中一个一定不是质数即可，剩下那个就无须重复判断了。

   - 从优化遍历次数方面考虑，对`divisor`和`n/divisor`两者中较小的那个进行判断时可以减少遍历次数。虽然无法直观比较这两个因子大小，但可以发现较小的那个因子一定是位于`[1, √n]`这个区间内。

     > 反证：如果`divisor > √n`，那么`1/divisor < 1/√n`，进而`n/divisor < n/√n = √n`，即`n/divisor < √n`；

   - 因此，外层循环的条件只须`diviosr * diviso <= n`即可。

   | n    | d    |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
   | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
   | 36   | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   | 12   | 13   | 14   | 15   | 16   | 17   | 18   | 19   | 20   |
   |      | 18   | 12   | 9    | x    | 6    | x    | x    | x    | x    | x    | 3    | x    | x    | x    | x    | x    | 2    | x    | x    |

### 二维数组示例

Pascal's Triangle (杨辉三角):

参考：[LeetCode - pascals-triangle](https://leetcode.cn/problems/pascals-triangle/); (Given an integer `numRows`, return the first numRows of Pascal's Triangle.)

```c++
vector<vector<int>> generate(int numRows) 
{
    vector<vector<int>> res(numRows);
    for(int i=0; i<numRows; ++i)
    {
        res[i] = vector<int>(i+1);
        res[i][0] = 1;
        for(int j=1; j<i; ++j)
        {
            res[i][j] = res[i-1][j-1] + res[i-1][j];
        }
        res[i][i] = 1;
    }
    return res;
}
// 不使用vector时
int** PascalTriangle(int n)
{
    int **p = new int*[n];
    for (int i=0; i< n; ++i)
    {
        p[i] = new int[i + 1];
        p[i][0] = 1;
        std::cout << p[i][0];
        for (int j = 1; j < i; ++j)
        {
            p[i][j] = p[i - 1][j - 1] + p[i - 1][j];
            std::cout << p[i][j];
        }
        p[i][i] = 1;
        std::cout << p[i][i] << std::endl;
    }
    return p;
}
```

### 动态数组实现

即手写`std::vector`中主要方法。一些知识点：

#### 扩容

动态数组的“动态”主要提现在可以自动扩容上。当容量不足时，动态数组会自动扩容；但当容量有很大富余时一般不进行缩容，本例中有缩容要求，因此在特定条件下也会动态缩减容量。

自动扩容时的增长因子：即扩容倍数；`新容量 = 旧容量 * 增长因子`；一般取`[1,2]`，VS2019用的是1.5，GCC用的是2；

缩减因子：`新容量 = 旧容量 / 缩减因子`;

> 参考: 
>
> - [HashMap的初始容量(initialCapacity)和装载因子(loadFactor)](https://blog.csdn.net/sunxuefen2009/article/details/17247833); 
> - [java 中使用v8引擎_深入V8引擎-AST(5)](https://blog.csdn.net/weixin_35755562/article/details/114516746);
> - [ArrayList, Vector, HashMap, HashSet的默认初始容量,加载因子,扩容增量](https://www.cnblogs.com/xiezie/p/5511840.html); 
> - [C++ STL vector扩容原理分析](https://blog.51cto.com/u_15127635/4165976); 
> - [HashMap中的初始容量和加载因子](https://blog.csdn.net/weixin_44723496/article/details/112387738); 
> - [HashMap中的初始容量和加载因子到底是表示什么意思呢?](https://blog.csdn.net/weixin_44560342/article/details/106119595); 

扩容两种情况：

1. 根据构造函数时的入参设置初始容量(initial_capacity)：调用`int DetermineCapacity(int capacity)`函数返回处理后的容量；
2. 数组使用过程中容量不足需要扩容：直接`新容量 = 旧容量 * 增长因子`；但为了统一，也包含在了`int DetermineCapacity(int capacity)`函数中。

> 当需要扩容时，无论是上述哪种情况，直接调用`DeterminCapacity`函数即可；

缩减容量只有1种情况：根据要求，当数组元素个数小于等于`容积 / 缩减因子`时，将容积缩减一半。直接`新容量 = 旧容量 / 2`;

```c++
static const int kMinCapacity = 16;			// 最小容量
static const int KGrowthFactor = 2;         // 增长因子；新容量 = 旧容量 * 增长因子
static const int kShrinkFactor = 4;			// 缩减因子；缩容条件满足：当前元素个数 <= 当前容量 / 缩减因子；

// 第一次调用该函数时（即构造函数时），参数capacity又称为初始容量(initial_capacity)
// 后面扩容/缩容前调用该函数时，
int DetermineCapacity(int capacity) const
{
    int true_capacity = kMinCapacity;

    while (capacity > true_capacity / KGrowthFactor)
    {
        true_capacity *= KGrowthFactor;
    }

    return true_capacity;
}
```

上述函数的作用：格式化+扩容；

即，将指定容量(`capacity`)格式化为符合“标准”内存大小（即`2^n`，如`16,32,64,128,256,512,1024...`）的“真正”容量；（因为分配的内存大小一般为`2^n`，所以如果输入的初始容量为`7、13、33`这种大小，就会调整为相应的`2^n`）

- 如果入参`capacity`已经是`2^n`，则计算结果等于`capacity * kGrowthFactor`；

- 如果入参`capacity`不是`2^n`，则计算后的结果会取“第一个大于`capacity`的那个`2^n`的`kGrowthFactor`倍”，结果不足`kMinCapacity`时取`kMinCapacity`；

  > 详细说明：
  >
  > - 先找到第一个比`capacity`大的`2^n`类型的数，它的**2倍**作为“真正”容量；
  >
  >   如：假设`kMinCapacity = 32, capacity=28`，第一个比28大的`2^n`就是32，那么32的2倍64就是“真正”容量；
  >
  > - 如果2倍后的结果小于“最小容量(`kMinCapacity`)”，则“最小容量”就是“真正”容量；
  >
  >   如：假设`kMinCapacity = 128, capacity=28`，第一个比28大的`2^n`就是32，但`32 < kMinCapacity`，因此“最小容量”128就是“真正”容量；

Sean注：1. 对于初始容量，因为它可能不是2^n，因此需要用该函数去调整；2. 对于非初始容量，它已经通过上一步变成2^n了，因此后续再调用该函数进行扩容只是简单的`kGrowthFactor`倍*原容量（即，当形参是2^n时，只是简单的乘以增长因子即可）；3. 该函数是把这2点整合到了一个函数里。
