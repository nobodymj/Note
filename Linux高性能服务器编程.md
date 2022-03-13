# Linux高性能服务器编程
## 主机字节序和网络字节序
大端字节序：一个整数的高位字节（23～31bit）存储在内存的低地址处，低位字节存储在内存的高地址处。大端字节序也称为网络字节序。
小端字节序：整数的高位字节存储在内存的高地址处，低位字节存储在内存的低地址处。现代PC大多采用小端字节序，因此又被称为主机字节序。
```
void byteOrder()
{
    union { 
        short value;
        char union_bytes[ sizeof(short) ];
    } test;

    test.value = 0x0102;
    if ( (test.union_bytes[0] == 1) && (test.union_bytes[1] == 2) ) {
        printf("big endian \n");
    } else if ( (test.union_bytes[0] == 2) && (test.union_bytes[1] == 1) ) {
        printf("litter endian \n");
    } else {
        printf("unknow... \n");
    }
}
```


## 高级I/O函数
### pipe函数
作用：用于创建一个管道，以实现进程间通信。
int pipe(int fd[2]);
fd[2]：分别构成管道的两端。fd[0]只能用于从管道读出数据，fd[1]只能用于往管道写入数据。
如果要实现双向的数据传输，就应该使用两个管道。
管道内部传输的数据是字节流，管道容量的大小默认是65535字节。可用fcntl函数修改管道容量。
### socketpair函数
作用：创建双向管道。
int socketpair(int domain, int type, int protocol, int fd[2]);//domain:AF_UNIX

### dup函数和dup2函数
作用：重定向输入输出。创建一个新的文件描述符，和原有文件描述符file_descriptor指向相同的文件/管道/网络连接。
dup返回的文件描述符总是取系统当前可用的最小整数值。
dup2返回第一个不小于file_descriptor_two的整数值。
dup和dup2创建的文件描述符并不继承原文件描述符的属性。
int dup(int file_descriptor);
int dup2(int file_descriptor_one, int file_descriptot_two);

### readv函数和writev函数
作用：readv函数将数据从文件描述符读到分散的内存块中，即分散读。
writev函数将多块分散的内存数据一并写入文件描述符中，即集中写。
ssize_t readv(int fd, const struct iovec* vector, int count);
ssize_t writev(int fd, const struct iovec* vector, int count);

### sendfile函数
作用：在两个文件描述符之间直接传递数据（完全在内核中操作），从而避免了内核缓冲区和用户缓冲区之间的数据拷贝，效率高，这被称为零拷贝。
ssize_t sendfile(int out_fd, int in_fd, off_t* offset, size_t count);

### mmap函数和munmap函数
mmap函数：用于申请一段内存空间，可以将这段内存作为进程间通信的共享内存，也可以将文件直接映射到其中。
munmap函数：释放由mmap创建的这段内存空间。
void* mmap(void* start, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void* start, size_t length);
prot：设置内存段的访问权限：PROT_READ/PROT_WRITE/PROT_EXEC/PROT_NONE
flags：控制内存段内容被修改后程序的行为：MAP_SHARED/MAP_PRIVATE/MAP_ANONYMOUS/MAP_FIXED/MAP_HUGETLB

### splice函数
作用：用于在两个文件描述符之间移动数据，也是零拷贝操作。
ssize_t splice(int fd_in, loff_t* off_in, int fd_out, loff_t* off_out, size_t let, unsigned int flags);
flags：控制数据如何移动：SPLICE_F_MOVE/SPLICE_F_NONBLOCK/SPLICE_F_MORE/SPLICE_F_GIFT
使用splice函数时，fd_in和fd_out必须至少有一个是管道文件描述符。

### tee函数
作用：用于在两个管道文件描述符之间复制数据，也是零拷贝操作。
它不消耗数据，因此源文件描述符上的数据仍然可以用于后续的读操作。
ssize_t tee(int fd_in, int fd_out, size_t let, unsigned int flags);

### fcntl函数
作用：提供了对文件描述符的各种控制操作。
int fcntl(int fd, int cmd, …);
fcntl支持的常用操作：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/66685527-E3BE-4B0C-ACDA-223523F2EDFA.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/%E6%88%AA%E5%B1%8F2021-09-13%20%E4%B8%8B%E5%8D%889.39.51.png)

## Linux系统日志
Linux系统日志：rsyslogd。既能接收用户进程输出的日志，又能接收内核日志。用户进程是通过调用syslog函数生成系统日志的。该函数将日志输出到一个AF_UNIX的socket类型的文件/dev/log中，rsyslogd则监听该文件以获取用户进程的输出。内核日志由printk等函数打印到内核的环形缓存中，环形缓存的内容直接映射到/proc/kmsg文件中。rsyslogd则通过读取该文件获得内核日志。
rsyslogd守护进程在接受到用户进程或内核输入的日志后，会把它们输出至某些特定的日志文件。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/157B2313-21A6-45B0-B3B0-6B1AC169BB2A.png)
### syslog函数
syslog：应用程序使用syslog函数与rsyslogd守护进程通信。
openlog可改变syslog的默认输出方式，进一步结构化日志内容。
setlogmask：设置日志掩码，使日志级别大于日志掩码的日志信息被系统忽略。
```
void syslog(int priority, const char* message, …);
void openlog(const char* ident, int logout, int facility);
int setlogmask(int masker);
void closelog();
```

## 用户信息
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/%E6%88%AA%E5%B1%8F2021-09-14%20%E4%B8%8B%E5%8D%8811.01.51.png)

## 进程间关系
### 进程组
获取指定进程的PGID（进程组ID）：
pid_t getpgid(pid_t pid);
每个进程组都有一个首领进程，其PGID和PID相同。进程组将一直存在，直到其中所有进程都退出，或者加入到其他进程组。
设置PGID（将PID为pid的进程的PGID设置为pgid。若pid和pgid相同则pid进程将被设置为进程组首领）：
int setpgid(pid_t pid, pid_t pgid);
### 会话
创建一个会话：
pid_t setsid(void);
pid_t getsid(pid_t pid);
### 系统资源限制
int getrlimit(int resource, struct limit *rlim);
int setrlimit(int resource, const struct rlimit* rlim);
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/2D8198A6-3074-438A-A988-5F4207A5112C.png)
### 工作目录
获取进程当前工作目录：
char* getcwd(char* but, size_t size);
改变进程工作目录：
int chdir(const char* path);
改变进程根目录：
int chroot(const char* path);
### 服务器程序后台化
int daemon(int nochdir, int no close);

