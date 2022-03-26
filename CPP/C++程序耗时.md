[toc]

# 前言

> 1秒(Second) = 10^3毫秒(millisecond) = 10^6微妙(microsecond) = 10^9纳秒(nanosecond)

在编程过程中，经常会需要计算某个操作的耗时情况。根据对精度的要求不同，精度通常分为**秒级**、**毫秒级**和**微秒级**，某些特定场景（如极速行情等）甚至需要**纳秒级**或更高。本文总结常用的各种精度下的常用方法，供日后参考使用。

## 总览

在记录程序耗时的时候，总是希望提高精度。一般地，建议使用`steady_clock`；但想提高精度时，建议使用`QPC - Windows`和`gettimeofday - Linux`；

| 方法             | Windows | Linux | 精度                | 备注              |
| ---------------- | ------- | ----- | ------------------- | ----------------- |
| time()           | √       | √     | 秒                  |                   |
| clock()          | √       | √     | 毫秒                |                   |
| steady_clock     | √       | √     | 秒&毫秒&微秒(&纳秒) | C++11             |
| timeGetTime()    | √       | ×     | 毫秒                | 需要添加库;不推荐 |
| GetTickCount64() | √       | ×     | 毫秒                |                   |
| QPC              | √       | ×     | 微秒                | 推荐              |
| gettimeofday()   | ×       | √     | 微秒                | 推荐              |
| timespec         | √       | √     | 纳秒                | C++17             |

下文用到的一个模拟耗时操作的函数：（以下代码在`Win10 + VS2019`和`Ubuntu16 - g++ 9.4.0`上验证，下同）

```c++
// 一个耗时的示例函数
double DoSomething(int n)
{
    double sum = 0;
    for (int i = 0; i < n; ++i)
    {
        sum += sqrt(i);
    }
    return sum;
}
```

# 1. 秒

**秒级**精度。

## 1.1 time()

**原型**：`time_t time(time_t* timer);`

The value returned generally represents **the number of seconds** since 00:00 hours, Jan 1, 1970 UTC (i.e., the current *unix timestamp*). 

**头文件**：`#include <ctime>`

**适用平台**：Windows, Linux; 

**精度**：精确到**秒**；

