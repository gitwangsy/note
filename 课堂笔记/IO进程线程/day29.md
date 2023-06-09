## 1. 多线程

### 线程概念

> 多线程（LWP）：线程是进程的更小粒度的单元，**进程是分配资源的最小单位，而线程是CPU调度的最小单位**，线程共享进程的资源。线程几乎不占用资源，只是占用了很少的程序执行的状态的资源（8k）。线程由于共用进程的资源，所以多线程没有多进程安全。多线程的切换要比多进程的切换开销小。在一个进程内至少会有一个线程（主线程）。[线程的所有的函数都调用的是第三方库libpthread.so](http://xn--libpthread-ji2pp39a1h5bl0k0tlo7aj9cbwgr46gzxfcad578rpggj3y702f1c2a.so/)，所以在使用前需要先安装线程函数的man手册。`sudo apt-get install manpages-posix manpages-posix-dev`

### 线程执行的先后顺序

**多线程执行没有先后顺序，采用的规则就是时间片轮询，上下文切换**

### 线程的内存空间文件

**多线程共享进程的资源，一个线程对全局变量的修改会影响另外一个线程**

### pthread_create()

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                  void *(*start_routine) (void *), void *arg);
功能：创建线程
参数：
    @thread:创建线程的线程号
    @attr:线程的属性，一般填写为NULL
        //https://docs.oracle.com/cd/E19253-01/819-7051/attrib-78961/index.html
    @start_routine:指向线程体（指向线程处理函数）
    @arg:向线程体传递的参数
返回值：成功返回0，失败返回错误码
```

```shell
gcc xxx.c -lpthread -lc
```

### pthread_self()

```c
pthread_t pthread_self(void);
功能：获取当前的线程号
参数：
    @无
返回值：总是会成功返回调用线程的线程号
```

### pthread_exit()

```c
void pthread_exit(void *retval);
功能：退出调用此函数的线程
参数：
    @retval：线程退出时的返回值
返回值：无
```

### pthread_join()

```c
int pthread_join(pthread_t thread, void **retval);
功能：阻塞等待子线程结束，并为其回收资源（结合态）
参数：
    @thread:线程号
	@retval:接收pthread_exit退出状态
返回值：成功返回0，失败返回错误码
```

### pthread_detach()

> 多线程有两种状态：结合态，分离态。使用线程默认属性创建的线程就是结合态
> 如果结合态的线程调用pthread_detach函数可以将线程设置为分离态，结合态的线程需要使用pthread_join来回收资源，而分离态的线程结束之后资源会被自动回收。

```c
int pthread_detach(pthread_t thread);
功能：将线程标记为分离态
参数：
    @thread:线程号
返回值：成功返回0，失败返回错误码
```

### pthread_cancel()

```c
int pthread_cancel(pthread_t thread);
功能：给线程发送取消的信号
参数：
    @thread:线程号
返回值：成功返回0，失败返回错误码
```

### pthread_setcancelstate()

```c
int pthread_setcancelstate(int state, int *oldstate);
功能：设置线程是否可被取消
参数：
    @state:
	PTHREAD_CANCEL_ENABLE ：线程可被取消
         PTHREAD_CANCEL_DISABLE：线程不可被取消
    @oldstate:返回设置前的状态
返回值：成功返回0，失败返回错误码
```

### pthread_setcanceltype()

```c
int pthread_setcanceltype(int type, int *oldtype);
功能：设置线程是否被取消的类型
参数：
    @type:类型
        PTHREAD_CANCEL_DEFERRED：延时取消（遇到取消点在取消（取消点：比如系统调用返回用户空间时））
        PTHREAD_CANCEL_ASYNCHRONOUS：异步取消（立即取消）
    @oldtype:返回设置前的类型
返回值：成功返回0，失败返回错误码
```

## 2. 多线程同步互斥

> 互斥：表示线程间争抢同一个资源的过程，不能保证哪个线程争抢到资源，但是最终只能有一个线程获取到资源。
> 同步：提前编排好线程的执行顺序，让他们顺序执行，这样就不会有同时访问的情况。同步一般用在**生产者和消费者模型**上。

### 线程互斥锁

```c
// 1.定义互斥锁
pthread_mutex_t mutex;

// 2.初始化线程互斥锁
//静态初始化
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
//动态初始化
int pthread_mutex_init(pthread_mutex_t * mutex,
                       				 const pthread_mutexattr_t * attr);
功能：初始化互斥锁
参数：
    @mutex:被初始化的锁
    @attr:锁的属性，一般填写为NULL(默认属性)
返回值：成功返回0，失败返回错误码
        