## 高性能服务器程序框架
### 服务器模型
#### C/S模型
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/%E6%88%AA%E5%B1%8F2021-09-16%20%E4%B8%8B%E5%8D%889.47.08.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/11699EEF-06E8-4591-B245-657893EA3A02.png)
#### P2P模型
Peer to Peer，点对点
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/D807C056-5E52-4BA8-ABA7-FA9144BE0368.png)

### 服务器编程框架
基本框架：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/8835C0DF-27D6-4703-931D-2E9F95EA912D.png)
|  模块                |     单个服务器程序                     |   服务器机群                         |
| —————————— | —————————————————————— | ——————————————————— | 
| I/O处理单元   |  处理客户连接，读写网络数据  | 作为接入服务器，实现负载均衡     |
| 逻辑单元         |  业务进程或线程                          |    逻辑服务器  |
| 网络存储单元 |  本地数据库、文件或缓存          |  数据库服务器    |
| 请求队列         |  各单元之间的通信方式              |   各服务器之间的永久TCP连接   |

![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/258BD33A-8E59-4187-BB2C-505CDBE58550.png)

### I/O模型
I/O复用：应用程序通过I/O复用函数向内核注册一组事件，内核通过I/O复用函数把其中就绪的事件通知给应用程序。Linux上常用的I/O复用函数是：select、poll和epoll_wait。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/11EC9B04-EA0D-40C0-8C65-F970D3F61BA3.png)

### 两种高效的事件处理模式
#### Reactor模式
Reactor模式：同步I/O模型通常用于实现Reactor模式。
Reactor模式要求主线程（I/O处理单元）只负责监听文件描述符上是否有事件发生，有的话立即通知工作线程（逻辑单元）。除此之外不做任何其他实质性工作。读写数据、接受新的连接、处理客户请求均在工作线程中完成。
工作流程：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/0F333291-6BF3-442F-A1D5-4D2D7B91974B.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/49E16A61-DE28-4C2B-88D9-82823C605FC1.png)

#### Proactor模式
Proactor模式：异步I/O模型用于实现Proactor模式。
Proactor模式将所有I/O操作都交给主线程和内核来处理，工作线程仅仅负责业务逻辑。
工作流程：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/161E4EAB-5483-4DAF-9C74-0589FD9B79CC.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/737C5B5E-CC13-418F-AC3A-618E14E398D1.png)

#### 使用同步I/O模拟Proactor模式
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/5F9970D8-5919-46CD-A59C-64A1E6AD62B2.png)

### 两种高效的并发模式
并发模式是指I/O处理单元和多个逻辑单元之间协调完成任务的方法。
#### 半同步/半异步模式
在并发模式中，同步指的是程序完全按照代码序列的顺序执行，异步指的是程序的执行需要由系统事件来驱动。常见的系统事件包括中断、信号等。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/9B80955F-459D-4C9D-9134-89ACFD401783.png)

半同步/半异步模式中，同步线程用于处理客户逻辑，异步线程用于处理I/O事件。异步线程监听到客户请求后，就将其封装成请求对象并插入请求队列中。请求队列将通知某个工作在同步模式的工作线程来读取并处理该请求对象。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/CC87C3A8-0B4C-4F95-8E3B-4B796C382FE8.png)

**半同步/半反应堆模式**：半同步/半异步模式的一种变体。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/%E6%88%AA%E5%B1%8F2021-09-16%20%E4%B8%8B%E5%8D%8811.11.48.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/A7467108-85A0-4319-B8E6-4D38393BFCBC.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/13B7C03C-7D82-427F-99B5-C96775D6E977.png)

**高效的半同步/半异步模式**
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/E4343130-F329-4624-BE09-B09B038F5C10.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/%E6%88%AA%E5%B1%8F2021-09-16%20%E4%B8%8B%E5%8D%8811.18.41.png)

#### 领导者/追随者模式
领导者/追随者模式是多个工作线程轮流获得事件源集合，轮流监听、分发并处理事件的一种模式。
在任意时间点，程序都仅有一个领导者线程，负责监听I/O事件。而其他线程则都是追随者，休眠在线程池中等待成为新的领导者。当前的领导者如果检测到I/O事件，首先要从线程池中推选出新的领导者线程，然后处理I/O事件。
新的领导者等待新的I/O事件。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/%E6%88%AA%E5%B1%8F2021-09-16%20%E4%B8%8B%E5%8D%8811.22.48.png)
HandleSet：句柄集
ThreadSet：线程集
EventHandler：事件处理器
ConcreteEventHandler：具体的事件处理器
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/7DA09BAD-96F1-46EB-931D-0CED8DD794B6.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/7FF48197-5A6D-43CA-BAB4-D8AFE484452E.png)
工作流程总结：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/090053FC-C714-4133-A4B5-47F1E5E34443.png)

### 有限状态机
逻辑单元内部的一种高效编程方法。

### 提高服务器性能的其他建议
#### 池
池是一组资源的集合，在服务器启动之初就创建好并初始化。当服务器运行时需要就可以直接从池中获取。
* 内存池：常用于socket的接收缓冲和发送缓冲
* 进程池和线程池：并发编程
* 连接池：通常用于服务器或服务器机群的内部永久连接
#### 数据复制
零拷贝、共享内存、指针等
#### 上下文切换和锁

## I/O复用
### select系统调用
用途：在一段指定时间内，监听用户感兴趣的文件描述符上的可读、可写和异常等事件。
**API**：
int select(int nfds, fd_set* readfds, fs_set* writefds, fd_set* excepties, struct timeval* timeout);
1）nfds：指定被监听的文件描述符总数，通常被设为select监听的所有文件描述符中最大值加1。
2）readfds、writefds、exceptfds：分别指向可读、可写和异常等事件对应的文件描述符集合。
访问fd_set结构：
FD_ZERO(fd_set* fdset);
FD_SET(int fd, fd_set* fdset);
FD_CLR(int fd, fd_set* fdset);
int FD_ISSET(int fd, fd_set* fdset);
3）timeval：设置超时时间。若给timeout的tv_sec和tv_usec成员都传递0，则select将立即返回。如果给timeout传NULL，则select将一直阻塞，直到某个文件描述符就绪。
struct timbal {
	long tv_sec;
	long tv_usec;
};
4）select成功时返回就绪文件描述符的总数。如果在超时时间内没有任何文件描述符就绪，select返回0。select失败时返回-1并设置errno。如果在select等待期间，程序接收到信号，则select立即返回-1，并设置errno为EINTR。
**文件描述符就绪条件**：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/AF621EAF-B32B-4578-A57F-05FD4A69B728.png)

