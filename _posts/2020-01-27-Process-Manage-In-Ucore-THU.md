---
layout: post
title: 清华ucore操作系统的进程管理解析
categories: OS
description: 清华ucore操作系统的进程管理解析
keywords: OS, 清华, ucore, 进程
---

> ucore是清华大学推出的一个用于教学目的的操作系统

# 1.ucore操作系统

ucore操作系统是清华大学计算机系为了课程需求而维护的一个简单的操作系统。ucore的[Github仓库地址](https://github.com/chyyuu/ucore_os_lab)，另外我维(学)护（习）的ucore [Github仓库地址](https://github.com/Neyzoter/ucore_os_lab)。不同于清华的ucore仓库，我的ucore仓库添加了许多中文注释，甚至修复了小的问题。不过，不得不说的是，清华大学的计算机课程真的很硬核！国内其他高校，甚至一些985高校都还需要进一步在教学上提高。

本文章从ucore操作系统的角度来解析操作系统是如何管理进程的。

# 2.ucore内核线程管理

内核线程不需要进行优先级切换，变量常驻内存当中，虚拟内存管理作用不大，故相对用户进程来说更加简单。而用户进程各自管理虚拟内存，互不干扰。当然，一个用户进程内部还可能包含多个线程，线程之间分享内存空间。而操作系统的内核线程通过线程控制块TCB来进行管理。线程控制块是一个数据结构，包含诸多线程相关信息，如名称、mm、pid等。

ucore的内核进程首先会创建一个空闲进程，而后会创建一个内核进程。（[ucore的Lab4](https://github.com/Neyzoter/ucore_os_lab/tree/master/labcodes_answer/lab4_result)，内核创建第0个进程即空闲进程，第1个内核进程）下面对该实验进行解析和理解。

## 2.1 内核线程总体运行流程

1. 每个内核线程都会对应一个线程控制块
2. 线程控制块通过链表链接起来，便于进行遍历、插入、删除等操作
3. 调度器通过县城控制块来使得不同内核线程在不同的时段占用CPU

## 2.2 内核线程运行细节

* **虚拟内存初始化**

  [具体说明](http://neyzoter.cn/2020/01/11/Memory-Manage-In-Ucore-TU/)

* **内核线程创建**

  主要在函数`proc_init @ proc.c`中运行

  1. 初始化PCB列表头

     此处也可以说是TCB。具体而言，初始化工作即将PCB的头尾相连

  2. 初始化PCB Hash表

     具体而言是将N个PCB列表头存放在数组中，对数组中每个PCB列表头进行初始化

     ```c
         for (i = 0; i < HASH_LIST_SIZE; i ++) {
             list_init(hash_list + i);
         }
     ```

     ```
         list1         list2       list3        list4        list5
     |------------|------------|------------|------------|------------|
       |_______|    |_______|    |_______|    |_______|    |_______|  
     ```

  3. 给空闲线程分配PCB空间并进行初始化

     `alloc_proc @ proc.c`

  4. 设置空闲线程为当前线程current

  5. 创建第1个内核线程

     1. 初始化trap frame

     2. fork

        分配PCB空间

        分配内核堆栈（物理）

        设置PID

        设置Hash 表

        PCB加入到PCB列表中

        设置为可运行

* **线程运行**

  空闲任务内，进行线程的调度`schedule @ sched.c`

# 3.ucore用户进程管理

## 3.1 用户进程管理涉及内容

* **内存管理**

  与内核线程不同，用户进程的内存管理部分增加了用户态虚拟内存的管理，把部分物理内存映射为用户态虚拟内存。

  1. 如果某进程执行过程中，CPU在用户态下执行（在CS段寄存器最低两位包含有一个2位的优先级域，如果为0，表示CPU运行在特权态；如果为3，表示CPU运行在用户态。），则可以访问本进程页表描述的用户态虚拟内存，但由于权限不够，不能访问内核态虚拟内存。
  2. 不同进程有着不同的页目录，所以即使虚拟地址相同，但是会映射到不同的物理地址，不会造成冲突

* **进程管理**

  1. 与内存管理相关的部分

     包括建立进程的页表和维护进程可访问空间（可能还没有建立虚实映射关系）的信息；加载一个ELF格式的程序到进程控制块管理的内存中的方法；在进程复制（fork）过程中，把父进程的内存空间拷贝到子进程内存空间的技术。

  2. 与用户态进程生命周期管理相关的部分

     包括让进程放弃CPU而睡眠等待某事件；让父进程等待子进程结束；一个进程杀死另一个进程；给进程发消息；建立进程的血缘关系链表

## 3.2 用户进程管理总体运行流程

* **虚拟内存初始化**

* **内核线程创建**

  包括第0个空闲内核线程（所有线程的祖先）和第1个内核线程

* **用户进程创建**

  第1个内核线程内会创建一个子（内核）线程，该内核线程实现的是执行一个用户进程，具体是**根据ld文件将程序的不同内容加载到某进程的用户空间中**。

## 3.3 用户进程运行细节

* **用户进程代码加载**

  Makefile文件会将用户进程的代码所在文件进行编译，用户程序会被bootloader加载到内存中（没有文件系统的情况下，如果有文件系统，则不需要通过此类方法加载），而且以全局变量的方式保存用户代码的起始地址和大小。

  在上述编译的时候，`user.ld`文件描述了用户程序的用户虚拟空间的执行入口虚拟地址，如下

  ```
  SECTIONS {
      /* Load programs at this address: "." means the current address */
      . = 0x800020;
  ```

  所有用户进程共享**内核**虚拟内存，映射到同样的物理内存空间。同时用户进程的虚拟内存空间地址范围是一样的，但是映射到不同的物理内存空间。具体见下图，`User Stack`是用户堆栈，`User Program & Heap`是用户进程代码和堆：

  ```
  /* *
   * Virtual memory map:                                          Permissions
   *                                                              kernel/user
   *
   *     4G ------------------> +---------------------------------+
   *                            |                                 |
   *                            |         Empty Memory (*)        |
   *                            |                                 |
   *                            +---------------------------------+ 0xFB000000
   *                            |   Cur. Page Table (Kern, RW)    | RW/-- PTSIZE
   *     VPT -----------------> +---------------------------------+ 0xFAC00000
   *                            |        Invalid Memory (*)       | --/--
   *     KERNTOP -------------> +---------------------------------+ 0xF8000000
   *                            |                                 |
   *                            |    Remapped Physical Memory     | RW/-- KMEMSIZE
   *                            |                                 |
   *     KERNBASE ------------> +---------------------------------+ 0xC0000000
   *                            |        Invalid Memory (*)       | --/--
   *     USERTOP -------------> +---------------------------------+ 0xB0000000
   *                            |           User stack            |
   *                            +---------------------------------+
   *                            |                                 |
   *                            :                                 :
   *                            |         ~~~~~~~~~~~~~~~~        |
   *                            :                                 :
   *                            |                                 |
   *                            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   *                            |       User Program & Heap       |
   *     UTEXT ---------------> +---------------------------------+ 0x00800000
   *                            |        Invalid Memory (*)       | --/--
   *                            |  - - - - - - - - - - - - - - -  |
   *                            |    User STAB Data (optional)    |
   *     USERBASE, USTAB------> +---------------------------------+ 0x00200000
   *                            |        Invalid Memory (*)       | --/--
   *     0 -------------------> +---------------------------------+ 0x00000000
   * (*) Note: The kernel ensures that "Invalid Memory" is *never* mapped.
   *     "Empty Memory" is normally unmapped, but user programs may map pages
   *     there if desired.
   *
   * */
  ```

* **用户进程创建**

  ucore可以通过系统调用`sys_exec @ syscall.c`来实现用户进程的创建工作，进而调用`do_execve @ proc.c`来实现具体的创建工作。具体而言，ucore操作系统会fork一个内核线程`user_main`，然后`user_main`进程会将通过系统调用`sys_exec`，进而调用`do_execve`来将该内核线程设置为用户线程。`do_execve`函数主要进行了：

  1. 清空mm

     ```c
     // [LAB5 SCC] 删除原来的内存空间
     // [LAB5 SCC] 如果是内核进程则mm是NULL
     if (mm != NULL) {
         lcr3(boot_cr3); // [LAB5 SCC] 设置为内核空间页表
         if (mm_count_dec(mm) == 0) {
             exit_mmap(mm);
             put_pgdir(mm);
             mm_destroy(mm);
         }
         current->mm = NULL;
     }
     ```

  2. 加载应用程序执行码到当前进程的新创建的用户态虚拟空间中

     `load_icode`完成这一步工作，包括：

     1. 创建和初始化mm数据结构

     2. 创建页目录表，拷贝为内核虚拟空间的页目录表
     3. 根据每个段（代码段、数据段、BSS段等）的起始位置和大小建立对应的vma结构，并将vma插入到mm
     4. 拷贝代码段、数据段、BSS段，建立物理地址和虚拟地址的映射关系
     5. 用户进程设置用户栈，为此调用`mm_mmap`函数建立用户栈的vma结构（会设置虚拟地址的start和end），并通过`pgdir_alloc_page`来创建物理地址空间和线性地址的映射关系
     6. cr3寄存器赋值为pgdir
     7. 设置中断帧，包括**栈顶USTACKTOP**等

     

*补充：*

<img src="/images/posts/2020-01-27-Process-Manage-In-Ucore-THU/QA1.png" width="700" alt="补充">

