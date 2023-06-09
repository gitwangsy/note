## 1. 进程相关函数

### getpid()/getppid()

```c
pid_t getpid(void);
功能：获取当前进程的进程号
参数：
    @无
返回值：总是会成功，成功返回pid
    
pid_t getppid(void);
功能：获取父进程的进程号
参数：
    @无
返回值：总是会成功，成功返回ppid
```

### exit()/_exit()

```c
void exit(int status);
功能：结束当前的进程（它是库函数，会刷新缓冲区）
参数：
    @status:进程结束的状态，将这个结果返回给父进程(终端可访问)，只有0-255区间的值可以返回，
         如果不是0-255区间的数stats&0377,转换到
         0-255之间的数，然后再返回
   //EXIT_FAILURE	1	/* Failing exit status.  */
   //EXIT_SUCCESS	0	/* Successful exit status.  */
返回值：无
         
void _exit(int status);
功能：结束当前的进程（它是系统调用，不会刷新缓冲区）
参数：
    @status:进程结束的状态，将这个结果返回给
         父进程，只有0-255区间的值可以返回，
         如果不是0-255区间的数stats&0377,转换到
         0-255之间的数，然后再返回
   //EXIT_FAILURE	1	/* Failing exit status.  */
   //EXIT_SUCCESS	0	/* Successful exit status.  */
返回值：无
```

### wait()/waitpid()

```c
pid_t wait(int *wstatus);
功能：用来阻塞回收子进程的资源,哪个子进程先结束wait就回收哪个进程的资源
参数：
    @wstatus:收到子进程退出的状态
返回值：成功返回回收掉的子进程的pid，失败返回-1置位错误码
WIFEXITED(wstatus)  //当子进程正常退出时候返回值
WEXITSTATUS(wstatus) //获取wstatus中8-15这8个bit位，代表子进程退出状态
WIFSIGNALED(wstatus)//当子进程被信号终止的时候它返回值
WTERMSIG(wstatus)   //获取wstatus中0-6这7个bit位，让子进程退出时候的信号号
         
pid_t waitpid(pid_t pid, int *wstatus, int options);
功能：可以指定回收进程的pid
参数：
 @pid:想要回收进程的pid
        < -1 ,首先对pid取绝对值，回收和绝对值相等的同组的进程的资源
 	 = 0  ,回收和调用进程同组的任意子进程的资源，如果子进程不和父进程同组了，
           父进程无法回收当前进程的资源
        =-1  ,回收任意子进程的资源，不管当前进程是否和父进程同组，父进程都能回
           收它的资源
        >0   ,回收指定进程号pid进程的资源
 @wstatus:进程退出状态指针和wait参数一样
 @options: 0阻塞回收，WNOHANG非阻塞回收
返回值：成功返回回收掉的进程的pid,如果是非阻塞方式没有回收掉子进程的资源返回0失败返回-1置位错误码
     
//都只能回收一个
```

## 2. 守护进程的创建

### 什么是守护进程

> 守护进程：守护进程是一个后台进程，守护进程不会随着终端的终止而终止。
可以**随着系统**的启动而启动，随着系统的终止而终止。类似windows上各种服务。

### 守护进程创建

```c
// 1. 创建孤儿进程
// 父进程退出，此时子进程就是孤儿进程(直接在else退出(exit(0)))
// 2. 设置孤儿进程的会话ID和组ID
pid_t setsid(void);
功能：将当前孤儿进程的会话id和组id都修改成pid的值
参数：
    无
返回值：成功返回会话id,失败返回-1，置位错误码
// 3. 切换到日志目录文件
int chdir(const char *path);
功能：切换目录
参数：
    @path:路径
返回值：成功返回0，失败返回-1置位错误码
// 4. 修改umask值(umask(0);)
mode_t umask(mode_t mask);
功能：修改文件的掩码
参数：
    @mask：掩码值
返回值：总是会成功，返回先前的掩码值
// 5. 创建日志文件
int fd = open("daemon.log",O_RDWR|O_CREAT|O_APPEND,0666);
// 6. 文件描述符重定向（实现日志记录）（同一文件可以被多个文件描述符指向）
dup2(fd, 0);
dup2(fd, 1);
dup2(fd, 2);
// 7. 开始自己的服务
while (1){
    printf("我正在测试守护进程\n");
    //重定向的日志文件是全缓冲文件，所以需要刷新，否则达到4K才刷新
    fflush(stdout); 
    sleep(1);
}
```

## 3. 进程内程序的替换

### 终端是如何执行a.out程序

> 终端是一个进程bash，如果它要执行a.out的话，首先它需要使用fork创建出来一个子进程，它会使用a.out可执行程序替换掉原理的可执行程序。

### system函数的使用

```c
int system(const char *command);
功能：在c程序中执行一条命令（fork创建一个进程(shell)，在进程中执行这条命令）
参数：
    @command:命令  "ls -l"
返回值：
       1.如果command为NULL，如果shell可用，则为非零值，如果没有shell可用，则为0。
       2.如果无法创建子进程，或者无法检索其子进程的状态，则返回值为-1。
       3.如果一个shell不能在子进程中执行，那么返回值就好像子进程通过调用_exit(2)终止，状态为127。
       4.如果所有系统调用都成功，则返回值是用于执行命令的子shell的终止状态。
        (shell的终止状态是它执行的最后一个命令的终止状态。)
       5.在最后两种情况下，返回值是一个“wstatus”，
        可以使用waitpid(2)中描述的宏来检查它。
        总结：成功返回0，失败返回-1
```

