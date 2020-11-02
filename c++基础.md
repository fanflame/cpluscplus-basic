# c++基础

**shared_ptr** **指针的实现**

```c++
template<typename T> class Shared_ptr {
	private:	
  	T *ptr;	
  	int *use_count;	// 用指针保证指向同一块地址public:	
	Shared_ptr() {		
		ptr = nullptr;		
		use_count = nullptr;		
		cout << "Created" << endl;	
	}	
	Shared_ptr(T *p){		
		ptr = p;		
		use_count = new int(1);		
		cout << "Created" << endl;	
	}	
	Shared_ptr(const Shared_ptr<T> &other) {		
		ptr = other.ptr;		
		++(*other.use_count);		
		use_count = other.use_count;		
		cout << "Created" << endl;	
  }	
  Shared_ptr<T>& operator=(const Shared_ptr &other) {		
    if(this == &other)	
      return *this;		
    (*other.use_count)++;		
    if(ptr && --(*use_count) == 0) {			
      delete ptr;			
      delete use_count;		
    }		
    ptr = other.ptr;		
    use_count = other.use_count;		
    return *this;	
  }	
  T& operator*() {	// 返回指针的引用		
    return *ptr;	
  }	
  T* operator->() {		
    return ptr;	
  }	
  int get_use_count() {		
    if(use_count == nullptr)	
      return 0;		
    return *use_count;	
  }	
  ~Shared_ptr() {		
    if(ptr && --(*use_count) == 0) {			
      delete ptr;			
      delete use_count;			
      cout << "Destroy" << endl;		
    }	
  }
};
```

## 构造函数不可以是虚函数

1. 从存储空间角度，虚函数对应一个虚函数表，而指向虚函数表的虚函数指针是存储区对象内存内的。如果构造函数是虚函数，则需要通过虚函数表来调用，而对象还没有构造出来，无法找到虚函数表。
2. 从使用角度，虚函数主要用于信息不全的情况下，使子类重写的函数能得到对应的调用。构造函数本身就是要初始化对象，所以用虚函数没有意义。

1. 

## i++ 和 ++i 的区别

**++i** 返回对象的引用，**i++** 必须产生一个临时对象保存更改前对象的值并返回，所以导致在大对象的时候产生的比较大的复制开销，效率较低。

```c++
// ++i
int& int::operator++() {	
	*this += 1;	
	return *this;
}
// i++
const int int::operator++(int) {	
	int old_value = *this;	
	++(*this);	
	return old_value;
}
```

## 拷贝构造函数

```c++
Node a;
Node b(a);
Node c = a;
```

这里的 b 和 c 都是一开始是不存在的，是通过 a 对象来构造和初始化的。

拷贝构造函数重载形式:

```c++
Node (const Node &other) {	
}
```

如果用户没有自定义拷贝构造函数并且用到了拷贝构造函数，则编译器会生成默认的拷贝构造函数。

在 C++ 中，这三种情况下拷贝构造函数会被使用：

1. 一个对象以值传递的形式传入函数内。
2. 一个对象以值传递的形式从函数返回。
3. 一个对象通过另一个对象初始化。

优点：可以很容易的复制对象。

缺点：对象的隐式拷贝是 C++ 中是错误和性能问题的来源之一。它也降低了代码的可读性，并使得对象子程序中的传递和改变变得难以跟踪。

## 赋值函数

```c++
Node a;
Node b;
b = a;
```

这里的 b 已经存在的，在通过 a 赋值给 b。

赋值函数重载形式：

```c++
Node& operator=(const Node &other) {
}
```

## 拷贝构造函数和赋值函数区别

1. 拷贝构造函数是在对象初始化时，分配一块空间并初始化，而赋值函数是对已经分配空间的对象进行赋值操作。
2. 实现上，拷贝构造函数是构造函数，通过参数的对象初始化产生一个对象。赋值函数则是把一个对象赋值给另一个对象，需要先判断两个对象是否是同一个对象，若是则什么都不做，直接返回，若不是则需要先释放原对象内存，在赋值。(可以参考 shared_ptr 实现)

总结：

