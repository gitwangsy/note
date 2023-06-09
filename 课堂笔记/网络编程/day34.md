## 1. 服务器模型

> 服务器模型分为两种：循环服务器、并发服务器
>
> **循环服务器：同一时刻只能处理一个客户端的请求**
>
> **并发服务器：可以同时处理多个客户端的请求**
>
> TCP服务器默认的就是一个循环服务器，因为他有两个阻塞的函数 accept recv 相互影响
>
> UDP服务器默认的就是一个并发服务器，因为只有一个阻塞的函数 recvfrom 

### TCP并发服务器

#### 多线程实现

> 主线程负责accept接收客户端的连接请求，一旦有客户端连接成功就创建一个子线程，专门用于和当前客户端通信

```c
typedef struct{
    int acceptfd;
    struct sockaddr_in client_addr;
} client_msg;

void* thread_func(void* arg){
    client_msg client = *(client_msg*)arg;
    printf("client ip:%s,port:%d\n",inet_ntoa(client.client_addr.sin_addr),
           			ntohs(client.client_addr.sin_port));
    char buf[128]={0};
    int ret;
    while(1){
        memset(buf,0,sizeof(buf));
        if(-1==(ret=recv(client.acceptfd,buf,sizeof(128),0))){
            perror("recv error");
            pthread_exit(-1);
        }else if(0==ret){
            printf("client close\n");
            break;
        }
        if(!strncasecmp("quit",buf,4)){
            printf("client quit\n");
            bread;
        }
        printf("client say:%s\n",buf);
        memset(buf,0,sizeof(buf));
        memcpy(buf,"ok",2);
        if(-1==send(client.acceptfd,buf,strlen(buf),0)){
            perror("send error");
            pthread_exit(-1);
        }
    }
    close(client.acceptfd);
    pthread_exit(NULL);
}
int main(int argc,const char *argv[]){
    // 检查参数
    if(argc != 3){
        printf("Usage: %s <IP> <PORT>\n",argv[0]);
        return -1;
    }
    // 1.创建流式套接字
    int sockfd = socket(AF_INET,SOCK_STREAM,0);
    if(-1==sockfd)
        PRINT_ERR("socket error");
    // 2.绑定网络信息结构体
    struct sockaddr_in server_addr={
        .sin_family=AF_INET;
        .sin_port=htons(atoi(argv[2]));
        .sin_addr.s_addr=inet_addr(argv[1]);
    }
    if(-1==bind(sockfd,(struct sockaddr*)&server_addr,sizeof(server_addr)))
        PRINT_ERR("bind error");
    // 3.监听
    if(-1==listen(sockfd,10))
        PRINT_ERR("listen error");
    // 4.循环接收客户端连接
    struct sockaddr_in client_addr;
    socklen_t len;
    int acceptfd,ret;
    pthread_t tid;
    client_msg client;
    while(1){
        if(-1==(acceptfd=accept(sockfd,(struct sockaddr*)&client_addr,&len))
            PRINT_ERR("accept error"); 
        client.acceptfd=acceptfd;
        client.client_addr=client_addr;
        if (0 != (ret = pthread_create(&tid, NULL, thread_func, &client))) {
            printf("pthread_create error: %s\n", strerror(ret));
            close(acceptfd);
            return -1;
        }
        if (0 != (ret = pthread_detach(tid))) {
            printf("pthread_create error: %s\n", strerror(ret));
            close(acceptfd);
            return -1;
        }
    }
    close(sockfd);
    
    return 0;
}
```

#### 多进程实现

> 父进程负责accept等待客户端连接，一旦有客户端连接就创建一个子进程，专门用于和当前客户端通信

