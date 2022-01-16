---
title: deque深度探索
date: 2020-04-30 15:47:48
tags: [C++,STL,数据结构]
categories: 文章
---

>  ***deque*** 是一种双向开口容器，不同于 ***stack*** 与  ***queue*** ，数据均可由它的两端进出。

#### STL 中的 ***deque*** 

> 在 STL 中， ***deque*** 的具体实现为下图所示：

<center>
{% asset_img deque.png deque内部结构示意图 %}
</center>
如上所示， ***deque*** 容器是由一个个 ***buffer*** 所构成，并且由一条顺序容器buf_map(***vector***)将其 ***buffer*** 串联起来用以记录 ***buffer*** 的地址和数量，最后加由 ***deque iterator*** 以构成连续空间的假象——

<!--more-->

- deque
  - buffer
  - buf_map
  - iterator

其中 ***buffer*** 就是一段连续内存，***buf_map*** 将决定 ***buffer*** 们在 ***deque*** 中的先后顺序。

#####  ***deque*** 中的 ***buffer***

<center>
{% asset_img buffer.png buffer内部结构示意图 %}
</center>

***buffer*** 是一组固定大小的序列，其中存放着 ***deque*** 应该储存的数据；

每一个 ***buffer*** 的长度由所存数据的大小决定——

在 GNUC 编译器中，一个 ***buffer*** 的长度是：

```C++
template <typename T>
buffer_size = sizeof(T) > 512 ? 1 : 512 / sizeof(T);
```

当所存数据的大小超过 512 字节时，仅将一个 ***buffer*** 的长度设置为 1，否则将其设为 512/sizeof(T) ,例如当 ***deque*** 容器所储存数据类型为 ***int(sizeof=4)*** 时，那么一个 ***buffer*** 应当存放 128 个数据。

#####  ***deque*** 中的 ***buf_map***

<center>
{% asset_img buf_map.png buf_map内部结构示意图 %}
</center>

***buf_map*** 是用于记录上述 ***buffer*** 所在位置的表，具体实现为 ***vertor*** ；

此 ***vector*** 中的每一个每一个值都储存着一个指针，用以指向每一个 ***buffer*** 的起点。

该 ***buf_map*** 中第一个指针指向 ***deque*** 中元素所占据第一个 ***buffer*** 的首位值。***buf_map*** 中指针指向的地址是与 ***deque*** 所存储数据的空间无关的，***buf_map*** 主要也是唯一的作用仅是用以记录组成 ***deque*** 容器中的 ***buffer***。

因而 ***buf_map*** 中所存储的指针数目即为 ***deque*** 容器中 ***buffer*** 的个数(*map_size*)。

当使用者将数据不断存储到 ***deque*** 的过程中，若某一 ***buffer*** 即将不能满足储存要求(存满了)， ***deque*** 将会开辟出一块新的内存空间作为新 ***buffer*** 并将其地址储存为 ***buf_map*** 的新元素。

需要注意的是，当 ***buf_map*** 也无法满足存储要求(其内部空间已经存满了指向 ***buffer*** 的指针)，***buf_map*** 将像正常的 ***vector*** 一样进行动态扩容，增长为其原内存的两倍(*GNUC*)大小；不过，在 ***vector*** 成功新开内存将原存储内容移动至新内存的过程中，***buf_map*** 会将原内容尽可能的存贮在新内存的中间位置，以确保在后续 ***deque*** 的数据存储过程中**两端均有足够的空间用以增长**。

#####  ***deque*** 中的 ***iterator***

<center>
{% asset_img iterator.png iterator内部结构示意图 %}
</center>

因 ***deque*** 是由一个个 ***buffer*** 和 ***buf_map*** 所拼凑出来的 “连续空间”，所以 ***deque iterator*** 的设计显得尤为重要！

如上图所示，一个 ***deque iterator*** 由四部分组成：

- deque iterator
  - cur[T*]
  - first[T*]
  - last[T*]
  - node[T**]

