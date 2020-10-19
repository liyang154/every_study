# 进程相关概念

## 1、程序和进程

​	**程序:**编译好的二进制文件，在磁盘上，不占用系统资源（CPU、内存、打开的文件、设备、锁.......）

​	**进程:**是一个抽象的概念，与操作系统原理联系紧密。进程是活跃的程序，占用系统资源，在内存中执行。

## 2、CPU和MMU

​	**MMU**:内存管理单元；负责处理CPU的内存访问请求的计算机硬件。位于CPU内部。

​	**功能：**虚拟地址到物理地址的转换、内存保护、cache的控制。

## 3、堆区栈区

​	堆的生长方向：从下往上

​	栈的生长方向：从上往下

## 4、进程控制块PCB

​	每个进程在内核中都有一个进程控制块（PCB）来维护进程相关的信息，linux内核的进程控制块是task_struct结构体。

**重点成员：**

- ​    进程id:系统中没哦一个进程有唯一的id. 非负整数。

- ​	进程的状态：就绪、运行、挂起、停止

- ​	进程切换时需要保存和恢复的一些CPU寄存器值。
- ​	描述虚拟地址空间的信息（虚拟地址空间与物理地址空间的对应关系）
- ​	描述控制终端的信息。
- ​	当前工作目录。
- ​	umask掩码--文件控制权限
- ​	文件描述符表
- ​	和信号相关的信息
- ​	用户id和组id
- ​	回话和进程组
- ​	进程可以使用的资源上限（查看资源上限命令：ulimit -a）

## 5、环境变量

#### **引入环境变量表：**

​		extern  char  ** environ

#### **getenv函数：**

​		**功能：**获取环境变量值

​		**函数原型:**char *getenv(const char *name)

​		**返回值:**成功返回环境变量的值；失败NULL

#### setenv函数：

​	设置环境变量的值

​	int setenv(const char *name ,const char *value ,int overwrite);		成功：0；失败：-1

​	overwrite:1：覆盖原来的环境变量

​						0：不覆盖

## 6、进程控制

#### fork函数

​	**头文件：**#include<sys/types.h>

​					#include<unistd.h>

​	**函数原型：**pid_t fork(void);

​	**函数功能：**创建子进程

​	**返回值：**

​		**成功：**该函数的每次调用都返回两次，在父进程中返回的是子进程的PID，在子进程中则返回0。可以根据改返回值判断当前是子进程还是父进程。

​		**失败：**返回-1，并设置errno.

​	**相关函数：**

```c
pid_t getpid(void);					//获取当前进程id
pid_t getppid(void);				//获取父进程id
```

测试：

```c
int main(){
    printf("XXXXXXXXXXXXXXX\n");
    pid_t pid;
    pid=fork();
    if(pid==-1){
        perror("fork error");
        exit(1);
    }else if(pid==0){
        printf("child,pid=%u,ppid=%u\n",getpid(),getppid());
    }else{
        printf("parent,pid=%u,ppid=%u\n",getpid(),getppid());
        sleep(1);
    }
    printf("YYYYYYYYYYYY\n");

    return 0;
}
```

创建多个子进程框架：谁先争夺到CPU，谁先执行（可以通过sleep控制执行的先后顺序）

```c
int CreateChildProcess(){
    pid_t pid;
    int i=0;
    //创建五个子进程
    for( i=0;i<5;++i){
        pid=fork();				//创建子进程
        if(pid==-1){
            perror("fork error");
            exit(1);
        }
        else if(pid==0)
            break;
    }
    //子进程调用
    if(i<5)
    {
        printf("i am child\n");
        sleep(i);
    }    
    else
    {
        printf("i am parent\n");
        sleep(i);
    }
    return 0;
}
```

#### 进程共享

​	刚fork之后：

​	父子相同之处：全局变量、.data、.text、栈、堆、环境变量、用户ID、宿主目录、进程工作目录、信号处理方式。。。。

​	父子不同之处：进程id、fork返回值、父进程id、进程运行时间、定时器、未决信号集

​	全局变量的值父子进程不能共享

​	父子进程间遵循读时共享写时复制的原则。

#### gdb调试

​	使用gdb调试的时候，gdb只能跟踪一个进程。可以在fork函数之前，通过指令设置gdb调试工具跟踪父进程或者是跟踪子进程。默认是跟踪父进程。

​	set follow-fork-mode child 命令设置在fork之后跟踪子进程

