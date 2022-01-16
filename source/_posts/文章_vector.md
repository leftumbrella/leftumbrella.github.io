---
title: vector深度探索
date: 2020-04-28 14:43:26
tags: [文章,STL,数据结构]
---

> vector 算是一种可扩充的`array`，用于储存它的内存与`array`一样，总是连续的。

#### 一个 vector 的内部结构可能是什么样子？ ####

- `vector`
  - `size[int]`——长度
    
  - `capacity[int]`——容量
    
  - `data[T*]`——数据项

如上所示，我们可以直接创建包含以上三个属性的`vector`类，其中`size`用以表示`vector`的长度，它可以是`int`型，`capacity`同理，它用来表示`vector`现在所有开辟出的内存空间的大小，而`data`自然为 ***指向模板参数类型的指针***。

因此在以上结构下，`sizeof(vector)` 得出的大小为两个 `int` 加 一根指针，这一数值在32位电脑上为`3*4=12`。

####  vector 是如何被创建出来的 ####

以下代码以 `copy constructor` 函数为例，展示了一个 `vector` 是如何创建出来的： 

```c++
template <typename T> vector(const vector<T>& v)
	:_capacity(2 * v.size()), _size(0) { //将新 vector 的容量设置为原 vector 长度的两倍
	_elem = new T[_capacity]; //开辟内存空间
	for (auto i : v) { //At C++11
		_elem[_size++] = i; //将原 vector 中的元素依次赋值给新 vector
	}
}
```

#### vector 的动态空间管理 

在对 `vector` 进行插入操作的过程中，容器中元素的长度随时有可能超出这个容器的总容量，因此需要在此时对`vector`进行扩容操作。

<!--more-->

`vector`遵循内存空间连续的规则，但直接在`vector`原内存的末尾开辟空间并不现实（空间可能已被使用），因此应当直接在现有内存中新开辟出一块更大的内存空间并将原`vector`中的所有元素整体搬过去。

如上文代码所见，在 `vector` 的拷贝构造函数中，我们直接***为新的 `vector` 分配原 `vector` 长度的两倍大小来储存数据***，这也是目前 GNUC  `vector`中的做法(我们将 `K=2` 称为`vector`扩容时的增长因子)：

> 在开辟新空间的时候，应当既要保证`vector`不会在短时间内再次溢出，又要保证这些空间中没有太多内存是多余的。
>
> 但是，当增长因子 `K=2` 时，每次扩展的新尺寸必然刚好大于之前分配的总和[例如：第一次扩容内存为`1*2=2`,第二次为`2*2=4`;那么恰好`4>1+(1*2)`，依此类推]
>
> 这将会导致从前`vector`所占用的内存空间在以后`vector`扩容的过程中永远不会被重新使用！
>
> 因此，最佳做法应当是将增长因子 k 设置为 `1 < K <2` 现在许多编译器已将这一因子设置为 1.5 。

```c++
template <typename T> void vector<T>::expand(){
    _capacity *= 2; //将新的容量设置为原先的2倍
	T* old_elem = _elem; //创建一个临时的指针指向原先vector所存储的数据
	_elem = new T[_capacity]; //开辟内存空间
	for (int i = 0; i < _size; ++i) {
		_elem[i] = old_elem[i]; //将原 vector 中的元素依次赋值给新 vector
	}
	delete old_elem; //删除之前所占用的内存
}
```

上述代码实例了`vector`的扩容操作，它应当在`size==capacity`时被调用。

相较`vector`的读取操作，扩容的时间复杂度时相当高昂的(`O(n)`)，但在实际使用中，扩容操作随着`vector`规模的不断增长，调用次数也不断减少，从而可将这一操作的复杂度分摊至所有操作(`O(1)`)。

#### STL 中的 vector

不同编译器乃至同一编译器的不同版本对于`vector`的实现都是各异的，但其内在无外乎以下类结构：

- `vector`
  - `start[iterator]`——指向`vector`容器的开始
  - `finish[iterator]`——指向`vector`容器的结束
  - `end_of_storage[iterator]`——指向`vector`所有容量的结束

如上所见，

当使用`vector.begin()`和`vector.end()`会分别直接返回`start`和`finish`迭代器；

当使用`vector.size()`会直接返回`end()-begin()`;

当使用`vector.empty()`会直接返回`end()==begin()`;

……

##### vector 中的 iterator

因`vector`所使用内存空间连续这一性质，`vector`的迭代器可直接使用 `T*`，原始指针已经可以满足各种对其的运算和移动操作。