计算耗时：配合 [difftime](http://www.cplusplus.com/reference/ctime/difftime/) （`double difftime (time_t end, time_t beginning);` Calculates the difference in seconds between beginning and end.）使用；

示例：

```c++
#include <cmath>       	// sqrt
#include <iostream>     // cout, endl
#include <ctime>       	// time_t, time, difftime

int main()
{
    time_t start_time = time(nullptr);
    std::cout << DoSomething(100000000) << std::endl;
    time_t end_time = time(nullptr);
    double time_spent = difftime(end_time, start_time);	// 需先end_time再start_time,否则结果是-3s
    std::cout << "time(): " << time_spent << " seconds" << std::endl;
    system("pause");
    return 0;
}
```

输出：

```c++
6.66667e+11
time(): 1 seconds			// Windows Release,Linux
time(): 1 seconds			// Linux
```

# 2. 毫秒

## 2.1 clock()

**原型**：`clock_t clock (void);`

Returns the processor time consumed by the program.

**头文件**：`#include <ctime>`

**适用平台**：Windows, Linux; 

**精度**：精确到**毫秒**；

计算耗时：得到前后时间差后，再**除以`CLOCKS_PER_SEC`**，得到耗时（精确到**`1/CLOCKS_PER_SEC`秒**）；

> `CLOCKS_PER_SEC`在Windows和Linux中的值分别是`1000`和`1000000`; (以`Win10 64bit`和`Ubuntu16 64bit`为例)

示例：

```c++
#include <cmath>       	// sqrt
#include <iostream>     // cout, endl
#include <ctime>       	// clock_t, clock, CLOCKS_PER_SEC

int main()
{
    clock_t start_time = clock();
    std::cout << DoSomething(100000000) << std::endl;
    clock_t end_time = clock();
    clock_t time_spent = end_time - start_time;
    std::cout << "clock(): " << ((float)time_spent)/CLOCKS_PER_SEC << " seconds" << std::endl;
    system("pause");
    return 0;
}
```

输出：

```c++
6.66667e+11
clock(): 1.183 seconds			// Windows Release
clock(): 0.677401 seconds		// Linux
```

## 2.2 steady_clock

[`chrono`库](http://www.cplusplus.com/reference/chrono/?kw=chrono) 是C++11新增的一个时间相关的库，`chrono`即`chronology`(年表;年代学)的简写。

参考：[C++日期和时间编程](https://paul.pub/cpp-date-time/); 

**头文件**：`#include <chrono>`

**适用平台**：Windows, Linux; 

**精度**：精确到**秒/毫秒/微秒/纳秒**；

理论上通过调整`std::chrono::duration_cast<std::chrono::milliseconds>`可以精确到秒、毫秒、微秒和纳秒。

示例：

```c++
#include <chrono>       // steady_clock, duration_cast
#include <cmath>        // sqrt
#include <iostream>     // cout, endl

int main()
{
    //std::chrono::time_point<std::chrono::steady_clock> start_time = std::chrono::steady_clock::now();
    auto start_time = std::chrono::steady_clock::now();

    std::cout << DoSomething(100000000) << std::endl;
    
    auto end_time = std::chrono::steady_clock::now();
    auto time_diff = end_time - start_time;
    auto elapsed_time = std::chrono::duration_cast<std::chrono::milliseconds>(time_diff).count();
    std::cout << "chrono: " << elapsed_time << " milliseconds" << std::endl;
    system("pause");
    return 0;
}
```

输出：

```c++
6.66667e+11
chrono: 996 milliseconds		// Windows
chrono: 678 milliseconds		// Linux
```

## 2.3 timeGetTime()

**原型**：`DWORD timeGetTime();`

The [timeGetTime function](https://docs.microsoft.com/en-us/windows/win32/api/timeapi/nf-timeapi-timegettime) retrieves the system time, in milliseconds. The system time is the time elapsed since Windows was started.

**适用平台**：Windows; 

**精度**：精确到**毫秒**；

使用时需要添加：

> 1. #include <windows.h>
> 2. #pragma comment(lib, "winmm.lib")

Note that the value returned by the **timeGetTime** function is a **DWORD** value. The return value wraps around to 0 every 2^32 milliseconds, which is about 49.71 days. This can cause problems in code that directly uses the **timeGetTime** return value in computations, particularly where the value is used to control code execution. You should always use the difference between two **timeGetTime** return values in computations. 【Sean注：比较时间需检查溢出情况】

示例：

```c++
#include <cmath>        // sqrt
#include <iostream>     // cout, endl
#include <windows.h>	// timeGetTime
#pragma comment(lib, "winmm.lib")		// timeGetTime

int main()
{
    DWORD start_time = timeGetTime();
    std::cout << DoSomething(100000000) << std::endl;
    DWORD end_time = timeGetTime();
    auto elapsed_time = end_time - start_time;
    std::cout << "timeGetTime(): " << elapsed_time << " milliseconds" << std::endl;
    system("pause");
    return 0;
}
```

输出：

```c++
6.66667e+11
timeGetTime(): 1049 milliseconds
```

## 2.4 GetTickCount64()

**原型**：`ULONGLONG GetTickCount64();`

The [GetTickCount64 function](https://docs.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-gettickcount64) Retrieves the number of milliseconds that have elapsed since the system was started.

**头文件**：`#include <windows.h>`

**适用平台**：Windows; 

**精度**：精确到**毫秒**；

示例：

```
#include <cmath>        // sqrt
#include <iostream>     // cout, endl
#include <windows.h>	// GetTickCount64

int main()
{
    unsigned long long start_time = GetTickCount64();
    std::cout << DoSomething(100000000) << std::endl;
    unsigned long long end_time = GetTickCount64();
    auto elapsed_time = end_time - start_time;
    std::cout << "GetTickCount64(): " << elapsed_time << " milliseconds" << std::endl;
    system("pause");
    return 0;
}
```

输出：

```c++
6.66667e+11
GetTickCount64(): 1203 milliseconds
```

# 3. 微秒

## 3.1 QPC

在Windows平台，参考 [Acquiring high-resolution time stamps](https://docs.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps) 获得高精度时间戳，其中 [**QueryPerformanceCounter (QPC)**](https://docs.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) 和 [QueryPerformanceFrequency](https://docs.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency) 在`Windows XP`及之后的系统中总是支持的。

**适用平台**：Windows; 

**精度**：精确到**微秒**；

示例：

```c++
#include <cmath>        // sqrt
#include <iostream>     // cout, endl
int main()
{
    LARGE_INTEGER start_time;
    LARGE_INTEGER end_time;
    LARGE_INTEGER elapsed_miscroseconds;
    LARGE_INTEGER frequency;                    // the number of ticks-per-second

    QueryPerformanceFrequency(&frequency);
    QueryPerformanceCounter(&start_time);

    std::cout << DoSomething(100000000) << std::endl;
    
    QueryPerformanceCounter(&end_time);
    elapsed_miscroseconds.QuadPart = end_time.QuadPart - start_time.QuadPart;   // the elapsed number of ticks
    
    //
	// We now have the elapsed number of ticks, along with the
	// number of ticks-per-second. We use these values
	// to convert to the number of elapsed microseconds.
	// To guard against loss-of-precision, we convert
	// to microseconds *before* dividing by ticks-per-second.
	//
    
    elapsed_miscroseconds.QuadPart *= 1000000;
    elapsed_miscroseconds.QuadPart /= frequency.QuadPart;

    std::cout << "QPC(): " << elapsed_miscroseconds.QuadPart << " microseconds" << std::endl;
    system("pause");
    return 0;
}
```

输出：

```c++
6.66667e+11
QPC(): 1228136 microseconds
```

## 3.2 gettimeofday()

**原型**：`int gettimeofday(struct timeval *restrict tv, struct timezone *restrict tz);`

```c++
struct timeval 
{
    time_t      tv_sec;     /* seconds */
    suseconds_t tv_usec;    /* microseconds */
};
struct timezone 
{
    int tz_minuteswest;     /* minutes west of Greenwich */
    int tz_dsttime;         /* type of DST correction */
};
```

The [gettimeofday() function](https://man7.org/linux/man-pages/man2/settimeofday.2.html) shall obtain the current time, expressed as seconds and microseconds since the Epoch, and store it in the **timeval** structure pointed to by *tp*.

**头文件**：`#include <sys/time.h>`

**适用平台**：Linux; 

**精度**：精确到**微秒**；

示例1：

```c++
#include <cmath>        // sqrt
#include <iostream>     // cout, endl
#include <sys/time.h>	// timeval, gettimeofday

int main()
{
    timeval start_time;
    gettimeofday(&start_time,nullptr);
    
    std::cout << DoSomething(100000000) << std::endl;
    
    timeval end_time;
    gettimeofday(&end_time,nullptr);
    
    auto time_diff_sec = end_time.tv_sec - start_time.tv_sec;
    auto time_diff_usec = end_time.tv_usec - start_time.tv_usec;
	auto elapsed_time = time_diff_sec*1000000 + time_diff_usec; 
    
    std::cout << "gettimeofday(): " << elapsed_time << " microseconds" << std::endl;

    return 0;
}
```

示例2：（利用`gettimeofday`封装成一个辅助函数）

```c++
#include <cmath>        // sqrt
#include <iostream>     // cout, endl
#include <sys/time.h>	// timeval, gettimeofday

// 获取当前时间的微秒数
long long GetNowTimeMicroseconds()
{
    timeval now;
    gettimeofday(&now, nullptr);
    return (now.tv_sec * 1000000 + now.tv_usec);
}

int main()
{
    long long start_time = GetNowTimeMicroseconds();
    
    std::cout << DoSomething(100000000) << std::endl;
    
    auto end_time = GetNowTimeMicroseconds();        
	auto elapsed_time = end_time - start_time; 
    
    std::cout << "gettimeofday(): " << elapsed_time << " microseconds" << std::endl;

    return 0;
}
```

输出：

```c++
6.66667e+11
gettimeofday(): 681585 microseconds
```

# 4. 纳秒

## 4.1 timespec

[`timespec`](https://en.cppreference.com/w/c/chrono/timespec) 是C++17新增的功能。

参考：[C++日期和时间编程](https://paul.pub/cpp-date-time/); [知乎 - C++日期和时间编程](https://zhuanlan.zhihu.com/p/373392670); 

**适用平台**：Windows, Linux; 

**精度**：精确到**纳秒**；

示例：

```c++
#include <cmath>        // sqrt
#include <iostream>     // cout, endl

int main()
{
    timespec start_time;
    timespec_get(&start_time, TIME_UTC);

    std::cout << DoSomething(100000000) << std::endl;

    timespec end_time;
    timespec_get(&end_time, TIME_UTC);
    
    auto time_diff_sec = end_time.tv_sec - start_time.tv_sec;
    auto time_diff_nsec = end_time.tv_nsec - start_time.tv_nsec;
    auto elapsed_time = time_diff_sec * 1000000000 + time_diff_nsec;
    
    std::cout << "timespec sec: " << time_diff_sec << " seconds" << std::endl;
    std::cout << "timespec nsec: " << time_diff_nsec << " nanoseconds" << std::endl;
    std::cout << "timespec: " << elapsed_time << " nanoseconds" << std::endl;
    
    return 0;
}
```

输出：(看起来在Windows下精度似乎并没有完全到纳秒)

```c++
// Windows Release
6.66667e+11
timespec sec: 1 seconds
timespec nsec: 68685900 nanoseconds
timespec: 1068685900 nanoseconds

    // Linux
6.66667e+11
timespec sec: 1 seconds
timespec nsec: -323589485 nanoseconds
timespec: 676410515 nanoseconds
```

