---
title: 如何分析App内存
date: 2017-11-18 11:33:39
tags: [iOS, Xcode, Instruments, 内存]
categories: Question

---

> Xcode下查看app内存使用情况有2种方法：

> - Navigator导航栏中的Debug navigator中的Memory

> - Instruments

 1. Debug navigator中的Memory
-----------------------

此方法是查看内存最简单直接有效的方法，真机调试时，通过Debug navigator中Memory查看app内存，入口如图

{% asset_img pic1.jpeg 图片来源见水印 %}

根据这个值查看app内存占用，这个内存是当前app占用的总内存，是堆栈内存、虚拟内存（OpenGL占用的显存算在虚拟内存中里面）的总和。

2. Instruments
-----------------------

启动Instruments的方法是，Product->Profile，经过漫长的编译时间后，出现Instruments界面，如下图

{% asset_img pic2.jpeg 图片来源见水印 %}

Instruments中，可以分析内存的工具有Activity Monitor、Allocations、Leaks。

> **Leaks**
Leaks是检测内存泄露的工具，很有用。Leaks运行中，看到下面这个红叉叉就表示有内存泄露了

> {% asset_img pic3.jpeg 图片来源见水印 %}  


----------


> **Allocations**
Allocations是检测堆栈内存的，下面的VM tracker检测虚拟内存。Allocations运行起来如下图

> {% asset_img pic4.jpeg 图片来源见水印 %}

> Allocations永远比Debug navigator Memory中显示的内存要小，就是因为Allocations中没有统计虚拟内存。
下图参考：
{% asset_img pic5.jpeg 图片来源见水印 %}

> 部分malloc出来的内存也算在虚拟内存中，下图参考自A look at how malloc works on the Mac
{% asset_img pic6.jpeg 图片来源见水印 %}


----------

> **Activity monitor**
Activity monitor看手机整体内存情况的，这里的显示app内存值和Debug navigator中的Memory显示的值是一样的
{% asset_img pic7.jpeg 图片来源见水印 %}


----------

> 其他
> --

> - App最多能占用多少内存不闪退
>> 占用机器内存的一半左右就会闪退，和系统版本、后台程序数有关。
{% asset_img pic8.jpeg 图片来源见水印 %}
不同渠道对内存有不同的要求，例如如下某渠道
{% asset_img pic9.jpeg 图片来源见水印 %}

> - iOS的App为什么内存没有泄露，内存却降不下来
>>  eg: 创建大概20个哥布林spine动画，此时内存占用46M，然后释放掉，内存占用竟然还是46M，以为是spine有内存泄露，Leaks检测没有发现内存泄露。反复加载释放20个哥布林，内存都没有超过48M，但是为毛内存没有下降，而是维持在46M左右
{% asset_img pic10.jpeg 图片来源见水印 %}
因为  
（1）图片加入了TextureCache，占用了部分内存  
（2）malloc出来的一部分内存算到了VM（虚拟内存）中，为了下次malloc速度更快，这部分内存虽然调用了free，但iOS系统依然没有将其回收。这就是上面说的部分malloc出来的内存也算在虚拟内存中。