- 对象不存在，没有通过别的对象来初始化，就是构造函数。
- 对象不存在，通过别的对象来初始化，就是拷贝构造函数。
- 对象存在，通过别的对象来初始化，就是赋值函数。

## 虚函数和内联函数

内联函数通常就是将它在调用处 **"内敛的"** 展开，消除反复调用的额外开销，但是如果函数太长，会使代码臃肿，反而降低效率。

虚函数可以是内联函数，无论是显式还是隐式，inline 都只是一个申请，最终由编译器决定是否内联。所以可以用 inline 修饰虚函数，但虚函数表现多态性时，不可以内联，只有当知道调用哪个类的函数时，才可以是内联。

## 空类的大小

在 C++ 中规定类的大小不为 0，空类大小为 1，当类不包含虚函数和非静态成员时，其对象大小也为 1。若存在虚函数，则需要存储一个虚函数指针大小，在 32 位上为 4 字节。

## 结构体字节对齐

```c++
class A {};
class B{
	public:	A x;
};
sizeof(B)  = 1
class B{public:	inline virtual fun() {	
}};
sizeof(B)  = 4
class B{public:	A x;	inline virtual fun() {	
}};
sizeof(B)  = 8
```

可以发现最后一个的 sizeof 并不是单纯的 1+4=5，而直接变成了 8，因为存在结构体的字节对齐规则。

结构体字节对齐的根本原因：1) 移植性更好，某些平台只能在特定的地址访问特定的数据。2) 提高存取数据的速度，CPU 通常按块读取数据会更加快速。

结构体字节对齐原则：

1. 无 **#pragma pack**

2. 1. 结构内部各成员首地址必然是自身大小的整数倍。
   2. sizeof 最终结果必然是结构内部最大成员的整数倍，不够补齐

3. 有 **#pragma pack(n)**

4. 1. 结构内部各成员首地址必然是 min(n, 自身大小) 的整数倍。
   2. sizeof 最终结果必然是 min(n, 结构内部最大成员) 的整数倍，不够补齐。

```c++
#include<bits/stdc++.h>
using namespace std;
class A {	
	char a;	
	int b;	
	short c;
};
class B {    
  int a;    
  char b;    
  short c;
};
int main() {   
  cout << sizeof(A) << endl;	// 12    
  cout << sizeof(B) << endl;	// 8    
  return 0;
}
```

造成不同结果的原理在于：

- 对于 class A 来说，内部字节为

| a    | b    | c    |
| :--- | :--- | :--- |
| 1*** | 1111 | 11** |

- 对于 class B 来说，内部字节为

| a    | b    | c    |
| :--- | :--- | :--- |
| 1111 | 1*   | 11   |

## 有 static、virtual 之类的一个类的内存分布

- static 修饰成员变量

- - 静态成员变量在 **全局存储区** 分配内存，本类的所有对象共享，在还没生成类对象之前也可以使用。

- static 修饰成员函数

- - 静态成员函数在 **代码区** 分配内存。静态成员函数和非静态成员函数的区别在于非静态成员函数存在 this 指针，而静态成员函数不存在，所以静态成员函数没有类对象也可以调用。

- virtual

- - 虚函数表存储在常量区，也就是只读数据段
  - 虚函数指针存储在对象内，如果对象是局部变量，则存储在栈区内。

## 使用宏定义求结构体成员偏移量

```c++
#include<bits/stdc++.h>
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE*)0)->MEMBER)
/*(TYPE*)0 将零转型成 TYPE 类型指针((TYPE*)0->MEMBER) 访问结构体中的成员&((TYPE*)0->MEMBER) 取出数据成员地址，也就是相对于零的偏移量(size_t) & ((TYPE*)0)->MEMBER) 将结果转换成 size_t 类型。*/
struct Node {	
  char a;	
  short b;	
  double c;	
  int d;
};
int main() {	
  printf("%d\n", offsetof(Node, a));	
  printf("%d\n", offsetof(Node, b));	
  printf("%d\n", offsetof(Node, c));	
  printf("%d\n", offsetof(Node, d));	
  return 0;
}
```

