```c
// 标准IO
fopen(2) fclose(1) perror(1) strerror(1) fgetc(1) fputc(2) fgets(3) fputs(2) fread(4) fwrite(4) feof(1) ferror(1) fseek(3) rewind(1) ftell(1)  15
// 接收数据的数组记得清零
// fopen失败返回NULL，置位错误码
// fgetc()失败返回EOF(-1)
// fgets()失败返回NULL
// fread()读取与否使用 while(!(feof(sfp) || ferror(sfp))) 判断
//    成功ret接收读取到的项目数，用于fwrite()写入
feof(fp); //如果这个函数返回真表示光标到了文件的结尾
ferror(fp);//如果这个函数返回真表示读的时候发生了错误
while (!feof(sfp) && !ferror(sfp)) { // !(feof(fp) || ferror(fp))
        ret = fread(buf, 1, sizeof(sfp), sfp);
        fwrite(buf, ret, 1, dfp);
}
// rewind() 将光标定位到开头
// ftell() 返回光标到文件开头的字节数
 
// 格式化函数、时间函数
sprintf(3n) snprintf(4n) fprintf(3n) time(NULL) localtime(1)  5
// 时间
time_t ts;//time_t是long int类型
struct tm *tm;//tm结构体
//1.获取系统时间
if((ts=time(NULL))==(time_t)-1)
    PRINT_ERR("get time error");
//2.将系统时间转换为本地时间
if((tm=localtime(&ts))==NULL)
    PRINT_ERR("localtime error");

// 文件IO
open read write close lseek  5

stat lstat getpwuid getgrgid 4

opendir readdir closedir  mkdir  4

fork getpid getppid exit _exit wait waitpid setsid chdir umask dup dup2 system 13

pthread_create pthread_self pthread_exit pthread_join pthread_detach pthread_cancel

pthread_cancelsetstate pthread_cancelsettype  8

pthread_mutext_init pthread_mutex_lock pthread_mutex_trylock pthread_mutex_unlock 

pthread_mutext_destroy 5

sem_init  sem_wait sem_post sem_destroy 4

pthread_cond_init pthread_cond_wait pthread_cond_signal pthread_cond_broadcast pthread_cond_destroy 5

pipe mkfifo signal kill raise alarm 6

msgget msgsnd msgrcv msgctl 4

shmget shmat shmdt shmctl 4

semget semctl semop 3
```

```c
// 将时间结构体转换为字符串
if (!strftime(timet, sizeof(timet), "%Y-%m-%d %H:%M:%S", tm)) // 默认补零对齐
	PRINT_ERR("strftime error");
```