### poll系统调用
用途：在指定时间内轮询一定数量的文件描述符，以测试其中是否有就绪者。
**API**：
int poll(struct pollfd* fds, nfds_t fds, int timeout);
1）fds：指定所有感兴趣的文件描述符上发生的可读、可写和异常等事件。
struct pollfd {
	int fd;               //文件描述符
	short events; //注册的事件
	short revents;//实际发生的事件，由内核填充
};
2）nfds：指定被监听事件集合fds的大小
3）timeout：指定poll的超时值，单位毫秒。timeout=-1时poll调用将永远阻塞，直到某个事件发生；当timeout=0时，poll调用将立即返回。
4）返回值含义与select相同

### epoll系列系统调用
epoll是Linux特有的I/O复用函数。
epoll使用一组函数来完成任务，而不是单个函数。epoll把用户关心的文件描述符上的事件放入内核里的一个事件表中，从而无须像select和poll那样每次调用都要重复传入文件描述符集或事件集。但epoll需要使用一个额外的文件描述符来唯一标识内核中的这个事件表（用epoll_create创建）。
**API**：
* int epoll_create(int size);
//size：只是告诉内核它的事件表需要多大。
//返回值：返回文件描述符，用于其他所有epoll系统调用的第一个参数，以指定要访问的内核事件表。
* int epoll_ctl(int epfd, int op, struct epoll_event* event);
//epfd：要操作的文件描述符
//op：指定操作类型：EPOLL_CTL_ADD、EPOLL_CTL_MOD、EPOLL_CTL_DEL
//event：指定事件。
struct epoll_event {
	__uint32_t events; //epoll事件类型
	epoll_data_t data;//用户数据
};
typedef union epoll_data {
	void* ptr;
	int fd;
	unit32_t u32;
	uint64_t u64;
} epoll_data_t;
//返回值：成功返回0，失败返回-1并设置errno
* int epoll_wait(int epfd, struct epoll_event* events, int max events, int timeout);
//timeout:与poll的timeout参数含义相同
//maxevents：指定最多监听多少个事件
//events：如果检测到事件，就将所有就绪的事件从内核事件表中复制到events指向的数组中。
//返回值：成功时返回就绪的文件描述符的个数，失败返回-1并设置errno。
**LT和ET模式**：
LT：Level Trigger，电平触发
ET：Edge Trigger，边沿触发
LT是默认的工作模式，此模式下相当于效率较高的poll。
当往epoll内核事件表中注册一个文件描述符上的EPOLLET事件时，epoll将以ET模式来操作该文件描述符。ET模式是epoll的高效工作模式。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/B0C13AE5-A25B-42DC-9B98-3DE560605A67.png)
**EPOLLONESHOT事件**：
对于注册了EPOLLONESHOT事件的文件描述符，操作系统最多触发其上注册的一个可读、可写或者异常事件，且只触发一次，除非使用epoll_ctl函数重置该文件描述符上注册的EPOLLONESHOT事件。

### select、poll和epoll的区别
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/%E6%88%AA%E5%B1%8F2021-09-18%20%E4%B8%8B%E5%8D%8810.09.34.png)

###  非阻塞connect
对非阻塞的socket调用connect，而连接又没有立即建立时，会返回errno值为EINPROGRESS的错误。
在这种情况下，我们可以调用select、poll等函数来监听这个连接失败的socket上的可写事件。当select、poll等函数返回后，再利用getsockopt来读取错误码并清除该socket上的错误。如果错误码时0表示连接成功建立，否则连接失败。
通过这种非阻塞connect方式，我们就能同时发起多个连接并一起等待。

### 超级服务xinetd
Linux因特网inetd是超级服务，它同时管理着多个子服务（即监听多个端口）。
xinetd是升级版本，增加了一些控制选项，并提高了安全性。
xinetd采用/etc/xinetd.conf主配置文件和/etc/xinetd.d目录下的子配置文件来管理所有服务。
xinetd管理的子服务类型：
1）标准服务：xinetd服务器在内部直接处理这些服务。比如日期服务daytime、回射服务echo和丢弃服务discard。
2）需要调用外部服务器程序来处理：通过fork和exec函数来加载运行这些服务器程序。比如telnet、ftp服务。
xinetd的工作流程：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/082247A3-F5D7-436B-9E64-E203EFD33834.png)

## 信号
信号是由用户、系统或者进程发送给目标进程的信息，以通知目标进程某个状态的改变或系统异常。
### Linux信号产生的条件：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/3EAA2FE4-539C-45FC-8CE9-0C0ACA0C258E.png)
### Linux发送信号
int kill(pid_t pid, int sig);//pid：为目标进程
pid参数含义：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/AEB196EC-1E9C-4EF3-A7B0-63A73C369EBD.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/02B9EBFA-DF3E-48A2-8E06-B0E208D3907E.png)
### 信号处理方式
typedef void (*__sighandler_t) (int);
信号处理函数只带有一个整形参数，用来指示信号类型。
信号处理函数应该是可重入的，否则很容易引发一些竞态条件。
**信号的两种其他处理方式**：
#define SIG_DFL  ((__sighandler_t) 0)  表示使用信号的默认处理方式
#define SIG_IGN  ((__sighandler_t) 1)   表示忽略目标信号
信号的默认处理方式：结束进程、忽略信号、结束进程并生成核心转储文件、暂停进程、继续进程
### Linux信号
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/FEF4B5A1-9C25-4223-A23D-79030841ACC2.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/A5BDACFD-A248-411B-94B8-853E3D4C0D0F.png)
### 中断系统调用
如果程序在执行处于阻塞状态的系统调用时接收到信号，并且我们为该信号设置了信号处理函数，则默认情况下系统调用将被中断，并且errno被设置为EINTR。
可以使用sigaction函数为信号设置SA_RESTART标志以自动重启被该信号中断的系统调用。
### 信号函数
* _sighandler_t signal(int sig, _sighandler_t _handler);
用途：为一个信号设置处理函数。
返回值：成功返回前一次调用signal函数时传入的函数指针或者sig对应的默认处理函数指针（第一次调用signal）；出错返回SIG_ERR，并设置errno。
* int sigaction(int sig, const struct sigaction* act, struct sigaction* oact);
//sig：要捕获的信号类型
//act：指定新的信号处理方式
//oact：输出信号先前的处理方式（如果不为NULL的话）
返回值：成功返回0，失败返回-1并设置errno
### 信号集
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/F984C37A-D8AB-41A9-A498-38AC618B2AA7.png)
### 进程信号掩码
int sigprocmask(int _how, _const sigset_t* _set, sigset_t* _oset);
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/A0AC5037-8FB5-4C8D-ACD1-4984114755A3.png)
如果_set为NULL，则进程信号掩码不变。此时可用_oset来获得进程当前信号掩码。
返回值：成功返回0，失败返回-1并设置errno。
### 被挂起的信号
int sigpending(sigset_t* set);
作用：获得进程当前被挂起的信号集。
返回值：成功返回0，失败返回-1并设置errno。
### SIGHUP
当挂起进程的控制终端时，SIGHUP信号将被触发。
对于没有控制终端的网络后台程序而言，它们通常利用SIGHUP信号来强制服务器重读配置文件。
### SIGPIPE
默认情况下，往一个读端关闭的管道或socket连接中写数据将引发SIGPIPE信号。需要在代码中捕获并处理该信号，或者至少忽略它。因为接收到SIGPIPE信号的默认行为是结束进程。
引起SIGPIPE信号的写操作将errno设置为EPIPE。
可以利用I/O复用系统调用来检测管道和socket连接的读端是否已经关闭。（以poll为例，管道读端关闭时，写端文件描述符上的POLLHUP事件将被触发，当socket连接被对方关闭时，socket上的PLLLRDHUP事件将被触发）
### SIGURG
用于内核通知应用程序带外数据到达。

