---
title: iterator的设计原则
date: 2020-04-28 17:51:45
tags: [文章,STL]

---

> 在 STL 中，`iterator` 是容器和算法之间的桥梁，`iterator`   必须满足算法所需要得知的相关内容方能使容器能够正常使用算法来进行一系列操作。

---

#### iterator

一般情况下，容器的`iterator` 必须定义如下 `typedef`：

- `iterator` 
  - `typedef ... value_type`
  - `typedef ... iterator_category`
  - `typedef ... difference_type`
  - `typedef ... pointer`
  - `typedef ... reference`

其中，各 `type` 表示的意义如下：

- `value_type` : `iterator` 所 *指* 元素的类型
- `iterator_category` : `iterator` 的分类，用以表示当前`iterator` 支持何种操作
  
  - `iterator_category`有以下分类[[信息来自cplusplus.com](http://www.cplusplus.com/reference/iterator/iterator_traits/)] : 
  - ***input_iterator_tag*** : 输入迭代器
  - ***output_iterator_tag*** : 输出迭代器
  - ***forward_iterator_tag*** : 转发迭代器
  - ***bidirectional_iterator_tag*** : 双向迭代器
  - ***random_access_iterator_tag*** : 随机访问迭代器



<!--more-->

<center>
{% asset_img category.png category功能示意图 %}
</center>
- `difference_type` : 任意两个`iterator` 之间的 ***最大差距***(注意不是距离，它包含了负值) 能够使用何种类型表示？

  - 一般情况下，皆使用 ` ptrdiff_t` 来代指这一 typedef；它在编译器中被定义为 signed int 。

- `pointer` : 指针

- `reference` : 引用

---

若如此定义好相关 typedef ，算法相关函数便可以通过 `iterator` 获取到该算法实现所需的内容；

但由于`iterator` 有退化的可能性【`iterator` 本身是一个普通的指针而非针对该容器所适配的类，例如`vector`容器】，因此 STL 增加了一个中间层类用以适配`iterator` 的各种情形——***iterator_traits***

#### iterator_traits

> iterator_traits 的主要实现为一个模板类，用以接受各个容器的`iterator` ；其中做了两个偏特化，以应对当 iterator_traits 接收到普通指针 `T*`  和 `const T*` 的状况。

在泛化的`iterator_traits`实现中，只是直接定义算法所需相关 type 为 `iterator` 的 typedef；

在偏特化的两个`iterator_traits`实现中有如下定义：

```C++
template <typename T*>
...
typedef T value_type;
typedef random_access_iterator_tag iterator_category;
typedef ptrdiff_t difference_type;
typedef T* pointer;
typedef T& reference;
```

```C++
template <typename const T*>
...
typedef const T* pointer;
typedef const T& reference;
```

> 上述代码需要注意的是，`value_type`的定义，`const T*`亦为 T ；这是因为在算法中，获取`value_type`的主要目的是为了声明变量，而声明 const 变量无任何意义。

如此，当某一容器使用算法时，算法内部便可以通过`iterator_traits`获取容器相关信息，用以进行相关操作;

***例如***：

```c++
template <typename corpusIter> std::pair<corpusIter, corpusIter> operator () ( corpusIter corpus_first, corpusIter corpus_last ) const {
	BOOST_STATIC_ASSERT (( boost::is_same<
		typename std::iterator_traits<patIter>::value_type, //use value_type
		typename std::iterator_traits<corpusIter>::value_type>::value ));//use value_type
    
	if ( corpus_first == corpus_last ) 
        return std::make_pair(corpus_last, corpus_last);
	if (pat_first ==pat_last ) 
        return std::make_pair(corpus_first, corpus_first);
	const difference_type k_corpus_length  = std::distance ( corpus_first, corpus_last );
	if ( k_corpus_length < k_pattern_length ) 
		return std::make_pair(corpus_last, corpus_last);
	return this->do_search ( corpus_first, corpus_last );
}

```

> 以上代码摘自 ***Boost*** 库中，算法`boyer_moore`的运算符重载，其中注释的两行使用了——
>
> `std::iterator_traits<patIter>::value_type`来获取到`iterator` 的`value_type`

