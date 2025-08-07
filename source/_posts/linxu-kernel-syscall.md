---
title: linxu-kernel-syscall
abbrlink: 54939
date: 2025-04-09 12:28:20
tags:
description:
categories:
cover:
swiper_index:
---




# 参考文章

{% link 『 Linux 』“ 一切皆文件 “, https://blog.csdn.net/2202_75303754/article/details/138967355,  https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp%} 

 

# 系统调用的基本概念
在linux中，操作系统负责硬件资源的封装，任务的创建、调度、读写磁盘
- μc/os中，采用OSinit-OSTaskCreate-创建一个任务
- Linux中，拥有权限管理来保证安全，划分内核态和用户态。通过系统调用，让APP某些运行陷入到内核态，以此来访问硬件设备

# 软中断
## 权限管理
- 程序的用户态、内核态
- 操作系统+ CPU软中断：swi/svc
- CPU的运行级别：普通权限(普通运行)、特权（陷入到内核态）
  - ARM32：
    - 普通模式：User
    - 特权模式：FIQ、IRQ、SVC、ABT、UND
  - ARM64：EL0、EL1、EL2、EL3
  - X86：ring0 ~ ring

## 系统调用号
- ARM：swi、svc
- ARM : swi、svc
  - 系统调用接口：read、write、printf
  - 内核中的实现：sys_read、sys_write
  - 系统调用号：
    -  32位ARM：3、4
    -  64位ARM：0、1
  

## 数据传递
- 软中断指令：– X86 int 80H – ARM swisvc
- 用户函数的参数传递
     - ARM：R0、R1、R2、R3、R4、R5、R6
     - ARM64：X1、X2、X3、X4、X5
- 系统调用号– ARM：R7 – ARM64：X8
- 内核函数的返回值– ARM：R0– ARM64：X

```powershell
      Arch/ABI    Instruction           System  Ret  Ret  Error    Notes

                                         call #  val  val2
       ───────────────────────────────────────────────────────────────────
       alpha       callsys               v0      v0   a4   a3       1, 6
       arc         trap0                 r8      r0   -    -
       arm/OABI    swi NR                -       a1   -    -        2
       arm/EABI    swi 0x0               r7      r0   r1   -
       arm64       svc #0                x8      x0   x1   -
       blackfin    excpt 0x0             P0      R0   -    -
       i386        int $0x80             eax     eax  edx  -
       ia64        break 0x100000        r15     r8   r9   r10      1, 6
       m68k        trap #0               d0      d0   -    -
       microblaze  brki r14,8            r12     r3   -    -
       mips        syscall               v0      v0   v1   a3       1, 6
       nios2       trap                  r2      r2   -    r7
       parisc      ble 0x100(%sr2, %r0)  r20     r28  -    -
       powerpc     sc                    r0      r3   -    r0       1
       powerpc64   sc                    r0      r3   -    cr0.SO   1
       riscv       ecall                 a7      a0   a1   -
       s390        svc 0                 r1      r2   r3   -        3
       s390x       svc 0                 r1      r2   r3   -        3
       superh      trap #0x17            r3      r0   r1   -        4, 6
       sparc/32    t 0x10                g1      o0   o1   psr/csr  1, 6
       sparc/64    t 0x6d                g1      o0   o1   psr/csr  1, 6
       tile        swint1                R10     R00  -    R01      1
       x86-64      syscall               rax     rax  rdx  -        5
       x32         syscall               rax     rax  rdx  -        5
       xtensa      syscall               a2      a2   -    -
```


ARM汇编系统调用：
```armasm
.text
.global _start

_start:
    mov r0, #1              /* stdout*/
    add r1, pc, #16         /* address of the string*/
    mov r2, #12            /* string length*/
    mov r7, #4             /*syscall for 'write'*/
    swi #0                  /* software interrupt*/  软中断调用sys_write来实现字符串的打印

_exit:
    mov r7, #1             /* syscall for 'exit'*/
    swi #0                 /* software interrupt*/

_string:
.asciz "Hello world\n"          @ our string, NULL terminated
```


## 系统调用接口的封装
写这些汇编非常麻烦，好在：
- C标准库包含一系列系统调用接口的封装– read、write、fork、open…

### syscall系统调用接口的封装
对于在C标准库中没有封装的系统调用
- syscall是一个库函数：`long syscall(long number, ...);`，如果想使用syscall来实现write系统调用，直接`syscall(1, 1, "helloworld\n", 12)`就行
- 封装了系统调用的汇编接口– 系统调用前保存CPU寄存器– 从系统调用返回后，恢复寄存器

syscall的汇编实现：
```armasm
000dad70 <syscall@@GLIBC_2.4>:
   dad70:	e1a0c00d 	mov	ip, sp
   dad74:	e92d00f0 	push	{r4, r5, r6, r7}
   dad78:	e1a07000 	mov	r7, r0
   dad7c:	e1a00001 	mov	r0, r1
   dad80:	e1a01002 	mov	r1, r2
   dad84:	e1a02003 	mov	r2, r3
   dad88:	e89c0078 	ldm	ip, {r3, r4, r5, r6}
   dad8c:	ef000000 	svc	0x00000000
   dad90:	e8bd00f0 	pop	{r4, r5, r6, r7}
   dad94:	e3700a01 	cmn	r0, #4096	; 0x1000
   dad98:	312fff1e 	bxcc	lr
   dad9c:	eafcf2c3 	b	178b0 <__libc_start_main@@GLIBC_2.4+0x278>
```

# 系统调用流程分析
以kill这个系统调用来分析： 

1. 接口封装: /usr/arm-linux-gnueabi/lib/libc.a
2. 系统调用号: arch/arm/include/generated/calls-eabi.S 这里定义了一个系统调用表，将系统调用号和指针对应起来，用户使用系统调用就能根据此表找到函数指针，从而跳转过去运行
3. 内核实现: kernel/signal.c 不同系统调用实现分布在不同的内核部分
4. 中断处理: arch/arm/kernel/entry-common.S  软中断

**实现过程就是根据系统调用号，从系统调用表中找到对应的入口函数指针，跳转执行，中间包含各种软中断的管理操作**
## 系统调用号：
```c
arch/arm/include/generated/uapi/asm/unistd-common.h
 #define __NR_kill (__NR_SYSCALL_BASE + 37)
```

## x系统调用实现：
```c
 kernel/signal.c :
 SYSCALL_DEFINE2(kill, pid_t, pid, int, sig)
 {
    struct kernel_siginfo info;
    prepare_kill_siginfo(sig, &info);
    return kill_something_info(sig, &info, pid);
 }
展开后相当于：
asmlinkage long sys_kill(pid_t pid, int sig)
```

## 系统调用函数实现：
```c
 include/linux/syscalls.h
 asmlinkage:GCC扩展，表示读取的参数来自栈中，而非寄存器
/* kernel/signal.c */
 asmlinkage long sys_restart_syscall(void);
 asmlinkage long sys_kill(pid_t pid, int sig);
 asmlinkage long sys_tkill(pid_t pid, int sig);
```
## 获取系统调用号
```armasm
 arch/arm/kernel/entry-common.S :  保护现场，获取系统调用号
ENTRY(vector_swi)
 addne scno, r7, #__NR_SYSCALL_BASE	@ put OS number in
 ldr tbl, sys_call_table
 ...
 invoke_syscall tbl, scno, r10, __ret_fast_syscall
   add  r1, sp, #S_OFF
 2: cmp  scno, #(__ARM_NR_BASE - 	__NR_SYSCALL_BASE)
   eor  r0, scno, #__NR_SYSCALL_BASE @ put OS number back
   bcs  arm_syscall
   mov  why, #0		 @ no longer a real syscall
   b   sys_ni_syscall		 @ not private func
 ...
 9001:
  sub lr, saved_pc, #4
  str lr, [sp, #S_PC]
  get_thread_info tsk
  b ret_fast_syscall   回到用户态kill，继续执行用户态代码
ENDPROC(vector_swi)

 syscall_table_start sys_call_table
  #define COMPAT(nr, native, compat) syscall nr, native
  #ifdef CONFIG_AEABI
    #include <calls-eabi.S>
  #else
    #include <calls-oabi.S>
  #endif
  #undef COMPAT
 syscall_table_end sys_call_table

 #define NATIVE(nr, func) syscall nr, func
```

## 系统调用表
```armasm
arch/arm/include/generated/calls-eabi.S :
 NATIVE(0, sys_restart_syscall)
 NATIVE(1, sys_exit)
 NATIVE(2, sys_fork)
 NATIVE(3, sys_read)
 NATIVE(4, sys_write)
 NATIVE(5, sys_open)
 NATIVE(6, sys_close)
 NATIVE(8, sys_creat)
 NATIVE(9, sys_link)
 NATIVE(10, sys_unlink)
 NATIVE(11, sys_execve)
 NATIVE(12, sys_chdir)
 NATIVE(14, sys_mknod)
 NATIVE(15, sys_chmod)
 NATIVE(16, sys_lchown16)
 NATIVE(19, sys_lseek)
 NATIVE(20, sys_getpid)
 NATIVE(21, sys_mount)
 NATIVE(23, sys_setuid16)
 NATIVE(24, sys_getuid16)
 NATIVE(26, sys_ptrace)
 NATIVE(29, sys_pause)
 NATIVE(33, sys_access)
 NATIVE(34, sys_nice)
 NATIVE(36, sys_sync)
 NATIVE(37, sys_kill)
 NATIVE(38, sys_rename)
 NATIVE(39, sys_mkdir)
其实就是定义一个函数入口指针  .long sys_kill
```


# 添加一个系统调用
## 增加内核对应的实现函数
```powershell
diff --git a/arch/arm/kernel/signal.c b/arch/arm/kernel/signal.c
index 585edbfcc..544c92bcb 100644
--- a/arch/arm/kernel/signal.c
+++ b/arch/arm/kernel/signal.c
@@ -723,3 +723,17 @@ asmlinkage void do_rseq_syscall(struct pt_regs *regs)
 	rseq_syscall(regs);
 }
 #endif
+/* add by wit */
+asmlinkage  void sys_hello(const char __user *buf, size_t count)
+{
+    char kernel_buf[100] = {0};
+    if(buf)
+    {
+        copy_from_user(kernel_buf, buf, (count < 100) ? count:100);
+        printk("sys_hello: %s\n", kernel_buf);
+    }
+}
+
+
+
```

## 在系统调用表中增加一个系统调用号及入口

```powershell
diff --git a/include/linux/syscalls.h b/include/linux/syscalls.h
index 37bea07c1..a43e2e0c6 100644
--- a/include/linux/syscalls.h
+++ b/include/linux/syscalls.h
@@ -674,6 +674,9 @@ asmlinkage long sys_sched_rr_get_interval(pid_t pid,
 asmlinkage long sys_sched_rr_get_interval_time32(pid_t pid,
 						 struct old_timespec32 __user *interval);
 
+/* add by wit: add a system call: hello */
+asmlinkage void sys_hello(const char __user *buf, size_t count);
+
 /* kernel/signal.c */
 asmlinkage long sys_restart_syscall(void);
 asmlinkage long sys_kill(pid_t pid, int sig);

--- a/arch/arm/include/generated/calls-eabi.S
+++ b/arch/arm/include/generated/calls-eabi.S

+ NATIVE(441, sys_hello)
```

# 系统调用的开销
## 主要的开销
- 中断，软中断也是中断，变态是有代价的，每次切换需刷新 CPU 流水线、TLB 和缓存，现代 CPU 需约 100-1000 时钟周期。
- 上下文保存与恢复，CPU 需保存用户态寄存器状态（如 PC、SP、EFLAGS）到内核栈，返回时再恢复。容易造成额外的内存访问
- 抢占系统、任务调度
- 同步，内核全局资源（如文件系统）可能需加锁，引发争用。
- IO等待，频繁拷贝（尤其是大块数据）会显著降低性能。

## 解决思路
- 快速系统调用指令：
x86 的 syscall（比 int 0x80 快 2-3 倍）。

- 虚拟系统调用：
  - vdso (Virtual Dynamic Shared Object)：
    将部分调用（如 gettimeofday()）映射到用户态，无需切换。
  - vsyscall（Virtual System Call）
    内核将部分系统调用的代码映射到固定的用户空间地址（如 0xffffffffff600000）。用户程序直接跳转到该地址执行，无需切换特权级。


### vsyscall
```powershell
$ cat /proc/self/maps
5611f7bb1000-5611f7bb3000 r--p 00000000 08:05 1704088                    /usr/bin/cat
5611f7bb3000-5611f7bb8000 r-xp 00002000 08:05 1704088                    /usr/bin/cat
5611f7bb8000-5611f7bbb000 r--p 00007000 08:05 1704088                    /usr/bin/cat
5611f7bbb000-5611f7bbc000 r--p 00009000 08:05 1704088                    /usr/bin/cat
5611f7bbc000-5611f7bbd000 rw-p 0000a000 08:05 1704088                    /usr/bin/cat
56123188b000-5612318ac000 rw-p 00000000 00:00 0                          [heap]
7f7dec15b000-7f7dec17d000 rw-p 00000000 00:00 0 
7f7dec17d000-7f7dec9ee000 r--p 00000000 08:05 1704009                    /usr/lib/locale/locale-archive
7f7dec9ee000-7f7deca10000 r--p 00000000 08:05 1706133                    /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f7deca10000-7f7decb88000 r-xp 00022000 08:05 1706133                    /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f7decb88000-7f7decbd6000 r--p 0019a000 08:05 1706133                    /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f7decbd6000-7f7decbda000 r--p 001e7000 08:05 1706133                    /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f7decbda000-7f7decbdc000 rw-p 001eb000 08:05 1706133                    /usr/lib/x86_64-linux-gnu/libc-2.31.so
7f7decbdc000-7f7decbe2000 rw-p 00000000 00:00 0 
7f7decbf5000-7f7decbf6000 r--p 00000000 08:05 1706106                    /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f7decbf6000-7f7decc19000 r-xp 00001000 08:05 1706106                    /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f7decc19000-7f7decc21000 r--p 00024000 08:05 1706106                    /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f7decc22000-7f7decc23000 r--p 0002c000 08:05 1706106                    /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f7decc23000-7f7decc24000 rw-p 0002d000 08:05 1706106                    /usr/lib/x86_64-linux-gnu/ld-2.31.so
7f7decc24000-7f7decc25000 rw-p 00000000 00:00 0 
7ffc30544000-7ffc30565000 rw-p 00000000 00:00 0                          [stack]
7ffc305d7000-7ffc305db000 r--p 00000000 00:00 0                          [vvar]
7ffc305db000-7ffc305dd000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]

```
可以看到程序段映射中有vsyscall段，这里保存了一些固定的系统调用。

**尽管 vsyscall 机制已被弃用，但 Linux 内核仍然在内存映射中保留**
ffffffffff600000-ffffffffff601000 这个区域（标记为 [vsyscall]）

### VDSO

---

## **3. 对比 `vsyscall` 和 `vDSO`**
| **特性**         | **`vsyscall`**                | **`vDSO`**                     |
|------------------|-------------------------------|--------------------------------|
| **地址分配**     | 固定 (`0xffffffffff600000`)   | 动态加载（ASLR 支持）          |
| **安全性**       | 低（固定地址易受攻击）        | 高（随机化地址）               |
| **内核支持**     | 旧版机制，已废弃              | 现代默认机制                   |
| **性能**         | 模拟执行（较慢）              | 直接用户态执行（最快）         |
| **调用方式**     | 硬编码地址                    | 通过 `glibc` 或 `dlopen` 调用  |

---

在内存映射中，可以看道vdso这个段，这个地址是随机的

源码在内核中实现
- arch/arm/kernel/vdso.c
- 关键函数:vdso_mremap、install_vvar
  - 速度最快
  - 开销最小,基本等价于函数调用开销


# 一切皆文件的哲学
**“一切皆文件”是 Linux 对系统资源的高度抽象，通过文件接口屏蔽底层差异，提供了简洁、一致的操作方式。**、
这种设计降低了系统复杂性，使得工具、脚本和应用程序能够以统一模式处理多样化资源，是 Linux 强大灵活性的重要基石。

简单来说，在Linux操作系统中，所有的资源（包括普通文件（文本、二进制文件等）、目录、设备（如磁盘、键盘）、进程信息、网络套接字、管道等）都被抽象为了文件。

在用户层面上，我们可以通过对对应的文件进行操作，进而完成对这些资源的操作。

这样做最明显的好处是，开发者仅需要使用一套 API 和开发工具，即可调取 Linux 系统中绝大部分的资源。

举个简单的例子，Linux 中几乎所有读（读文件，读系统状态，读PIPE）的操作都可以用read 函数来进行；几乎所有更改（更改文件，更改系统参数，写 PIPE）的操作都可以用 write 函数来进行。

每一种设备都有用于描述自身的读写方法与属性等(在对应的数据结构中)，将这些方法的地址赋值给对应的函数，将属性抽象成文件的内容，就可以用访问文件的方式来访问这些资源。

虽然在访问这些设备时所调用的函数都是文件的 read 和 write 等，但实际上调用的却是对应设备的读写函数。

按照面向对象语言的视角来理解就是：struct file 是一个抽象类，而各种设备继承自 struct file 并各自实现了读写等方法。在较高的层次就可以将这些设备都看作文件来处理。


![一切皆文件](https://image.aruoshui.fun/i/2025/04/15/vpxf4e-0.webp)


## 硬件识别机制
在Linux当中的`/dev`目录下可以看到存在许多文件;
```powershell
$ ls /dev
AliSecGuard      initctl           
autofs           input              
block            kmsg               
btrfs-control    log                
bus              loop-control       
char             mapper              
console          mcelog              

```

而这些文件被称为设备文件;
同时这些设备文件代表着系统当中的各种硬件设备,它们将为用户的程序提供一个接口;
用户可以通过这些口从而间接的调用硬件;

**驱动程序等后续会说到**

- 驱动程序负责管理和控制硬件,而驱动程序本身也是被OS进行管理的;
- 设备文件本身也是为用户提供一个与用户与驱动交互的接口;
- 而驱动程序则为为设备文件提供的一个与硬件交互的一个接口;
- 这些抽象的接口本身对于OS来说是不知情的;
- OS只知道在调用对应的硬件时只需要去调用对应的设备文件即可;这样就把硬件设备抽象成执行文件了
