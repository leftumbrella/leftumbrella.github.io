---
title: list深度探索
date: 2020-04-29 16:55:22
tags: [文章,STL,数据结构]
---

>  ***list*** 和 ***vector*** 一样，也是一种线性数据结构，不同于 ***vector*** 的是， ***list*** 所占用的内存空间并不是连续的，但它通过各种手段制造出一种连续内存空间的*假象*； ***list*** 因其内部结构的某些特点，在动态操作上能够消耗及其短暂的时间。

####  ***list*** 是由什么构成的？

通常来说，一个 ***list*** 类仅需存放一根指向 ***list node*** 的指针；表示 ***list*** 节点的 ***list node*** 中才为应储存的内容。

-  ***list*** 

  -  *** *list node*** —— 指向列表节点的指针

    -  *** *prev[ list node\* ]*** —— 指向列表中上一个元素位置的指针
    -  *** *next[ list node\* ]*** —— 指向列表中下一个元素位置的指针
    -  ***data[ T ]*** —— 列表中存放的数据

<!--more-->

#### STL 中的 ***list*** 

<center>
{% asset_img list.png list内部结构 %}
</center>

如上图所示，在 STL 中， ***list*** 所呈现的内部结构为 **双向环状列表** ；

并且，此 ***list*** 首节点的 ***prev*** 指针与末节点的 ***next*** 指针均指向一个并不存在于 ***list*** 容器内部的节点，从而形成环状。

#####  ***list*** 容器的 ***iterator***

因 ***list*** 内部存储的非连续性，因而无法像 ***array*** 或 ***vector*** 一样使用正常的指针来充当 ***iterator***。

- ***list iterator***
```
typedef T value_type;
typedef T* pointer;
typedef T& reference;
typedef bidirectional_iterator_tag iterator_category;
typedef ptrdiff_t difference_type;
```
上表列出了 ***list iterator*** 定义的五个 ***iterator_traits*** 所需 type；除此以外， ***list iterator*** 内部还应存在一根指向 ***list node*** 的指针：

`list_node<T>* node;`

######  ***list iterator*** 中运算符的重载

为了在使用中模拟连续空间， ***list iterator*** 对与寻常指针所支持的一系列运算符进行了适应自身的重载。

- operator++

  - ```c++
    template <typename T> list_iterator<T>& operator++(){
        node = node->next;
        return *this;
    }
    ```

- operator*

  - ```c++
    template <typename T> T operator*(){
            return node->data;
    }
    ```

##### ***list*** 中多余出来的节点

如上图所示，呈环状的 ***list*** 中存在一个不属于 ***list*** 本身的节点，当调用`list.end()`时返回的 ***iterator*** 指向的便是这个节点。

在 ***GNUC++ 7.4*** 版本编译器中；经测试 `list.end() == (--list.begin())`，这也证实了这一现象；

不过，按理说这个不在 ***list*** 内部的节点所存放的数据、 ***\*prev*** 和 ***\*next*** 应该都为空，但事实上，该节点两根指针又分别指向了 ***list*** 的尾首；并且该节点的 ***data*** 是有值的，该值为 ***list*** 模板参数类型，并且会随着 ***list*** 所存储内容变化而变化，暂未发现规律……

> 这一现象在 ***GNUC*** 编译器里究竟代表着什么呢？
>
> *阅读源码，未完待续……*