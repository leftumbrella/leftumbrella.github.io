---
title: C++11 之 Variadic Templates(数量不定的模板参数)
date: 2019-3-20 18:05:50
tags: [泛型编程,C++11,C++]
categories: 文章
---


>" . . . "

在 C++11 中，引入了新的特性，允许在模板参数中使用数量不确定的参数。

当然，这个函数需要 **至少有一个固定参数。**
##### 例如：
``` 
template <typename T, typename... Types>
void autoPrint(const T& a, const Type&... args){
    cout << a << endl;
    autoPrint(args...);
}
```
如以上程序所示，在函数模板中， Types 代表着暂不确定数量的参数。

因此，当使用者调用起 autoPrint 函数时，如果参数大于一个，在执行完函数内容以后，会递归调用自身，直到 *没有参数* 为止。
##### 例如：
```
autoPrint(1,"hello",2.37,ture);
//以下为输出：
1
hello
2.37
ture
```
<!--more-->
需要注意的是，因为会递归调用，而函数本身没有出口。当递归到 *没有参数* 的时候，无法再次调用自身。

因而在构建这样的函数模板时 **必须重载一个与本函数重载且无参的函数**

所以，上面的写法，在编译过程中无法通过，因为当递归到 *args...* 没有参数的时候，程序无法继续进行。
```
void autoPrint(){

}
```
只有与如上的无参函数 **同时存在** ，程序才会正常执行。

```
sizeof...(args)
```

如果想要得知那些 *不定参数* 具体有多少个的时候，就需要使用 *sizeof...()* 来获得。

---
当然，在函数中不必非要递归调用自己，亦可进行想要的操作。譬如进行组合继承等更加复杂的操作。