​	set follow-fork-mode parent 设置跟踪父进程

**注意**：一定要在fork函数之前设置才可以。

#### exec函数

​	fork创建子进程后执行的是和父进程相同的程序（但有可能执行不同的代码分支），子进程往往要调用一种exec函数以执行另一程序。当进程调用exec函数时，该进程的用户空间代码和数据完全被新程序替换，从新程序的启动例程开始执行。调用exec并不创建新进程，所以调用exec前后该进程的id并未改变。

​	将当前进程的.text、.data替换成所要加载的程序的.text、.data，然后让进程从新的.text第一条指令开始执行，但进程ID不变。换核不换壳。

​	**成功不返回原程序，失败才返回。**

#### execlp函数

头文件：#include<unistd.h>

加载一个进程，借助PATH环境变量

int execlp(const char * file,const char * arg,...，NULL);成功：无返回；失败：-1   //最后需要跟上NULL

参数1：要加载的程序的名字。该函数需要配合PATH环境变量来使用，当PATH中所有目录搜索后没有参数1则出错返回。

arg；命令以及命令参数

该函数通常用来调用系统程序。如ls、date、cp、cat等命令。

```c
pid_t pid=fork();
    if(pid==-1){
        perror("fork error");
        exit(1);
    }else if(pid==0){
        printf("child proccess\n");
        execlp("ls","ls","-l",NULL);                    //最后一位必须是NULL
    }else{
        printf("parent proccess\n");
    }
    return 0;
```

#### execl函数

**功能**：加载一个进程，通过 路径+程序名来加载

**函数原型**：

int execl(const char *path,const char *arg,....,NULL);

成功：无返回；失败：-1

path:要加载程序的路径

**对比使用**

execl("/bin/ls","ls","-l",NULL)；

execlp("ls","ls","-l",NULL);

#### execvp函数

加载一个进程，使用自定义环境变量env

int execvp(const char*file,const char *argv[]);

evecvp与execlp参数形式不同，原理相同

**使用：**

char *argv[]={"ls","-l",NULL};

execvp("ls",argv);

#### 练习

将当前系统中的进程信息，打印到文件中

```c
int main()
{
    int fd;
    fd = open("ps.out", O_WRONLY | O_CREAT, 0644);
    if (fd < 0)
    {
        perror("file create error");
        exit(1);
    }
    //将stdout的文件描述符指向ps.out文件，这样stdout的东西就能输出到ps.out文件中
    dup2(fd, STDOUT_FILENO);
    execlp("ps", "ps", "ax", NULL);
    close(fd);
    return 0;
}
```

### 回收子进程

#### 孤儿进程

父进程先于子进程结束，则子进程成为孤儿进程，子进程的父进程成为init进程，称为init进程领养孤儿进程。

#### 僵尸进程	

​	进程终止，父进程尚未回收，子进程残留资源（PCB）存放于内核中，变成僵尸进程。

​	**注意：**僵尸进程是不能使用kill命令清除掉的。因为kill命令只是用来终止进程的，而僵尸进程已经终止。想要处理僵尸进程，可以使用wait函数。

#### wait函数

​	一个进程在终止时会关闭所有文件描述符，释放在用户空间分配的内存，但它的PCB还保留着，内核在其中保存了一些信息：如果是正常终止则保存着退出状态，如果是异常终止则保存着导致该进程终止的信号是哪个。这个进程的**父进程**可以调用**wait或waitpid**获取这些信息，然后彻底清除这个进程。

​	父进程调用wait函数可以回收子进程终止信息。该函数有三个功能：

​	（1）阻塞等待子进程退出

​	（2）回收子进程残留资源

​	（3）获取子进程结束状态（退出原因）

pid_t wait(int *status);

成功：返回清理掉的子进程id；失败:-1

参数：status:传出参数子进程的退出状态

**多进程时，哪个子进程先结束就回收哪个**

```C 
//回收所有子进程
if(pid>0){
		//父进程
		while(wait(NULL)>0);
		//while(waitpid(-1,NULL,0));
}
```



​	当进程终止时，操作系统的隐式回收机制会：

------

​	**1.关闭所有文件描述符**

​	**2.释放用户空间分配的内存。内存的PCB仍存在，它保存该进程的退出状态。**

------

​	可使用wait函数传出参数status来保存进程的退出状态。借助宏函数进一步判断进程终止的具体原因。宏函数可以分为以下三组。

------