## 定时器
定时是指在一段时间之后触发某段代码的机制。可以在这段代码中依次处理所有到期的定时器。
### Linux提供的三种定时方法
* socket选项SO_RCVTIMEO和SO_SNDTIMEO
* SIGALRM信号
* I/O复用系统调用的超时参数

### socket选项SO_RCVTIMEO和SO_SNDTIMEO
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/A483DEE9-F861-4557-9269-856986B4912B.png)

### SIGALRM信号
由alarm和setitimer函数设置的实时闹钟一旦超时，将触发SIGALRM信号。
一般而言，SIGALRM信号按照固定的频率生成，即由alarm或setitimer函数设置的定时周期T保持不变。

### 高性能定时器
#### 时间轮
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/857A15D7-7E44-4B68-9DFB-67D9D44C5477.png)
时间轮内，指针指向轮子上的一个槽（slot），它以恒定的速度顺时针转动，每次转动称为一个滴答（tick）。一个滴答的时间称为时间轮的槽间隔si（slot interval），即心博时间。该时间轮共N个槽，转动一周时间为N*si。每个槽指向一条定时器链表，每条链表上的定时器具有相同的特征：它们的定时时间相差N*si的整数倍。加入现在指针指向槽cs，要添加一个定时时间为ti的定时器，则该定时器将被插入槽ts对应的链表中：ts = ( cs + (ti / si) ) % N。
检测到期的定时器是，执行其上的回调函数。
#### 时间堆
考虑：将所有定时器中超时时间最小的一个定时器的超时值作为心博间隔。这样，一旦心博函数tick被调用，超时时间最小的定时器必然到期，就可以在tick函数中处理该定时器。然后再次从剩余的定时器中找出超时时间最小的一个，作为下一次心博间隔。
用最小堆实现的定时器为时间堆。

## I/O框架概述
各种I/O框架库的实现原理基本相似，要么以Reactor模式实现，要么以Proactor模式实现，要么同时以这两种模式实现。
基于Reactor模式的I/O框架库包含组件：句柄（Handler）、事件多路分发器（EventDemultiplexer）、事件处理器（EventHandler）和具体的事件处理器（ConcreteEventHandler）、Reactor。
**组件关系如下**：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/2C05C615-D70A-42F6-B77E-62C9789567A2.png)
1）句柄：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/CD17F61E-DA1B-4A8D-8E8F-0117AE928237.png)
2）事件多路分发器：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/CC89A1BD-9631-4C98-A199-FB93787A485C.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/E8B9CE9C-505C-4BE3-8A39-1EAADB68F737.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/54B3086C-97E6-4D3B-8385-6E1A6D62C8D4.png)
3）事件处理器和具体事件处理器：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/AC3921A0-B350-445B-BFA7-359EBDE5DADD.png)
4）Reactor：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/5D9E182F-1580-42E2-9E3E-F79CA7B58152.png)
**I/O框架库的工作时序**：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/4540097A-7D40-4671-90FA-4E0FEDBA068B.png)

### Libevent
Libevent是开源社区的一款高性能的I/O框架库。
使用Libevent的有：高性能的分布式内存对象缓存软件memcached、Google浏览器Chromium的Linux版本。
Libevent的特点：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/E7AEEB3E-5D7E-46FF-B3F2-A72AD92B6355.png)