size_t 在可以理解成 unsigned\ \ int，在不同平台下被 typedef 成不同类型。

## C++ 中哪些函数不可以是虚函数

1. 普通函数(非成员函数)：普通函数不属于类成员，不能被继承。普通函数只能被重载，不能被重写，因此声明成虚函数没有意义。
2. 构造函数：构造函数本来就是为了初始化对象而存在的，没有定义为虚函数的必要。而且对象还没构造出来，不存在虚函数指针，也无法成为虚函数。
3. 内联成员函数：内联函数是为了在代码中直接展开，减少调用函数的代价，是在编译的时候展开。而虚函数需要动态绑定，这不可能统一。只有当编译器知道调用的是哪个类的函数时，才可以展开。
4. 静态成员函数：对于所有对象都共享一个函数，没有动态绑定的必要。
5. 友元函数：C++ 不支持友元函数的继承，自然不能是虚函数。

## main 函数前后还会执行什么

全局对象的构造在 main 函数之前完成，全局对象的析构在 main 函数之后完成。

## const 和 define

1. define 在预编译阶段起作用，const 在编译、运行的时候起作用。
2. define 只是简单的替换功能，没有类型检查功能，const 有类型检查功能，可以避免一些错误。
3. define 在预编译阶段就替换掉了，无法调试，const 可以通过集成化工具调试。

## inline 和 define

1. inline 在编译时展开，define 在预编译时展开。
2. inline 可以进行类型安全检查，define 只是简单的替换。
3. inline 是函数，define 不是函数。
4. define 最好用括号括起来，不然会产生二义性，inline 不会。
5. inline 是一个建议，可以不展开，define 一定要展开。

## inline 函数的要求

1. 含有递归调用的函数不能设置为 inline
2. 循环语句和 switch 语句，无法设置为 inline
3. inline 函数内的代码应很短小。最好不超过５行

## 声明和定义

- 变量定义：为变量分配空间，还可以为变量指定初始值。
- 变量声明：向程序表明变量的类型和名字，但不分配空间。可以通过 extern 关键字来声明而不定义，extern 告诉编译器变量在别的地方定义了。

1. 定义也是声明，声明不是定义。例如：

2. - **extern int i** 声明且不定义 **i** 变量，**int i** 声明且定义了 **i** 变量。

3. 声明有初始值时，为当成定义

4. - **extern int i=10** 此时看成定义了 **i** 变量。

5. 函数的声明和定义，区别在于是否带有花括号。

## 面向过程和面向对象

面向过程就是分析出解决问题所需要的步骤，然后用函数把步骤一步一步的实现。优点在于性能高。

面向对象就是构成问题事务分解成各个对象，建立对象的目的不是为了完成一个步骤，而是为了描述某个对象在整个解决问题的步骤中的行为。优点在于易维护、易复用、易扩展，使系统更加灵活。

## 堆区和自由存储区

基本上所有的 C++ 编译器默认使用堆来实现自由存储，也就是缺省的 new/delete 是通过 malloc/free 方式来实现的，这时候可以说他从堆上分配内存，也可以说他从自由存储区分配内存。但程序员可以通过重载操作符，改用其他内存实现自由存储。

堆区是操作系统维护的一块内存，而自由存储区是 C++ 中用于 new/delete 动态分配和释放对象的 **抽象概念**。

## 堆区和栈区

|          | 堆区                              | 栈区                   |
| :------- | :-------------------------------- | :--------------------- |
| 管理方式 | 由程序员控制                      | 系统主动分配           |
| 空间大小 | 空间很大，可以达到4G              | 默认1M                 |
| 碎片问题 | 频繁的 malloc/free 会造成大量碎片 | 不会存在这个问题       |
| 生长方向 | 向着内存地址增加的方向            | 向着内存地址减小的方向 |
| 分配效率 | 速度比较慢                        | 速度比较快             |

## #ifndef 和 #endif 的作用

在大的软件工程中，可能多个文件包含同一个头文件，当这些文件链接成可执行文件的时候，就会造成大量的 "重定义" 错误，可以通过 **#ifndef** 来避免这个错误。

