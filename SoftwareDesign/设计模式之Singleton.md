> *设计模式用来解决反复出现的问题，是解决特点问题的方法。对于设计模式，不要过度思考，不要强制使用设计模式。*

#### 设计模式分类
---

设计模式分为以下以下三大类：

* **创建型模式**
创建型模式专注于如何初始化对象，包括单例模式、简单工厂模式、工厂方法模式、抽象工厂模式等。

* **结构型模式**
结构型模式主要关注对象组合，也就是关注实体之间如何互相使用，包括适配器模式、桥接模式、组合模式、代理模式等。

* **行为型模式**
行为型设计模式关心对象之间的责任分配，不仅仅关注对象间的组合，而且还概述了它们之间的消息传递/通信的模式，主要包括迭代器模式、观察者模式、访问者模式。

#### 单例模式
---

##### 设计思想

需要保证一个类只有一个实例，在类中要构造一个实例，就必须调用类的构造函数，而为了防止在外部调用类的构造函数而构造实例，需要将构造函数的访问权限标记为protected或private；最后，需要提供全局访问点，就需要在类中定义一个static函数，返回在类内部唯一构造的实例。

##### 实现方法

* 静态成员实例的懒汉模式

使用双重锁定检查保证线程安全，智能指针自动释放对象。但是，加锁解锁操作将成为一个性能的瓶颈。

```cpp
#include <mutex>
#include <memory>

class Singleton {
public:
    static Singleton& Instance() {
        if(m_pInstance == nullptr) {
            //lock_guard创建对象时自动加锁，析构对象时自动解锁
            std::lock_guard<std::mutex> g_lock(_mutex); 
            if(m_pInstance == nullptr) {
                m_pInstance = std::shared_ptr<Singleton>(new Singleton());
            }
        }
        return *m_pInstance; //返回对象的引用
    }

private:
    Singleton() {}
    Singleton(const Singleton& rhs) {}
    Singleton& operator = (const Singleton& rhs) {}
   
    static std::mutex _mutex; //互斥量
    static std::shared_ptr<Singleton> m_pInstance; //智能指针
};

std::shared_ptr<Singleton> Singleton::m_pInstance = nullptr;
std::mutex Singleton::_mutex;
```

* 饿汉模式

考虑到加锁解锁带来的开销，有以下改进方法：
 ```cpp
#include <memory>

class Singleton {
public:
    static Singleton& Instance() {
        return *m_pInstance;
    }

private:
    Singleton() {}
    Singleton(const Singleton& rhs) {}
    Singleton& operator = (const Singleton& rhs) {}
   
    static std::shared_ptr<Singleton> m_pInstance;
};

std::shared_ptr<Singleton> Singleton::m_pInstance = std::shared_ptr<Singleton>(new Singleton());
```

静态数据成员初始化在程序开始时，也就是进入主函数之前，由主线程以单线程方式完成了初始化，所以静态初始化实例保证了线程安全性。在性能要求比较高时，就可以使用这种方式，从而避免频繁的加锁和解锁造成的资源浪费。

#### 单例模式隐藏的问题

* **保护**

为了保证一个类只有一个实例，需要将构造函数、拷贝构造函数、赋值运算符声明为private，或者使用delete声明为删除的函数。因为：
1. 在这些成员没有被声明的情况下，编译器可能不会生成这些函数，而是使用这些默认行为：对实例的构造就是分配一部分内存，而不对该部分内存做任何事情；对实例的拷贝或者赋值也是对原来实例所拥有的各信息进行内存按位拷贝。
2. 但是，在有些情况下，编译器就需要生成这些成员函数的默认实现：类成员或者其基类提供了由用户定义的构造函数，类中声明了虚函数，类的各个派生类中拥有一个虚基类。

* **返回引用还是指针**

Singleton返回的实例的生存期是由Singleton本身所决定的，而不是用户代码。
指针和引用在语法上的一个区别就是指针可以为NULL，并可以通过delete运算符删除指针所指的实例，而引用则不可以。由此引申出的语义区别之一就是这些实例的生存期意义：通过引用所返回的实例，生存期由非用户代码管理，而通过指针返回的实例，其可能在某个时间点没有被创建，或是可以被删除的，但是这两条Singleton都不满足。因此在这里返回引用而不是指针。