WIFEXITED（status）为非0   ---->   进程正常结束

WEXITSTATUS（status）如上宏为真，使用此宏 ----->   获取进程退出状态（exit的参数）

------

WIFSIGNALED（status）为非0  ---->   进程异常终止

WIERMSIG（status）如上宏为真，使用此宏 ----->  取得使进程终止的那个信号的编号

------

WIFSTOPPED（status）为非0 ----->  进程处于暂停状态

WSTOPSIG（status）如上宏为真，使用此宏----->  取得使进程暂停的那个信号的编号

WIFCONTINUED（status）为真 ----->  进程暂停后已经继续运行

------

#### waitpid函数

作用和wait函数一样————回收子进程，但是可指定pid进程清理，可以不阻塞。

pid_t waitpid(pid_ pid,int *status,int options)

成功：返回清理掉的子进程ID；

失败：-1（无子进程）

参数：

​	pid:	 **>0 回收指定ID的子进程**

​				**-1 回收任意子进程（相当于wait）**

​				0 回收和当前调用waitpid一个组的所有子进程

​				<-1  回收指定进程组内的任意子进程

​	options:    当取值为WNOHANG时（常用），waitpid调用为非阻塞状态。 

​					  当取值为0时，waitpid调用为阻塞状态。

​				     如果参数3位WNOHANG，且子进程没有被回收，则返回0。

```C 
do{
	wpid=waitpid(-1,NULL,WNOHANG);			//==wait(NULL);
	if(wpid>0){
		n--;			//n回收子进程的个数
	}
	//if wpid==0  说明子进程正在运行
	sleep(1);
}while(n>0);
```

**注意**：一次wait或waitpid调用只能清理一个子进程，清理多个子进程应该使用循环。

#### 作业

​	父进程fork3个子进程，三个子进程一个调用ps命令，一个调用自定义程序1，一个调用自定义程序2（会出现段错误）。父进程使用waitpid对其子进程进行回收。

## 7、进程间通信——IPC

#### 常用方法

​	管道（使用最简单）、信号（开销最小）、共享映射区（共享内存）（无血缘关系）、本地套接字（最稳定）（实现比较复杂）

#### 管道

​	管道是一种基本的IPC机制，作用域有血缘关系的进程之间，完成数据传递。调用pipe系统函数即可创建一个管道。有如下特质：

​	1、其本质是一个伪文件（实为内核缓冲区）

​	2、由两个文件描述符引用，一个表示读端，一个表示写端。

​	3、规定数据从管道的写端流入管道，从读端流出。

​	管道的原理：管道实为内核使用环形队列机制，借助内核缓冲区（4k）实现。

​	管道的局限性：

​		（1）数据自己读不能自己写（一端读另一端写）

​		（2）数据一旦被读走，便不再管道中存在，不可反复读取。

​		（3）由于管道采用半双工通信方式（数据只能单向流动）。因此，数据只能在一个方向上流动。

​		（4）只能在公共祖先的进程间使用管道。

##### 	 pipe函数

​	  创建管道

​		int pipe(int pipefd[2]);				成功：0；失败：-1，设置errno

​		函数调用成功返回r/w两个文件描述符。无需open，但需手动close。规定：f[0] -->  r;   f[1] --> w，就像0对应标准输入，1对应标准输出一样。向管道文件读写数据其实是在读写内核缓冲区。

​		管道创建成功以后，创建该管道的进程（父进程）同时掌握着管道的读端和写端。实现父子间进程通信通常可以采用如下步骤：关闭子进程的写端和父进程的读端。

  	**实例**

```c
int main()
{
    int fd[2];
    pid_t pid;
    int ret = pipe(fd);
    if (ret == -1)
    {
        perror("pipe error:");
        exit(1);
    }
    pid = fork();
    if (pid == -1)
    {
        perror("fork error:");
        exit(1);
    }
    else if (pid == 0)
    {
        //子进程
        close(fd[1]); //子进程关闭写端
        char buf[1024];
        ret = read(fd[0], buf, sizeof(buf)); //read返回读取的字节数，如果读到文件末尾返回0
        printf("read len=%d\n", ret);
        if (ret == 0)
        {
            printf("----\n"); //读到文件末尾
        }
        write(STDOUT_FILENO, buf, ret);
    }
    else
    {
        //父进程
        close(fd[0]); //父进程关闭读端，这样形成管道内数据的单向流动
        //父进程写数据
        write(fd[1], "hello pipe \n", strlen("hello pipe \n"));
    }
    return 0;
}
```

