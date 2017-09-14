### 关于I/O多路复用
---

select，poll，epoll都是IO多路复用的机制，I/O多路复用就是一个进程可以监视多个描述符，一旦某个描述符就绪（读就绪或写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，并且这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

### select、poll和epoll
---

#### select

> select的函数原型：int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

select 函数管理的描述符分3类，分别是writefds、readfds和exceptfds。调用select函数后会阻塞，直到有描述副就绪（有数据可读、可写或者有except）或者超时（timeout指定等待时间，如果立即返回设为null即可），select就返回。当select函数返回后，可以通过遍历描述符集合来找到就绪的描述符。

select的一个缺点是单个进程能够监视的描述符的数量存在最大限制，在Linux上一般为1024，可以通过修改宏定义甚至重新编译内核的方式提升这一限制。

### poll

和select相比，poll没有最大描述符的数量限制，但是和select一样，poll返回后，需要轮询来获取就绪的描述符。

select和poll都需要在返回后，**通过遍历文件描述符来获取已经就绪的socket**。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。

#### epoll

相对于select和poll来说，epoll更加灵活，没有描述符数量限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间只需要一次拷贝。

* **epoll的三个接口**

epoll操作需要三个接口：epoll_create、epoll_ctl和epoll_wait。

1. int epoll_create(int size)

> *参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议*

创建一个epoll的句柄。

2. int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)

> *- epfd：epoll_create的返回值。 - op：表示op操作，用三个宏来表示：添加EPOLL_CTL_ADD，删除EPOLL_CTL_DEL，修EPOLL_CTL_MOD，分别添加、删除和修改对fd的监听事件。- fd：需要监听的fd（文件描述符）- epoll_event：告诉内核需要监听什么事件*

对指定描述符fd执行op操作。

3. int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout)

等待epfd上的IO事件，最多返回maxevents个事件。

events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。

* **epoll工作模式**

epoll对文件描述符的操作有两种模式：LT（level trigger，水平触发）和ET（edge trigger，边沿触发），LT模式是默认模式。

1. LT模式

LT(level triggered)是缺省的工作方式，并且同时支持block和no-block socket。

当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件，在下次调用epoll_wait时，会再次响应该事件并通知应用程序。（也就是说，如果不做任何操作，内核还是会继续通知的）

2. ET模式

ET是高速工作方式，只支持no-block socket。

当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应此事件了，直到这个描述符的状态再次发送变化，内核才会重新发出通知。（如果进行了处理，但数据没有读取完，也不会再得到通知）

ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞socket，因为ET模式下的读写需要一直读或写直到出错（对于读，当读到的实际字节数小于请求字节数时就可以停止），如果是阻塞的socket，这个一直读或一直写会在最后一次阻塞，这样就不能在阻塞在epoll_wait上，造成其他文件描述符的任务饿死。

当使用epoll的ET模型来工作时，当产生了一个EPOLLIN事件后，读数据的时候需要考虑的是当recv()返回的大小如果等于请求的大小，那么很有可能缓冲区还有数据未读完，也意味着该次事件还没有处理完，所以还需要再次读取。

#### 总结：Select、Poll与Epoll区别

三者的区别主要体现在以下三点：

* **支持的最大连接数**

select是1024个，poll和epoll理论上没有限制，取决于系统内存大小。

* **IO效率**

select和poll每次都需要轮询所有的描述符，所以IO效率随着要管理的描述符数量增多而下降。

epoll不同于select和poll轮询的方式，而是通过监听回调的机制。epoll事先通过epoll_ctl()来注册一 个描述符，一旦某个文件描述符就绪时，内核会采用类似callback的回调机制来迅速激活这个文件描述符，当进程调用epoll_wait() 时便得到通知。所以只有就绪的描述符才会得到通知。

* **描述符的拷贝**

select和poll每次调用都要把描述符集合从用户态往内核态拷贝，而epoll只需要拷贝一次，将用户态的文件描述符的事件存放到内核的一个事件表中，避免多次复制。