## 多进程编程
### fork系统调用
pid_t fork(void);
作用：创建新进程。每次调用都返回两次，在父进程中返回的是子进程的pid，在子进程中返回0。调用失败时返回-1。
父进程和子进程：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/9A16BF43-279C-414F-9AFE-4B4A04FDA402.png)
### exec系列系统调用
作用：在子进程中执行其他程序，即替换当前进程映像。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/7D347AE8-B3ED-4EFD-8FA2-3384570BC0D9.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/7900C85D-F412-4E3E-8145-981DBFE54401.png)
### 处理僵尸进程
子进程进入僵尸态的情况：
1）子进程结束运行后，父进程读取其退出状态之前，子进程处于僵尸态。
2）父进程结束或者异常终止，而子进程继续运行。此时子进程的PPID被操作系统设置为1（即init进程）。init进程接管了该子进程，并等待它结束。在子进程退出之前，处于僵尸态。
处于僵尸态的子进程占据着内核资源。
**API**：
```
pid_t wait(int* stat_loc);
pid_t waitpid(pid_t pid, int* stat_loc, int options);
```
wait函数将阻塞进程，直到该进程的某个子进程结束运行为止。它返回结束运行的子进程的PID，并将该子进程的退出状态信息存储于stat_loc中。
**子进程的退出状态信息**：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/FDF83758-360A-45E0-BAED-B4F16FEA02F7.png)
waitpid只等待由pid指定的子进程。（pid=-1时与wait函数相同）
options参数控制waitpid的行为。取WNOHANG时waitpid调用是非阻塞的：如果pid指定的目标子进程还没有结束或意外终止，则waitpid立即返回0；如果目标子进程确实正常退出了，则返回子进程的PID。
对waitpid函数而言，最好在某个子进程退出之后再调用它。
**SIGCHLD**：
当一个进程结束时，它将给其父进程发送一个SIGCHLD信号。
我们可以在父进程中捕获SIGCHLD信号，并在信号处理函数中调用waitpid函数以“彻底结束”一个子进程。
### 管道
管道也是父进程和子进程间通信的常用手段。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/201C1912-682A-4067-B810-1963A777FA5E.png)
要实现父、子进程之间的双向数据传输，必须要使用两个管道。
### 信号量
假设有信号量SV，则对它的P、V操作含义如下：
* P(SV)：如果SV的值大于0，就将它减1；如果SV的值为0，则挂起进程的执行。
* V(SV)：如果有其他进程因为等待SV而挂起，则唤醒之；如果没有，则将SV加1。
信号量的取值可以时任何自然数，但最常用最简单的是二进制信号量。
**二进制信号量保护关键代码段**：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/17D68019-62B0-4ECA-9D46-45543B57F4C0.png)
当关键代码段可用时，二进制信号量SV的值为1，进程A和B都有机会进入关键代码段。如果此时进程A执行了P(SV)操作将SV减1，则进程B若再执行P(SV)就会被挂起。知道进程A离开关键代码段，并执行V(SV)操作将SV加1，关键代码段才重新变得可用。如果此时进程B因为等待SV而处于挂起状态，则它将被唤醒，并进入关键代码段。
**信号量API**：
定义在sys/sem.h头文件中，主要包含3个系统调用：semget、semop和semctl。
#### semget系统调用
int semget(key_t key, int num_sems, int sem_flags);
参数：
    key：键值，用来标识一个全局唯一的信号量集。
    num_sems：指定要创建/获取的信号量集中信号量的数目。
    sem_flags：指定一组标志。它低端的9个比特是该信号量的权限，其格式和含义与open系统调用的mode参数相同。它还可以和IPC_CREAT标志做按位“或”运算以创建新的信号量集（已存在时不产生错误）。可以联合使用IPC_CREAT和IPC_EXCL标志来确保创建一组新的、唯一的信号量集（已存在时返回错误）。
返回值：成功时返回一个正整数值标识信号量集；失败时返回-1并设置errno。
#### semop系统调用
作用：改变信号量的值，即执行P、V操作。
与每个信号量关联的一些重要内核变量：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/B244C61F-DDD6-4BE3-B924-5FFF63FC3305.png)
int semop(int sem_id, struct sembuf* sem_ops, size_t num_sem_ops);
参数：
    1）sem_id：由semget调用返回的信号量集标识符，用以指定被操作的目标信号量集。
    2）semops：指向一个sembuf结构体类型的数组：
struct sembuf{
	unsigned short int sem_num;//信号量集中信号量的编号
	short int sem_op;//指定操作类型，每种类型的操作的行为又收到sem_flg成员的影响
	short int sem_flg;//IPC_NOWAIT或SEM_UNDO
};
sem_op和sem_flg将按照如下方式来影响semop的行为：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/56D1B6A6-609B-4E9C-AD20-9EAA6F41B8E4.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/90767465-8539-472B-9B1B-218530ABFEAA.png)
    3）num_sem_ops：指定要执行的操作个数，即sem_ops数组元素个数。
semop对数组sem_ops中的每个成员按照数组顺序依次执行操作，并且该过程是原子操作，以避免别的进程在同一时刻按照不同的顺序对该信号集汇总的信号量执行semop操作导致的竞态条件。
返回值：成功返回0，失败返回-1并设置errno。失败时sem_ops数组中指定的所有操作都不被执行。
#### semctl系统调用
作用：允许调用者对信号量进行直接控制。
int semctl(int sem_id, int sem_num, int command, …);
参数：
    sem_id：semget调用返回的信号量集标识符，指定被操作的信号量集。
    sem_num：指定被操作的信号量在信号量集中的编号。
    command：指定要执行的命令。有的命令需要传递第4个参数。
**支持的command参数**：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/1FCB6DFB-4CAC-4057-820C-1C4EEBD448E9.png)
#### 特殊键值IPC_PRIVATE
semget的调用者可以给其key参数传递一个特殊的键值IPC_PRIVATE（其值为0），无论信号量是否已存在，都将创建一个新的信号量。

### 共享内存
共享内存是最高效的IPC机制，因为它不涉及进程之间的任何数据传输。
我们必须用其他辅助手段来同步进程对共享内存的访问，否则会产生竞态条件。
#### shmget系统调用
作用：用于创建一段新的共享内存，或者获取一段已存在的共享内存。
int shmget(key_t key, size_t size, int shmflg);
参数：
    key：标识一段全局唯一的共享内存
    size：指定共享内存的大小，单位字节。
    shmflg：同semget的sem_flags参数，额外支持SHM_HUGETLB（大页）和SHM_NORESERVE（不保留交换分区）。
返回值：成功返回共享内存的标识符，失败返回-1并设置errno。
#### shmat和shmdt系统调用
* void* shmat(int shm_id, const void* shm_addr, int shmflg);
参数：
    shm_id：由shmget返回的共享内存标识符
    shm_addr：指定将共享内存关联到进程的哪块地址空间。最终效果还受到shmflg参数的可选标志SHM_RND的影响：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/4A5E9099-E909-464E-BCA1-46EDC17A586D.png)
    shmflg：除SHM_RND外还支持如下：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/909A7C73-D753-444A-B50B-4A0C9BC3A780.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/00300249-2058-4333-A488-52BDC27D40F3.png)
返回值：成功返回共享内存被关联到的地址，失败返回(void*)-1并设置errno。
* int shmdt(const void* shm_addr);
作用：将关联到shm_addr处的共享内存从进程中分离。
返回值：成功返回0，失败返回-1并设置errno。
#### shmctl系统调用
int shnctl(int shm_id, int command, struct shmid_ds* buf);
参数：
    shm_id：由shmget返回的共享内存标识符。
    command：指定要执行的命令。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/C5EB1056-3FE5-4779-8E70-B5D9EAFFCB42.png)
