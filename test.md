```makefile
modname?=demo
KERNELDIR:=/home/linux/linux-5.10.61
all:
	make -C $(KERNELDIR) M=$(PWD) modules
clean:
	make -C $(KERNELDIR) M=$(PWD) clean
obj-m:=$(modname).o
```

```c
#include <linux/init.h>
#include <linux/module.h>

char* demo_str = "hello";
module_param(demo_str, charp, 0644);
MODULE_PARM_DESC(demo_str, "demo str");

static int __init demo_init(){
    printk(KERN_INFO "demo_init\n"); // 打印信息
    return 0;
}
static void __exit demo_exit(){
    printk(KERN_INFO "demo_exit\n"); // 打印信息
}

module_init(demo_init);
module_exit(demo_exit);
MODULE_LICENSE("GPL");
```

```shell
FSMP1A> pri bootcmd
bootcmd=tftp 0xc2000000 uImage; tftp 0xc4000000 stm32mp157a-fsmp1a.dtb;bootm 0xc2000000 - 0xc4000000
FSMP1A> pri bootargs
bootargs=root=/dev/nfs nfsroot=192.168.1.210:/home/linux/rootfs,tcp,v4 rw console=ttySTM0,115200 init=/linuxrc ip=192.168.1.10
```

```c
#define KERN_ERR  "3" /* error conditions */
#define KERN_WARNING  "4" /* warning conditions */
#define KERN_NOTICE  "5" /* normal but significant condition */
#define KERN_INFO  "6" /* informational */
#define KERN_DEBUG  "7" /* debug-level messages */
```

SGI：软

PPI：私有

SPI：共享
