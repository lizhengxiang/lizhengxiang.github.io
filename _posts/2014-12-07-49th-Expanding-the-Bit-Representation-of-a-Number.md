---
date: 2014-12-07 22:52:12+00:00
layout: post
title: CS-APP 49th Expanding the Bit Representation of a Number
thread: 4
categories: 读书笔记
tags: CS-APP
---

##关于C中的类型转换

对于C中的类型转换一直停留在《C和指针》中的那张类型提升表中，其中给出了类型的提升顺序，也就是说如果一个表达式中同时出现了int和short，short类型的变量会自动提升为int。但是对其中的具体缘由，从来没有去详细的想过，之前看书比较粗糙，没有细读一本书，理解了整体的意思就觉得自己真的理解的作者的意图，最近在[coursera]上看《[史记]》的开放课程的时候老师讲到一个看怎样读经书的方法，讲到一篇经典的文章的时候，都会去揣测每一个字的含义，以及为什么要用这个字并且出现在这个位置。然后自己去翻了一下前人写的那些个注疏,发现好多的经典我们都没有真正的理解其中的含义。

言归正传，继续写我的读书笔记，当一个无符号类型的值从较小的数据类型转换为一个较大的数据类型的时候，高位补零就可以了，这里也叫作零扩展(zero extersion)。如果将一个有符号的值从较小类型转换成较大的类型的时候会发生什么的？难道也是一样的吗？非也，这里的概念我也是第一次看到，这里会发生符号扩展(sign extension),也就是说高位补什么是根据他的最高位决定的。

##关于类型转换的一些建议

关于类型转我觉得C++之父的那句话永远是真理,C风格的隐时类型转换正的很邪恶，经常会发生一些和你的直觉不相符的事情。这也就是C程序员永远要考虑的一个问题。下面就是一段很邪恶的代码。
```
```
 
  	3 float sum_elements(float a[], unsigned int length)
	4 {
  	5    float sum = 0.0;
  	6
  	7    int i;
  	8    for(i = 0; i < length-1; i++)
  	9       sum += a[i];
 	10
 	11    return sum;
 	12 
	}
我使用如下的调用会
```
```

	17 float s[] = {1,2,3,4};
	18 float result = sum_elements(s, 4);
 	22 result = sum_elements(s, 0);

你猜会发生什么？
>qian@qian-vm:~/excise/csap/ch2$ ./a.out <br>
>result of sum is 6.000000<br>
>Segmentation fault (core dumped)

-END
[coursera]:http://coursera.org/     "coursera"
[史记]:https://class.coursera.org/shiji-002 "史记"
