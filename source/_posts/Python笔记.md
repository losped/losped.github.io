---
title: Python笔记
date: 2019-10-02 14:13:17
tags: Py
categories: Py
---

### 1.**str**和**repr**的区别

**str**
把值转换为合理形式的字符串，给用户看的。str实际上类似于int，long，是一种类型。
```
>>> print str("Hello,  world!")
Hello,  world!            
>>> print str(1000L)
1000                         
>>> str("Hello, world!")
'Hello, world!'               # 字符串转换之后仍然是字符串
>>> str(1000L)
'1000'
1
2
3
4
5
6
7
8
**repr()**
```

创建一个字符串，以合法python表达式的形式来表示值。repr()是一个函数。
```
>>> print repr("Hello,  world!")
'Hello,  world!'
>>> print repr(1000L)
1000L
>>> repr("Hello,  world!")
"'Hello,  world!'"
>>> repr(1000L)
'1000L'
```
### 2.可更改(mutable)与不可更改(immutable)对象
在 python 中，strings, tuples, 和 numbers 是不可更改的对象，而 list,dict 等则是可以修改的对象。

不可变类型：变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变a的值，相当于新生成了a。

可变类型：变量赋值 la=[1,2,3,4] 后再赋值 la[2]=5 则是将 list la 的第三个元素值更改，本身la没有动，只是其内部的一部分值被修改了。

python 函数的参数传递：

不可变类型：类似 c++ 的值传递，如 整数、字符串、元组。如fun（a），传递的只是a的值，没有影响a对象本身。比如在 fun（a）内部修改 a 的值，只是修改另一个复制的对象，不会影响 a 本身。

可变类型：类似 c++ 的引用传递，如 列表，字典。如 fun（la），则是将 la 真正的传过去，修改后fun外部的la也会受影响

python 中一切都是对象，严格意义我们不能说值传递还是引用传递，我们应该说传不可变对象和传可变对象。
