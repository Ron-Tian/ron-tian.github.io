---
id: cpp-smart-pointer
title: "C++ 智能指针"
date: 2026-07-18
tags: C++,内存管理
excerpt: "auto_ptr、unique_ptr、shared_ptr、weak_ptr 四种智能指针的原理、用法与最佳实践。"
cover: linear-gradient(135deg, #89f7fe 0%, #66a6ff 100%)
readingTime: 52
type: post
---

**0 什么是智能指针**

智能指针就是智能/自动化的管理指针所指向的动态资源的释放。并且可以像操控指针一样去使用。

**1 要解决的问题**

C/C++程序的显示管理内存在为其在性能上带来一定优势，但是比较容易出错，在传统C++里我们一般使用new和delete或者new[]和delete[]去管理动态开辟的内存,并且必须匹配使用,『记得』手动释放资源，总不是最佳实践。由于人为引起的内存管理不佳从而引起程序的一系列问题时有发生且很不容易排查,比如野指针(指针被释放仍在使用)、指针重复释放(释放已经释放过的指针)、内存泄露(指针没有释放)等。因此C++中提供智能指针去解决此类问题。

**2 基本原理：**

RAII规范(资源获取即初始化)处理资源：C++编程语言的一种管理资源、避免泄漏的惯用法，利用的就是C++构造的对象最终会被销毁的原则。RAII的做法是使用一个对象，在其构造时获取对应的资源，在对象生命期内控制对资源的访问，使之始终保持有效，最后在对象析构的时候，释放构造时获取的资源。(常用的lock_guard对mutex互斥锁的管理、智能指针对内存的管理等都是利用这一规范)。

**3 C++ 中的智能指针**

包含在头文件<memory>中，分别有4个智能指针，分别为auto_ptr、unique_ptr、shared_ptr、weak_ptr。

**3.1 auto_ptr--权限转移法(C++11已废弃)**

为了解决对象中指针的重复释放，早期C++98采用管理权转移的方式。即在解决对象的赋值、拷贝构造时，比如：a = b；将a.ptr的地址空间释放，然后将b.ptr的指针赋给a.ptr，最后将地址空间的管理权交付于a.ptr,并将b.ptr致null。因此，在赋值过后，将不能使用原来指针，缺点就是不能有几个指针指向同一块内存，一个智能指针只能指向一块内存。

**3.1.1 构造函数：构造函数提供一个裸指针构造方式**

explicit auto_ptr(_Ty *_Ptr = 0)
: _Myptr(_Ptr){// construct from object pointer}

**3.1.2 拷贝构造函数：支持拷贝构造，并将原auto_ptr指向清空**

auto_ptr(_Myt& _Right)
: _Myptr(_Right.release())
{
}
   _Ty *release()
{// return wrapped pointer and give up ownership   _Ty *_Tmp = _Myptr;
_Myptr = 0;
return (_Tmp);
}

**3.1.3 析构函数:默认调用delete**

~auto_ptr() _NOEXCEPT
{// destroy the objectdelete _Myptr;
}

**3.1.4 使用场景(不建议使用)**

(1)赋值和拷贝后后，原有auto_ptr不能访问空间了,造成程序结构混乱;

(2)当指针做函数实参传递时，则该智能指针在函数中再不能访问;

(3)C++11引入了右值引用后，move语义有了语法层面的支持，这给auto_ptr最初的设计理念提供了语言层面的支持，最终标准库提供了资源独占型的智能指针unique_ptr。

(4)不支持数组和定制删除器，即不能管理对象数组(析构函数只会调用delete释放)

管理数组会造成内存泄露,如下:

class CTest{
public:
CTest(){
cout << "CTest" << endl;
}
~CTest(){
cout << "~CTest" << endl;
}
};
void TestAuto_ptr(){
auto_ptr<CTest> p(new CTest[5]);
}
输出：
CTestCTestCTestCTestCTest~CTest

**3.2 unique_ptr(推荐使用)**

通过删除拷贝构造函数和赋值运算符的方式禁止智能指针之间的拷贝，同时提供了移动构造函数来转移智能指针内实际指针的所有权，以此实现了资源独占。

**3.2.1 构造函数**

   explicit unique_ptr(pointer _Ptr)
: _Mybase(_Ptr)
{// construct with pointerstatic_assert(!is_pointer<_Dx>::value,
"unique_ptr constructed with null deleter pointer");
}

**3.2.2 拷贝构造函数(禁止拷贝构造)**

unique_ptr(const _Myt&) = delete;

**3.2.3 赋值运算符重载(禁止赋值运算符)**

_Myt& operator=(const _Myt&) = delete;

**3.2.4 移动构造函数和移动赋值运算符重载(std::move转移给其他unique_ptr)**

   unique_ptr(unique_ptr&& _Right) _NOEXCEPT
: _Mybase(_Right.release(),
_STD forward<_Dx>(_Right.get_deleter()))
{// construct by moving _Right}
_Myt& operator=(_Myt&& _Right) _NOEXCEPT
{// assign by moving _Rightif (this != &_Right)
{// different, do the movereset(_Right.release());
this->get_deleter() = _STD forward<_Dx>(_Right.get_deleter());
}
return (*this);
}

**3.2.5 使用场景**

unique_ptr支持自定义deleter,并且支持管理数组对象

**(1)管理指针**

unique_ptr<CTest> p3(new CTest());//普通写法unique_ptr<CTest> p4(make_unique<CTest>());//推荐写法-安全且效率高

**(2)管理数组(支持定制删除器)**

unique_ptr为其支持数组匹配，实现数组版本的模板特化，并在构造unique_ptr单个对象时，默认删除器为default_delete，构造数组对象时，默认删除器为default_delete<_Ty[]>;并且支持自定义删除器。(附3)

写法1：使用数组特化版本按默认管理数组
unique_ptr<CTest[]> p1(new CTest[10]);
等于：unique_ptr<CTest[], decltype(default_delete<CTest[]>())> up2(new CTest[5], default_delete<CTest[]>());
写法2：自定义删除器管理数组,需要传入自定义删除器类型
unique_ptr<CTest, decltype(default_delete<CTest[]>())> up4(new CTest[5], default_delete<CTest[]>());
unique_ptr<CTest, void(*)(CTest*)> up3(new CTest[5], [](CTest *p) { delete[] p; });

**(3)unique_ptr管理权的转移，提供移动构造函数，如下用法**

unique_ptr<CTest> p(make_unique<CTest>(6));
unique_ptr<CTest> p2;
//p2 = p;  //编译出错p2 = std::move(p);//p为空,p转移至p2

**3.3 shared_ptr(推荐使用)**

基本想法是对于动态分配的对象，进行引用计数技术(每当增加一次对同一个对象的引用，那么引用对象的引用计数就会增加一次，每删除一次引用，引用计数就会减一，当一个对象的引用计数减为零时，就自动删除指向的堆内存),它能够记录多少个shared_ptr共同指向一个对象，从而消除显示的调用delete，当引用计数变为零的时候就会将对象自动删除,shared_ptr可以通过get() 方法来获取原始指针，通过reset()来减少一个引用计数， 并通过use_count()来查看一个对象的引用计数。

**3.3.1 构造函数：支持提供裸指针方式构造**

template<class _Ux>
explicit shared_ptr(_Ux *_Px)
{// construct shared_ptr object that owns _Px_Resetp(_Px);
}

**3.3.2 拷贝构造函数**

赋值后新指针引用计数加1，若之前原指针有指向其他shared_ptr,减少其引用计数

shared_ptr(const shared_ptr<_Ty>& _Other) _NOEXCEPT
{// construct shared_ptr object that owns same resource as _Otherthis->_Reset(_Other);
}
template<class _Ty2>
void _Reset(const _Ptr_base<_Ty2>& _Other)
{// release resource and take ownership of _Other._Ptr_Reset(_Other._Ptr, _Other._Rep);
}
void _Reset(_Ty *_Other_ptr, _Ref_count_base *_Other_rep)
{// release resource and take _Other_ptr through _Other_repif (_Other_rep)
_Other_rep->_Incref();//拷贝之后shared_ptr引用计数加1_Reset0(_Other_ptr, _Other_rep);
}
void _Reset0(_Ty *_Other_ptr, _Ref_count_base *_Other_rep)
{// release resource and take new resourceif (_Rep != 0)
{
//若之前原指针有指向其他shared_ptr,减少其引用计数，并将新指针赋值_Rep->_Decref();}
_Rep = _Other_rep;
_Ptr = _Other_ptr;
}

**3.3.6 使用场景**

**(1)管理指针**

shared_ptr<CTest> p10(new CTest());  //普通写法shared_ptr<CTest> p11(make_shared<CTest>());//C++规范和unique_ptr一致，推荐使用

**(2)管理数组:和unique_ptr相比没有特化数组版本，但可以自定义删除器，使用如下**

(a)使用std数组对象删除器（实现见附3）
shared_ptr<CTest> p3(new CTest[5], default_delete<CTest[]>());
(b)自己定义删除器
shared_ptr<CTest> p5(new CTest[5], [](CTest *p) { delete[] p; });
或者：
template< typename T >
struct array_deleter{
void operator ()(T const * p)
{
delete[] p;
}
};
shared_ptr<CTest> p4(new CTest[5], array_deleter<CTest>());

**3.4 weak_ptr(本身不管理内存,只能用share_ptr构造)**

如果你仔细思考std::shared_ptr就会发现依然存在着资源无法释放的问题。看下面这个例子：

struct A;
struct B;
struct A {
std::shared_ptr<B> pointer;
       ~A() {
std::cout << "A 被销毁" << std::endl;
}
};
struct B {
std::shared_ptr<A> pointer;
       ~B() {
std::cout << "B 被销毁" << std::endl;
}
};
int TestPtr() {
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();
       a->pointer = b;  //b的引用计数增加为2       b->pointer = a;  //a的引用计数增加为2return 0;//离开作用域后各自引用计数减1，最后都不会释放}

运行结果是 A, B 都不会被销毁，这是因为 a,b 内部的 pointer 同时又引用了 a,b，这使得 a,b 的引用计数均变为了 2，而离开作用域时，a,b 智能指针被析构，却只能造成这块区域的引用计数减一，这样就导致了 a,b 对象指向的内存区域引用计数不为零，而外部已经没有办法找到这块区域了，也就造成了内存泄露，

解决这个问题的办法就是使用弱引用指针std::weak_ptr，std::weak_ptr是一种弱引用（相比较而言 std::shared_ptr就是一种强引用）。std::weak_ptr 没有*运算符和->运算符，所以不能够对资源进行操作，它的唯一作用就是用于检查std::shared_ptr是否存在，其 expired() 方法能在资源未被释放时，会返回 false，否则返回 true。

弱引用不会引起share_ptr引用计数增加，当换用弱引用时候，weak_ptr不控制对象的生命期，但是它知道对象是否还活着。如果对象还活着，那么它可以提升为有效的shared_ptr；如果对象已经死了，提升会失败，返回一个空的shared_ptr。

**3.4.1 构造和拷贝构造、赋值运算符重载:只能通过waek_ptr或者share_ptr构造,本身不管理内存**

weak_ptr(const weak_ptr& _Other) _NOEXCEPT
{// construct weak_ptr object for resource pointed to by _Otherthis->_Resetw(_Other);
}
weak_ptr(const shared_ptr<_Ty2>& _Other) _NOEXCEPT
{// construct weak_ptr object for resource owned by _Otherthis->_Resetw(_Other);
}

**3.4.1 拷贝构造函数（赋值运算符类似,只会增加弱引用计数，强引用计数不增加）**

weak_ptr(const shared_ptr<_Ty2>& _Other) _NOEXCEPT
{// construct weak_ptr object for resource owned by _Otherthis->_Resetw(_Other);
}
void _Resetw(const _Ptr_base<_Ty2>& _Other)
{// release weak reference to resource and take _Other._Ptr_Resetw(_Other._Ptr, _Other._Rep);
}
void _Resetw(_Ty2 *_Other_ptr, _Ref_count_base *_Other_rep)
{// point to _Other_ptr through _Other_repif (_Other_rep)
_Other_rep->_Incwref();   //引用基类弱引用引用数增加(配合附3代码)if (_Rep != 0)
_Rep->_Decwref();//释放旧的weak_ptr弱引用_Rep = _Other_rep;
_Ptr = _Other_ptr;
}

**3.4.2 使用场景**

(1)当你想使用对象，但是并不管理对象，并且在需要时可以返回对象的shared_ptr时，则使用

(2)解决shared_ptr的循环引用问题

**如开头A和B修改如下即可：**

struct A{
weak_ptr<B> pointerA;
~A() {
cout << "A 被销毁" << endl;
}
};
struct B{
weak_ptr<A> pointerB;
~B() {
cout << "B 被销毁" << endl;
}
};

**4 总结**

C++使用智能指针管理内存，在一定程度上消除了对new/delete的滥用，是一种更加成熟的编程范式,另外如果使用 shared_ptr和unique_ptr使用 new 来调用，这使得代码出现了某种程度上的不对称。make_shared和make_unique就能够用来消除显式的使用new，在日常程序编写中应使用行为更加良好的unique_ptr和shared_ptr/weak_ptr智能指针来进行自动的堆内存释放。

**5 部分源码**

**附1 std默认删除器源码**

template<class _Ty>//单个对象删除器
struct default_delete{// default deleter for unique_ptrtypedef default_delete<_Ty> _Myt;
default_delete() _NOEXCEPT
{// default construct}
template<class _Ty2,class = typename enable_if<is_convertible<_Ty2 *, _Ty *>::value,void>::type>
default_delete(const default_delete<_Ty2>&) _NOEXCEPT
{// construct from another default_delete}
void operator()(_Ty *_Ptr) const _NOEXCEPT
{// delete a pointerstatic_assert(0 < sizeof (_Ty),
"can't delete an incomplete type");
delete _Ptr;
}
};
template<class _Ty>//数组对象删除器
struct default_delete<_Ty[]>
{// default deleter for unique_ptr to array of unknown sizetypedef default_delete<_Ty> _Myt;
default_delete() _NOEXCEPT
{// default construct}
template<class _Other>
void operator()(_Other *) const = delete;
void operator()(_Ty *_Ptr) const _NOEXCEPT
{// delete a pointerstatic_assert(0 < sizeof (_Ty),
"can't delete an incomplete type");
delete[] _Ptr;
}
};

**附2 unique_ptr模板和数组特化模板析构部分代码**

// TEMPLATE CLASS unique_ptr SCALAR//匹配单个对象template<class _Ty,class _Dx>// = default_delete<_Ty>
class unique_ptr: private _Unique_ptr_base<_Ty, _Dx,is_empty<_Dx>::value|| is_same<default_delete<_Ty>, _Dx>::value>{// non-copyable pointer to an objectpublic:
~unique_ptr() _NOEXCEPT
{// destroy the objectif (this->_Myptr != pointer())
this->get_deleter()(this->_Myptr); //获取删除器删除unique_ptr}
};
// TEMPLATE CLASS unique_ptr ARRAY  //匹配数组对象template<class _Ty,class _Dx>class unique_ptr<_Ty[], _Dx> : private _Unique_ptr_base<_Ty, _Dx,is_empty<_Dx>::value|| is_same<default_delete<_Ty[]>, _Dx>::value>{// non-copyable pointer to an array objectpublic:
~unique_ptr() _NOEXCEPT
{// destroy the object_Delete();
}
void _Delete()
{// delete the pointerif (this->_Myptr != pointer())
this->get_deleter()(this->_Myptr);  //获取删除器删除unique_ptr}
};

**附3 share_ptr和weak_ptr引用计数基类**

class _Ref_count_base{// common code for reference countingprivate:
virtual void _Destroy() = 0;
virtual void _Delete_this() = 0;
private:
_Atomic_counter_t _Uses;//强引用shared_ptr引用计数个数_Atomic_counter_t _Weaks;//弱引用waek_ptr引用计数个数protected:
_Ref_count_base()
{// construct_Init_atomic_counter(_Uses, 1);
_Init_atomic_counter(_Weaks, 1);
}

unsigned int _Get_uses() const{// return use countreturn (_Get_atomic_count(_Uses));
}

void _Incref()//增加shared_ptr引用计数{// increment use count_MT_INCR(_Mtx, _Uses);
}

void _Incwref()//增加weak引用计数{// increment weak reference count_MT_INCR(_Mtx, _Weaks);
}

void _Decref()
{// decrement use countif (_MT_DECR(_Mtx, _Uses) == 0)
{// destroy managed resource, decrement weak reference count_Destroy();
_Decwref();
}
}
void _Decwref()
{// decrement weak reference countif (_MT_DECR(_Mtx, _Weaks) == 0)
_Delete_this();
}

long _Use_count() const{// return use countreturn (_Get_uses());
}

bool _Expired() const{// return true if _Uses == 0return (_Get_uses() == 0);
}

virtual void *_Get_deleter(const _XSTD2 type_info&) const    {   // return address of deleter objectreturn (0);
    }
};
