### malloc和new的区别（free和delete）
---

首先，new可以分为operator new( )、placement new( )和new operator，operator new( )和placement new( )是函数，而new operator是通常我们所说的new操作符。其中operator new( )函数执行和malloc相同的任务，即分配内存，但是不调用构造函数；而 new 操作符则调用operator new( )函数，分配内存后再调用类的构造函数进行对象的构造。placement new( )函数，就是operator new( )函数的一个重载版本，允许在一个已经分配好的内存中构造一个新的对象。
* **new操作符和malloc( )的区别**

1. malloc与free是C++/C语言的标准库函数，new/delete是C++的运算符，**它们都可以用于动态申请内存/释放内存（相同点）**。
2. **操作的对象有所不同。**对于非内部数据类型的对象而言，光用maloc/free 无法满足动态对象的要求。ADT对象在创建的时候要执行构造函数， 消亡之前要执行析构函数。由于malloc/free 是库函数而不是运算符，不在编译器控制权限之内，无法执行构造函数和析构函数的任务。
3. C++需要一个能完成**动态内存分配和对象初始化工作**的运算符new，以一个能完成**对象清理与释放内存工作**的运算符delete。
4. **返回类型的安全性以及是否需要指定内存大小上不同。**new操作符申请内存分配时无须指定内存块的大小，得到的是对象类型的的指针，而malloc返回的都是void*指针，因为malloc 函数本身并不识别要申请的内存是什么类型，它只关心内存的总字节数，需要进行类型转换。
5. new可以认为是malloc加构造函数的执行，delete可以认为析构函数加free函数的执行。new将调用constructor，而malloc不能；delete将调用destructor，而free不能。
6. new内存分配失败时，会抛出bac_alloc异常，它不会返回NULL；malloc分配内存失败时返回NULL。**习惯在使用malloc分配内存后判断分配是否成功，对于new应该使用异常机制。**
7. 使用malloc分配内存后，如果在使用过程中发现内存不足，可以使用realloc函数进行内存重新分配实现内存的扩充。new不能扩充。

