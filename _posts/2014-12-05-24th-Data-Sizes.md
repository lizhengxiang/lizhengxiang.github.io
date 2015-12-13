---
date: 2014-12-06 00:18:12+00:00
layout: post
title: CS-APP 24th Data Sizes读书笔记
thread: 2
categories: 读书笔记
tags: CS-APP
---
##在32位机器和64位机器直接移植的时候的注意问题：
由于在32位机器中我们有时候将指针直接赋给要一个int变量，但是在64位机器中int占4个字节而指针为8个字节，这一点还是需要注意的。

##关于大段小段：
说实话这是一个很不好记得概念，每次碰到这个问题后都要查一下怎么对应的，最搞笑的是面试的时候经常会问到，请将下面的数用大段和小段的格式表示出来。。。。。。，这都是一些很无聊的面试题，如果你正的想答对这个题的话可以记住，我们常用(intel处理器)的是小段，就是小地址存放地位的值，位置和权值对应。

说到面试，也是有好多的苦水要吐吐了，为什么面试每个公司的时候面试官的手里明明拿着我们的简历和还要要求做一下自我介绍呢？难道你在考验我对自己的熟练程度吗？

练习题，大小端的转换:
```
```

	#include <stdio.h>
	#define MASK 0xFF
	int main()
	{
   		int little = 0;
   		scanf("%d", &little);
   		int big = 0;
   		int i;
  		for(i = 0; i < 4; i++){
      		big |= (MASK & (little >> i * 8)) << (24 - i*8);
  		 }
  		 printf("little endian = %08x\nbig endian = %08x\n", little, big);
   		return 0;
	}

