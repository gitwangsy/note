## 1.获取文件属性

### stat结构体

```c
struct stat{
    dev_t st_dev;	//磁盘设备号 主次磁盘设备号函数：major(st_dev)  minor(st_dev)
    ino_t st_ino;	//inode号(文件编号)，文件系统识别文件的唯一编号（命令：ls -i）
    mode_t  st_mode;	//文件的类型和权限  （查看命令：man 7 inode）（第7节）
    nlink_t   st_nlink;       //硬链接数  
    uid_t     st_uid;         //用户id   （id 用户名） （/etc/passwd 文件存放用户信息）
    gid_t     st_gid;         //组id
    dev_t     st_rdev;       //设备号，linux内核识别设备驱动的唯一的编号
   off_t     st_size;        //文件大小，单位是字节  1023
   blksize_t st_blksize;     //一个块的大小（常见512字节）  512
   blkcnt_t  st_blocks;      //块的个数
   struct timespec st_atim;  /* Time of last access */
   struct timespec st_mtim;  /* Time of last modification */
   struct timespec st_ctim;  /* Time of last status change */
}
    
//在st_mode中bit12-bit15代表的是文件的类型
if((st_mode & 0170000) == 0100000){
    printf("普通文件");
}
//宏定义
    S_IFMT     0170000   bit mask for the file type bit field(文件类型掩码)
    S_IFSOCK   0140000   socket
    S_IFLNK    0120000   symbolic link
    S_IFREG    0100000   regular file
    S_IFBLK    0060000   block device
    S_IFDIR    0040000   directory
    S_IFCHR    0020000   character device
    S_IFIFO    0010000   FIFO(管道文件)
linux 文件类型：(7种)
    - : 普通文件(regular file)
    d : 目录文件(directory)
    l : 软链接文件(symbolic link)
    b: 块设备文件(block device)
    c: 字符设备文件(character device)(有关系统调用)
    p: 管道文件(pipe)(用于进程间通信)(命令：mkfifo filename 创建管道文件)
    s: 套接字文件(socket)(用于网络通信)
        
//在st_mode中bit0-bit8代表的是文件的权限(9位)
文件权限 = st_mode & 0777
        
// 设备号（32bit） = 主设备号（高12）+次设备号（低20）
主设备号 = devno >> 20； (取高位)
次设备号 = devno & (1<< 20 - 1);  (取低位)
主设备号：它是哪一类设备
次设备号：同类中哪个设备
```

### stat()

```c
int stat(const char *pathname, struct stat *statbuf);
//如果想要获取软连接文件属性使用lstat,用法和stat一样
功能：获取文件的属性
参数：
    @pathname:文件的路径及名字 "/usr/include/head.h"
    @statbuf:获取到的属性结构体
返回值：成功返回0，失败返回-1置位错误码
```

## 2.获取用户名和组名

### getpwuid()/getgrgid()

```c
struct passwd *getpwuid(uid_t uid);
功能：根据uid获取用户的信息
参数：
    @uid:用户的id
返回值：成功passwd的结构体指针，失败返回NULL置位错误码
       struct passwd {
           char   *pw_name;       //用户名
           char   *pw_passwd;     //用户的密码
           uid_t   pw_uid;        //uid
           gid_t   pw_gid;        //gid
           char   *pw_gecos;      //用户信息
           char   *pw_dir;        //用户主目录 /home/linux
           char   *pw_shell;      //命令行解析器
       };

struct group *getgrgid(gid_t gid);
功能：根据gid获取组信息
参数：
    @gid:组号
返回值：成功返回group结构体指针，失败返回NULL,置位错误码
   struct group {
       char   *gr_name;        //组名
       char   *gr_passwd;      //组的密码
       gid_t   gr_gid;         //组id
   };
```

## 3.目录操作

### opendir()

```c
DIR *opendir(const char *name);
功能：打开目录
参数：
    @name:目录名
返回值：成功返回DIR的结构体指针，失败返回NULL置位错误码
```

### readdir()

```c
struct dirent *readdir(DIR *dirp);
功能：读取目录下的文件
参数：
    @dirp：目录的指针
返回值：成功返回dirent结构体指针，失败返回的NULL，置位错误码
struct dirent {
   char           d_name[256]; //文件名
   ino_t          d_ino;      //inode号
   unsigned short d_reclen;   //结构体大小
   unsigned char  d_type;     //文件类型
          DT_BLK      This is a block device.
          DT_CHR      This is a character device.
          DT_DIR      This is a directory.
          DT_FIFO     This is a named pipe (FIFO).
          DT_LNK      This is a symbolic link.
          DT_REG      This is a regular file.
          DT_SOCK     This is a UNIX domain socket.
};
```

### chdir()

```c
int chdir(const char *path);
功能：切换目录
参数：
    @path:路径
返回值：成功返回0，失败返回-1置位错误码
```

### closedir()

```c
int closedir(DIR *dirp);
功能：关闭目录
参数：
    @dirp：目录的指针
返回值：成功返回0，失败返回-1置位错误码
```

## 4. 库的制作及使用

windows库：
	xxx.dll 动态库

​	xxx.lib 静态库

linux库:

​	libxxx.so :动态库（lib前缀，xxx库名，.so后缀）

​	libxxx.a :静态库 （lib前缀，xxx库名，.a后缀）

### 静态库

> 静态库它的后缀格式是.a，静态库是由不包含main函数的xxx.c文件编译而来的。
> 在使用静态库的时候会将用户程序和库文件中的函数的实现编译生成一个可执行程序，这个**可执行程序的体积比较大，但是效率比较高。缺点就是更新麻烦。**

```shell
# -L :指定库的路径
# -l :指定链接的库名
# -I(大i) :指定头文件路径
# 创建
gcc -c ./src/linklist.c  -o linklist.o -I ./inc/
ar -cr liblinklist.a linkkist.o
# 使用
gcc ./src/main.c -L ./lib/ -llinklist -I inc/ -o ./bin/a.out
```

### 动态库

> 动态库它的后缀格式是.so，动态库是由不包含main函数的xxx.c文件编译而来的。
>
> 在使用动态库的时候会将用户程序和库文件中的函数的符号表编译生成一个可执行
>
> 程序，这个**可执行程序的体积比较小，更新容易，缺点效率比较低。动态库可以被**
>
> **多个程序共享使用，所以动态库又名共享库。**

```shell
# -fPIC      忽略文件的位置
# -shared  生成的库是共享库
# 创建
gcc -fPIC -shared ./src/linklist.c -o ./lib/liblinklist.so -I ./inc/
# 使用
gcc ./src/main.c -L ./lib/ -llinklist -I ./inc/ -o ./bin/a.out
sudo vi /etc/ld.so.conf.d/libc.conf
sudo ldconfig
```

