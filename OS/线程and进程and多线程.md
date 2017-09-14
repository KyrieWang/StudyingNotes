#### 进程和线程
---

* **进程和线程的区别**

总结起来，进程和线程有以下几点区别：

1. 进程是资源分配的最小单位，而线程是cpu调度的最小单位，一个进程由一个或者多个线程组成，线程是一个进程里面代码的不同执行路线，线程不能独立运行，必须依附在进程里面；
2. 进程有独立的地址空间，不同的进程之间相互独立。比如在Linux下面启动一个进程，那么系统会分配给进程独立的地址空间，建立一些数据表来维护他的代码段、堆栈段、数据段。线程没有单独的地址空间，对于运行在一个进程中的线程，他们共享进程的地址空间，但是每个线程拥有他自己的局部变量和堆栈；
3. 线程之间的通信比较方便，同一个进程下的线程可以共享全局变量、静态变量这些数据，但是要考虑到线程安全的问题；进程间的通信需要使用进程通信的方式进行，比如通过共享内存、管道、套接字这些方式。

* **线程的状态**

线程有以下5种状态：

1. 新建状态
创建一个线程，线程还没有开始运行。

2. 就绪状态
处于就绪状态的线程需要和其他线程竞争cpu时间。

3. 运行状态
当线程获得CPU时间后，它才进入运行状态。

4. 阻塞状态
阻塞状态是正在运行的线程没有运行结束，暂时让出CPU，这时其他处于就绪状态的线程就可以获得CPU时间，进入运行状态。线程运行过程中，可能由于各种原因进入阻塞状态，比如说：
* 线程通过调用sleep方法进入睡眠状态
* 线程调用一个I/O阻塞的操作，操作完成之前不会返回到它的调用者
* 线程试图得到一个锁，而该锁正被其他线程持有
* 线程在等待某个触发条件

5. 死亡状态
线程正常退出或者中间发生了错误。

* **进程间通信-IPC**

> *Linux下常用的进程间通信方法：管道、消息队列、共享内存、信号量、套接字等*

1. **管道**

管道是进程间通信中最古老的方式，包括无名管道和有名管道。无名管道用于父进程和子进程间的通信，有名管道可以用于运行在同一台机器上的任意两个进程间的通信。

2. **消息队列**

消息队列用于运行于同一台机器上的进程间通信，它和管道很相似，是一个在系统内核中用来保存消息的队列，它在系统内核中是以消息链表的形式出现的。正在逐渐被淘汰。

3 **共享内存**

 共享内存是运行在同一台机器上的进程间**通信最快**的方式，因为数据不需要在不同进程间复制，通常是由一个进程创建一块共享内存区，其他进程对这块内存区进行读写。

4. **信号量**

信号量是用来协调不同进程间的数据对象的，本质上是一个计数器，用来记录对某个资源的存取状况，比如共享内存。

5. **套接字**

套接字可以用在不同的计算机之间的进程通信，也可以用在本地同一台机器的进程间通信。

* **Linux下的fork**

Linux下的进程机制：

1. Linux下每个进程有唯一的PID标识，PID是从1到32768的正整数，其中1一般是特殊进程init，其他进程从2开始编号；
2. Linux中有一个叫做进程表的结构来存储当前正在运行的进程，可以用ps -aux命令来查看；
3. Linux中的进程是树状结构的，init进程是根节点，其他进程都有父进程（父进程就是启动一个进程的进程）；
4. fork的作用是复制一个和当前进程一样的进程，新进程的所有数据，包括变量、环境变量、程序计数器等都和原来进程的当前值一样，但是是一个全新的进程，并作为原进程的子进程。
5. 创建新进程成功后，系统中出现两个基本完全相同的进程，这两个进程执行没有固定的先后顺序，哪个进程先执行要看系统的进程调度策略。

fork的一个例子：