![image-20201016111423595](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201016111423595.png)

1.父进程调用pipe函数创建管道，得到两个文件描述符fd[0]、fd[1]指向管道的读端和写端。

2.父进程调用fork创建子进程，那么子进程也有两个文件描述符指向同一管道。

3.父进程关闭管道读端，子进程关闭管道写端。父进程可以向管道中写入数据，子进程将管道中的数据读出。由于管道是利用环形队列实现的，数据从写端流入管道，从读端流出，这样就实现了进程间通信。

##### 管道的读写行为

使用管道需要注意以下4种特殊情况（假设都是阻塞I/O操作，没有设置O_NONBLOCK标志）：

1. 如果所有指向管道写端的文件描述符都关闭了（管道写端引用计数为0），而仍然有进程从管道的读端读数据，那么管道中剩余的数据都被读取后，再次read会返回0，就像读到文件末尾一样。

2. 如果有指向管道写端的文件描述符没关闭（管道写端引用计数大于0），而持有管道写端的进程也没有向管道中写数据，这时有进程从管道读端读数据，那么管道中剩余的数据都被读取后，再次read会阻塞，直到管道中有数据可读了才读取数据并返回。

3. 如果所有指向管道读端的文件描述符都关闭了（管道读端引用计数为0），这时有进程向管道的写端write，那么该进程会收到信号SIGPIPE，通常会导致进程异常终止。当然也可以对SIGPIPE信号实施捕捉，不终止进程。具体方法信号章节详细介绍。

4. 如果有指向管道读端的文件描述符没关闭（管道读端引用计数大于0），而持有管道读端的进程也没有从管道中读数据，这时有进程向管道写端写数据，那么在管道被写满时再次write会阻塞，直到管道中有空位置了才写入数据并返回。

##### 管道缓冲区大小

​	可以使用ulimit –a 命令来查看当前系统中创建管道文件所对应的内核缓冲区大小。通常为：

​          pipe size      (512 bytes, -p) 8

​     也可以使用fpathconf函数，借助参数  选项来查看。使用该宏应引入头文件<unistd.h>

​         long fpathconf(int fd, int name);   成功：返回管道的大小    失败：-1，设置errno

##### 管道的优劣

​	优点：简单，相比信号，套接字实现进程间通信，简单很多。

​     缺点：1. 只能单向通信，双向通信需建立两个管道。

​        		2. 只能用于父子、兄弟进程(有共同祖先)间通信。该问题后来使用fifo有名管道解决。

##### 总结

① 读管道： 1. 管道中有数据，read返回实际读到的字节数。

​                     2. 管道中无数据：

​								(1) 管道写端被全部关闭，read返回0 (好像读到文件结尾)

​                    		    (2) 写端没有全部被关闭，read阻塞等待(不久的将来可能有数据递达，此时会让出cpu)

② 写管道： 1. 管道读端全部被关闭， 进程异常终止(也可使用捕捉SIGPIPE信号，使进程不终止)

​                     2. 管道读端没有全部关闭： 

​								(1) 管道已满，write阻塞。

​                     		   (2) 管道未满，write将数据写入，并返回实际写入的字节数。

------

##### 兄弟间进程通信

​	使用管道实现兄弟间 进程通信。兄：ls  第：wc -l 父：等待回收子进程

```c
int main()
{
    int fd[2];
    pid_t pid;
    int ret = pipe(fd);
    if (ret == -1)
    {
        perror("pipe error:");
        exit(1);
    }
    //循环创建两个子进程
    int i = 0;
    for (i; i < 2; ++i)
    {
        pid = fork();
        if (pid == -1)
        {
            perror("fork error");
            exit(1);
        }
        else if (pid == 0)
        {
            //子进程
            break;
        }
    }
        //兄进程
    if (i == 0)
    {
        close (fd[0]);                              //兄进程关闭读端，操作写端
        dup2(fd[1],STDOUT_FILENO);                  //将标准输出指向管道写端
        execlp("ls","ls","-l",NULL);
        perror("execlp error");                     //execlp执行失败提示
    }
    else if(i==1)
    {
        //第进程
        close(fd[1]);                               //第进程关闭写端，操作读端
        dup2(fd[0],STDIN_FILENO);                   //将标准输入指向管道读端
        execlp("wc","wc","-l",NULL);
        perror("execlp error");                     //execlp执行失败提示
    }
    else
    {
        //父进程关闭读端写端
        close(fd[0]);
        close(fd[1]);
        //回收子进程
        wait(NULL);
        wait(NULL);
    }
    return 0;
}
```