返回值：成功的返回值取决于command参数，失败返回-1并设置errno。
#### 共享内存的POSIX方法
mmap函数利用它的MAP_ANONYMOUS标志可以实现父子进程之间的匿名内存共享。
mmap在无关进程之间共享内存的方式：通过shm_open创建或打开一个POSIX共享内存对象。
int shm_open(const char* name, int flag, mode_t mode);
参数：
    name：指定要创建/打开的共享内存对象（应该使用“/somename”的格式）
    oflag：指定创建方式，可以是下列标志中的一个或多个的按位或：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/EC9F19BF-C2EA-46C7-9D84-F336926B3C2D.png)
返回值：成功返回一个文件描述符，可用于后续的mmap将共享内存关联到调用进程。失败返回-1并设置errno。
int shm_unlink(const char* name);
作用：删除使用完的共享内存对象。该函数将name参数指定的共享内存对象标记为等待删除。当所有使用该共享内存对象的进程都使用unmap将它从进程中分离之后，系统将销毁这个共享内存所占据的资源。

### 消息队列
消息队列是在两个进程之间传递二进制块数据的一种简单有效的方式。每个数据块都有一个特定的类型，接收方可以根据类型来有选择地接收数据。
#### msgget系统调用
作用：创建一个消息队列，或者获取一个已有的消息队列。
int msgget(key_t key, int msgflg);
参数：与semget参数相同
返回值：成功返回消息队列的标识符，失败返回-1并设置errno。
#### msgsnd系统调用
作用：把一条消息添加到消息队列中。
int msgsnd(int msqid, vonst void* msg_ptr, size_t msg_sz, int msgflg);
参数：
    msqid：由msgget调用返回的消息队列标识符
    msg_ptr：指向一个准备发送的消息，消息类型为msgbuf
struct msgbuf {
	long mtype;   //指定消息的类型
	char mtext[512];//消息数据
};
    msg_sz：消息的数据部分（mtext）的长度。可以为0标识没有消息数据。
    msgflg：控制msgsnd的行为，通常仅支持IPC_NOWAIT标志，即以非阻塞的方式发送消息。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/66C7B645-6D6A-4C48-90FC-AC8254DD5A7C.png)
返回值：成功返回0，失败返回-1并设置errno。
#### msgrcv系统调用
作用：从消息队列中获取消息。
int msgrcv(int msqid, void* msg_ptr, size_t msg_sz, long int msgtype, int msgflg);
参数：
    msqid：由msgget调用返回的消息队列标识符。
    msg_ptr： 用于存储接收的消息
    msg_sz：指的是消息数据部分的长度
    msgtype：指定接收何种类型的消息。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/60A716A2-C1A4-4F1C-B747-3555D1E843C4.png)
    msgflg：控制msgrcv函数的行为。可以是如下一些标志的按位或：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/B13DC19D-7AD9-4048-8A96-6790EF67367A.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/F4B6FBAD-29B4-4D31-9DB9-5549830EEE8D.png)
返回值：成功返回0，失败返回-1并设置errno。
#### msgctl系统调用
作用：控制消息队列的某些属性。
int msgctl(int msqid, int int command, struct msqid_ds* buf);
参数：
    msqid：由msgget返回的消息队列标识符。
    command：指定要执行的命令
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/5085DCCC-8EA2-4F58-A06E-862DA0ADA30F.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/051BBF5D-D769-425F-A536-4AE1EC78238D.png)
返回值：成功时取决于command参数，失败返回-1并设置errno。

### IPC命令
Linux提供了ipcs命令，以观察当前系统上拥有哪些共享资源实例。
ipcrm命令可以删除遗留在系统中的共享资源。

### 在进程间传递文件描述符
注意：传递一个文件描述符并不是传递一个文件描述符的值，而是要在接收进程中创建一个新的文件描述符，并且该文件描述符和发送进程中被传递的文件描述符指向内核中相同的文件表项。
在Linux下，可以利用UNIX域socket在进程间传递特殊的辅助数据，以实现文件描述符的传递。

## 多线程编程
### Linux线程模型
线程是程序中完成一个独立任务的完整执行序列，即一个可调度的实体。
根据运行环境和调度者的身份，线程可分为内核线程和用户线程。
**内核线程**运行在内核空间，由内核调度；
**用户线程**运行在用户空间，由线程库调度。
当进程的一个内核线程获得CPU的使用权时，它就加载并运行一个用户线程。可见内核线程相当于用户线程运行的“容器”。
一个进程可以拥有M个内核线程和N个用户线程，其中M<=N。
在一个系统的所有进程中，M和N的比值都是固定的。

按照M:N的取值，线程的实现方式分为三种模式：完全在用户空间实现、完全由内核调度和双层调度。
* **完全在用户空间实现的线程**：
线程库负责管理所有执行线程，利用longjmp切换线程的执行。一个进程的所有执行线程共享该进程的时间片，它们对外表现出相同的优先级。因此，N=1，即M个用户空间线程对应1个内核线程，而内核线程实际上就是进程本身。
优点：创建和调度线程都无须内核的干预，速度快；不占用额外的内核资源，不影响系统性能。
缺点：对于多处理器系统，一个进程的多个线程无法运行在不同的CPU上，因为内核是按照其最小调度单位来分配CPU的。线程的优先级只对同一进程中的线程有效。
* **完全由内核调度**：
将创建、调度线程的任务都交给了内核，运行在用户空间的线程库无须执行管理任务。完全由内核调度的线程实现方式满足M:N=1:1，即1个用户空间线程被映射为1个内核线程。
优缺点与完全在用户空间实现的线程正好相反。
* **双层调度模式**：
内核调度M个内核线程，线程度调度N个用户线程。
结合了前两者的优点：不会消耗过多的内核资源，线程切换速度也很快，同时它可以充分利用多处理器的优势。

### 创建线程和结束线程
#### pthread_create
int pthread_create(pthread_t* thread, const pthread_attr_t* attracted, void* (*start_routine)(void*), void* arg);
返回值：成功返回0，失败返回错误码。
一个用户可以打开的线程数量不能超过RLIMIT_NPROC软资源限制。
系统上所有用户能创建的线程总数也不能超过/proc/sys/kernel/threads-max内核参数所定义的值。
线程一旦被创建好，内核就可以调度内核线程来执行start_routine所指向的函数了。
#### pthread_exit
void pthread_exit(void* retail);
作用：线程函数在结束时调用，以确保安全、干净地退出。
通过retval参数向线程的回收者传递其退出信息。
它执行完之后不会返回到调用者，而且永远不会失败。
#### pthread_join
作用：一个进程中的所有线程都可以调用pthread_join函数来回收其他线程（前提目标线程是可回收的），即等待其他线程结束。
int pthread_join(pthread_t thread, void** retval);
参数：
     thread：目标线程的标识符
     retval：目标线程返回的退出信息