```c++
#ifndef _A_H
#define _A_H
#endif
```

这样就可以保证 a.h 被定义一次。

## explicit 关键字

类的构造函数存在隐式转换，如果想要避免这个功能，就可以通过 explicit 关键字来将构造函数声明成显式的。

## 菱形继承

```c++
class A {	
  public:		
  void fun() {			
    cout << "!!!" << endl;		
  }
};
class B : public A{};class C : public A{
  
};
class D : public B, public C {
  
};

int main() {	
  // freopen("in", "r", stdin);	
  D d;	
  d.fun();
  return 0;
}
```

像上面的继承关系，当继承关系形成菱形时，D 中保存了两份 A，在调用 d.fun() 时就会出现调用不明确的问题。

这种情况由两种解决方法：

1. 使用域限定访问的函数。也就是将 d.fun 修改成 d.B::fun 或者 d.C::fun。
2. 第二种方法就是使用虚继承。虚继承解决了从不同途径继承来的同名数据成员在内存中有不同的拷贝造成数据不一致的问题，将共同基类设置成虚基类，这时从不同路径继承过来的同名数据成员在内存中就只有一个拷贝。操作方法就是在 B 和 C 的继承处加上 virtual 修饰。

虚继承底层实现一般通过虚基类指针和虚基类表实现。每个虚继承的子类都有一个虚基类指针和一个虚基类表，虚表中记录了虚基类与本类的偏移地址。通过偏移地址，这样就找到了虚基类成员，节省了存储空间。

## weak_ptr 指向的对象被销毁

首先 weak_ptr 是一种不控制对象生存周期的智能指针，它指向一个 shared_ptr 管理的对象。一旦最后一个指向对象的 shared_ptr 被销毁，那么无论是否有 weak_ptr 指向该对象，都会释放资源。

weak_ptr 不能直接指向对象，需要先调用 lock，而 lock 会先判断该对象是否还存在。

如果存在 lock 就会返回一个指向该对象的 shared_ptr，并且该对象的 shared_ptr 的引用计数加一。

如果在 weak_ptr 获得过程中，原本的所有 shared_ptr 被销毁，那么该对象的生命周期会延长到这个临时 shared_ptr 销毁为止。

## lambda 表达式

lambda 表达式定义一个匿名函数，并且可以捕获一定范围内的变量，基本格式如下：

```
[capture](params) mutable ->ReturnType {statement}
```

- [capture]：捕获列表，可以捕获上下文的变量来供 lambda 函数使用

- - [var]：值传递的方式捕获 var
  - [&var]：引用传递的方式捕获 var
  - [=]：值传递的方式捕获父作用域的所有变量。
  - [&]：引用传递的方式捕获父作用域的所有变量。
  - [this]：值传递的方式捕获当前 this 指针。

- (params)：参数列表，和普通函数参数列表一致，如果不传参数可以和 () 一起忽略。

- mutable 修饰符号，默认情况下，lambda 表达式是一个 const 函数，可以用 mutable 取消他的常量性。若使用 mutable 修饰，参数列表不可省略。

- **->**ReturnType：返回值类型。若是 void，可以省略。

- {statement}：函数主体，和普通函数一样。

lambda 表达式优点在于代码简洁，容易进行并行计算。缺点在于在非并行计算中，效率未必有 for 循环快，并且不容易调试，对于没学过的程序员来说可读性差。

```c++
#include<bits/stdc++.h>
using namespace std;
int main() {    
  vector<int> g{9, 2, 1, 2, 5, 6, 2};    
  int ans = 1;    
  sort(g.begin(), g.end(), [](int a, int b) ->bool{
    return a>b;
  });    
  for_each(g.begin(), g.end(), [&ans](int x){
    cout << x << " ", ans = ans*x;
  });    
  cout << "\nmul = " << ans << endl;    
  return 0;
}
/*9 6 5 2 2 2 1mul = 2160*/
```

## 存在全局变量和局部变量时，访问全局变量

可以通过全局作用符号 :: 来完成

```c++
int a = 10;
int main() {	
// freopen("in", "r", stdin);	
	int a = 20;	
  cout << a << endl;	
  cout << ::a << endl;	
  return 0;
}
```

