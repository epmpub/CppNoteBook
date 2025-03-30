# Linux POSIX Semaphore的使用

# 目的

Semaphores are not used to transfer data between processes. Instead, they allow processes to synchronize their actions. One common use of a semaphore is to synchronize access to a block of shared memory in order to prevent one process from accessing the shared memory at the same time as another process is updating it.
---- Chapter 47: System V Semaphore <<The Linux Programming Interface>>
简而言之，信号量的使用是为了进程同步的用途，避免一个进程访问一个共享内存时候被另一个进程更新。

对于POSIX semaphore而言，信号量可以允许进程和线程进行同步，而访问共享的资源。

# 原理

semaphore是内核维护的一个整数，它的值被严格限制大于或者等于0. **信号量是非负整数。** 那么我们可以想一下维护一个数字需要做什么？ 赋值，增加，减少，判断，那么对应信号量的操作

- 设置它为一个正整数的值
- 增加这个数字
- 减少这个数字
- 等待这个数字变为0
  **在有一些书上，会把信号量的操作叫做PV操作，为啥是P，因为这个是荷兰计算机科学家Dijkstra，就是那个最短路径遍历算法的提出者发明的。在荷兰语中P表示Prolaag表示减少， V表示Verhoog是增加的意思。P表示Wait， V表示Signal。**

# 类型

SUSv3 阐释了两种POSIX 信号量：

## 命名信号量，named semaphores

这种信号量有一个名字，通过其名字来调用sem_open()函数，无关的进程也可以访问相同的信号量

## 非命名信号量，unamed semephores

这种信号量并没有一个名字，他们存在于一个约定好的内存空间。无名信号量可以被进程间，或者一组线程们进行共享。被进程共享的时候，信号量存在于共享内存的区域。但是被进程共享时，信号量存在于被进程共享的内存区域里。

------

# 关于Named Semaphores

## 操作

- The sem_open() function opens or creates a semaphore, initializes the semaphore
  if it is created by the call, and returns a handle for use in later calls.
- The sem_post(sem) and sem_wait(sem) functions respectively increment and decrement
  a semaphore’s value.
- The sem_getvalue() function retrieves a semaphore’s current value.
- The sem_close() function removes the calling process’s association with a semaphore
  that it previously opened.
- The sem_unlink() function removes a semaphore name and marks the semaphore
  for deletion when all processes have closed it.

### sem_open



```cpp
sem_t *sem_open(const char *name, int oflag, mode_t mode, ..., unsigned int value */ ); //Returns pointer to semaphore on success, or SEM_FAILED on error
```

#### 打开(创建)新的信号量

第二个参数是open flag，它是一个bit mask参数。

- 如果oflag为0，我们会访问一个已经存在的信号量。
- 如果oflag为 O_CREAT，系统会根据第一个参数(name)判断这个名称的信号量是否存在，如果不存在创建一个新的信号量。 即不存在则创建新的。
- 如果oflag为O_CREAT | O_EXCL, 就是两个条件都要满足进行创建。那么如果第一个参数name的信号量存在，sem_open会返回-1失败，原因是O_EXCL表示 exclusive 排他的。所以如果调用sem_open("xxx", O_CREAT | O_EXCL，...) == -1,表示这个信号量已经存在

#### 打开已经存在的信号量

sem_open打开一个已经存在的信号量，那么只需要两个参数(name, oflag)。如果O_CREAT存在在oflag中，那么还需要两个参数，就是mode 和value. 这个很明显么，你想创建一个新的信号量，那么你就要提供创建所需要的信息。(如果信号量存在的话，这两个参数会被忽略)

#### mode参数

mode也是一个bit mask的参数，这个bit value和打开文件的相似。一般来说一共有三种Open Mode，即 O_RDONLY, O_WRONLY, O_RDWR，这个前缀O表示Open。
在打开一个semaphore是需要进行部署sem_post和sem_wait，所以需要Read 和Write权限。

#### perms参数

Linux permissions 参数是以Linux中权限组来划分的，一般来说Linxu把所有用户分为 所有者USER, 群组GROUP， 其他人OTHERS 还有超级用户ROOT，但是ROOT这里一般不参与讨论。
这里直接用图表示更加直接。

