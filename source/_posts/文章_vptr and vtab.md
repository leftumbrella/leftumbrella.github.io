---
title: C++ 面向对象之 virtual pointer 和 virtual tab
date: 2019-3-23 15:30:25
tags: C++
categories: 文章
---

>*在下文中：*<br>
 *vptr == virtual pointer == (虚指针)* <br>
 *vtab == virtual table == (虚表)*
## Part 1:
>多态

当一个类中存在虚函数时， **这个类的大小会比数据本身多一个指针的空间** 。
```
class A{
public:
    virtual void fun1(){}
    virtual void fun2(){}
    void fun3(){}
    void fun4(){}
private:
    int a,b,c;
};
class B : public A{
public:
    virtual void fun1(){}
    void bFun(){}
};
class C{
public:
    void cFun1(){}
    void cFun2(){}
    void cFun3(){}
private:
    int a,b,c;
};
```
<!--more-->
如上图所示，在 ***class A*** 中，存在两个虚函数 。

子类 ***class B*** 重写了虚函数 ***fun1()*** ，同时拥有一个自身的成员函数 ***bFun()*** 。 

这两个个类同时拥有 ***class A*** 中的 —— 三个 ***int*** 类型的成员。

而 ***class C*** 独立存在，并且拥有 **三个非虚成员函数** ，同样拥有三个 ***int*** 类型的成员。
```
int main() {
	std::cout << sizeof(A) << std::endl;
	std::cout << sizeof(B) << std::endl;
	std::cout << sizeof(C) << std::endl;
	return 0;
}
```
众所周知，在 32 位系统中，一个 ***int*** 类型的数据占据四个字节。当我们 ***sizeof()*** 这三个类时，应当得到三个 ***int*** 的大小也就是 ***12*** 个字节数。

但由于虚函数的存在， ***class A*** 与 ***class B*** 应当多出一个指针大小，亦为四字节。
<center>
{% asset_img 1.png 运行效果 %}
</center>
<!-- ![运行效果](./images/writings_vptr_1.png) -->

如图所示， ***class A*** 的子类 ***class B*** 继承了虚函数以及一个指针大小内存。

而在没有任何虚函数的 ***class C*** 中，所占内存只有成员大小。

并且， **无论存在多少个虚函数，所占用的内存始终为一个指针大小** 。
## Part 2:
>在含有虚函数的类中多出来的虚指针指向的位置

**虚指针指向一张记载有虚函数地址的表** ，这张记载有虚函数位置的表称为 ***virtual table*** (虚表)。

***virtual table*** 中的每一个函数指针都指向一个虚函数，也就是说 ***virtual table*** 中的函数指针个数等同于该类中的虚函数的个数。

继承的类和父类分别拥有各自的 ***virtual table*** ，这些 ***virtual table*** 指向他们各自所需要调用的虚函数的位置，因而当分别使用某一个类的对象来调用虚函数的时候，会自动寻找到适当的函数进而调用。
<center>
{% asset_img 2.png 示意图 %}
</center>
## Part 3:
>动态绑定

如上图所示，假设有一根指向 ***class B*** 对象的指针 ***P*** ，当程序需要调用 ***p*** 所指对象的虚函数 ***fun1()*** 时，会执行如下步骤。
1. 通过指针 ***P*** 找到 ***class B*** 的 ***vptr*** 
2. 通过 ***vptr*** 找到 ***vtab***
3. 在 ***vtab*** 中，找到所需虚函数的位置
4. 调用该函数

我们把这种通过 ***vptr*** , ***vtab*** 的绑定方式称为 **动态绑定** 。

而若将这一步骤转换为静态绑定的调用方式则为如下所示：

 ***( * (p->vtab[n]) )(p)***

其中， ***n*** 便是所需函数位于 ***virtual table*** 中的位置。

同样的，当一个 ***class B*** 的对象 ***b*** 调用 ***fun1()*** 时，底层的操作为：

*b.fun1();*

***( * (this->vtab[n] ) (this))***

反观静态绑定，只能通过早先汇编语言中 ***call*** 的方式根据特定的地址来调用各自的函数。
>*本文验证所用编译器为 Visual Studio 2019*<br>
*编译和运行环境为 windows x86*