其中 ***cur***、***first*** 和 ***last*** 均为指向所存储数据类型的指针，分别指向——

***cur***：**当前 *iterator* 所指元素**的位置

***first***：当前 *iterator* 所指元素 **所在 *buffer* 的起始位置**

***last***：当前 *iterator* 所指元素 **所在 *buffer* 的末尾位置**

而 ***node*** 是 **指向该 *iterator* 所指元素所在的 *buffer* 在 *buf_map* 中存储的位置**；

因 ***buf_map*** 储存的是指向一个 ***buffer*** 的指针(*T\**),所以 ***node*** 的类型应为 ***pointer to (pointer to T)***(T**)。

在本文章的首图中，以 ***deque*** 的两个特殊 ***iterator***(***start & finish***) 为例子：

***start*** 中 ***cur*** 指向 ***deque*** 容器第一个元素；***first*** 指向 ***deque*** 容器中第一个 ***buffer***的开始 ，这也是 ***buf_map*** 中存储的第一根指针指向的位置；***end*** 指向 ***deque*** 容器中第一个 ***buffer*** 的结束；***node*** 指向 ***buf_map[0]***。

***finish*** 中 ***cur*** 指向 ***deque*** 容器最后一个元素的下一个元素的位置(*deque.end()*)，首图恰好展示了一个特殊情况—— ***deque*** 中的数据恰好占据完一整个 ***buffer***，因此 ***deque*** 重新开辟出一块 ***buffer***，所以自然 ***first*** 指向这个新 ***buffer*** ；***end*** 指向此 ***buffer*** 的结束；***node*** 指向 ***buf_map[buf_map.size()-1]***。

###### ***deque iterator*** 中的运算符重载

为了模拟连续空间， ***deque iterator*** 针对寻常指针可操作的部分运算符进行了重载以支持相关操作：

> 以下代码皆为简化版本，仅表达内部逻辑，实际操作中应遵循设计规则，参数类型等可能需要调整

***operator++***

```c++
template <typename T>
deque<T>::iterator operator++(){
    ++node; //将 iterator 中指向元素的指针移动至下一个
    if(cur==last){ //如果此时移动至了此 buffer 的末尾
        ++node; //将指向 buf_map 某位置的指针移动至下一个以定位到下一个 buffer
        cur = first = *node; //将新 buffer 的首元素地址赋值给 cur 和 first
        last = first+buf_size; //将 last 指针设置为新 buffer 的结尾(first+一个buffer容纳的数量)
    }
    return *this;
}
```

***operator+=***

```c++
template <typename T>
deque<T>::iterator operator+=(int n){
    int skip_num = n + (cur-first); //加上该元素当前所在buffer之前的所有数量
    if(skip_num>=0 && skip_num<buf_size){ //此时若加上 n 是否还在本 buffer 内(考虑n为负数)
        cur +=n ;
    }else{
        int buf_skip_num = skip_num>0 ? skip_snum/buf_size //若n为正数,该+=操作应跳跃的buffer数目
            : -((skip_num/buf_size)+1); //若n为负数,该+=操作应跳跃的buffer数目
        node += buf_skip_num; //将指向 buf_map 某位置的指针移动至下一个以定位到+=操作后iterator应指向位置所在的 buffer
        first = node; //重设first
        last = first+buf_size; //重设last
        cur = first + (skip_num - buf_skip_num*buf_size ); //将所cur指针移动至+=操作后在该buffer内应指向的位置
    }
    return *this;
}
```

***operator-***

```c++
template <typename T>
int operator-(deque<T>::iterator n){
    //(node - n.node - 1)为两iterator之间间隔的buffer数目
    //(cur - first)为当前iterator所指元素到所处buffer结尾的元素数目
    //(n.last - n.cur)为减号后iterator所指元素到所处buffer起始的元素数目
    return buf_size * (node - n.node - 1) + (cur - first) + (n.last - n.cur);
}
```



operator*

```c++
template <typename T>
T& operator*(){
    return *cur;
}
```