## 全局变量的初始化的顺序

同一文件中的全局变量按照声明顺序，不同文件之间的全局变量初始化顺序不确定。

如果要保证初始化次序的话，需要通过在函数使用静态局部变量并返回来实现。

```c++
class FileSystem{...};
FileSystem & tfs(){	
  static FileSystem fs;//定义并初始化一个static对象	
  return fs;
}
```

## 浅拷贝和深拷贝

- 浅拷贝：源对象和拷贝对象共用一份实体，仅仅是引用的变量名不同。对其中任意一个修改，都会影响另一个对象。
- 深拷贝：源对象和拷贝对象相互独立。对其中一个对象修改，不会影响另一个对象。
- 两个对象指向同块内存，当析构函数释放内存时，会引起错误。

从这个例子可以看出，b 通过默认拷贝函数进行初始化，然而进行的是浅拷贝，导致对 a 进行修改的时候，b 的存储值也被修改。

```c++
struct Node {	
  char* s;	
  Node(char *str) {		
    s = new char[100];		
    strcpy(s, str);	
  }	
  /*	手动实现拷贝构造函数	
  Node(const Node &other) {		
  	s = new char[100];		
  	strcpy(s, other.s);	
  }	*/
};
int main() {	
  // freopen("in", "r", stdin);	
  Node a("hello");	
  Node b(a);	
  cout << b.s << endl;	
  strcpy(a.s, "bad copy");	
  cout << b.s << endl;	
  return 0;
}
```

正确的写法应该自己写一个拷贝函数，而不是用默认的，应该尽量的避免浅拷贝。

## memcpy 和 memmove 的实现

memcpy 可以直接通过指针自增赋值，但要求源地址和目的地址无重合。

```c++
void mymemmove1(void* s, const void* t, size_t n) {	
  char *ps = static_cast<char*>(s);	
  const char *pt = static_cast<const char*>(t);	
  while(n--) {		
    *ps++ = *pt++;	
  }
}
```

如果源地址和目的地址存在重合，会因为地址的重合导致数据被覆盖，所以要通过 memmove 来实现，需要从末尾往前自减赋值。

为了加快速度还可以使用 4 字节赋值的方式

```c++
// 直接按字节进行 copy
void mymemmove1(void* s, const void* t, size_t n) {	
  char *ps = static_cast<char*>(s);	
  const char *pt = static_cast<const char*>(t);	
  if(ps<=pt && pt<=ps+n-1) {		
    ps = ps+n-1;		
    pt = pt+n-1;		
    while(n--) {			
      *ps-- = *pt--;		
    }	
  } else {		
    while(n--) {			
      *ps++ = *pt++;		
    }	
  }
}
// 加快速度，每次按 4 字节进行 copy
void mymemmove2(void *s, const void *t, size_t n) {	
  int *ts = static_cast<int*>(s);	
  const int *tt = static_cast<const int*>(t);	
  char *ps = static_cast<char*>(s);	
  const char *pt = static_cast<const char*>(t);	
  int x = n/4, y = n%4;	
  if(ps<=pt && pt<=ps+n-1) {		
    ps = ps+n-1;		
    pt = pt+n-1;		
    while(y--) {			
      *ps-- = *pt--;		
    }		
    ps++, pt++;		
    ts = reinterpret_cast<int*>(ps);		
    tt = reinterpret_cast<const int*>(pt);		
    ts--, tt--;		
    while(x--) {			
      *ts-- = *tt--;		
    }	
  } else {		
    while(y--) {			
      *ps++ = *pt++;		
    }		
    ts = reinterpret_cast<int*>(ps);		
    tt = reinterpret_cast<const int*>(pt);		
    while(x--) {			
      *ts++ = *tt++;		
    }	
  }
}
```

## 通过重载 new/delete 来检测内存泄漏的简易实现

讲每次 new 产生的内存记录，并在 delete 的时候删去记录，那么最后剩下的就是发生内存泄漏的代码。