// 3.上锁
//尝试获取锁，如果锁资源存在那就占用锁，如果锁资源不可利用，立即返回。
int pthread_mutex_trylock(pthread_mutex_t *mutex);
//获取锁
int pthread_mutex_lock(pthread_mutex_t *mutex);
功能：上锁（如果线程获取不到锁的资源，线程阻塞，直到其他的线程将锁释放）
参数：
      @mutex:执行锁的指针
返回值：成功返回0，失败返回错误码
       
// 4.解锁
int pthread_mutex_unlock(pthread_mutex_t *mutex);
功能：解锁
参数：
      @mutex:执行锁的指针
 返回值：成功返回0，失败返回错误码 
          
// 5.销毁锁
int pthread_mutex_destroy(pthread_mutex_t *mutex);
功能：销毁互斥锁
参数：
    @mutex:执行锁的指针
返回值：成功返回0，失败返回错误码
```

### 线程互斥锁死锁问题

> 1. 产生死锁的四个必要条件互斥，请求保持，不可剥夺，循环等待
>
> 2. 死锁的规避方
> 	1. 指定线程获取锁的顺序
> 	2. 尽量避免锁的嵌套使用
> 	3. 给线程上锁指定超时时间（pthread_mutex_timedlock())
> 	4. 在全局位置指定锁是否被使用的状态，如果被使用就不再获取

### 线程同步之无名信号量

```c
// 1. 定义无名信号量
sem_t sem;

// 2. 初始化无名信号量
int sem_init(sem_t *sem, int pshared, unsigned int value);
功能：初始化无名信号量
参数：
    @sem:指向无名信号量的指针
    @pshared:0 线程的同步
           		1 进程的同步（亲缘关系进程）
    @value:信号的初值  1
返回值：成功返回0，失败返回-1置位错误码
        
// 3.获取信号量(P操作)
int sem_wait(sem_t *sem);
功能：申请资源（让信号量的值减去1，然后和0比较如果结果为0，表示获取锁成功了）    
    如果在调用sem_wait的时候获取不到资源，sem_wait会阻塞
参数：
    @sem:指向无名信号量的指针
返回值：成功返回0，失败返回-1置位错误码
        
// 4.释放信号量(V操作)
int sem_post(sem_t *sem);
功能：释放资源
参数：
    @sem:指向无名信号量的指针
返回值：成功返回0，失败返回-1置位错误码
        
// 5.销毁无名信号量
int sem_destroy(sem_t *sem);
功能：销毁无名信号量
参数：
    @sem:指向无名信号量的指针
返回值：成功返回0，失败返回-1置位错误码
```

### 线程同步之条件变量

> 条件变量也是实现多线程间的同步机制的，无名信号量适合使用在线程个数比较少的线程中实现同步。而条件变量适合用在多线程的同步机制，定义一个条件变量即可，不需要像 无名信号量一样随着线程数的增多而增多。

```c
// 1.定义条件变量
pthread_cond_t cond;

// 2.初始化条件变量
//静态初始化	
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
//动态初始化
int pthread_cond_init(pthread_cond_t * cond,const pthread_condattr_t * attr);
功能：动态初始化一个条件变量
参数：
   @cond：条件变量的指针
   @attr:NULL使用默认属性
返回值：成功返回0，失败返回非0

// 3.阻塞等待条件变量
int pthread_cond_wait(pthread_cond_t * cond,pthread_mutex_t * mutex);
功能：阻塞等待条件变量，在条件变量中维护了一个队列，这里的互斥锁就是为
    了解决在往队列中放线程的时候出现竞态问题的。
使用的步骤：
    1.使用pthread_mutex_lock上锁
    2.调用pthread_cond_wait
        2.1将当前线程放入队列
        2.2休眠
        2.3解锁
        2.4获取锁
        2.5休眠状态退出
    3.你的程序
    4.使用pthread_mutex_unlock解锁
参数：
    @cond:条件变量的地址
    @mutex:互斥锁
返回值：成功返回0，失败返回非零
        
// 4.给休眠的线程发信号或者广播
int pthread_cond_signal(pthread_cond_t *cond);
功能：唤醒(至少)一个休眠的线程
参数：
    @cond:条件变量的地址
返回值：成功返回0，失败返回非零
int pthread_cond_broadcast(pthread_cond_t *cond);
功能：唤醒所有休眠的线程
参数：
    @cond:条件变量的地址
返回值：成功返回0，失败返回非零     
       
// 5.销毁条件变量     
int pthread_cond_destroy(pthread_cond_t *cond);
功能：销毁条件变量
参数：
     @cond:条件变量的地址
返回值：成功返回0，失败返回非零
```