```c
void sig_handler(int signo){
	wait(NULL);
}
int main(int argc ,const char* argv[]){
	//判断参数
	if(argc!=3){
		printf("Usage:%s <IP> <PROT>\n",argv[0]);
		return -1;
	}
	// 1.创建流式套接字
	int sockfd = socket(AF_INET,SOCK_STREAM,0);
	if(-1==sockfd)
		PRINT_ERR("socket error");
	// 2.绑定
	struct sockaddr_in server_addr={
		.sin_family = AF_INET;
		.sin_port = htons(atoi(argv[2]));
		.sin_addr.s_addr = inet_addr(argv[1]);
	};
	if(-1==bind(sockfd,(struct sockaddr*)&server_addr,sizeof(server_addr)))
		PRINT_ERR("bind error");
	// 3.监听
	if(-1==listen(sockfd,10))
		PRINT_ERR("listen error");
	// 4.循环接收客户端连接
	int acceptfd,ret;
	pid_t pid;
	char buf[128]={0};
	
	if(signal(SIGUSR1,sig_handler)==SIG_ERR)
		PRINT_ERR("signal error");
	while(1){
		if(-1==(acceptfd=accept(sockfd,NULL,NULL)))
			PRINT_ERR("accept error");
		// 5.创建子进程
		if(-1==(pid=fork()))
			PRINT_ERR(fork error);
		if(0==pid){// 6.子进程通信
			close(sockfd);
			while(1){
				memset(buf,0,sizeof(buf));
				if(-1==(ret=recv(acceptfd,buf,sizeof(buf),0))){
					perror("recv error");
					exit(-1);
				}else if(0==ret){
					printf("client close\n");
					break;
				}
				if(!strncasecmp("quit",buf,4)){
					printf("client quit\n");
					break;
				}
				printf("client say:%s\n",buf);
				memset(buf,0,sizeof(buf));
				memcpy(buf,"ok",3);
				if(-1==send(acceptfd,buf,strlen(buf),0)){
					perror("send error");
					exit(-1);
				}
			}
			close(acceptfd);
			kill(getppid(),SIGUSR1);
			exit(0);
		}else{// 7.父进程
			close(acceptfd);
		}	
	}
	close(sockfd);
	
	return 0;		
}
```

#### 多路IO复用

>TCP的服务器中有两个阻塞的函数  accept  recv
>也就是说，由于**文件描述符 sockfd  acceptfd的缓冲区**没有内容，就会导致阻塞
>也就是说，accept  recv相互影响本质就是  sockfd 和多个客户端的 acceptfd 相互影响
>所以，可以使用多路IO复用来解决。

```c
int main(int argc,const char* argv[]){
	if(argc!=3){
		printf("Usage:%s <IP> <PORT>\n",argv[0]);
		return -1;
	}
	int sockfd=socket(AF_INET,SOCK_STREAM,0);
	if(-1==sockfd)
		PRINT_ERR("socket error");
	struct sockaddr_in addr={
		.sin_famliy=AF_INET;
		.sin_port=htons(atoi(agrv[2]));
		.sin_addr.s_addr=inet_addr(argv[1]);
	};
	if(-1==bind(sockfd,(struct sockaddr*)&addr,sizeof(addr)))
		PRINT_ERR("bind error");
	if(-1==listen(sockfd,10))
		PRINT_ERR("listen error");
	// 创建文件描述符集合
	fd_set readfds,readfds_temp;
	FD_ZERO(&readfds);
	FD_ZERO(&readfds_temp);
	FD_SET(sockfd,&readfds);
	int maxfd=sockfd;
	
	// 循环接收客户端连接
	int acceptfd,ret,num,i;
	struct sockaddr_in client_addr;
	socklen_t len;
	char buf[128]={0};
	while(1){
		readfds_temp=readfds;
		ret = select(maxfd+1,&readfds_temp,NULL,NULL,NULL);
		for(i=0;i<maxfd+1 && ret>0;i++){
			if(FD_ISSET(i,&readfds_temp)){
				if(i==sockfd){
					if(-1==(acceptfd=accept(i,(struct sockaddr*)&client_addr,&len)))
						PRINT_ERR(accept error);
					printf("client ip: %s, port: %d\n", inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));
                    FD_SET(acceptfd,&readfds);
					maxfd=acceptfd>maxfd?acceptfd:maxfd;
				}else{
					memset(buf,0,sizeof(buf));
					if(-1==(num=recv(i,buf,sizeof(buf),0))){
						PRINT_ERR("recv eror");
					}else if(0==num){
						printf("client close\n");
						close(i);
						FD_CLR(i,&readfds);
						continue;
					}
					if(!strncasecmp("quit",buf,4)){
						printf("client quit\n");
						close(i);
						FD_CLR(i,&readfds);
						continue;
					}
					printf("client ip: %s, port: %d, msg: %s\n", inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port), buf);
                    memcpy(buf, "OK", 3);
                    if(-1==send(i, buf, strlen(buf)+1, 0))
						PRINT_ERR("send error");
				}
				ret--;
			}
		}
	}
	close(sockfd);

	return 0;
}
```

## 2. 网络超时检测