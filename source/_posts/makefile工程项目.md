---
title: makefile工程项目
abbrlink: 26988
date: 2024-02-26 20:03:31
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


# 什么是makefile
- 描述了整个工程了的编译、链接规则 
- 软件项目自动化编译 

# 程序的编译以及链接
## 程序的存储与运行
![计算机架构](https://image.aruoshui.fun/i/2025/02/26/xf9l23-0.webp)

### 嵌入式系统架构
![架构](https://image.aruoshui.fun/i/2025/02/26/xg6ill-0.webp)

## 程序文件的分类
- 二进制文件bin，用途依系统或应用而定
- elf文件，用于二进制文件、可执行文件、目标代码、共享库和核心转储格式文件的文件格式。
    - 可执行文件
    - 可重定位文件、可组装文件
    - 共享库文件

## 动态库和静态库
库就是目标文件的归档
- 静态库：在编译链接的过程中就链接到了可执行文件中，但是如果部分库重复使用，就会导致重复链接，造成文件大小太大
  ![静态库](https://image.aruoshui.fun/i/2025/02/26/xqq5p8-0.webp)

- 动态库：可以看到入口地址，就像我们安装文件的时候，安装包里有很多dll文件，在运行时会随可执行文件一起加载到内存中去，程序运行到该位置会从内存中动态加载此库，从而减少程序的体积
  ![动态库](https://image.aruoshui.fun/i/2025/02/26/xpipv7-0.webp)

{% note success modern %}success 

问题：
一个C文件中，如果一行代码后面忘记; 是报编译错误还是链接错误？ 
**编译器在语法分析阶段直接报错**
• 一个C文件中，引用了一个在另一个C文件中定义的函数,但是没有声明，编译会成功吗？会出现什么错误或者警告，如何去除这个错误或警告？ 
**可能通过但产生 警告（如隐式函数声明警告），也可能直接报错（取决于编译器严格性）**
• 一个C文件中，引用了一个没有被定义的函数，是否会编译通过？是报编译错误，还是链接错误？

**编译阶段：可能通过但有警告（隐式声明），生成目标文件。**
**错误阶段：最终报 链接错误，因为函数定义不存在于任何链接的文件中。**

{% endnote %}

## makefile的基本语法
### 规则

#### 目标
1. 一个规则中可以无目标依赖，仅仅实现某种操作
    比如：
    ```makefile
    test1 :
     @echo "Just for test1:$@"
    test2:
     @echo "Just for test2"
    ```
2. 一个规则中可以没有命令，仅仅描述依赖关系
    ```makefile
    all:test1
    all:test2
    ```
    这两条命令都依赖test1或2来执行
3. 默认目标：可以有多个目标，但是以第一个为默认目标 
   
4. 多目标：一个规则中可以有多个目标，多个目标具有相同的生成命令 
   
5. 多规则目标:多个规则可以是同一目标，make在解析的过程中会将多个规则的依赖文件合并
    ```makefile
    all:test1
    all:test2
    ```
    还是一样的在make中会将所有的all合并一起

6. 伪目标：无条件执行，可以看做一种标签，实行某种操作
   ```makefile
   .PONHY: clean
   clean:
	rm -f lcd.o hello player.o
   ```
#### 目标依赖
1. 时间戳机制：makefile其实就是根据时间戳来判断目标依赖文件是否需要更新的
   - 在上次make之后修改过的C文件，会被重新编译 
   - 在上次make之后修改过的头文件，依赖此头文件的会被重新编译
2. 自动产生依赖
   Gcc –M命令生成该文件要依赖的文件
3. 模式匹配

#### 生成命令
由shell命令组成，每条命令make都会开一个进程并执行。
命令也支持并发执行命令 `make -j4`

### 变量
#### 变量基础
清晰易懂
```makefile
STR = hello
STR2 = hello
STR2 += world!

test1 = a
test1 ?= b
test2 ?= b

all:
	@echo "STR = $(STR)"
	@echo "STR2 = $(STR2)"
	@echo "test1 = $(test1)"
	@echo "test2 = $(test2)"
```
#### 变量分类
1. 立即展开变量：在解析阶段直接赋值常量字符串
   使用`:=`赋值操作符
2. 延迟展开变量：在运行阶段，实际使用变量时再进行求值 
   使用`=`赋值操作符
```makefile
.PHONY:all

HELLO = Good
TIME = morning!
STRING = $(HELLO) $(TIME)
# STRING ：= $(HELLO) $(TIME)
$(info $(STRING))
TIME = afternoon!
$(info $(STRING))

all:
	@echo "done"
```
这里如果使用立即展开变量，在makefile解析生成依赖树的时候，**直接变成常量字符串了**

• 一般在目标、目标依赖中使用立即展开变量 
• 在命令中一般使用延迟展开变量

#### 追加、条件赋值
#### 目标变量
默认为全局变量，在所有依赖的规则中都可以使用，可以做到文件级的编译选项
```makefile
.PHONY:clean

OBJS = player.o lcd.o
BIN  = mp3
N = 1
$(BIN): N = 2
$(BIN):$(OBJS)
	@echo "BIN: N = $(N)"
	gcc -o $(BIN) $(OBJS)
player.o:N= 3
player.o:player.c
	@echo "player.o: N = $(N)"
	gcc -o player.o -c player.c
lcd.o:lcd.c # 这个N是由于 bin依赖lcd，所以N取 bin中定义的N = 2
	@echo "lcd.o: N = $(N)"
	gcc -o lcd.o -c lcd.c
clean:
	@echo "clean: N = $(N)"
	rm -f  $(BIN) $(OBJS)
```

#### 模式变量
```makefile
.PHONY:clean
N = 1
OBJS = player.o lcd.o
BIN  = mp3
$(BIN):$(OBJS)
	@echo "BIN:N=$(N)"
	gcc -o $(BIN) $(OBJS)
$(BIN): N = 2
%.o: N = 3
player.o:player.c
	@echo "player.o:N=$(N)"
	gcc -o player.o -c player.c
lcd.o:lcd.c
	@echo "lcd.o:N=$(N)"
	gcc -o lcd.o -c lcd.c
clean:
	@echo "clean: N = $(N)"
	rm  $(BIN) $(OBJS)

##########################
player.o:N=3
gcc -o player.o -c player.c
lcd.o:N=3
gcc -o lcd.o -c lcd.c
BIN:N=2
gcc -o mp3 player.o lcd.o
```
从结果可以知道所有.o后缀的目标都以N = 3为值

#### 自动变量
```txt
自动变量是局部变量 
• 目标 
$@ 
• 所有目标依赖 
$^ 
• 第一个依赖 
$< 
• 使用举例 
gcc  -o $@ $^ 
```

```makefile
.PHONY:clean
OBJS = player.o lcd.o
BIN  = mp3
$(BIN):$(OBJS)
	@echo "BIN------------$@:$^"
	gcc -o $@ $^ 
player.o:player.c
	@echo "------------$@:$^"
	gcc -o $@ -c $^
lcd.o:lcd.c
	@echo "---------$@:$^"
	gcc -o $@ -c $^
clean:
	rm -f $(BIN) $(OBJS)
```

#### 系统环境变量
作用范围 
- 变量在make开始运行时被载入到Makefile文件中 
- 对所有的Makefile都有效。 
- 若Makefile中定义同名变量，系统环境变量将被覆盖 
- 命令行中传递同名变量，系统环境变量将被覆盖 

常见的系统环境变量 
- CFLAGS 
- SHELL 
- MAKE 

#### 变量的传递
```txt
├── lcd
│   └── makefile
├── makefile
└── test
    └── makefile
```
```makefile
.PHONY:all

export N = 3
#N = 3
all:
	@echo "build...."
#	cd test && make N=$(N)
	cd test && make
	make -C lcd 
```
这里变量以export设置为系统环境变量，其他makefile也能访问到

### 条件执行
1. 关键字
   • ifeq、else、endif 
   • ifneq
2. 使用
   条件语句从ifeq开始，括号与关键字用空格隔开
```makefile
.PHONY:all

DEBUG = true
ifeq ($(DEBUG),true)
VERSION = debug
CC = gcc -g
else
VERSION = release
CC = gcc
endif

hello:hello.c
	@echo "build $(VERSION) mode"
	$(CC) -o $@ $^
clean:
	rm hello
```
进行选择debug模式或者release模式切换

### 函数
直接查手册就行
#### 文本处理函数
```makefile
.PHONY:all
SRCS = player.c lcd.c usb.c media.c hello.h main.txt
OBJS = $(subst .c,.o,$(strip $(SRCS)))
DEPS = $(patsubst %.c,%.d,$(SRCS))
DEPS2 = $(SRCS:.c=.d)
FIND = $(findstring usb,$(SRCS))
FILTER = $(filter %.c %.h, $(SRCS))
all:
	@echo "OBJS = $(OBJS)"	
	@echo "DEPS = $(DEPS)"
	@echo "DEPS2 = $(DEPS2)"
	@echo "FIND = $(FIND)"
	@echo "FILTER = $(FILTER)"

#########################################
OBJS = player.o lcd.o usb.o media.o hello.h main.txt
DEPS = player.d lcd.d usb.d media.d hello.h main.txt
DEPS2 = player.d lcd.d usb.d media.d hello.h main.txt
FIND = usb
FILTER = player.c lcd.c usb.c media.c hello.h
```

#### 文件名处理函数
```makefile
.PHONY:all
LIB = /home/hello/libhello.a
LIB1 = $(dir $(LIB))
LIB2 = $(notdir $(LIB))
LIB3 = $(suffix $(LIB))
LIB4 = $(basename $(LIB))
LIB5 = $(addsuffix .c,$(LIB4))
LIB6 = $(addprefix /usr/lib/,$(LIB2))
SRCS = $(wildcard *.c)
all:
	@echo "LIB1 = $(LIB1)"
	@echo "LIB2 = $(LIB2)"
	@echo "LIB3 = $(LIB3)"
	@echo "LIB4 = $(LIB4)"
	@echo "LIB5 = $(LIB5)"
	@echo "LIB6 = $(LIB6)"	
	@echo "SRCS = $(SRCS)"

#########################################
LIB1 = /home/hello/
LIB2 = libhello.a
LIB3 = .a
LIB4 = /home/hello/libhello
LIB5 = /home/hello/libhello.c
LIB6 = /usr/lib/libhello.a
SRCS = hello.c 3.c main.c
```
#### 常用函数 
foreach
```makefile
A = 1 3 4 5 6 7 8 9
B = $(foreach i,$(A),$(addprefix 0.,$(i)))
C = $(foreach i,$(A),$(addsuffix .0,$(i)))


all:
@echo "A = $(A)"
@echo "B = $(B)"
@echo "C = $(C)"
```
   
shell
```makefile
.PHONY:all clean
$(shell mkdir -p s1)
$(shell mkdir -p s2)
all:
	@echo "hello world"
clean:
	rm -r s1 s2
```
### 库的生成和使用
#### 静态库的生成
都知道库是给别人用的，所以做库时，头文件(.h)必须暴露，源文件(.c)必须隐藏。
1. 将需要形成库的文件编译成.o文件
   ```dotnetcli
   ├── hello.c  - 做成库
   ├── hello.h
   └── main.c   -调用库
   ```
   `gcc -c hello.c -o hello.o`
2. 然后使用指令`ar -rc libhello.a hello.o`来生成库
   注意**形成库文件前缀必须是lib，后缀必须是.a，后面可以加版本号。**

#### 静态库的使用
我这里已经成了一个静态库，现在交付给其他人使用
首先拿到别人给的头文件和库文件，我们需要去系统路径下安装别人给的头文件和库文件： 

安装头文件：sudo cp*.h /usr/include/
安装库文件：sudo cp libmy_stdio.a /lib64/

```txt
├── hello.c
├── hello.h
├── hello.o
├── libhello.a
└── main.c
```
简单例子来看现在main.c需要使用库，就可以使用` gcc main.c -L ./ -l hello` ，运行生成的可执行文件即可
`-l`参数使用来指定第三方库的
`-L`参数使用来指定库的路径的，由于我这个简单的例子没有将库放入系统库文件，所以使用`-L`来指定位置

#### makefile中的静态库生成
制作：
```makefile
.PHONY:clean

libmath.a:libmath.o
	ar rcs $@ $^
libmath.o:libmath.c libmath.h
clean:
	rm libmath.a libmath.o
```

使用：
```makefile
.PHONY: clean

hello:main.o
	gcc -o $@ $^ -L./ -lmath
main.o:main.c
	gcc -o $@ -c $^
clean:
	rm hello main.o
```

#### 动态库的生成
有了静态库，动态库也不难理解
```makefile
.PHONY:clean
libdll.so:dll.o
	gcc -o $@ -shared $^
dll.o:dll.c
	gcc -o $@ -fPIC -c $^ 
clean:
	rm libdll.so dll.o
```

使用方法跟静态库类似
```makefile
.PHONY:clean
hello:main.o
	gcc -o hello main.o -L./ -ldll
main.o:main.c
	gcc -o $@ -c -fPIC $^

clean:
	rm -f  main.o hello
```
这里可以使用`ldd`系统命令来查看依赖情况

#### Linux系统中，os可以根据环境变量查动态库
`export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:` +路径也可以：
如果想永久加env可以`vim .bash.src`中加入export ...
记得刷新就行

## makefile小tips
学完了基本的语法，就可以开始实践makefile了，下面是开发中经常遇到的初级及tips
### 自动生成依赖关系

问题抛出可以看：
{% link 自动生成依赖关系, https://blog.csdn.net/qq_52484093/article/details/122765782,  https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp%} 
执行 make 时，首先查看 include，然后才是执行顶层目标所对应的规则。


### 使用目录管理源文件
源文件散着放在一起会造成混乱，少一点还能理清楚，但是源文件多起来就不那么方便了，这里使用目录来管理源文件

{% link 利用Makefile给多文件、多目录C源码建立工程, https://zhuanlan.zhihu.com/p/422891037,  https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%} 


## 实战案例
{% link Makefile工程实践, https://blog.csdn.net/qq_55299368/article/details/122071652,  https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp%} 
