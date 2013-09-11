---
layout: post
title: 漫谈C语言整型类型转换规则
category : C/C++
tags : [C/C++, 类型转换]
---
{% include JB/setup %}

整型类型包括long long，long，int，short，char（包括有符号以及无符号unsigned）类型。

或许你觉得整型类型转换规则很简单，那么请先看以下这道不定项选择题：

    unsigned char a = 0xe0;
    int b = a;
    char c = a;
    A. a>0 && c>0 为真   B. a == c 为真   C. b 的十六进制表示是：0xffffffe0   D. 上面都不对

这道题目还可以这样变形：
    
    char a =0xe0;
    unsigned int b = a;
    unsigned char c = a;
    A. a>0 && c>0 为真   B. a == c 为真   C. b 的十六进制表示是：0xffffffe0   D. 上面都不对

你能马上给出很确定的答案吗？如果可以，那么下面的内容你可以不用看了。如果你跟我一样，第一次看到这道题目有点懵了，那么请仔细阅读。这道题目我是在某公司的笔试中碰到（签了保密协议不便透露具体公司），以前编程一般不会去注意这些细节，因为用不着。不过想想这些点还是搞清楚比较好，下面将学习笔记贴上。

接下来从整体上把转换分成两类：任何类型转换成无符号类型 以及 任何类型转换成带符号类型。

<!--break-->

###任何类型转换成无符号类型
----------------------------------------------------------------

**任何整形转换为某种特定的无符号类型数的方法是：以该无符号数类型能够表示的最大值加1为模，找出与此整数同余的最小的非负值。（摘自《K&R C》第173页）**

下面分情况详细说（其实上面那句话已经高度概括了）：

* 待转换数为带符号正数或者是无符号类型数的话，规则为短类型扩充为长类型则高位补零，长类型转换为短类型则截取相应低位字节

如下：

    int int32 = 0x00010002;
    short int16 = int32;（则int16 = 0x0002）

    short int16 = 0x0001;
    int int32 = int16;（则int32 = 0x00000001）

* 如果待转换数为带符号负数，又分以下三种情况

如果无符号类型的位模式较宽，则将待转换数填入无符号类型的低位字节，高位字节则全部补1。如以下情况：

    short int16 = 0x8765;//-30875
    unsigned int uint32 = int16;（则uint32 = 0xffff8765，4294936421);

如果无符号类型的位模式较窄，则相当于截取低位字节。如以下情况

    int int32 = 0x87654321;//-2023406815
    unsigned short uint16 = int32;（则uint16 = 0x4321，17185）；

如果无符号类型与待转换位宽一样，则每个位复制过去。但要注意两者的值是不同的。如以下情况：

    int int32 = 0x87654321;//值为-2023406815
    unsigned int uint32 = int32;（则uint32 = 0x87654321，2271560481） 

以下为实现代码：
    
    #include <stdio.h>
    int main()
    {
        unsigned int uint32;
        int int32;
        unsigned short uint16;
        short int16;

        int16 = 0x8765;
        uint32 = int16;
        printf( "int16=%hd\t\tuint32=%u\tint16=%hx\tuint32=%x\n", int16, uint32, int16, uint32 );

        int32 = 0x87654321;
        uint16 = int32;
        printf( "int32=%d\tuint16=%hu\t\tint32=%x\tuint16=%hx\n", int32, uint16, int32, uint16);

        int32 = 0x87654321;
        uint32 = int32;
        printf( "int32=%d\tuint32=%u\tint32=%x\tuint32=%x\n", int32, uint32, int32, uint32 );

        return 0;
    }

###任何类型转换成带符号类型
------------------------------------------------------
将任何整数转换为带符号类型时，如果它可以在新类型中表示出来，则其值保持不变，否则它的值同具体的实现有关。（摘自《K&R C》第173页）**为什么跟具体实现有关，不作规定呢？**

下面分情况详细说（以下结果为在GCC编译器所得）

* 转换后的位模式较窄，则截断低字节字段。

如以下两种情况：   

    int int32 = 0x87654321;
    short int16 = int32;（int16 = 0x4321）
    unsigned int uint32 = 0x87654321;
    short int16 = uint32;（int16 = 0x4321）

* 转换后的位模式较宽，分两种情况：

转换前的数是无符号数，将无符号数填入转换后的低字节，高字节填0补充。如下：

    unsigned short uint16 = 0x8765;
    int int32 = uint16;（int32 = 0x00008765）
转换前的数是带符号数，将带符号数填入转换后的低字节，高字节按转换前的符号位填充。如下：

    short int16 = 0x8765;//符号位为1
    int int32 = int16;（int32 = 0xffff8765）

* 转换前后的位数一样，则每个位复制过去。但要注意两者的值是不同的。如下：


    unsigned int uint32 = 0x87654321;//值为2271560481
    int int32 = uint32;（int32 = 0x87654321，值为-2023406815）

以下为实现代码

    #include <stdio.h>
    int main()
    {
        unsigned int uint32;
        int int32;
        unsigned short uint16;
        short int16;

        int32 = 0x87654321;
        int16 = int32;
        printf( "%d %hd %x %hx \n", int32, int16, int32, int16 );
        uint32 = 0x87654321;
        int16 = uint32;
        printf( "%u %hd %x %hx \n", uint32, int16, uint32, int16 );

        int16 = 0x8765;
        int32 = int16;
        printf( "%hd %d %hx %x \n", int16, int32, int16, int32 );
        uint16 = 0x8765;
        int32 = uint16;
        printf( "%hu %d %hx %x \n", uint16, int32, uint16, int32 );

        uint32 = 0x87654321;
        int32 = uint32;
        printf( "%u %d %x %x \n", uint32, int32, uint32, int32 );

        return 0;
    }

以上写得有点乱，不过还算详细。每一种情况提到。

回到题目：
    
    unsigned char a = 0xe0;
    int b = a;
    char c = a;
    A. a>0 && c>0 为真   B. a == c 为真   C. b 的十六进制表示是：0xffffffe0   D. 上面都不对

个人觉得正确答案为D。A显然不对，a>0但是c<0；B也是错误，具体下面讲；C，b的十六进制表示应该是：0x000000e0；所以选D。

B选项为什么错误呢？a c两者在内存上表示相同，但是两者比较的时候，需要先把a跟c都隐式转换成int型，再进行比较。这个过程，a就变成0x000000e0，c就变成0xffffffe0，所以不相等。（觉得奇怪的话，可以试试sizeof（a+c）的值是否等于sizeof（int）的值。）至于为什么要把a跟c都转换成int型呢？相关知识可以搜索“整形提升”以及“普通算数类型转换”，《K&R C》第173页有讲，限于篇幅，这里就不展开了。（不知不觉这篇笔记已经有了三千多字了）。

需要注意以下情况：

    unsigned int uint32 = 0xe0000001；
    int int32 = 0xe0000001；
这个时候 int32 == uint32。跟上面的情况是不同的。关键就在于“整形提升”。

变形题目：
    
    char a =0xe0;
    unsigned int b = a;
    unsigned char c = a;

个人觉得答案应该是C。A跟B选项的解释可以套用上一题。至于C选项，根据上面的规则，b确实是0xffffffe0。