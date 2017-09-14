### 六大组件
---

STL可分为容器(containers)、迭代器(iterators)、空间配置器(allocator)、适配器器(adaptors)、算法(algorithms)、仿函数(functors)六个部分，容器和算法是最核心的部分。

### 容器
---

#### vector

* **vector实现**

vector内部使用动态数组的方式实现，其元素是连续存储的。当没有多余的空间来容纳新元素，就会动态分配新的空间来保存新旧元素，将已有元素从旧位置移动到新空间中，然后添加新的元素，释放旧的空间。它的内部使用allocator类进行内存管理，大部分vector采用的分配策略，是在每次需要重新分配内存空间时，是当前容量capacity的1.5或2倍。

vector由于数组的增长只能向前，所以也只提供了后端插入和后端删除， 
也就是push_back和pop_back。当然在前端和中间要操作数据也是可以的， 
用insert和erase，但是前端和中间对数据进行操作必然会引起数据块的移动， 
这对性能影响是非常大的。**最大的优势是随机访问的速度很快**

以下是vector的数据结构

```cpp
template<class T,class Alloc=alloc>
class vector{
public:
    typedef T  value_type;
    typedef value_type* point;
    typedef value_type* iterator;
    typedef value_type& reference;
    typedef size_t size_type;
...
protected:
    iterator start;//目前使用空间的头部
    iterator finish;//目前使用空间的尾
    iterator end_of_storage;//能够使用空间的尾
...
};
```

注意到vector中的三个迭代器：start，finish，end_of_storage。begin()，end()，size()这些操作其实都是通过维护这三个迭代器得到的。

