---
title: GDB调试工具
abbrlink: 14250
date: 2024-03-05 20:08:01
tags:
description:
categories:
cover:
swiper_index:
---


# 参考文章
{% link 知乎, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%} 

{% link CSDN, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp%} 

{% link github, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/01/13/p2416y-0.webp%} 

最经常用的就用例子来介绍：
```powershell
(gdb) l                      <-- 显示带行号的源代码
1 #include <stdio.h>
2 int main ()
3 {
4     unsigned long long int n, sum;
5     n = 1;
6     sum = 0;
7     while (n <= 100)
8     {
9         sum = sum + n;
10         n = n + 1;
(gdb)                      <-- 默认情况下，l 选项只显示 10 行源代码，如果查看后续代码，安装 Enter 回车即可                                                               
11     }
12     return 0;
13 }
(gdb) b 7               <-- 在第 7 行源代码处打断点
Breakpoint 1 at 0x400504: file main.c, line 7.
(gdb) r                   <-- 运行程序，遇到断点停止
Starting program: /home/mozhiyan/demo1/main.exe

Breakpoint 1, main () at main.c:7
7     while (n <= 100)
Missing separate debuginfos, use: debuginfo-install glibc-2.17-55.el7.x86_64
(gdb) p n               <-- 查看代码中变量 n 的值
$1 = 1                   <-- 当前 n 的值为 1，$1 表示该变量所在存储区的名称
(gdb) b 12             <-- 在程序第 12 行处打断点
Breakpoint 2 at 0x40051a: file main.c, line 12.
(gdb) c                  <-- 继续执行程序
Continuing.

Breakpoint 2, main () at main.c:12
12     return 0;
(gdb) p n               <-- 查看当前 n 变量的值
$2 = 101               <-- 当前 n 的值为 101
(gdb) q                  <-- 退出调试
A debugging session is active.

Inferior 1 [process 3080] will be killed.

Quit anyway? (y or n) y                 <-- 确实是否退出调试，y 为退出，n 为不退出
[root@bogon demo]#
```

## GDB break设置断点
刚才的例子中已经涉及到了断点，`b`就指的是`break`

### break
`break <location>`这里location有这几种值：
- linenum	linenum 是一个整数，表示要打断点处代码的行号。要知道，程序中各行代码都有对应的行号，可通过执行 l（小写的 L）命令看到。
- filename:linenum	filename 表示源程序文件名；linenum 为整数，表示具体行数。整体的意思是在指令文件 filename 中的第 linenum 行打断点。
- \+ offset
  \- offset	offset 
  为整数（假设值为 2），+offset 表示以当前程序暂停位置（例如第 4 行）为准，向后数 offset 行处（第 6 行）打断点；-offset 表示以当前程序暂停位置为准，向前数 offset 行处（第 2 行）打断点。
- function	function 表示程序中包含的函数的函数名，即 break 命令会在该函数内部的开头位置打断点，程序会执行到该函数第一行代码处暂停。
- filename:function	filename 表示远程文件名；function 表示程序中函数的函数名。整体的意思是在指定文件 filename 中 function 函数的开头位置打断点。

还有一种方法：`(gdb) break ... if cond`
... 可以是前边`location`所有参数的值，用于指定打断点的具体位置；cond 为某个表达式。整体的含义为：每次程序执行到 ... 位置时都计算 cond 的值，如果为 True，则程序在该位置暂停；反之，程序继续执行。

```powershell
(gdb) l
1 #include<stdio.h>
2 int main(int argc,char* argv[])
3 {
4     int num = 1;
5     while(num<100)
6     {
7         num *= 2;
8     }
9     printf("num=%d",num);
10   return 0;
(gdb)
11 }
(gdb) b 4          <-- 程序第 4 行打断点
Breakpoint 1 at 0x1138: file main.c, line 4.
(gdb) r              <-- 运行程序，至第 4 行暂停
Starting program: /home/ubuntu64/demo/main.exe

Breakpoint 1, main (argc=1, argv=0x7fffffffe078) at main.c:4
4     int num = 1;
(gdb) b +1        <-- 在第 4 行的基础上，在第 5 行代码处打断点
Breakpoint 2 at 0x55555555513f: file main.c, line 5.
(gdb) c             <-- 继续执行程序，至第 5 行暂停
Continuing.

Breakpoint 2, main (argc=1, argv=0x7fffffffe078) at main.c:5
5     while(num<100)
(gdb) b 7 if num>10     <-- 如果 num>10 在第 7 行打断点
Breakpoint 3 at 0x555555555141: file main.c, line 7.
(gdb) c               <-- 继续执行
Continuing.

Breakpoint 3, main (argc=1, argv=0x7fffffffe078) at main.c:7
7         num *= 2;       <-- 程序在第 7 行暂停
(gdb) p num      <-- p 命令查看 num 当前的值
$1 = 16             <-- num=16
```
### tbreak
tbreak 命令可以看到是 break 命令的另一个版本，tbreak 和 break 命令的用法和功能都非常相似，唯一的不同在于，使用 tbreak 命令打的断点仅会作用 1 次，即使程序暂停之后，该断点就会自动消失。

### GDB rbreak 命令
和 break 和 tbreak 命令不同，rbreak 命令的作用对象是 C、C++ 程序中的函数，它会在指定函数的开头位置打断点。

rbreak 命令的使用语法格式为：
`(gdb) rbreak regex`

其中 regex 为一个正则表达式，程序中函数的函数名只要满足 regex 条件，rbreak 命令就会其内部的开头位置打断点。值得一提的是，rbreak 命令打的断点和 break 命令打断点的效果是一样的，会一直存在，不会自动消失。