参考[链接1](http://www.cnblogs.com/renyuan/archive/2013/05/30/3108656.html)、[链接2](http://blog.jobbole.com/102002/)

> *c++内存布局：C++内存区分为五个区，分别是堆、栈、自由存储区、全局/静态存储区、常量存储区。其中，堆（heap）是C语言和操作系统的术语，是操作系统维护的一块动态内存，而自由存储是C++中基于new与delete运算符的抽象概念，new所申请的内存区域在C++中称为自由存储区。从静态存储区分配的内存在编译时就已经分配好，在程序运行期都存在；在栈上分配的内存，栈内存分配的运算内置与处理器的指令集，效率高，内存分配是连续的，从高地址向低地址扩展；堆内存受限于虚拟内存大小，分配速度比较慢，容易产生内存碎片*

* **关于placement new**

Placement new( )是operate new( )的一个重载版本，作用在已分配的对象上，该对象已有正确的大小和正确的赋值，这包括建立虚函数表和调用构造函数。

placement new就是在用户指定的内存位置上构建新的对象，这个构建过程不需要额外分配内存，只需要调用对象的构造函数即可。placement new有两点好处：
1. 在已经分配好的内存上构建对象，速度很快；
2. 可以重复利用已经分配好的内存，有效的避免内存碎片问题。

* **关于堆和栈的区别**

主要有以下几点区别：申请后系统的响应、申请大小的限制、申请效率的比较、存储内容的不同。

### 关于string类
---

* **string中的一些坑**
1. **结构体中string的赋值问题**
对于以下代码
```cpp
struct flowRecord            
{  
    string app_name;                                                              
    struct flowRecord *next;  
};  
flowRecord *fr = (flowRecord*)malloc(sizeof(flowRecord));  
fr->app_name = "hello";  
cout << fr->app_name << endl;
```

编译运行会出现“Segmentation fault (core dumped)”错误。问题在于动态分配内存是使用的是malloc而不是new。new在分配内存时会调用默认的构造函数，而malloc不会，string在在赋值之前需要调用默认的构造函数来初始化string后才能使用（如赋值、打印等操作）。如果使用malloc分配内存，就不会调用string默认的构造函数来初始化结构体中的app_name字符串，因此这里给其直接赋值是错误的，应该使用new操作符。

> *用C++开发程序时，就尽量使用C++中的函数，不要C++与C混合编程，导致使用混淆而引起错误，比如有时候new分配的内存却用free释放*

2. **c_str( )函数的问题**

c_str函数用于string和const char*之间的转换。string的c_str函数返回的指针是由string管理的，所以该指针的生命周期是string对象的生命周期；string类的实现实际上封装这一个char*类型的指针，c_str( )函数直接返回该指针的引用，因此，**如果string对象被改变，将会直接影响已经执行过的c_str函数返回的指针引用**。

3. **字符串字面值和标准库的string不是同一种类型**

* **实现自己的string类**
正确的实现string类，要实现构造函数、拷贝构造函数、拷贝赋值操作符、析构函数，还有+、==、[ ]等运算符，还包括size( )、c_str( )成员函数。String类声明如下：

```cpp
#ifndef STRING_H
#define STRING_H


class String
{
    public:
        String(const char *str = nullptr);
        String(const String &other);
        String& operator=(const String& other);
        String operator+(const String& str) const;
        bool operator==(const String& str) const;
        char& operator[](int n) ；
        const char& operator[](int n) const;
        ~String();
        
        size_t size() const;
        const char* c_str() const;

    private:
        char* m_pStr;
};

#endif // STRING_H
```

定义如下:

```cpp
#include "String.h"
#include <cstddef>

String::String(const char *str = nullptr)
{
    if(str == nullptr)
    {
        m_pStr = new char[1];
        *m_pStr = '\0';
    }
    else
    {
        size_t length = strlen(str);
        m_pStr = new char[length+1];
        strcpy(m_pStr, str);
    }
}

String::String(const String &other)
{
    size_t length = strlen(other.m_pStr);
    m_pStr = new char[length+1];
    strcpy(m_pStr, other.m_pStr);
}

String& String::operator=(const String& other)
{
    if(this == &other)
        return *this;
    
    delete[] m_pStr;
    size_t length = strlen(other.m_pStr);
    m_pStr = new char[length+1];
    strcpy(m_pStr, other.m_pStr);
    
    return *this;
}

String String::operator+(const String& str) const
{
    String newStr;
    size_t newLen = strlen(m_pStr) + str.size();
    newStr.m_pStr = new char[newLen+1];
    strcpy(newStr.m_pStr, m_pStr);
    strcat(newStr.m_pStr, str.m_pStr);
    return newStr;
}

bool String::operator==(const String& str) const
{
    if(strlen(m_pStr) != str.size())
        return false;
    return strcmp(m_pStr, str.m_pStr) ? false:true;
}

char& String::operator[](int n)
{
    return m_pStr[n];
}

const char& String::operator[](int n) const
{
    return m_pStr[n];
}

size_t String::size() const
{
    return strlen(m_pStr);
}

const String::char* c_str() const
{
    return m_pStr;
}

String::~String()
{
    delete[] m_pStr;
}
```

### static关键字
---
static主要有3个作用：局部静态变量、外部静态变量/函数、类的静态数据成员/成员函数。static 的静态存储方式使得同一函数的所有static变量的“副本”都共用一个该变量。所以使用了static变量的函数一般是“不可再入”的，不是“线程安全”的。

C++是C的超集，所以static在C中的用法 对于C++来说是全盘接受的，而两者的不同与C++面向对象的特性有关，也就是static在“类”中的意义和作用。

1. 函数体内static变量的作用范围为该函数体，不同于自动变量，该变量的内存只被分配一次，因此其值在下次调用时仍维持上次的值；
2. 在模块内的static全局变量可以被模块内所用函数访问，但不能被模块外其它函数访问；
3. 在模块内的static函数只可被这一模块内的其它函数调用，这个函数的使用范围被限制在声明它的模块内；
4. 在类中的static成员变量属于整个类所拥有，对类的所有对象只有一份拷贝，在编译时创建并初始化，在该类的任何对象建立之前就存在；
5. 在类中的static成员函数属于整个类所拥有，这个函数不接收this指针，因而只能访问类的static成员变量。静态成员函数可定义为inline函数，如需访问非静态成员，需要将对象作为参数，通过对象名访问该对象的非静态成员。

### extern关键字
---

1. extern可以置于变量或者函数前，以标示变量或者函数的定义在别的文件中，提示编译器遇到此变量和函数时在其他模块中寻找其定义。比如，B模块(编译单元)要是引用模块(编译单元)A中定义的全局变量或函数时，它只要包含A模块的头文件即可，在编译阶段，模块B虽然找不到该函数或变量， 但它不会报错，它会在链接时从模块A生成的目标代码中找到此变量或函数。
2. 关于extern “C”。C++支持函数重载而C不支持，函数被C++编译器编译后再库中的名字与C语言的不同。为了在C++中调用被C编译后的函数，指示编译器按C的编译规则去翻译函数名称而不是按C++的，这样就可以解决名字匹配的问题。

### const关键字
---

const关键字一般有以下几种作用：
1. 欲阻止一个变量被改变，可以使用const关键字。在定义该const变量时，通常需要对它进行初始化，因为以后就没有机会再去改变它了；
2. 对指针来说，可以指定指针本身为const，也可以指定指针所指的数据为const，或二者同时指定为const；
3. const可以修饰函数的参数，在函数内部不能改变其值；
4. 对于类的成员函数，若指定其为const类型，则表明其是一个常函数，**不能修改对象的non-static成员变量，只能由const类对象调用该函数**；但是，可以在类声明里使用关键字**mutable**修饰数据成员，以指定一个特定的数据成员可以在一个const对象里被改变。*两个函数如果只是常量性不同，可以被重载，*所以可以运用const成员函数实现其non-const版本的孪生兄弟来避免代码重复（使用两次类型转换，第一次使用static_cast为*this添加const属性，第二次使用const_cast移除返回值的const属性），但是const成员调用non-const是错误的行为。
5. 对于类的成员函数，有时候必须指定其返回值为const类型，以使得其返回值不为“左值”。例如：const classA operator*(const classA& a1,const classA& a2);
6. C默认const是外部连接的，C++默认cosnt是内部连接的；
7. 构造函数和析构函数都不是const成员函数，因为它们在初始化和清理时，总是对对象作些修改。
8. 类中定义const数据成员只能在初始化列表里对const成员初始化，不能再构造函数体内赋值。

### volatile关键字
---

volatile关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素更改，比如：操作系统、硬件或者其它线程等。由于访问寄存器的速度要快过RAM，所以编译器一般都会作减少存取外部RAM的优化。**遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。**

对于以下代码：
```cpp
volatile int i = 10; 
int a = i;
... //其他代码，并未明确告诉编译器，对i进行过操作
int b = i;
```

volatile 指出 i是随时可能发生变化的，每次使用它的时候必须从i的地址中读取，因而编译器生成的汇编代码会重新从i的地址读取数据放在b中。而优化做法是，由于编译器发现两次从i读数据的代码之间的代码没有对i进行过操作，它会自动把上次读的数据放在b中，而不是重新从i里面读。这样以来，如果i是一个寄存器变量或者表示一个端口数据就容易出错，所以说volatile可以保证对特殊地址的稳定访问。

### explicit关键字
---

explicit修饰构造函数，将禁止构造函数作为转换函数，即禁止构造函数自动进行隐式类型转换

### 深拷贝（值拷贝）与浅拷贝（位拷贝）

* 深拷贝和浅拷贝可以简单理解为：如果一个类拥有资源，当这个类的对象发生复制过程的时候，如果资源重新分配，这个过程就是深拷贝，反之，没有重新分配资源，就是浅拷贝。
* **默认的拷贝构造函数和缺省的赋值拷贝操作符**均采用位拷贝而非值拷贝的方式来实现，倘若类中含有指针变量成员，这两个函数注定将出错。

### 基于多态的引申--对象切片
---

当使用多态来处理对象时，传地址与传值有明显的不同。地址都有相同的长度，传派生类型（它通常稍大一些）对象的地址和传基类（它通常小一点）对象的地址是相同的，使用多态的目的是让对基类对象操作的代码也能操作派生类对象。 如果使用对象而不是使用指针或引用进行向上映射，会发生向上的类型转换，导致对象被**切片**。

比如，一个函数的参数是父类类型的对象（传值方式），如果传递派生类对象的参数给该函数，编译器将接收该参数，但只会拷贝这个派生类对象对应于其父类的部分，切除掉对象的派生部分。在值传递时，会调用父类的拷贝构造函数，该构造函数会初始化vptr指向父类的虚函数表，并且只拷贝对象的父类部分。

另外，虚函数机制在构造函数中是不工作的，原因是vptr的初始化是分阶段进行的。

### 关于重载
---

可作为函数重载判断依据的有：参数个数、参数类型、**const修饰符**；
不可以作为重载判断依据的有：返回类型。

### 结构体在C和C++中的区别
---

1. C中的结构体内不允许有函数存在，C++允许有内部成员函数，且允许该函数是虚函数。所以C的结构体是没有构造函数、析构函数、和this指针的；
2. C的结构体对内部成员变量的访问权限只能是public，而C++允许public,protected,private三种；
3. C语言的结构体是不可以继承的，C++的结构体是可以从其他的结构体或者类继承过来的

以上都是表面的区别，实际区别就是**面向过程和面向对象编程思路**的区别。
> *C的结构体只是把数据变量给封装起来了，并不涉及算法。而C++是把数据变量及对这些数据变量的相关方法给封装起来，并且给对这些数据和方法不同的访问权限。C语言中是没有类的概念的，但是C语言可以通过结构体内创建函数指针实现面向对象思想。*

### C++中struct和class
---

1. 成员的默认访问权限不同。class的成员默认是private权限，struct默认是public权限；
2. 默认继承权限不同，如果不指定，来自class的继承按照private继承处理，来自struct的继承按照public继承处理。

### C++序列化方法
---

> *将程序数据转化成能被存储并传输的格式的过程称为序列化，其逆过程被称为反序列化。即序列化就是将对象实例的状态转换为可保持或传输的格式，反序列化就是根据流重建对象。比如，可以序列化一个对象然后通过HTTP在客户端和服务器之间传输该对象。*

参考链接：
[Boost Serialization 库](https://www.ibm.com/developerworks/cn/aix/library/au-boostserialization/)
[Boost C++库 - 序列化](http://zh.highscore.de/cpp/boost/serialization.html#serialization_archive)
[Boost - 序列化 (Serialization)](http://blog.csdn.net/zj510/article/details/8105408)
[Boost - Serialization序列化](http://stlchina.huhoo.net/boost/libs/serialization/doc/serialization.html#classoperators)

[](http://blog.csdn.net/lanxuezaipiao/article/details/24845625#)[](http://blog.csdn.net/lanxuezaipiao/article/details/24845625#)[](http://blog.csdn.net/lanxuezaipiao/article/details/24845625#)[](http://blog.csdn.net/lanxuezaipiao/article/details/24845625#)[](http://blog.csdn.net/lanxuezaipiao/article/details/24845625#)[](http://blog.csdn.net/lanxuezaipiao/article/details/24845625#)

### defaul和delete关键字
---

* **default用法**
1. 指示编译器生成该函数的默认实现，如使用=default生成默认构造函数、默认拷贝构造函数、默认赋值操作符、默认析构函数；
2. **只能对具有合成版本的成员函数使用=default**。

* **delete用法**
1. 最大用途是阻止对象拷贝。在函数参数列表后面加上=delete来将函数声明为***删除的函数**，不能以任何方式使用他们。所以，将拷贝构造函数和拷贝赋值运算符定义为删除的函数可以阻止对象被拷贝；
2. **除析构函数以外，任何函数都可以被定义为删除的函数**，因为析构函数被删除，对象就无法销毁。如果一个类或是类的某个成员删除了析构函数，编译器将不允许创建该类的变量或成员，但动态分配该类型的对象是可以的，只是不能释放这些对象；
2. =delete标准出来之前，是通过将拷贝构造函数和拷贝赋值运算符声明为private来阻止拷贝的。

### override和final关键字
---

* **override用法**

1. 派生类使用override显示的注明它使用某个成员函数覆盖了它从父类继承的虚函数。

* **final用法**
1. 在类名后加上关键字final，可以阻止该类被继承，即不能作为基类；
2. 将类中某个函数指定为final，防止后续的派生类覆盖该函数。

### 关于右值引用
---

C++11 引用一个新的引用类型叫右值引用类型，它是绑定到右值的，如临时对象或字面量。
增加右值引用的主要原因是为了实现 move 语义。与传统的拷贝不同，move 的意思是目标对象获取原对象的资源，并将源对象置于“空”状态，所以move后不能使用一个源对象的值，但是可以重新给它赋值或销毁它。当拷贝一个对象时，代价昂贵且无必要，move 操作就可以替代它。如在 string 交换的时候，使用 move 就有巨大的性能提升。

### 关于大端Big-Endian、小端Little-Endian存储
---

* **Big-Endian和Little-Endian的定义**
Little-Endian就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端；而Big-Endian就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。比如，数字0x12 34 56 78在内存中的表示形式：
1. Big-Endian模式：
低地址 -----------------> 高地址
0x12  |  0x34  |  0x56  |  0x78
2. Little-Endian模式
低地址 ------------------> 高地址
0x78  |  0x56  |  0x34  |  0x12

2. 判断机器的字节顺序是Big-Endian还是Little-Endian

指针方法：
```cpp
int x = 1;  
if(*(char *)&x == 1)  
    printf("little-endian\n");  
else  
    printf("big-endian\n");
```

### 关于bool、int、float和指针变量与“零值”的比较
---

bool型变量：if (! val)
int型变量：if (var == 0)
float型变量：const float EPSINON = 0.00001;
                           if (var >= – EPSINON && var <= EPSINON)
指针变量：if (var == nullptr)

> *如果想判断一个变量的“真”、“假”，应直接使用if(var)、if(!var)，表明其为“逻辑”判断；如果用if判断一个数值型变量(short、int、long等)，应该用if(var==0)，表明是与0进行“数值”上的比较；而判断指针则适宜用if(var==NULL)；浮点型变量并不精确，所以不可将float变量用“==”或“!=”与数字比较，应该设法转化成“>=”或“<=”形式。如果写成if (x == 0.0)就错了*

### 关于异常
---

* **构造函数和析构函数抛出异常**

1. 构造函数抛异常

不会发生资源泄漏。假设在operator new()时抛出异常，那么将会因异常而结束此次调用，内存分配失败，不可能存在内存泄露。假设在别处(operator new() )执行之后抛出异常，此时析构函数调用，已构造的对象将得以正确释放，且自动调用operator delete()释放内存 。

2. 析构函数抛异常

可以抛出异常，但该异常必须留在析构函数；若析构函数因异常退出，可能使得已分配的对象未能正常析构，造成内存泄露。

* **栈展开**

抛出异常时，将暂停当前函数的执行，开始查找匹配的catch子句。首先检查throw本身是否在try块内部，如果是就检查与该try相关的catch子句，看是否可以处理该异常。如果不能处理，就退出当前函数，并且释放当前函数的内存并销毁局部对象，继续到上层的调用函数中查找，直到找到一个可以处理该异常的catch。

### C++的转换机制
---

* **static_cast**

static_cast基本上拥有与C语言中旧式转型相同的作用和意义，以及相同的限制。但是，该类型转换操作符不能移除常量性，因为有一个专门的操作符const_cast用来移除常量性。并且，当需要把一个较大的算数类型赋值给较小的类型时，static_cast很有用，或者用来执行编译器无法自动执行的类型转换，比如用static_cast来找回存在于void*中的值：
```cpp
void *p = &var;
double *dp = static_cast<double *>(p);
```

* **const_cast**

 const_cast用来去掉运算对象的**底层const性质**。比如：
```cpp
const  char  pc*;
char  *p = const_cast<char *>(pc)
```

* **dynamic_cast**

dynamic_cast支持运行时类型识别（RTTI）,用来将基类的指针或引用安全地转换成派生类的指针或引用。如果转型失败，当转型对象是指针的时候会返回一个null指针；当转型对象是reference会抛出一个异常exception。dynamic_cast无法应用在缺乏虚函数的类型上，也不能改变类型的常量性。

* **reinterpret_cast**

reinterpret_cast的常用于转换函数指针类型，而且转换结果几乎总是和编译器平台相关，不具有移植性，使用非常危险。
```cpp
typedef void(*FuncPtr)();
int doSomething();
FuncPtr funcPtr = reinterpret_cast<FuncPtr>(&doSomething);
```

### 深拷贝和浅拷贝

* **浅拷贝**

如果在类中没有显式地声明一个拷贝构造函数，那么，编译器将会**根据需要**生成一个默认的拷贝构造函数，完成对象之间的位拷贝，即只对对象中的数据成员进行简单的赋值。default memberwise copy即称为浅拷贝。编译器在以下4种情况下才会生成默认的拷贝构造函数：
1. 类包含的成员变量是object，并且该object的类有拷贝构造函数；
2. 该类继承于一个父类，这个父类有拷贝构造函数；
3. 类声明了虚函数；
4. 类继承于一个父类，这个父类有虚函数或虚基类。

* **深拷贝**

在某些状况下，类内成员变量需要动态开辟堆内存，如果实行位拷贝，也就是把对象里的值完全复制给另一个对象，如A=B。这时，如果B中有一个成员变量指针已经申请了内存，那A中的那个成员变量也指向同一块内存。如果此时B中执行析构函数释放掉指向那一块堆的指针，这时A内的指针就将成为悬挂指针。因此，这种情况下不能简单地复制指针，而应该复制“资源”，也就是再重新开辟一块同样大小的内存空间。

### 指针和引用的区别
---
1.主要有以下区别：
* 指针是一个变量，存储的是一个地址，指向内存中的一个存储单元；而引用是变量的一个别名，实质上和变量是一个东西；
* 指针的值可以为空，并且非const的指针可以被重新赋值来指向不同的对象；而引用的值不能为空，并且引用在定义的时候必须初始化，一旦初始化就和原变量绑定在一起，不能再重新绑定到别的对象；
* 对指针执行sizeof( )操作得到的是指针本身的大小；对引用执行sizeof( )得到的是被引用对象的大小；
* 对指针的自增（++）运算表示对地址的自增，自增的大小取决于所指向的数据的类型；引用的自增运算表示对被引用对象的值的自增；
* 还有就是作为函数参数进行传递时的区别。指针作为函数参数是值传递的方式，函数内部的指针形参是指针实参的一份拷贝，改变指针形参不能改变指针实参的值，但是可以通过解引用（*）运算符来更改指针所指向内存单元里的数据；而引用在作为函数参数传递时，实质上传递的是实参本身，也就是说传递进来的不是实参的拷贝，所以对形参的修改就是对实参的修改。使用引用传递能减少开销。

2. 使用指针还是引用

大多数情况下应该使用引用而不是指针，特别是数据对象是比较大的结构，虽然使用指针或引用都可以，但是类对象在数据结构和语义上来说更适合引用，对象的引用还支持多态性；引用还比指针安全，不存在无效的引用；还有就是重载某个操作符时也应该使用引用。

但是，如果考虑到可能不指向任何对象或者是需要指向不同的对象，就必须使用指针，比如当动态分配内存的时候。还有就是考虑谁拥有内存这个问题，如果是接受变量的代码负责释放相关对象的内存，就必须使用指针，如果接收变量的代码不需要释放内存，就应该使用引用。

### 虚函数的效率问题
---

虚函数影响效率的原因有两点。
第一，调用虚函数的cache命中率不够好，代码总是不停的跳转到无关的位置，虚函数不在cache里的概率很高。
第二，编译器不好做优化。不同于普通的函数，在编译期间就可以确定函数调用的地址，调用虚函数的地址要到运行期间才能确定，编译器无法知道更多的细节，没办法做优化。