```c++
#include <bits/stdc++.h>
using namespace std;
class TraceNew {
  public:	class TraceInfo {	
    private:		
    const char* file;		
    size_t line;	public:		
    TraceInfo();		
    TraceInfo(const char *File, size_t Line);		
    ~TraceInfo();		
    const char* File() const;		
    size_t Line();	
  };    
  TraceNew();    
  ~TraceNew();    
  void Add(void*, const char*, size_t);    
  void Remove(void*);	
  void Dump();
  private:    
  map<void*, TraceInfo> mp;
} trace;
TraceNew::TraceInfo::TraceInfo() {}
TraceNew::TraceInfo::TraceInfo(const char *File, size_t Line) : file(File), line(Line) {}
TraceNew::TraceInfo::~TraceInfo() {	
  delete file;
}
const char* TraceNew::TraceInfo::File() const {	
  return file;
}
size_t TraceNew::TraceInfo::Line() {	
  return line;
}
TraceNew::TraceNew() {    
  mp.clear();
}
TraceNew::~TraceNew() {	
  Dump();	
  mp.clear();}
void TraceNew::Add(void *p, const char *file, size_t line) {	
  mp[p] = TraceInfo(file, line);
}
void TraceNew::Remove(void *p) {	
  auto it = mp.find(p);	
  if(it != mp.end())	
    mp.erase(it);
}
void TraceNew::Dump() {	
  for(auto it : mp) {		
    cout << it.first << " " << "memory leak on file: " << it.second.File() << " line: " << it.second.Line() << endl;	
  }
}
void* operator new(size_t size, const char *file, size_t line) {	
  void* p = malloc(size);	
  trace.Add(p, file, line);	
  return p;
}
void* operator new[](size_t size, const char *file, size_t line) {	
  return operator new(size, file, line);
}
void operator delete(void *p) {	
  trace.Remove(p);	
  free(p);
}
void operator delete[](void *p) {	
  operator delete(p);	
}
#define new new(__FILE__,__LINE__)
int main() {	
  int *p = new int;	i
    nt *q = new int[10]; 	
  return 0;
}
/*0xa71850 memory leak on file: a.cpp line: 900xa719b8 memory leak on file: a.cpp line: 91*/
```

## 垃圾回收机制

之前使用过，但现在不再使用或者没有任何指针再指向的内存空间就称为 "垃圾"。而将这些 "垃圾" 收集起来以便再次利用的机制，就被称为“垃圾回收”。

垃圾回收机制可以分为两大类：

1. 基于引用计数的垃圾回收器

2. - 系统记录对象被引用的次数。当对象被引用的次数变为 0 时，该对象即可被视作 "垃圾" 而回收。但难以处理循环引用的情况。

3. 基于跟踪处理的垃圾回收器

4. - **标记-清除**：对所有存活对象进行一次全局遍历来确定哪些对象可以回收。从根出发遍历一遍找到所有可达对象(活对象)，其它不可达的对象就是垃圾对象，可被回收。
   - **标记-缩并**：直接清除对象会造成大量的内存碎片，所以调整所有活的对象缩并到一起，所有垃圾缩并到一起，然后一次清除。
   - **标记-拷贝**：堆空间分为两个部分 From 和 To。刚开始系统只从 From 的堆空间里面分配内存，当 From 分配满的时候系统就开始垃圾回收：从 From 堆空间找出所有的活对象，拷贝到 To 的堆空间里。这样一来，From 的堆空间里面就全剩下垃圾了。而对象被拷贝到 To 里之后，在 To 里是紧凑排列的。接下来是需要将 From 和 To 交换一下角色，接着从新的 From 里面开始分配。

## 通过位运算实现加减乘除取模

- 加法操作

对于每一位而言，在不考虑进位的情况下，可以得到

```
0+0 = 0\\
0+1 = 1\\
1+0 = 1\\
1+1 = 0
```

显然，上面的情况符合 **异或** 操作且只有第四种情况发生了进位，进位情况符合 **与** 操作。在所有发生进位处，应该在更高的一位处加一，这个值可以通过 **左移** 操作实现。那么就可以得到

```
x+y = x \oplus y + (x \& y)<<1
```

