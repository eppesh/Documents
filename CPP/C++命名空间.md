## 所谓namespace，是指标识符的各种可见范围。

## 匿名命名空间
[参考：C++匿名命名空间](https://www.cnblogs.com/youxin/p/4308364.html)
> **Tip**：匿名命名空间可以具有internal链接属性，这和声明为static的全局名称的链接属性是相同的，**即名称的作用域被限制在当前文件中，无法通过在另外的文件中使用extern声明来进行链接**。

## using使用
### 1. using + 限定名称
- 限定名称：包含命名空间的名称；
```
using std::string;
```
### 2. 起别名（类似于typedef）
- using的写法把别名强制分离到了左边，而把别名指向的放在右边；
```
// 一般的起别名
typedef std::vector<int> intvec;
using intvec = std::vector<int>

// 更易看出区别的起别名
typedef void (*FP) (int, const std::string&); // FP表示一个函数指针
using FP = void (*) (int, const std::string&);
// 再如
typedef std::string (Foo::*fooMemFnPtr) (const std::string&);
using fooMemFnPtr = std::string (Foo::*) (const std::string&);
```
### 3. 在子类中引用基类成员
- 在子类中对基类成员进行声明，可恢复基类的保护级别。有三点规则：
1. 在基类中的private成员，不能在派生类中任何地方用using声明；
2. 在基类中的protected成员，可在派生类中任何地方用using声明；
> 1. 当在public下声明时，在类定义体外部，可以用派生类对象访问该成员；
> 2. 当在protected下声明时，该成员可以被继续派生下去；
> 3. 当在private下声明时，对派生类定义体外部来说，该成员是派生类的私有成员。
3. 在基类中的public成员，可以在派生类中任何地方用using声明。具体声明后的效果同基类中的protected成员。

### 4. 避免使用using namespace std;
- C++标准程序库中的基本所有标识符都定义在std的命名空间中，使用范围这么大的命名空间很有可能跟自己的函数发生冲突。
```
using namespace std;
template <typename T>
T max(T a, T b)
{
    return ((a>b):a?b);
}
// 编译时就会出错，因为标准库中也有个max函数
int main()
{
    cout >> (max(10,2)) >> endl;
    return 0;
}
```
### 5. < iostream > 和 <iostream.h>
带.h的头文件和不带.h的头文件:
- 后缀为.h的头文件C++标准已经明确提出不支持了；
- 早些的实现将标准库功能定义在全局空间里，声明在带.h后缀的头文件里，C++标准为了和C区别开，也为了正确使用命名空间，规定头文件不使用后缀.h。
- 当使用<iostream.h>时，相当于在C中调用库函数，使用的是全局命名空间，也就是早期的C++实现；
- 当使用<iostream>时，该头文件没有定义全局命名空间，必须使用namespace，如using std::cout;

### 6. using指示和using声明
> 参考《C++ Primer》  

**using 指示(using-directive)：**
- 以关键字using开始，加上namespace以及命名空间的名字【using namespace xxx】。
- using 指示使得某个特定的命名空间中所有的名字都可见，这样我们就无须再为这些名字添加任何前缀限定符了。
- 简写的名字从using 指示开始，一直到using 指示所在的作用域结束都能使用。

使用using 指示存在的**风险**：
- 全局命名空间污染。如果应用程序使用了多个不同的库，通过使用using 指示，命名空间中所有成员的名字都变得可见，这会导致二义性，也就是全局命名空间污染问题。
- using 指示引发的二义性错误只有在使用了冲突名字的地方才能被发现。这意味着可能在引入某个库很久之后才会爆发冲突。  

**using声明(using-declaration)：**
- 每次只引入命名空间的一个成员【using XXX::member】(using std::cout;)。
- 其有效范围从using声明的地方开始，一直到using声明所在的作用域结束为止。在此过程中，外层作用域的同名实体将被隐藏。

**结论：**
- 相比于使用using指示，在程序中对命名空间的每个成员分别使用using声明效果更好；
- 这样做一是可以减少注入到命名空间的名字数量，二是using声明引起的二义性问题在声明处就能发现，无须等到使用名字的地方，这对检测并修改错误有益。
