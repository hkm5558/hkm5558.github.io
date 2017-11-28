---
title: KVC机制如何通过key找到value
date: 2017-11-28 19:02:54
tags: [iOS, KVC]
categories: Question
---

- **setValue:forKey**:



> 1.  
首先搜索`setKey:`方法。  
（key指成员变量名，首字母大写）  
2.   
上面的setter方法没找到, 如果类方法`accessInstanceVariablesDirectly`返回`YES`。  
那么按 `_key`, `_isKey`, `key`, iskey的顺序搜索成员名。（`NSKeyValueCodingCatogery`中实现的类方法，默认实现为返回`YES`）  
3.   
如果没有找到成员变量，调用`setValue:forUnderfinedKey:`

- **valueForKey**:

> 1. 
首先按`getKey`，`key`，`isKey`的顺序查找getter方法，找到直接调用。如果是`BOOL`、`int`等内建值类型，会做`NSNumber`的转换。  
2.  
上面的getter没找到，查找`countOfKey`、`objectInKeyAtindex、KeyAtindexes`格式的方法。如`果countOfKey`和另外两个方法中的一个找到，那么就会返回一个可以响应`NSArray`所有方法的代理集合的`NSArray`消息方法。  
3.  
还没找到，查找`countOfKey`、`enumeratorOfKey`、`memberOfKey`格式的方法。如果这三个方法都找到，那么就返回一个可以响应`NSSet`所有方法的代理集合。  
4.  
还是没找到，如果类方法`accessInstanceVariablesDirectly`返回`YES`。那么按 `_key`，`_isKey`，`key`，`iskey`的顺序搜索成员名。    
5.  
再没找到，调用`valueForUndefinedKey`。  