可以发现，后面的式子变成了一个新的加法式，那么只要递归计算即可。当 (x & y)<<1 == 0 时，就消除了加法式。

- 
- 
- 
- 
- 
- 

```
ll add(ll x, ll y) {	ll newx = x^y;	ll newy = (x&y)<<1;	if(newy == 0)	return newx;	return add(newx, newy);}
```

- 减法操作

减法操作可以看成







同样可以通过加法式得到

- 
- 
- 

```
ll sub(ll x, ll y) {	return add(x, add(~y, 1));}
```

- 乘法操作

假设 y=1010，则可以关注于二进制上的 1 位，那么可以将 x*y 做出拆解

\begin{aligned} x*y &= x*1010\ &= x*1000 + x*0010
\end{aligned}

而这个当乘数只有一个 1 时，可以通过二进制的左移操作实现。

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
ll mul(ll x, ll y) {	ll ans = 0;	int flag = x^y;	x = x<0 ? add(~x, 1) : x;	y = y<0 ? add(~y, 1) : y;	for(int i=0; (1ll<<i)<=y; i++) {		if(y&(1ll<<i)) {			ans = add(ans, x<<i);		}	}	return flag<0 ? add(~ans, 1) : ans;}
```

- 除法操作

和乘法操作思想一样，枚举答案每一位是否为 1，通过左移来得到乘积并减去。先从大的开始找，如果有一位是 1，那么就在答案将这一位设成 1。

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
ll divide(ll x, ll y) {	ll ans = 0;	int flag = x^y;	x = x<0 ? add(~x, 1) : x;	y = y<0 ? add(~y, 1) : y;	for(int i=31; i>=0; i--) {		if(x >= (1ll*y<<i)) {			ans |= (1ll<<i);			x = sub(x, 1ll*y<<i);		}	}	return flag<0 ? add(~ans, 1) : ans;}
```

- 取模操作

已经得到了除法的结果，那么取模操作也很容易实现了

- 
- 
- 
- 
- 

```
ll mod(ll x, ll y) {	x = x<0 ? add(~x, 1) : x;	y = y<0 ? add(~y, 1) : y;	return sub(x, mul(y, divide(x, y)));}
```

## 为什么子类对象可以赋值给父类对象而反过来却不行

- 子类继承于父类，它含有父类的部分，又做了扩充。如果子类对象赋值给父类变量，则使用该变量只能访问子类的父类部分。
- 如果反过来，这个子类变量如果去访问它的扩充成员变量，就会访问不到，造成内存越界。

## 为什么 free 时不需要传指针大小

free 要做的事是归还 malloc 申请的内存空间，而在 malloc 的时候已经记录了申请空间的大小，所以不需要传大小，直接传指针就可以。

## 手写链表实现 LRU

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
class LRU {private:	struct Node {		int val;		Node *pre, *suf;		Node() {			pre = suf = nullptr;		}		Node(int _val) {			val = _val;			pre = suf = nullptr;		}	};	int size;	int capacity;	Node *head;	unordered_map<int, Node*> mp;	Node* find(int val) {		if(mp.count(val)) {			return mp[val];		} else {			return nullptr;		}	}	void del(Node *node) {		if(node == nullptr)	return ;		node->pre->suf = node->suf;		node->suf->pre = node->pre;		mp.erase(node->val);		if(node == head)	head = head->suf;		size--;	}	void add(Node *node) {		if(head == nullptr) {			head = node;			head->suf = head;			head->pre = head;			mp[node->val] = node;		} else {			node->suf = head;			node->pre = head->pre;			head->pre = node;			node->pre->suf = node;			mp[node->val] = node;			head = node;		}		size++;	}public:	LRU() {		mp.clear();		head = nullptr;		size = capacity = 0;	}	void reverse(int _capacity) {		capacity = _capacity;	}	void insert(int val) {		Node *node = new Node(val);		Node *selectnode = find(val);		del(selectnode);		if(size == capacity) {			del(head->pre);		}		add(node);	}	void view() {		Node *p = head;		do {			cout << p->val << " ";			p = p->suf;		} while(p != head);		cout << endl;	}}lru;
```