![fork的一个例子](http://upload-images.jianshu.io/upload_images/7109298-5bbb280c4724c2ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

fork语句将程序分成了两个部分，整个程序运行如下：

1. 执行该程序，生成了进程P，P执行完part.A的代码；
2. 执行到pid = fork()时，P启动一个子进程Q，Q继承了进程P的所有变量、环境变量、程序计数器的当前的值；
3. 在P进程中，fork将Q的PID返回给变量pid，然后继续执行part.B的代码；
4. 在进程Q中，将0赋给pid，并继续执行part.B部分的代码

> *在这个过程中，P执行了所有程序，而Q继承了fork语句执行时当前的环境，而不是程序的初始环境，所以Q只执行了part.B部分，并且Q中的fork语句不启动新的进程，仅仅返回0*

fork函数的一个特性就是**它仅仅被调用一次，却能够返回两次**，可能有三种不同的返回值：
1. 在父进程中，fork返回新创建子进程的进程ID；
2. 在子进程中，fork返回0；
3. 如果出现错误，fork返回一个负值（比如当前进程数已经达到系统规定的上限，或者内存不足）。

#### 线程安全

如果多线程的程序，运行结果是可以预期到的，而且和单线程的程序运行结果相同，那么就是线程安全的。

#### 多线程中的锁

多线程中有以下几种锁：互斥锁、条件锁（条件变量）、自旋锁、读写锁、递归锁。

* **互斥锁Mutex**

互斥锁是用来避免多个线程在某个时间同时操作一个共享资源。在一个时间点上，只有一个线程可以获得互斥量，在释放之前其他线程都不能获得这个互斥量，只能以阻塞的方式等待。

C++11提供了4种互斥量：
1. mutex，最基本的 Mutex 类
2. recursive_mutex，递归 Mutex 类
3. time_mutex，定时 Mutex 类
4. recursive_timed_mutex，定时递归 Mutex 类

mutex可以直接通过他的lock, unlock, trylock等方法来管理锁，C++11还提供两种管理锁对象的类：
1. lock_guard

lock_guard 是一个范围锁，本质是 RAII的思想，在他的对象构造的时候自动对互斥量加锁，对象析构的时候自动解锁，即使程序抛出了异常也能保证正确解锁，不需要手动进行加锁和解锁。

2. unique_lock

unique_lock和lock_guard一样，也能自动加锁和自动解锁；不同的是unique_lock是以独占所有权的方式来管理互斥量的加锁和解锁，而且还提供lock, unlock, trylock这些方法，支持手动的加锁解锁操作，比lock_guard要灵活一些。

* **条件锁**

条件锁就是条件变量。条件变量可以让一个线程等待其他线程的通知，也可以给其他线程发送通知。比如说在一个线程池里，没有任务的时候任务队列是空的，这个时候线程池里的线程因为“任务队列为空”这个条件而处于阻塞的状态。一旦有任务被添加进来，就会以信号量的方式唤醒一个线程来处理任务。

C++11里提供了condition_variable 这个类来实现条件变量。condition_variable 对象需要使用前面提到的unique_lock来获得锁，当它的wait方法被调用的时候，当前线程会一直被阻塞，wait方法会自动释放所，其他被阻塞在锁竞争上的线程得以继续执行。直到其他的线程在相同的condition_variable 对象上调用了唤醒函数来唤醒当前线程，收到唤醒信号后，当前线程会重新对互斥量加锁。

* **自旋锁**

自旋锁也是一种用来保护多线程间共享资源的锁，**当一个线程尝试获取自旋锁的所以权时，线程不会阻塞，而是以忙等待的方式不断的循环检查锁是否可用，直到获得这个自旋锁为止**。

对于互斥锁来说，一个线程试图获取锁而发生阻塞时，线程会被放到等待队列，此时cpu可以去处理其他的任务；而对于自旋锁来说，线程会占用cpu来一直不断的循环请求这个锁，比较销毁处理器资源。

所以如果持有锁的时间比较短，适合使用自旋锁。

#### 死锁

死锁是指两个或两个以上的线程在执行的过程中，因为争夺资源而而造成一种互相等待的现象。比如说，线程1锁住了A，然后尝试对B进行加锁，同时线程2锁住了B，尝试对A进行加锁，这时就发送了死锁。线程1永远得不到B，线程2也永远得不到A，他们会一直阻塞下去。

产生死锁有四个必要条件：
1. 互斥条件：一个资源每次只能被一个线程占用，如果还有其他线程请求资源，只能阻塞等待；
2. 请求和保持条件：如果一个线程因为请求资源而阻塞了，那么它不会释放原来已经占有的资源；
3. 不剥夺条件：进程已经获得的资源，在没有使用完之前不能被其他线程剥夺，只能使用完后由自己释放；
4. 循环等待条件：多个线程之间形成一种头围相接，循环等待资源的关系。

避免死锁：
* 加锁顺序
线程按照一定的顺序加锁和解锁，但是需要事先知道所有可能会用到的锁。
```
Thread 1:
  lock A 
  lock B

Thread 2:
   wait for A
   lock C (when A locked)

Thread 3:
   wait for A
   wait for B
   wait for C
```

* 加锁的时间限制
线程尝试获取锁的时候加上一定的超时时间限制，若一个线程没有在给定的时限内成功获得所有需要的锁，则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试。这段等待时间让其它线程有机会尝试获取相同的这些锁，并且让程序在没有获得锁的时候可以继续运行。

* 死锁检测
不太懂。。。

#### 线程同步和线程互斥

* **线程同步**

线程同步是指线程之间所具有的一种制约关系，一个线程的执行依赖于另一个线程的消息，当它没有得到另一个线程的消息通知时应该等待，直到信息到达时才被唤醒。

线程间的同步方法大体可以分为两类：用户模式和内核模式。内核模式就是利用系统内核对象的单一性来进行同步，使用时需要切换内核态和用户态，内核模式的方法有**事件、信号量、互斥量**；用户模式不需要切换到内核态，只在用户态完成操作，用户模式下的方法有原子操作、临界区。

* **线程互斥**

线程互斥是指对于共享的进程系统资源，在各个线程访问时的排他性，当有若干个线程都要使用某些共享资源时，任何时刻最多只允许一个线程去使用这些资源，其他要使用这个资源的线程必须等待，直到占用资源的线程释放资源。线程互斥可以看成一种特殊的线程同步。

#### 临界资源和临界区

在一段时间内只允许一个线程访问的资源就被称为临界资源或独占资源，他们被要求互斥的访问。

每个进程中访问临界资源的代码就被称为临界区。如果有多个线程试图同时访问临界区，那么在有一个线程进入临界区后，其他所有试图访问临界区的线程都会被挂起，一直持续到金融临界区的线程离开。临界区被释放后，其他线程可以继续抢占。

#### cache对多线程的影响

#### volatile

#### 线程池

线程池最简单的就是生产者消费者模型。池里的每个线程都是消费者，用于消费（处理）一个任务。直接上代码了，细节后面在解释。

```cpp
#include <vector>
#include <queue>
#include <memory>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <future>
#include <atomic>
#include <functional>
#include <stdexcept>

class ThreadPool {
public:
    ThreadPool(size_t size);
    template <class F, class... Args>
    auto commit(F&& f, Args&&... args)->std::future<typename std::result_of<F(Args...)>::type>;
    int threadNum() {return threads_nums;};
    ~ThreadPool();

private:
    std::vector<std::thread> workers; //线程池
    std::queue<std::function<void()>> tasks; //任务队列
    std::mutex queue_mutex; //互斥量
    std::condition_variable condition; //条件变量
    std::atomic<bool> stop; //是否关闭提交，即线程池是否运行
    std::atomic<int> threads_nums;
};
```

先来看构造函数：
```cpp
ThreadPool::ThreadPool(size_t size = 4) : stop(false) {
    threads_nums = size < 1 ? 1 : size;
    //初始化线程数量
    for(size_t i = 0; i < threads_nums; ++i)
        workers.emplace_back(
            [this]
            {
                while(!this->stop)
                {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> u_lock(this->queue_mutex);
                        condition.wait(u_lock, [this]{return this->stop.load() || !tasks.empty();}); //阻塞等待，直到有task或关闭了提交

                        if(this->stop && this->tasks.empty())
                            break;

                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    --threads_nums;
                    task();
                    ++threads_nums;
                }
            }
        );
}
```

构造函数中，emplace_back( [this] {...} ) 构造一个线程对象，每一个线程执行lambda调度函数，调度函数中循环获取task来执行，mutex互斥量保证添加和移除task的互斥性，条件变量保证获取task的同步性。

再来看添加任务的commit函数：
```cpp
template<typename F, typename... Args>
auto ThreadPool::commit(F&& f, Args&&... args)->std::future<typename std::result_of<F(Args...)>::type> {
    using return_type = typename std::result_of<F(Args...)>::type; //result_of用于推导函数的返回值

    //packaged_task包装一个可调用的对象，并且允许异步获取该可调用对象产生的结果，结果保存在future对象
    auto task = std::make_shared<std::packaged_task<return_type()>>(
                std::bind(std::forward<F>(f), std::forward<Args...>(args)...) );
    std::future<return_type> res = task->get_future(); //get_future用来获取与共享状态相关联的future对象
    {
        std::unique_lock<std::mutex> u_lock(queue_mutex);

        if(stop.load())
            throw std::runtime_error("threadpool stopped");

        tasks.emplace([task](){ (*task)(); });
    }

    condition.notify_one();
    return res;
}
```

commit函数使用了C++11的可变参数模板，提交task可以带任意多的参数，第一个参数是可调用对象F，后面是F的参数Args。使用智能指针来管理资源，packaged_task来包装可调用的对象，以便异步获得线程执行的结果，结果保存在future对象中。

最后是析构函数：
```cpp
ThreadPool::~ThreadPool() {
    stop.store(true);
    condition.notify_all();

    for(auto &worker : workers)
        if(worker.joinable())
            worker.join();
}
```

线程池析构的时候，对每一个线程调用join方法，保证等待所有线程都执行完。

下面是一个线程池使用的简单例子：
```cpp
#include <iostream>
#include <chrono>

#include "ThreadPool.h"

using namespace std;

int main()
{
    ThreadPool pool(4);
    vector<future<int>> results;

    for(int i = 0; i < 10; ++i) {
        results.emplace_back(
            pool.commit([i] {
                        cout << "hello" << i << endl;
                        this_thread::sleep_for(chrono::seconds(2));
                        cout << "world" << i << endl;
                        return i * i;
                        })
                             );
    }

    for(auto &result : results)
        cout << result.get() << endl;
    
    return 0;
}
```