该函数会一直阻塞，直到被回收的线程结束为止。
返回值：成功返回0，失败返回错误码：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/BC735907-2CA3-4B99-A277-2FC8111CD6DD.png)
#### pthread_cancle
* 作用：异常终止一个线程，即取消线程。
int pthread_cancle(pthread_t thread);
* 接收到取消请求的目标线程可以决定是否允许被取消以及如何取消：
int pthread_setcanclestate(int state, int* oldstate);
参数：
    state：设置线程的取消状态（是否允许取消）
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/6854DFF8-4A02-4159-A9B1-4450CE9EAE76.png)
    oldstate：记录线程原来的取消状态
int pthread_setcancletype(int type, int* oldtype);
参数：
    type：设置线程的取消类型（如何取消）
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/17CD94A2-6610-415D-A35D-F6D9CA4A1E69.png)
    oldtype：记录线程原来的取消类型。
返回值：两个函数成功时返回0，失败时返回错误码

### 线程属性
pthread_attr_t结构体定义了一套完整的线程属性：
```
#include <bits/pthreadtypes.h>
#define _SIZEOF_PTHREAD_ATTR_T 36
typedef union {
	char _size[_SIZEOF_PTHREAD_ATTR_T];
	long int _align;
} pthread_attr_t;
```
各种线程属性全部包含在一个字符数组中。

线程库定义了一系列函数来操作pthread_attr_t类型的变量，便于获取和设置线程属性：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/6BF09DE3-0F4F-4DE0-B534-348C7D5D6763.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/92675806-9887-4EAF-9B0C-16069F10C52A.png)

每个线程属性的含义：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/07E22FF3-350D-4E3A-97D7-5FDCD0BA66D9.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/11F1B1BB-5FE6-439D-A638-37C6110C47BB.png)

### POSIX信号量
```
int sem_init(sem_t* sem, int pshared, unsigned int value);
int sem_destroy(sem_t* sem);
int sem_wait(sem_t* sem);
int sem_trywait(sem_t* sem);
int sem_post(sem_t* sem);
```
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/A0929450-0A03-4A29-8FC8-C80EEA9E5F0C.png)

### 互斥锁
#### 互斥锁API
```
int pthread_mutex_init(pthread_mutex_t* mutex, const pthread_mutexattr_t* mutexattr);
int pthread_mutex_destroy(pthread_mutex_t* mutex);
int pthread_mutex_lock(pthread_mutex_t* mutex);
int pthread_mutex_trylock(pthread_mutex_t* mutex);
int pthread_mutex_unlock(pthread_mutex_t* mutex);
```
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/31F6358A-FAA8-4425-919E-BB9E751BB048.png)
#### 互斥锁属性
pthread_mutexattr_t结构体定义了一套完整的互斥锁属性。
线程库提供了一系列函数来操作pthread_mutexattr_t类型的变量，以便于获取和设置互斥锁属性：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/D8F0BB01-F9A1-4AA4-A79A-E9E087F00B54.png)
* 互斥锁常见属性pshared：指定是否允许跨进程共享互斥锁。可选值如下：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/F16ADCA0-E577-40FD-ADC8-2955F6E7AE9C.png)
* 互斥锁常见属性type：指定互斥锁的类型。Linux支持如下4种类型：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/C7F48A9F-B3AF-4406-996A-921EB4552BDA.png)

### 条件变量
用于在线程之间同步共享数据的值。
条件变量提供了一种线程间的通知机制：当某个共享数据达到某个值的时候，唤醒等待这个共享数据的线程。
```
int pthread_cond_init(pthread_cond_t* cond, const pthread_condattr_t* cond_attr);
int pthread_cond_destroy(pthread_cond_t* cond);
int pthread_cond_broadcast(pthread_cond_t* cond);
int pthread_cond_signal(pthread_cond_t* cond);
int pthread_cond_wait(pthread_cond_t* cond, pthread_mutex_t* mutex);
```
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/8525349C-BBBB-45BF-BDD6-3D73B3EB00F8.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/38838489-C50C-4779-9A0C-24824DF08407.png)

### 多线程环境
#### 可重入函数
如果一个函数能被多个线程同时调用且不发生竞态条件，则称它是线程安全的，或者说可重入函数。
在多线程程序中调用库函数，一定要使用其可重入版本。
#### 线程和进程
如果一个多线程程序的某个线程调用了fork函数，那么新创建出来的子进程只拥有一个执行线程，它是调用fork那么线程的完整复制。并且子进程将自动继承父进程中互斥锁（条件变量与之类似）的状态。也就是说，父进程中已经被加锁的互斥锁在子进程中也是被锁住的。问题：子进程可能不清楚从父进程继承而来的互斥锁的具体状态。
pthread提供了专门的函数pthread_atfork确保fork调用后父进程和子进程都拥有一个清楚的锁状态。
int pthread_atfork(void (*prepare)(void), void (*parent)(void), void (*child)(void));
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/C5C21AB8-974F-47C0-B58A-218862BF47CA.png)
#### 线程和信号
* 多线程环境下设置线程信号掩码函数：
int pthread_sigmask(int how, const sigset_t* newmask, sigset_t* oldmask);
函数的参数定义同sigprocmask。
* 多线程环境下处理信号的正确方式：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/9D58921D-6984-4ED2-8A3F-992896A5FC56.png)
* 多线程环境下，明确将一个信号发送给指定线程的函数：
int pthread_kill(pthread_t thread, int sig);
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/4E213103-C478-4E58-B261-9FBAE1298E08.png)

## 进程池和线程池
进程池和线程池相似。
进程池是由服务器预先创建的一组子进程，这些子进程数目在3～10之间。线程池中的线程数量应该和CPU数量差不多。
进程池的一般模型：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/4E1E6656-B4E2-4BB7-BEDF-D8A441F8CCA6.png)
### 处理多客户
在使用进程池处理多客户任务时，需要考虑：监听socket和连接socket是否都由主进程来统一管理；一个客户连接上的所有任务是否始终由一个子进程来处理。如果客户任务存在上下文关系，最好一直用同一个子进程来为止服务。


## 服务器调制、调试和测试
### 最大文件描述符数
系统分配给应用程序的文件描述符数量是有限制的，所以我们必须总是关闭那些已经不再使用的文件描述符，以释放它们占用的资源。
比如作为守护进程运行的服务器程序总是关闭标准输入、标准输出和标准错误这三个文件描述符。

