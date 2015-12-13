---
date: 2014-12-17 22:52:12+00:00
layout: post
title: CS-APP 146th Call Stack and Frame
thread: 6
categories: 读书笔记
tags: CS-APP
---

##关于C中的调用栈帧
之前对C的理解确实比较浮潜，一直停留在《C和指针》上面，最后一章好像讲到了函数调用的过程，恰好也是从汇编的角度去讲解的，但是一直没有详细的去读那一部分。相关的知识也停留在“每次函数调用的时候，每个函数都有一个自己的函数栈”，但是其中调用者与被调用者的栈的关系是什么？他么是怎么完成函数的传递的一直没有详细的去研究。先按照书中的sample code分析如下。

##Linux的栈结构
我们知道Linux中每一个进程都有一个虚拟的地址空间，分为用户空间和内核空间，我们的程序都是在用户空间中，而在用户空间中，我们的程序又分为若干段。一下是一个本节用到的sample code执行size后的结果。
   >text     data     bss     dec     hex filename <br>
   >1105     276       4    1385     569 a.out

简单的可以看出一个程序访问代码段，数据段，堆栈段。详细的可以参看微机原理类的书籍，哪里写汇编代码的部分应该将的很详细，这里主要还是C相关的东西。其实数据段又分为初始化数据段和未初始化的数据段，像C中的全局变量就在初始化的数据段(这也是为什么全局变量的初始值为0的原因)，在C语言中堆和栈又是两个现对独立的东西，如果程序中需要使用堆空间我们就要像系统请求(malloc)，等使用完后需要哦归还给操作系统。而栈就不同了，栈主要用于保存函数的局部变量和在函数见传递参数使用。
如下是一个简单的sample code

```
```

  	1 #include <stdio.h>
  	2 #include <stdlib.h>
  	3
  	4 int globalx = 12;
  	5 void print()
  	6 {
  	7    int localx = 0;
  	8    printf("address of function print's localx=%p\n", &localx);
  	9 }
 	10
 	11 int main()
 	12 {
 	13    int localx = 0l;
 	14    int *mallocx = (int *)malloc(sizeof(int));
 	15    printf("address of function main's localx=%p\n", &localx);
 	16    printf("address of globalx=%p\n", &globalx);
 	17    printf("address of mallocx=%p\n", &mallocx);
 	18    printf("address of print=%p\n", &print);
 	19    print();
 	20
 	21    printf("pid = %d\n", getpid());
 	22
 	23    //LOOP
 	24    while(1);
 	25    return 0;
 	26 }

我们在这里需要干什么呢？首先我们可以看到我们列印了许多函数或者变量的地址出来，通过查看每个变量或者函数的地址我们可以更好的理解进程的地址空间。我们执行a.out后的输出结果如下:
>address of function main's localx=0xbff50128<br>
>address of globalx=0x804a028<br>
>address of mallocx=0x8229008<br>
>address of print=0x804847d<br>
>address of function print's localx=0xbff500fc<br>
>pid = 5477

从其中我们也可以得出程序的大致存储空间如下图。
![function_stack]
关于整个进程的地址空间的的话我们可以查看/proc/$PID/maps文件得到。
![call_process]
从其中我们也更清晰的看到了程序中的每个段在整个进程空间中的位置。

##函数调用到底发生了什么
通过上面的例子，我们大概了解了C中的函数栈结构，下面就来看看，具体的函数调用的时候发生了什么？不过在往下看之前你明白一下几点。

* push和pop的原理
* 发生一次函数调用的时候%eax，%edx，%ecx是由调用者保存的，而%ebx，%esi，%edi是由被调用者保存的

清楚以上两点后我们就可以继续前进了。作为一个实例考虑以下的代码：
```
```

	1 int swap_add(int *xp, int *yp)
	2 {
	3    int x = *xp;
	4    int y = *yp;
	5
	6    *xp = y;
	7    *yp = x;
	8    return x + y;
	9 }
	10
	11 int caller()
	12 {
	13    int arg1 = 534;
	14    int arg2 = 1057;
	15    int sum = swap_add(&arg1, &arg2);
	16    int diff = arg1 - arg2;
	17
	18    return sum * diff;
	19 }

使用`gcc -S swap_call.c`可以得到相应的汇编代码,为了方便理解我增加了部分注释。
```
```

  	5 swap_add:
	8    pushl %ebp
 	11    movl  %esp, %ebp

 	13    subl  $16, %esp
 	14    movl  8(%ebp), %eax
 	15    movl  (%eax), %eax
 	16    movl  %eax, -8(%ebp)
 	17    movl  12(%ebp), %eax
 	18    movl  (%eax), %eax
 	19    movl  %eax, -4(%ebp)
 	20    movl  8(%ebp), %eax
 	21    movl  -4(%ebp), %edx
 	22    movl  %edx, (%eax)
 	23    movl  12(%ebp), %eax
 	24    movl  -8(%ebp), %edx
 	25    movl  %edx, (%eax)
 	26    movl  -4(%ebp), %eax
 	27    movl  -8(%ebp), %edx
		  ;将结果放入eax中，调用函数从eax中取出结果
 	28    addl  %edx, %eax
 	29    leave
 
 	38 caller:
 	39 .LFB1:
		  ;保存当前函数的ebp
 	41    pushl %ebp
 	44    movl  %esp, %ebp
		  ;为函数局部变量开辟一个空间
 	46    subl  $24, %esp
          ;int arg1 = 534;
 	47    movl  $534, -16(%ebp)
          ;int arg2 = 1057;
 	48    movl  $1057, -12(%ebp)
          
		  ；通过栈来函数间传递参数
 	49    leal  -12(%ebp), %eax
 	50    movl  %eax, 4(%esp)
 	51    leal  -16(%ebp), %eax
 	52    movl  %eax, (%esp)
          ;调用函数
 	53    call  swap_add
          ;将计算结果从eax中取出，放入ebp-8的位置
 	54    movl  %eax, -8(%ebp)
          ；计算arg1 - arg2的结果并将其放入ebp-4中
 	55    movl  -16(%ebp), %edx
 	56    movl  -12(%ebp), %eax
 	57    subl  %eax, %edx
 	58    movl  %edx, %eax
 	59    movl  %eax, -4(%ebp)
          ；计算sum * diff的结果并将其放入eax中 
 	60    movl  -8(%ebp), %eax
 	61    imull -4(%ebp), %eax
 	62    leave

下面是一个简单的call stack图

![call_stack]

##通过这个过程我学到了什么？
做所有的事情我们都应到学到一些东西，尤其是看书，如果一无所获，那就是纯粹在浪费时间，通过这调用栈的分析，我更清楚了理解了Linux中函数的调用过程，每个函数间的参数是怎样传递的，每个函数的局部栈是怎么管理的。

-END
[call_stack]: ../album/ch3/call_stack.jpg "call_stack"
[function_stack]:  ../album/ch3/function_stack.png "function_stack"
[call_process]: ../album/ch3/call_process.png "call_process"		