![img](https://upload-images.jianshu.io/upload_images/3053840-37d37a8484b1e612.png?imageMogr2/auto-orient/strip|imageView2/2/w/1024/format/webp)

image.png

![img](https://upload-images.jianshu.io/upload_images/3053840-53bfcff2299f0aca.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

image.png

对于Liux的文件stat.h中的定义，首先可以参考这个[Permission-Bits](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.gnu.org%2Fsoftware%2Flibc%2Fmanual%2Fhtml_node%2FPermission-Bits.html)
常用的有 , 这里S前缀表示Set



- S_IRUSR 用户读权限，
- S_IWUSR 用户写权限
- 没有 用户读写权限
- S_IRWXU 用户读，写，执行权限

------

- S_IRGRP 群组读权限
- S_IWGRP 群组写权限
- S_IRWXG 群组读，写，执行权限

------

- S_IROTH 其他人读权限
- S_IWOTH 其他人写权限
- S_IRWXO 其他人读，写，执行权限

根据上图中的映射关系，S_IRUSR 表示二进制， 0b100000000, 这个数的十进制是 2^8 = 256， 在linux试一下跟预期一样
`printf("%d", S_IRUSR);`,输出256.

------

当然每一个权限是可以拆开的，比如用户的读，写，执行可以分别表示为 S_IRUSR S_IWUSR S_IXUSR,同理其他也一样。 事实上，在stat.h文件里面是这样定义的。



```cpp
//宏定义的基础定义，
#define __S_IREAD   0400    /* Read by owner.  */
#define __S_IWRITE  0200    /* Write by owner.  */
#define __S_IEXEC   0100    /* Execute by owner.  */
//用户
#define S_IRUSR __S_IREAD   /* Read by owner. 这个宏就是 0x 04  */
#define S_IWUSR __S_IWRITE  /* Write by owner.  */
#define S_IXUSR __S_IEXEC   /* Execute by owner.  */
#define S_IRWXU (__S_IREAD|__S_IWRITE|__S_IEXEC) /* Read, write, and execute by owner.  */
//群组
#define S_IRGRP (S_IRUSR >> 3)  /* Read by group.  */
#define S_IWGRP (S_IWUSR >> 3)  /* Write by group.  */
#define S_IXGRP (S_IXUSR >> 3)  /* Execute by group.  */
#define S_IRWXG (S_IRWXU >> 3)  /* Read, write, and execute by group.  */
//其他人
#define S_IROTH (S_IRGRP >> 3)  /* Read by others.  */
#define S_IWOTH (S_IWGRP >> 3)  /* Write by others.  */
#define S_IXOTH (S_IXGRP >> 3)  /* Execute by others.  */
#define S_IRWXO (S_IRWXG >> 3) /* Read, write, and execute by others.  */
```

可以看到，最基础的是 0400 这个数字，这个数字在C语言因为是以0开头，所以是八进制表示，它表示 4 * (8^2) = 256; 因此在赋值的时候也可以直接用数字0400赋值。
所以 S_IRUSR 表示 二进制 0b100000000, 八进制 0400, 十六进制 0x100,对于S_IRGRP 表示对于S_IRUSR 进行右移3位，即0b100000000变成 0b100000[000]最后三位去掉，变成0b000100000,这个数字是32. 如果变成S_IROTH,就是再向右移动3bit，变成0b100，即4. 所以右移也是整除2的操作，移动一次表示除以2. 其他可以类似推到

#### value

就是需要传给新的信号量的初始值，这个数字是非负整数。

#### 特别注意

如果sem_open失败，返回的是 叫做 `SEM_FAILED`的一个值，这个值可不是简单的-1.我们看一下Linux环境下的源代码



```cpp
#if __WORDSIZE == 64
# define __SIZEOF_SEM_T 32
#else
# define __SIZEOF_SEM_T 16
#endif

/* Value returned if `sem_open' failed.  */
#define SEM_FAILED      ((sem_t *) 0)

typedef union
{
  char __size[__SIZEOF_SEM_T];
  long int __align;
} sem_t;
```

sem_t是一个共用体，SEM_FAILED 是一个共用体指针。
SEM_FAILED 可能被定义为 *((sem_t *) 0) or ((sem_t *) –1)*, Linux中是以 (sem_t *) 0) 定义的

### sem_close



```cpp
//Returns 0 on success, or –1 on error
int sem_close(sem_t *sem); 
```

这个是关闭信号量，并没有删除它，如果要删除，需要调用sem_unlink()函数

### sem_wait

如果信号量当前的值是大于0，sem_wait会立即返回。
如果信号量的值就是0，sem_wait会阻塞. 直到这个值增加超过0，然后sem_wait会立即返回，就是说，这个函数是检测信号量的值，如果信号量为正整数，表示这个资源可以被访问，因此直接通过，如果为0则阻塞， 表示没有资源可以被访问。

## sem_trywait

sem_trywait函数是sem_wait的非阻塞版本。

### sem_post

信号量的值加1

### sem_getvalue

`int sem_getvalue(sem_t *sem, int *sval);`
获取当前信号量

------

# Unnamed Semaphores 非命名信号量

非命名信号量使用相同的函数, sem_wait(), sem_post(), sem_getvalue()
两个不一样的函数sem_init 和 sem_destroy()

## sem_init



```cpp
//Returns 0 on success, or –1 on error, 这个和sem_open不一样，sem_open函数 如果失败返回的是SEM_FAILED,不是-1
int sem_init(sem_t *sem, int pshared, unsigned int value);
```



```csharp
- If pshared is 0, then the semaphore is to be shared between the threads of the
calling process. In this case, sem is typically specified as the address of either a
global variable or a variable allocated on the heap. A thread-shared semaphore
has process persistence; it is destroyed when the process terminates.
- If pshared is nonzero, then the semaphore is to be shared between processes. In
this case, sem must be the address of a location in a region of shared memory (a
POSIX shared memory object, a shared mapping created using mmap(), or a
System V shared memory segment). The semaphore persists as long as the
shared memory in which it resides. (The shared memory regions created by
most of these techniques have kernel persistence. The exception is shared
anonymous mappings, which persist only as long as at least one process maintains
the mapping.) Since a child produced via fork() inherits its parent’s memory
mappings, process-shared semaphores are inherited by the child of a fork(), and
the parent and child can use these semaphores to synchronize their actions.
```

这里可知，

1. pshared 这个参数如果是0，是被线程进行共享。sem这个参数指向一个全局变量或者分配在heap(堆)上的一个变量，这里就说明，这个变量大概率是通过malloc分配的
   为非负数，因为linux c里面没有c++ new这样的操作符。找了个例子.



```cpp
#include <stdlib.h>
int main()
{
  int 1 = 1;
  int *pi = (int*)malloc(sizeof(int)); //this variable pi is allocated at heap region
  *pi = 2;
  //...
  free(pi);
}
```

并且这个信号量在进程期间都会存在，只有当进程结束才会被释放。

1. pshared 为非负整数，信号量被进程共享。sem指向共享内存(POSIX shared memory object， a shared mapping created using mmap(), or a
   System V shared memory segment)。