## 实时监控变量值的变化情况
当我们需要监控某个变量或者表达式的值，通过值的变化情况判断程序的执行过程是否存在异常或者Bug。这种情况下，break 命令显然不再适用，推荐使用 `watch` 命令
`(gdb) watch cond`这个 cond 就是监控的变量或者表达式
```powershell
(gdb) l           <--列出要调试的程序源码
1 #include<stdio.h>
2 int main(int argc,char* argv[])
3 {
4     int num = 1;
5     while(num<=100)
6     {
7         num *= 2;
8     }
9     printf("%d",num);
10     return 0;
(gdb)
11 }
(gdb) b 4       <-- 使用 break 命令打断点
Breakpoint 1 at 0x115c: file main.c, line 4.
(gdb) r           <-- 执行程序
Starting program: /home/ubuntu64/demo/main.exe

Breakpoint 1, main (argc=1, argv=0x7fffffffe088) at main.c:4
4     int num = 1;
(gdb) watch num   <-- 监控程序中 num 变量的值
Hardware watchpoint 2: num
(gdb) c            <-- 继续执行，当 num 值发生改变时，程序才停止执行
Continuing.

Hardware watchpoint 2: num

Old value = 0
New value = 2
main (argc=1, argv=0x7fffffffe088) at main.c:5
5     while(num<=100)
(gdb) c           <-- num 值发生了改变，继续执行程序
Continuing.

Hardware watchpoint 2: num

Old value = 2
New value = 4
main (argc=1, argv=0x7fffffffe088) at main.c:5
5     while(num<=100)
(gdb)
```
通过借助 watch 命令监控 num 的值，后续只要 num 的值发生改变，程序都会停止。

## 捕捉断点
和前 2 种断点不同
- 普通断点作用于程序中的某一行，当程序运行至此行时停止执行
- 观察断点作用于某一变量或表达式，当该变量（表达式）的值发生改变时，程序暂停。

而捕捉断点的作用是，监控程序中某一事件的发生，例如程序发生某种异常时、某一动态库被加载时等等，一旦目标时间发生，则程序停止执行。
`(gdb) catch event`



|event 事件	| 含 义 |
| ---       | ---  |
|throw [exception] |	当程序中抛出 exception 指定类型异常时，程序停止执行。如果不指定异常类型（即省略 exception），则表示只要程序发生异常，程序就停止执行。|
catch [exception] |	当程序中捕获到 exception 异常时，程序停止执行。exception 参数也可以省略，表示无论程序中捕获到哪种异常，程序都暂停执行。
| load [regexp] unload [regexp]| 其中，regexp 表示目标动态库的名称，load 命令表示当 regexp 动态库加载时程序停止执行；unload 命令表示当 regexp 动态库被卸载时，程序暂停执行。regexp 参数也可以省略，此时只要程序中某一动态库被加载或卸载，程序就会暂停执行。

## 单步调试
在最开始的例子中，借助 next 命令可以控制 GDB 单步执行程序。所谓单步调试，就是通过一行一行的执行程序，观察整个程序的执行流程，进而尝试发现一些存在的异常或者 Bug。

根据实际场景的需要，GDB 调试器共提供了 3 种可实现单步调试程序的方法，即使用 next、step 和 until 命令。换句话说，这 3 个命令都可以控制 GDB 调试器每次仅执行 1 行代码，但除此之外，它们各自还有不同的功能。

### until
next和step都很好理解，until之前并没有使用过

不带参数的 until 命令 ： `(gdb) until`
可以使 GDB 调试器快速运行完当前的循环体，并运行至循环体外停止。注意，until 命令并非任何情况下都会发挥这个作用，只有当执行至循环体尾部（最后一行代码）时，until 命令才会发生此作用；反之，until 命令和 next 命令的功能一样，只是单步执行程序。
```c
#include <stdio.h>
int print(int num){
    int ret = num * num;
    return ret;
}
int myfunc(int num){
    int i = 1;
    int sum = 0;
    while(i <= num){
        sum += print(i);
        i++;
    }
    return sum;
}
int main(){
    int num =0;
    scanf("%d", &num);
    int result = myfunc(num);
    printf("%d", result);
    return 0;
}
```


```powershell
(gdb) b 17
Breakpoint 1 at 0x1201: file main.c, line 17.
(gdb) r
Starting program: ~/demo/main.exe

Breakpoint 1, main () at main.c:17
17     scanf("%d", &num);
(gdb) u
3
18     int result = myfunc(num);
(gdb) step
myfunc (num=3) at main.c:7
7     int i = 1;
(gdb) u
8     int sum = 0;
(gdb) u
9     while(i <= num){
(gdb) u
10         sum += print(i);
(gdb) u
11         i++;
(gdb) u                                 <-- 执行 i++ 操作
9     while(i <= num){
(gdb) u                                 <-- 快速执行完循环体
13     return sum;
(gdb) p sum
$1 = 14
```
可以看到，这里当程序单步执行完第 11 行代码时，借助 until 命令快速执行完了整个循环体，并在第 13 行代码处停止执行。根据 p 命令输出的 num 变量的值可以确认，整个循环过程确定完整地执行完了。

## GDB finish和return命令
实际调试时，在某个函数中调试一段时间后，可能不需要再一步步执行到函数返回处，希望直接执行完当前函数，这时可以使用 finish 命令。与 finish 命令类似的还有 return 命令，它们都可以结束当前执行的函数。

finish 命令和 return 命令的区别是，finish 命令会执行函数到正常退出；而 return 命令是立即结束执行当前函数并返回，也就是说，如果当前函数还有剩余的代码未执行完毕，也不会执行了。除此之外，return 命令还有一个功能，即可以指定该函数的返回值。

# 多线程程序的调试
待定...