Linux对应用程序能打开的最大文件描述符数量有两个层次的限制：用户级限制和系统级限制。
用户级限制：指目标用户运行的所有进程总共能打开的文件描述符数；
系统级限制：指所有用户总共能打开的文件描述符数。

一些常用命令：
* 查看用户级文件描述符数限制：
ulimit -n 
* 将用户级文件描述符数设定为max-file-number（临时，只在当前session中有效）：
ulimit -SHn max-file-number 
* 永久修改用户级文件描述符数限制：
在/etc/security/limits.conf加入如下两项：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/C6C99C33-C340-4121-88F8-1A8E7F32D053.png)
* 修改系统级文件描述符数限制（临时）：
sysctl -w fs.file-max=max-file-number
* 永久更改系统级文件描述符数限制：
在/etc/sysctl.conf中添加 fs.file-max=max-file-number，然后执行sysctl -p使生效。

### 调整内核参数
几乎所有的内核模块，包括内核核心模块和驱动程序，都在/proc/sys文件系统下提供了某些配置文件以供用户调整模块的属性和行为。通常一个配置文件对应一个内核参数，文件名就是参数的名字，文件的内容就是参数的值。
命令sysctl -a可以查看所有这些内核参数。
#### /proc/sys/fs目录
/proc/sys/fs目录下的内核参数都与文件系统相关。
* /proc/sys/fs/file-max：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/BA8C0C2B-4068-41EB-872E-5A83E9213D7A.png)
* /proc/sys/fs/epoll/max_user_watches：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/E0019C93-D735-4739-9319-6085AFA22F15.png)
#### /proc/sys/net目录
内核中网络模块的相关参数都位于/proc/sys/net目录下，其中和TCP/IP协议相关的参数主要位于如下三个子目录：core、ipv4和ipv6。
和服务器性能相关的部分参数如下：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/6DA2FDE8-CA2C-439A-A115-C9E8E62F512D.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/88D86AE1-A35B-481C-BE97-87213D004451.png)
除了通过直接修改文件的方式修改这些系统参数外，也可以使用sysctl命令来修改它们。这两种都是临时的。永久的修改方法是在/etc/sysctl.conf文件中加入相应网络参数及其数值，并执行sysctl -p使之生效。

### gdb调试
#### 用gdb调试多进程程序
调试子进程的方法有两种：
* 单独调试子进程：
找到子进程的PID，然后gdb attach 子进程
* 使用调试器选项follow-fork-mode
follow-fork-mode允许我们选择程序在执行fork系统调用后是继续调试父进程还是调试子进程。
用法：
(gdb) set follow-fork-mode mode
其中mode可选值是parent和child，分别表示调试父进程和子进程。
#### 用gdb调试多线程程序
gdb有一组命令可辅助多线程程序的调试：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/3E5CD0AE-88B7-4E52-8BD9-71AD1A7316D8.png)
**关于调试进程池和线程池程序的一个不错方法**：
先将池中的进程个数或线程个数减少至1，以观察程序的逻辑是否正确。然后逐步增加进程或线程的数量，以调试进程或线程的同步是否正确。

### 压力测试
压力测试程序有多种实现方式：比如I/O复用方式，多线程、多进程并发编程方式，以及这些方式的结合使用。
单纯的I/O复用方式的施压程度是最高的，因为线程和进程的调度本身也要占用一定CPU时间。

## 系统检测工具
### tcpdump
tcpdump提供大量选项，用以过滤数据包或定制输出格式。常见选项如下：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/785E7EBC-315B-416D-8985-F297F9787F42.png)
tcpdump还支持用表达式来进一步过滤数据包。
表达式的操作数分为3种：类型type、方向dir和协议proto。
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/CA2178CC-F9C2-4765-8405-9E4FA6A4B0EE.png)
支持逻辑操作符：包括and（&&）、or（||）、not（!）
如果表达式比较复杂，可以使用括号将它们分组。
允许直接使用数据包中的部分协议字段的内容来过滤数据包。

### lsof
lsof（list open file）是一个列出当前系统打开的文件描述符的工具。
常用选项：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/055D0816-2FD7-4E18-8E83-220B48F39313.png)
lsof输出内容：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/1C7A3EE9-7BEA-4CD6-995D-5FD0C0C008B1.png)

### nc
nc（netcat）
主要用来快速构建网络连接。
可以让它以服务器方式运行，监听某个端口并接收客户连接，因此可用来调试客户端程序；可以让它以客户端方式运行，向服务器发起连接并收发数据，因此可用来调试服务器程序。
nc常用的选项：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/39794A70-43F0-44A2-AB0C-F20C467A85A5.png)

### strace
strace是测试服务器性能的重要工具。它跟踪程序运行过程中执行的系统调用和接收到的信号，并将系统调用名、参数、返回值及信号名输出到标准输出或者指定的文件。
常用选项：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/EA4074AF-A322-4827-88FB-3321A4E54531.png)

### netstat
netstat是一个功能很强大的网络信息统计工具。
它可以打印本地网卡接口上的全部连接、路由表信息、网卡接口信息等。
常用的选项：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/5FD1147D-6332-49FB-8684-8B05175A7C4A.png)
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/217D029D-E779-4B03-B29A-779DD54660CC.png)
netstat的每行输出字段：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/EFB9A6C8-CECC-4FA1-8536-EBEBFFCCD203.png)

### vmstat
vmstat（virtual memory statistics），它能实时输出系统的各种资源的使用情况，比如进程信息、内存使用、CPU使用率以及I/O使用情况。
常用选项和参数：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/2077EF54-4317-4EC0-B29F-9205F2AB4422.png)
输出信息：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/315CFCD0-C093-4E89-AA7A-E26AE7283E97.png)
不过，可以使用iostat命令获得磁盘使用情况的更多信息，可以使用mpstat获得CPU使用情况的更多信息。vmstat主要用于查看系统内存的使用情况。

### ifstat
ifstat（interface statistics），是一个简单的网络流量监测工具。
常用选项和参数：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/D6BC9891-FB9F-40A7-8F7A-9FDA8965D76F.png)
 
### mpstat
mpstat（multi-processor statistics），能实时监测多处理器系统上每个CPU的使用情况。
mpstat和iostat命令通常都集成在包sysstat中。
典型用法：
![](Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/1FF90E85-E386-48D8-B4FB-60E4CBB882EE.png)