**注意：**

​	允许一个pipe有一个写端，多个读端

#### FIFO

​	FIFO常被称为**命名管道**，以区分管道(pipe)。管道(pipe)只能用于“有血缘关系”的进程间。但通过FIFO，不相关的进程也能交换数据。

​     FIFO是Linux基础文件类型中的一种。但，FIFO文件在磁盘上没有数据块，仅仅用来标识内核中一条通道。各进程可以打开这个文件进行read/write，实际上是在读写内核通道，这样就实现了进程间通信。

创建方式：

1. 命令：mkfifo  管道名

2. 库函数：int mkfifo(const char *pathname,  mode_t mode); 成功：0； 失败：-1

   pathname:自定义命名管道名

3. 头文件

   #include<sys/stat.h>

​     一旦使用mkfifo创建了一个FIFO，就可以使用open打开它，常见的文件I/O函数都可用于fifo。如：close、read、write、unlink等。

​	命名管道的操作方式类似于文件的操作方式。

读：fifo_r.c

```c
#include <stdio.h>
#include <stdlib.h>

#include <unistd.h>
#include <sys/unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
void sys_err(const char *str)
{
    perror(str);
    exit(1);
}

int main(int args, cha *argv[])
{
    int fd,len;
    char buf[4096];
    
    if(argc<2)
    {
        printf("error");
        return -1;
    }
    //argv[1]为输入的命名管道名称
    fd = open(argv[1], O_RDONLY);
    if(fd < 0)
    {
        sys_err("open");
    }
    while(1)
    {
        len = read(fd, buf ,sizeof(buf));
        write(STDOUT_FILENO, buf, len);
        sleep(3);		//多端读时应该增加睡眠秒数
    }
    close(fd);
    return 0;
}

```

写：fifo_w.c

```c
#include <stdio.h>
#include <stdlib.h>

#include <unistd.h>
#include <sys/unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
void sys_err(const char *str)
{
    perror(str);
    exit(1);
}

int main(int args, cha *argv[])
{
    int fd,len;
    char buf[4096];
    
    if(argc<2)
    {
        printf("error");
        return -1;
    }
    //argv[1]为输入的命名管道名称
    fd = open(argv[1], O_RDONLY);
    if(fd < 0)
    {
        sys_err("open");
    }
    i=0;
    while(1)
    {
        sprintf(buf, "hell0 %d\n" ,i++);
        write(fd, buf, strlen(buf));
        sleep(1);		
    }
    close(fd);
    return 0;
}
```

#### 共享存储映射

------

​	**无血缘关系的进程可以打开同一文件进行通信。需要使用sleep()函数控制文件读写**

------

#### 存储映射I/O

​	存储映射I/O (Memory-mapped I/O) 使一个磁盘文件与存储空间中的一个缓冲区相映射。于是当从缓冲区中取数据，就相当于读文件中的相应字节。于此类似，将数据存入缓冲区，则相应的字节就自动写入文件。这样，就可在不适用read和write函数的情况下，使用地址（指针）完成I/O操作。

​	使用这种方法，首先应通知内核，将一个指定文件映射到存储区域中。这个映射工作可以通过mmap函数来实现。

![image-20201016193613564](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201016193613564.png)

####  mmap函数

​	**创建共享内存映射**

​	void *mmap(void *adrr, size_t length, int prot, int flags, int fd, off_t offset); 

返回：成功：返回创建的映射区首地址；**失败：MAP_FAILED宏**

参数：  

​     addr:    建立映射区的首地址，由Linux内核指定。使用时，直接传递NULL，表示系统自动分配

​     length： 欲创建映射区的大小。小于等于文件的实际大小

​     prot：   映射区权限PROT_READ、PROT_WRITE、PROT_READ|PROT_WRITE

​     flags：   标志位参数(常用于设定更新物理区域、设置共享、创建匿名映射区)

​                    MAP_SHARED: 会将映射区所做的操作反映到物理设备（磁盘）上。   

​                    MAP_PRIVATE: 映射区所做的修改不会反映到物理设备。

​     fd：     用来建立映射区的文件描述符

​     offset：  映射文件的偏移(4k的整数倍)。默认0，表示映射文件全部。