![vector](http://upload-images.jianshu.io/upload_images/7109298-dfa7427c21755968.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* **容器大小操作**

1. capacity()：容器在不扩张内存的情况下可以容纳多少各元素
2. size()：容器中元素的个数，与capacity是不一样的
3. reserve()：通知容器它应该准备保存多少个元素的空间

* **vector迭代器失效**

1. 添加元素后，如果重新分配了存储空间，指向容器的迭代器、指针、引用都会失效；如果存储空间没有重新分配，指向插入位置之前元素的迭代器、指针、引用仍然有效，指向插入位置之后的都会失效。
2. 删除元素后，指向删除位置之前元素的迭代器、指针、引用仍然有效，之后的无效（删除元素，尾后迭代器总会失效）

* **提升vector性能的使用方法**

1. 使用reserve来提前分配足够的空间以避免不必要的重新分配和复制周期
*如果由一个容量为 0 的 vector 开始，往里面添加元素会花费大量的运行性能，包括分配、回收、拷贝和释放等操作，还会引起迭代器失效。如果预先就知道 vector 需要保存多少元素，就应该提前为其分配足够的空间。*

2. 使用 shrink_to_fit() 释放 vector 占用的内存
*使用 erase() 或 clear() 从 vector 中删除元素并不会释放分配给 vector 的内存，如果在代码中到达某一点，不再需要 vector 时候，应该使用 std::vector::shrink_to_fit() 方法释放掉它占用的内存。*

3. 在填充或者拷贝到 vector 的时候，应该使用赋值而不是 insert() 或push_back()
*从一个 vector 取出元素来填充另一个 vector 的时候，常有三种方法 ： 把旧的 vector 赋值给新的 vector，使用基于迭代器的 std::vector::insert() ，或者使用基于循环的  std::vector::push_back()。三者的性能差距：vector 赋值比 insert() 快了 55.38%，比 push_back() 快了 89%。想高效填充 vector，首先应尝试使用 assignment，然后再考虑基于迭代器的 insert()，最后考虑 push_back。如果需要从其它类型的容器拷贝元素到 vector 中，赋值的方式不可行，最好考虑基于迭代器的 insert() 。*

4. 遍历 std::vector 元素的时候，避免使用 std::vector::at() 函数
*访问vector的元素有三种方法：迭代器、std::vector::at() 成员函数和下标[ ]运算符。其中 std::vector::at() 函数访问 vector 元素是最慢的。*

5. 尽量避免在 vector 前部插入元素
*任何在 vetor 前部部做的插入操作其复杂度都是 O(n) 的，十分低效，因为 vector 中的每个元素项都必须为新的元素腾出空间而被后移。显然，vector 包含元素越多，插入性能会越差。*

#### list

相对于vector的连续线性空间，list就显得复杂许多，它的好处就是插入或删除一个元素，就配置或删除一个元素空间。**list的优势对于任何位置的元素的插入或删除，速度都很快**。list是一个双向环状链表：

![list](http://upload-images.jianshu.io/upload_images/7109298-80c6c66546d6ccdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### deque

* **deque原理**

deque和vector的最大差异，一在于deque允许常数时间内对起头端进行插入或移除操作，二在于deque没有所谓容量(capacity)概念，因为它是以分段连续空间组合而成，随时可以增加一段新的空间连接起来。

deque由一段一段连续空间组成，一旦有必要在deque的前端或尾端增加新空间，便配置一段连续空间，串接在整个deque的前端或尾端。deque的最大任务，便是在这些分段的连续空间上，维护其整体连续的假象，并提供随机存取的接口，避开了“重新配置、复制、释放”的轮回，代价是复杂的迭代器结构。

由于deque是通过链接若干片连续的空间实现的，所以均衡了vector和list的特点。

* **deque迭代器**

迭代器首先必须指出分段连续空间在哪里，其次它必须能够判断自己是否已经处在缓冲区的边缘，如果是，一旦前进或后退就必须跳跃下一个缓冲区，为了能够正常跳跃，deque必须随时掌握管控中心。
deque采用一块map（不是STL的map容器）作为中控器，其为一小块连续空间，其中每个元素都是指针，指向另一段缓冲区，中控器、缓冲区、迭代器的结构如下：

![迭代器](http://upload-images.jianshu.io/upload_images/7109298-66aa2d405bd0a482.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假如deque中已经包含了20个元素了，缓冲区大小为8，则内存布局如下：

![内存布局](http://upload-images.jianshu.io/upload_images/7109298-a74b48eaf75b7f4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### stack

stack是一种先进后出(First In Last Out，FILO)的数据结构，它只有一个出口。stack允许增加元素、移除元素、取得最顶端元素。但除了最顶端外，没有任何其他方法可以存取，stack的其他元素，换言之，stack不允许有遍历行为。stack默认以deque为底层容器。

#### set

set的所有元素都会根据元素的键值自动排序。set的元素不像map那样可以同时拥有实值(value)和键值(key)，set元素的键值就是实值，实值就是键值，set不允许有两个相同的元素。**Set元素不能改变，**在set源码中，set<T>::iterator被定义为底层TB-tree的const_iterator，杜绝写入操作，也就是说，set iterator是一种constant iterators(相对于mutable iterators)。

#### map

map的所有元素都会根据元素的键值自动排序。map的所有元素都是pair，同时拥有实值(value)和键值(key)。pair的第一元素为键值，第二元素为实值。map不允许有两个相同的键值。

**不能通过map的迭代器改变元素的键值，因为map元素的键值关系到map元素的排列规则。任意改变map元素键值都会破坏map组织。如果修改元素的实值，这是可以的，因为map元素的实值不影响map元素的排列规则。**因此，map iterator既不是一种constant iterators，也不是一种mutable iterators

#### 底层数据结构：hashtable

* **实现原理**

C++的hashtable使用链地址法来解决碰撞冲突，buckets使用的是vector数据结构，可以看出**hashtable的实质就是vector+链表**。当插入一个元素时，通过哈希函数计算出散列值，找到该插入哪个buckets的插槽，然后遍历该插槽指向的链表，如果有相同的元素，就返回；否则的话就将该元素插入到该链表的头部。(如果是multi版本的话，是可以插入重复元素的，此时插入过程为：当插入一个元素时，找到该插入哪个buckets的插槽，然后遍历该插槽指向的链表，如果有相同的元素，就将新节点插入到该相同元素的后面；如果没有相同的元素，产生新节点，插入到链表头部)

![hashtable结构](http://upload-images.jianshu.io/upload_images/7109298-e9113cd31994f60d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **rehash**
> *参考Redis的rehash，解决rehash带来的性能下降问题*

hashtable里的元素越来越多，冲突几率越来越高，会进行rehash操作。rehash从宏观上看都是这样的操作：分配一个容量更大的哈希表，从原来的哈希表上将数据移动到新的哈希表上，释放旧的哈希表。

Redis是事件驱动模型，对实时性性的要求很高，采用的是渐进式的rehash方式来解决操作延迟问题。首先会创建一个两倍长度的哈希表，如果正在进行rehash：
1. 当进行SET操作时，它将首先移动一个旧元素到新哈希表，再把K-V放入新的哈希表中；
2. 当进行GET操作时，也会首先移动一个旧元素到新哈希表，再依次从旧数组、新数组中搜索Value。

上述操作中每次只移动一个就元素到新的哈希表中。移动的过程如下：
1. 使用rehashidx下标来表示下一个需要rehash的项在原哈希表中的索引，开始时为0；
2. 每次在[rehashidx, rehashidx+10]这个范围遍历原哈希表，如果找到就重新计算哈希值并放到新哈希表中，没找到就等待下一次机会，处理一个值rehashidx就加一，rehash 全部完成后就置为 -1；
3. 通过上述操作，每次在进行SET/GET操作时，都会保证向前遍历旧哈希表1～10步，最终旧哈希表将被遍历完。

#### hash_map

hash_map以hashtable为底层结构，由于hash_map所提供的操作接口，hashtable都提供了，所以几乎所有的hash_map操作行为都是转调用hashtable的操作行为结果。RB-tree有自动排序功能而hashtable没有，反映出来的结果就是，map的元素有自动排序功能而hash_map没有。

#### STL常见问题
[链接1](http://blog.csdn.net/weiyuefei/article/details/52089724)、[链接2](http://blog.csdn.net/shenya1314/article/details/54923558)、[链接3](http://blog.csdn.net/u010126059/article/details/50708056)
