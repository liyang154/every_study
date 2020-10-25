#  进程相关概念

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

**全局变量父子进程不共享**

**主线程和子线程共享全局变量**  

****

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

WEXITSTATUS（status）如上宏为真，使用此宏 ----->   **获取进程退出状态（exit的参数）**

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
    int ret = pipe(fd);						//先pipe,后fork
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

#### 共享存储映射mmap

------

​	**无血缘关系的进程可以打开同一文件进行通信。需要使用sleep()函数控制文件读写**

------

##### 存储映射I/O

​	存储映射I/O (Memory-mapped I/O) 使一个磁盘文件与存储空间中的一个缓冲区相映射。于是当从缓冲区中取数据，就相当于读文件中的相应字节。于此类似，将数据存入缓冲区，则相应的字节就自动写入文件。这样，就可在不适用read和write函数的情况下，使用地址（指针）完成I/O操作。

​	使用这种方法，首先应通知内核，将一个指定文件映射到存储区域中。这个映射工作可以通过mmap函数来实现。

![image-20201016193613564](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201016193613564.png)

#####  mmap函数

​	**创建共享内存映射**

​	头文件：#include<sys/mman.h>

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

**示例：**

```c++
void testMmap(){
    char *p=NULL;
    int fd;
    fd=open("testMap",O_RDWR|O_CREAT|O_TRUNC,0644);
    if(fd==-1){
        throw("open error");
    }
    //拓展文件大小
    /*lseek(fd,10,SEEK_END);
    write(fd,"\0",1);*/

    ftruncate(fd,10);               //
    //获取文件长度
    int len=lseek(fd,0,SEEK_END);
    p=(char *)mmap(NULL,len,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
    //失败
    if(p==MAP_FAILED){
       throw("mmap error");
    }
     
    //使用p对文件进行读写操作
    strcpy(p,"hello world");
    cout<<p<<endl;

    int ret=munmap((void *)p,len);
    if(ret==-1){
        throw("munmap error");
    }
}
```

**注意事项：**

1.用于创建映射区的文件大小为0，实际指定大小非0，会出现总线错误。

2.用于创建映射区的文件大小为0，实际指定0大小创建映射区，会出现无效参数。

3.如果open时O_RDONLY, mmap时PROT参数指定PROT_READ|PROT_WRITE会怎样？

​	会出现无效参数

4.文件描述符fd在mmap创建映射区完成即可关闭。后续访问文件，用地址访问。

5.如果文件偏移量不是4k的整数倍，不垂涎无效参数的错误。

6.对申请的内存，不能越界访问。

7.不能对内存进行++操作。

8.需要检查mmap函数的返回值。	

9.映射区访问权限为“私有”MAP_PRIVATE,对内存所做的所有修改，只在内存有效，不会反应到物理磁盘上。

10.映射区访问权限为MAP_PRIVATE,只需要open文件是，有读权限，用于创建映射区即可。

11.创建映射区，需要read权限。当访问权限指定为“共享”MAP_SHARE时，mmap的读写权限，应该<=文件的open权限，只写不行。

**mmap保险调用方式**

1.fd=open("文件名"，O_RDWR)

2.mmap(NULL，有效文件大小，PROT_READ|PROT_WRITE，MAP_SHARED，fd ,0);

**总结**：

1. 创建映射区的过程中，隐含着一次对映射文件的读操作。

2. 当MAP_SHARED时，要求：映射区的权限应 <=文件打开的权限(出于对映射区的保护)。而MAP_PRIVATE则无所谓，因为mmap中的权限是对内存的限制。

3. 映射区的释放与文件关闭无关。只要映射建立成功，文件可以立即关闭。

4. 特别注意，当映射文件大小为0时，不能创建映射区。所以：用于映射的文件必须要有实际大小！！  mmap使用时常常会出现总线错误，通常是由于共享文件存储空间大小引起的。

5. munmap传入的地址一定是mmap的返回地址。坚决杜绝指针++操作。

6. 如果文件偏移量必须为4K的整数倍

7. mmap创建映射区出错概率非常高，一定要检查返回值，确保映射区建立成功再进行后续操作。

##### munmap函数

释放映射区

同malloc函数申请内存空间类似的，mmap建立的映射区在使用结束后也应调用类似free的函数来释放。

int munmap(void *addr, size_t length); 成功：0； 失败：-1

##### mmap建立父子间进程通信

先创建映射区，在创建映射空间

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/wait.h>

int var = 100;

int main(void)
{
    int *p;								//p必须为指针类型，才能够修改内存
    pid_t pid;

    int fd;
    fd = open("temp", O_RDWR|O_CREAT|O_TRUNC, 0644);
    if(fd < 0){
        perror("open error");
        exit(1);
    }
    unlink("temp");				//删除临时文件目录项,使之具备被释放条件.
    //扩展文件大小
    ftruncate(fd, 4);

    p = (int *)mmap(NULL, 4, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
    //p = (int *)mmap(NULL, 4, PROT_READ|PROT_WRITE, MAP_PRIVATE, fd, 0);
    if(p == MAP_FAILED){		//注意:不是p == NULL
        perror("mmap error");
        exit(1);
    }
    close(fd);					//映射区建立完毕,即可关闭文件

    pid = fork();				//创建子进程
    if(pid == 0){
        *p = 2000;
        var = 1000;
        printf("child, *p = %d, var = %d\n", *p, var);
    } else {
        sleep(1);
        printf("parent, *p = %d, var = %d\n", *p, var);
        wait(NULL);

        int ret = munmap(p, 4);				//释放映射区
        if (ret == -1) {
            perror("munmap error");
            exit(1);
        }
    }

    return 0;
}
```

**父子进程使用mmap进程间通信流程**

​	父进程先创建映射区。	open(O_RDWR)   mmap(MAP_SHARED)

​	指定MAP_SHARED 权限

​	fork()创建子进程

​	一个进程读，另一个进程写。

**无血缘关系进程间mmap通信**

​	两个进程打开同一个文件，创建映射区。

​	指定flags为MAP_SHARED。

​	一个进程写入，另一个进程读出。

​	【注意】：无血缘关系进程间通信。**mmap:数据可以重复读**。

​															  **fifo:数据只能一次读取**。 

**匿名映射**

​	使用**MAP_ANONYMOUS** (或MAP_ANON)， 如: 

​     int *p = mmap(NULL, 4, PROT_READ|PROT_WRITE, MAP_SHARED|MAP_ANONYMOUS, -1, 0); 

 	 "4"随意举例，该位置表大小，可依实际需要填写。

需注意的是，MAP_ANONYMOUS和MAP_ANON这两个宏是Linux操作系统特有的宏。在类Unix系统中如无该宏定义，可使用如下两步来完成匿名映射区的建立。

​     ① fd = open("/dev/zero", O_RDWR);

​     ② p = mmap(NULL, size, PROT_READ|PROT_WRITE, MMAP_SHARED, fd, 0);

#### 信号

##### **信号的产生**

1. 按键产生，如：Ctrl+c、Ctrl+z、Ctrl+\

   Ctrl + c → 2) SIGINT（终止/中断）  "INT" ----Interrupt

   Ctrl + z → 20) SIGTSTP（暂停/停止） "T" ----Terminal 终端。

   Ctrl + \ → 3) SIGQUIT（退出） 

2. 系统调用产生，如：kill、raise、abort

3. 软件条件产生，如：定时器alarm

4. 硬件异常产生，如：非法访问内存(段错误)、除0(浮点数例外)、内存对齐出错(总线错误)

5. 命令产生，如：kill命令 

**递达**：递送并且到达进程

**未决**：产生和递达之间的状态。主要由于阻塞(屏蔽)导致该状态。 

**信号的处理方式:** 

1. 执行默认动作 

2. 忽略(丢弃) 

3. 捕捉(调用户处理函数)

Linux内核的进程控制块PCB是一个结构体，task_struct, 除了包含进程id，状态，工作目录，用户id，组id，文件描述符表，还包含了信号相关的信息，主要指阻塞信号集和未决信号集。

**阻塞信号集(信号屏蔽字)**： 将某些信号加入集合，对他们设置屏蔽，当屏蔽x信号后，再收到该信号，该信号的处理将推后(解除屏蔽后)

**未决信号集**: 

1. 信号产生，未决信号集中描述该信号的位立刻翻转为1，表信号处于未决状态。当信号被处理对应位翻转回为0。这一时刻往往非常短暂。

2. 信号产生后由于某些原因(主要是阻塞)不能抵达。这类信号的集合称之为未决信号集。在屏蔽解除前，信号一直处于未决状态。

**信号四要素**：

1. 编号   2. 名称   3. 事件   4. 默认处理动作

**linux常规信号一览表**

1) **SIGHUP**: 当用户退出shell时，由该shell启动的所有进程将收到这个信号，默认动作为终止进程

2) **SIGINT**：当用户按下了<Ctrl+C>组合键时，用户终端向正在运行中的由该终端启动的程序发出此信号。默认动

作为终止进程。

3) **SIGQUIT**：当用户按下<ctrl+\>组合键时产生该信号，用户终端向正在运行中的由该终端启动的程序发出些信

号。默认动作为终止进程。

4) SIGILL：CPU检测到某进程执行了非法指令。默认动作为终止进程并产生core文件

5) SIGTRAP：该信号由断点指令或其他 trap指令产生。默认动作为终止里程 并产生core文件。

6) SIGABRT: 调用abort函数时产生该信号。默认动作为终止进程并产生core文件。

7) **SIGBUS**：非法访问内存地址，包括内存对齐出错，默认动作为终止进程并产生core文件。

8) **SIGFPE**：在发生致命的运算错误时发出。不仅包括浮点运算错误，还包括溢出及除数为0等所有的算法错误。默认动作为终止进程并产生core文件。

9) **SIGKILL**：无条件终止进程。**本信号不能被忽略，处理和阻塞**。**默认动作为终止进程**。它向系统管理员提供了可以杀死任何进程的方法。

10) **SIGUSE1**：用户定义 的信号。即程序员可以在程序中定义并使用该信号。默认动作为终止进程。

11) **SIGSEGV**：指示进程进行了无效内存访问。**默认动作为终止进程并产生core文件**。**段错误**。

12) **SIGUSR2**：另外一个用户自定义信号，程序员可以在程序中定义并使用该信号。默认动作为终止进程。

13) **SIGPIPE **：Broken pipe向一个没有读端的管道写数据。默认动作为终止进程。

14) **SIGALRM**: 定时器超时，超时的时间 由系统调用alarm设置。**默认动作为终止进程**。

15) **SIGTERM**：程序结束信号，与SIGKILL不同的是，**该信号可以被阻塞和终止**。通常用来要示程序正常退出。执行shell命令Kill时，缺省产生这个信号。默认动作为终止进程。

16) SIGSTKFLT：Linux早期版本出现的信号，现仍保留向后兼容。默认动作为终止进程。

17) SIGCHLD：子进程结束时，父进程会收到这个信号。**默认动作为忽略这个信号**。

18) SIGCONT：如果进程已停止，则使其继续运行。默认动作为继续/忽略。

19) **SIGSTOP**：停止进程的执行。信号不能被忽略，处理和阻塞。**默认动作为暂停进程**。

20) SIGTSTP：停止终端交互进程的运行。按下<ctrl+z>组合键时发出这个信号。默认动作为暂停进程。

21) SIGTTIN：后台进程读终端控制台。默认动作为暂停进程。

22) SIGTTOU: 该信号类似于SIGTTIN，在后台进程要向终端输出数据时发生。默认动作为暂停进程。

23) SIGURG：套接字上有紧急数据时，向当前正在运行的进程发出些信号，报告有紧急数据到达。如网络带外数据到达，默认动作为忽略该信号。

24) SIGXCPU：进程执行时间超过了分配给该进程的CPU时间 ，系统产生该信号并发送给该进程。默认动作为终止进程。

25) SIGXFSZ：超过文件的最大长度设置。默认动作为终止进程。

26) SIGVTALRM：虚拟时钟超时时产生该信号。类似于SIGALRM，但是该信号只计算该进程占用CPU的使用时间。默认动作为终止进程。

27) SGIPROF：类似于SIGVTALRM，它不公包括该进程占用CPU时间还包括执行系统调用时间。默认动作为终止进程。

28) SIGWINCH：窗口变化大小时发出。默认动作为忽略该信号。

29) SIGIO：此信号向进程指示发出了一个异步IO事件。默认动作为忽略。

30) SIGPWR：关机。默认动作为终止进程。

31) SIGSYS：无效的系统调用。默认动作为终止进程并产生core文件。

34) SIGRTMIN ～ (64) SIGRTMAX：LINUX的实时信号，它们没有固定的含义（可以由用户自定义）。所有的实时信号的默认动作都为终止进程。

 **默认动作：**

​          Term：终止进程

​          Ign： 忽略信号 (默认即时对该种信号忽略操作)

​          Core：终止进程，生成Core文件。(查验进程死亡原因， 用于gdb调试)

​          Stop：停止（暂停）进程

​          Cont：继续运行进程

这里特别强调了**9) SIGKILL 和19) SIGSTOP信号，不允许忽略和捕捉，只能执行默认动作。甚至不能将其设置为阻塞。**

**另外需清楚，只有每个信号所对应的事件发生了，该信号才会被递送(但不一定递达)，不应乱发信号！！**

------

##### kill函数/命令产生信号

kill命令产生信号：kill -SIGKILL pid

杀死进程组 kill -SIGKILL -pid(进程组ip)

查看进程组 ps ajx

kill函数：给指定进程发送指定信号(不一定杀死)

头文件：#include <sys/types.h>

​				#include<signal.h>

 int kill(pid_t pid, int sig);  成功：0；失败：-1 (ID非法，信号非法，普通用户杀    init进程等权级问题)，设置errno

​     sig：不推荐直接使用数字，**应使用宏名**，因为不同操作系统信号编号可能不				同，但名称一致。

​	 pid > 0: 发送信号给指定的进程。

​     pid = 0: 发送信号给 与调用kill函数进程属于同一进程组的所有进程。

​     pid < 0: 取|pid|发给对应进程组。

​     pid = -1：发送给进程有权限发送的系统中所有进程。

**进程组**：每个进程都属于一个进程组，进程组是一个或多个进程集合，他们相互关联，共同完成一个实体任务，每个进程组都有一个进程组长，默认进程组ID与进程组长ID相同。

##### raise和abort函数

#include<signal.h>

raise 函数：给当前进程发送指定信号(自己给自己发)  raise(signo) == kill(getpid(), signo);

​       int raise(int sig); 成功：0，失败非0值

------

#include<stdlib.h>

abort 函数：给自己发送异常终止信号 6) SIGABRT 信号，终止并产生core文件

​       void abort(void); 该函数无返回

##### alarm函数

使用自然计时法（不常用，一般用sleep或者usleep）

设置定时器(闹钟)。在指定seconds后，内核会给当前进程发送14）SIGALRM信号。进程收到该信号，默认动作终止。

**每个进程都有且只有唯一个定时器。**

unsigned int alarm(unsigned int seconds); 返回0或上个定时器剩余的秒数，无失败。

​     常用：取消定时器alarm(0)，返回旧闹钟余下秒数。

**实际执行时间 = 系统时间 + 用户时间 + 等待时间**

------

##### 信号集操作函数

内核通过读取未决信号集来判断信号是否应被处理。信号屏蔽字mask可以影响未决信号集。而我们可以在应用程序中自定义set来改变mask，以达到屏蔽指定信号的目的。

**信号集设定**

头文件：#include<signal.h>

sigset_t set;     // typedef unsigned long sigset_t; 

int sigemptyset(sigset_t *set);             将某个信号集清0             成功：0；失败：-1

int sigfillset(sigset_t *set);                 将某个信号集置1            成功：0；失败：-1

int sigaddset(sigset_t *set, int signum);      将某个信号加入信号集       成功：0；失败：-1

 int sigdelset(sigset_t *set, int signum);      将某个信号清出信号集       成功：0；失败：-1

int sigismember(const sigset_t *set, int signum);判断某个信号是否在信号集中   返回值：在集合：1；不在：0；出错：-1 

 sigset_t类型的本质是位图。但不应该直接使用位操作，而应该使用上述函数，保证跨系统操作有效。  对比认知select 函数。

**sigprocmask函数**

用来屏蔽信号、解除屏蔽也使用该函数。其本质，读取或修改进程的信号屏蔽字(PCB中)

  **严格注意，屏蔽信号：只是将信号处理延后执行(延至解除屏蔽)；而忽略表示将信号丢处理。**

int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);  成功：0；失败：-1，设置errno

参数：

​          set：传入参数，是一个位图，**set中哪位置1，就表示当前进程屏蔽哪个信号。**

​          oldset：传出参数，保存旧的信号屏蔽集。

​          how参数取值：  假设当前的信号屏蔽字为mask

​			1.    SIG_BLOCK: 当how设置为此值，set表示需要屏蔽的信号。相当于 mask = mask|set

2. SIG_UNBLOCK: 当how设置为此，set表示需要解除屏蔽的信号。相当于 mask = mask & ~set

3. SIG_SETMASK: 当how设置为此，set表示用于替代原始屏蔽及的新屏蔽集。相当于 mask = set若，调用sigprocmask解除了对当前若干个信号的阻塞，则在sigprocmask返回前，至少将其中一个信号递达。（不推荐使用该方法）

**sigpending函数**

读取当前进程的**未决**信号集

int sigpending(sigset_t *set); set传出参数。  返回值：成功：0；失败：-1，设置errno

------

mask:信号屏蔽字

set:用户自定义的信号集，可以用在mask上

------

##### 信号捕捉

**signal函数**

**注册**一个信号捕捉函数：

头文件#include<signal.h>

typedef void (*sighandler_t)(int);

sighandler_t signal(int signum, sighandler_t handler);

**signum:**某个系统信号

​     该函数由ANSI定义，由于历史原因在不同版本的Unix和不同版本的Linux中可能有不同的行为。因此应该尽量避免使用它，取而代之使用sigaction函数。

  void (*signal(int signum, void (*sighandler_t)(int))) (int);

  能看出这个函数代表什么意思吗？ 注意多在复杂结构中使用typedef。

```c
void sys_err(const char * str){
    perror(str);
    exit(1);
}
//信号捕捉函数
void sig_cath(int signo){
    printf("catch you!!! %d\n",signo);
}
int main(){
    signal(SIGINT,sig_cath);
    while(1);			//该循环只是为了保证有足够的时间来测试函数特性
    return 0;
}
```

**sigaction函数**  

头文件：\#include <bits/sigaction.h> 

​				#include<signal.h>

修改信号处理动作（通常在Linux用其来注册一个信号的捕捉函数）

  int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact); 成功：0；失败：-1，设置errno

参数：

act：传入参数，新的处理方式。

​          oldact：传出参数，旧的处理方式。              

**struct sigaction结构体**

  struct sigaction {

​    void   (*sa_handler)(int);

​    void   (*sa_sigaction)(int, siginfo_t *, void *);

​    sigset_t  sa_mask; 

​    int    sa_flags; 

​    void   (*sa_restorer)(void);

  };

​     sa_restorer：该元素是过时的，不应该使用，POSIX.1标准将不指定该元素。(弃用)

​     sa_sigaction：当sa_flags被指定为SA_SIGINFO标志时，使用该信号处理程序。(很少使用) 

重点掌握：

​     ① **sa_handler**：指定信号捕捉后的处理函数名(即注册函数)。也可赋值为SIG_IGN表忽略 或 SIG_DFL表执行默认动作

​     ② **sa_mask**: 调用信号处理函数时，所要屏蔽的信号集合(信号屏蔽字)。注意：仅在处理函数被调用期间屏蔽生效，是临时性设置。

​     ③ sa_flags：通常设置为0，表使用默认属性。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

/*自定义的信号捕捉函数*/
void sig_int(int signo)
{
	printf("catch signal SIGINT\n");//单次打印
    sleep(10);
    printf("----slept 10 s\n");
}

int main(void)
{
	struct sigaction act;		

	act.sa_handler = sig_int;
	act.sa_flags = 0;
	sigemptyset(&act.sa_mask);		//不屏蔽任何信号
    //sigaddset(&act.sa_mask, SIGQUIT);		

	sigaction(SIGINT, &act, NULL);

    printf("------------main slept 10\n");
    sleep(10);

	while(1);//该循环只是为了保证有足够的时间来测试函数特性

	return 0;
}
```

**信号捕捉特性**

1. 进程正常运行时，默认PCB中有一个信号屏蔽字，假定为☆，它决定了进程自动屏蔽哪些信号。当注册了某个信号捕捉函数，捕捉到该信号以后，要调用该函数。而该函数有可能执行很长时间，在这期间所屏蔽的信号不由☆来指定。**而是用sa_mask来指定**。调用完信号处理函数，再恢复为☆。

2. XXX信号捕捉函数执行期间，XXX信号自动被屏蔽。但是其他信号仍然可以被捕捉。

3. **阻塞的常规信号不支持排队，产生多次只记录一次**。（后32个实时信号支持排队）

**内核实现信号捕捉过程**

![image-20201023194234168](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201023194234168.png)

用户态可以通过中断、系统调用进入内核态

**SIGCHLD信号**（17）

产生条件：

​	子进程终止时

​	子进程接收到SIGSTOP信号停止时

​	子进程处在停止态，接受到SIGCONT后唤醒时

```c

//回收子进程
void catch_child(int signo){
    pid_t wpid;
    while(wpid=wait(NULL)!=-1)          //循环回收子进程，防止出现僵尸进程
        printf("catch chile id %d\n",wpid);
}
int main(){
    pid_t pid;
    int i;
    for ( i = 0; i < 5; i++)
    {
        if((pid=fork())==0)
            break;                  //子进程
    }
    if(i==5){
        //父进程
        printf("parent,pid=%d\n",getpid());
        //在父进程回收子进程
        struct sigaction act;
        act.sa_handler=catch_child;                             //注册信号处理函数
        sigemptyset(&act.sa_mask);
        act.sa_flags=0;
        sigaction(SIGCHLD,&act,NULL);                         //捕捉信号，并处理；SIGCHILD，子进程结束时会收到这个信号
        while(1);
    }else{
        printf("child,pid=%d\n",getpid());
    }
     
}
```

#### 会话

创建会话

创建一个会话需要注意以下6点注意事项：

1. 调用进程不能是进程组组长，该进程变成新会话首进程(session header)

2. **该进程成为一个新进程组的组长进程。**

3. 需有root权限(ubuntu不需要)

4. 新会话丢弃原有的控制终端，该会话没有控制终端

5. **如果该调用进程是组长进程，则出错返回**

6. 建立新会话时，先调用fork, 父进程终止，子进程调用setsid

setsid函数

创建一个会话，并以自己的ID设置进程组ID，同时也是新会话的ID。

​     pid_t setsid(void); 成功：返回调用进程的会话ID；失败：-1，设置errno

​     调用了setsid函数的进程，既是新的会长，也是新的组长。                               

练习：fork一个子进程，并使其创建一个新会话。查看进程组ID、会话ID前后变化

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
    pid_t pid;

    if ((pid = fork())<0) {
        perror("fork");
        exit(1);

    } else if (pid == 0) {

        printf("child process PID is %d\n", getpid());
        printf("Group ID of child is %d\n", getpgid(0));
        printf("Session ID of child is %d\n", getsid(0));

        sleep(10);
        setsid();       //子进程非组长进程，故其成为新会话首进程，且成为组长进程。该进程组id即为会话进程

        printf("Changed:\n");

        printf("child process PID is %d\n", getpid());
        printf("Group ID of child is %d\n", getpgid(0));
        printf("Session ID of child is %d\n", getsid(0));

        sleep(20);

        exit(0);
    }

    return 0;
}
```

#### 守护进程

Daemon(精灵)进程，是Linux中的后台服务进程，**通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件**。一般采用以d结尾的名字。**不受用户登录注销影响。**

eg : httpd，sshd

Linux后台的一些系统服务进程，没有控制终端，不能直接和用户交互。不受用户登录、注销的影响，一直在运行着，他们都是守护进程。如：预读入缓输出机制的实现；ftp服务器；nfs服务器等。

  创建守护进程，最关键的一步是调用setsid函数创建一个新的Session，并成为Session Leader。

#### 创建守护进程

1. 创建子进程，父进程退出

   所有工作在子进程中进行形式上脱离了控制终端

2. 在子进程中创建新会话

　　     setsid()函数

​			使子进程完全独立出来，脱离控制

3. 改变当前目录为根目录

   chdir()函数

   **防止占用可卸载的文件系统**

   **也可以换成其它路径**

4. 重设文件权限掩码

　　     umask()函数

　　     防止继承的文件创建屏蔽字拒绝某些权限

　　     增加守护进程灵活性

5. 关闭文件描述符

　　     继承的打开文件不会用到标准输入，标准输出，标准错误文件描述符。浪费系统资源，无法卸载

6. 开始执行守护进程核心工作

   守护进程退出处理程序模型

```c
void daemonize(void)
{
    pid_t pid;
    /*
     * * 成为一个新会话的首进程，失去控制终端
     * */
    if ((pid = fork()) < 0) {
        perror("fork");
        exit(1);
    } else if (pid != 0) /* parent */
        exit(0);
    setsid();				//创建新会话
    /*
     * * 改变当前工作目录到/目录下.
     * */
    if (chdir("/") < 0) {
        perror("chdir");
        exit(1);
    }
    /* 设置umask为0 */
    umask(0);			//改变文件访问权限掩码
    /*
     * * 重定向0，1，2文件描述符到 /dev/null，因为已经失去控制终端，再操作0，1，2没有意义.
     * */
    close(STDIN_FILENO);				//STDIN_FILENO 0
    open("/dev/null", O_RDWR);
    dup2(0, STDOUT_FILENO);				//STDOUT_FILENO 1
    dup2(0, STDERR_FILENO);				//STDERR_FILENO 2
    while(1);							//模拟守护进程业务
}
```

# 线程

LWP：light weight process 轻量级的进程，本质仍是进程(在Linux环境下)

![img](file:///C:/Users/yang/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)    

​	 进程：独立地址空间，拥有PCB

​     线程：也有PCB，但没有独立的地址空间(共享)

​     区别：在于是否共享地址空间。    独居(进程)；合租(线程)。

​     Linux下：    

​				 线程：最小的执行单位

​                 进程：最小分配资源单位，可看成是只有一个线程的进程

ps  -Lf 进程ip :查看线程号LWP。

**线程可看做寄存器和栈的集合**

## 线程共享资源

​     1.文件描述符表

​     2.每种信号的处理方式

​     3.当前工作目录

​     4.用户ID和组ID

​     5.内存地址空间 (.text/.data/.bss/heap/共享库)

## 线程非共享资源

​     1.线程id

​     2.处理器现场（寄存器的值）和**栈指针**(内核栈)

​     3.独立的栈空间(用户空间栈)

​     4.errno变量

​     5.信号屏蔽字

​     6.调度优先级

## 线程控制

------

线程中出错检测不能再用perror,

而是用fprintf(stderr,"error string:%s\n,strerror(ret))

### pthread_self函数

​	获取线程ID。其作用对应进程中 getpid() 函数。

​	线程id是在进程地址空间内部，用来标识线程身份的id号。

​     pthread_t pthread_self(void); 返回值：成功：id；   失败：无！

​     线程ID：pthread_t类型，本质：在Linux下为无符号整数(%lu)，其他系统中可能是结构体实现

​     线程ID是进程内部，识别标志。(两个进程间，线程ID允许相同)

​     注意：不应使用全局变量 pthread_t tid，在子线程中通过pthread_create传出参数来获取线程ID，而应使用pthread_self。

### pthread_create函数

创建一个新线程。         其作用，对应进程中fork() 函数。

​     int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);

​     返回值：成功：0；   失败：错误号    -----Linux环境下，所有线程特点，失败均直接返回错误号。

参数：  

​     pthread_t：当前Linux中可理解为：typedef unsigned long int pthread_t;

参数1：传出参数，保存系统为我们分配好的线程ID

​     参数2：通常传NULL，表示使用线程默认属性。若想使用具体属性也可以修改该参数。

​     参数3：函数指针，指向线程主函数(线程体)，该函数运行结束，则线程结束。

​     参数4：线程主函数执行期间所使用的参数。

在一个线程中调用pthread_create()创建新的线程后，当前线程从pthread_create()返回继续往下执行，而新的线程所执行的代码由我们传给pthread_create的函数指针start_routine决定。start_routine函数接收一个参数，是通过pthread_create的arg参数传递给它的，该参数的类型为void *，这个指针按什么类型解释由调用者自己定义。start_routine的返回值类型也是void *，这个指针的含义同样由调用者自己定义。start_routine返回时，这个线程就退出了，其它线程可以调用pthread_join得到start_routine的返回值，类似于父进程调用wait(2)得到子进程的退出状态，稍后详细介绍pthread_join。

pthread_create成功返回后，新创建的线程的id被填写到thread参数所指向的内存单元。我们知道进程id的类型是pid_t，每个进程的id在整个系统中是唯一的，调用getpid(2)可以获得当前进程的id，是一个正整数值。线程id的类型是thread_t，它只在当前进程中保证是唯一的，在不同的系统中thread_t这个类型有不同的实现，它可能是一个整数值，也可能是一个结构体，也可能是一个地址，所以不能简单地当成整数用printf打印，调用pthread_self(3)可以获得当前线程的id。

#### 循环创建线程

```c
void create_more_pthread()
{
    int i;
    int ret;
    pthread_t tid;
    for (i = 0; i < 5; ++i)
    {
        ret = pthread_create(&tid, NULL, tfn, (void *)i);
        if (ret != 0)
        {
            sys_err("pthread_create error");
        }
    }
    sleep(i); //给子线程流出执行时间
}
```

​     **主线程和子线程共享全局变量**          

​	**全局变量父子进程不共享**

### pthread_exit函数

将当前线程退出

​     void pthread_exit(void *retval);   参数：retval表示线程退出状态，通常传NULL

思考：使用exit将指定线程退出，可以吗？                                              【pthrd_exit.c】

​     结论：**线程中，禁止使用exit函数，exit代表退出进程，它会导致进程内所有线程全部退出。**

​     在不添加sleep控制输出顺序的情况下。pthread_create在循环中，几乎瞬间创建5个线程，但只有第1个线程有机会输出（或者第2个也有，也可能没有，取决于内核调度）如果第3个线程执行了exit，将整个进程退出了，所以全部线程退出了。

​     所以，**多线程环境中，应尽量少用，或者不使用exit函数，取而代之使用pthread_exit函数，将单个线程退出**。任何线程里exit导致进程退出，其他线程未工作结束，主控线程退出时不能return或exit。

另注意，pthread_exit或者return返回的指针所指向的内存单元必须是全局的或者是用malloc分配的，不能在线程函数的栈上分配，因为当其它线程得到这个返回指针时线程函数已经退出了。

### pthread_join函数

**阻塞等待线程退出，获取线程退出状态**      其作用，对应进程中 waitpid() 函数。

​     int pthread_join(pthread_t thread, void **retval); 成功：0；失败：错误号

​     参数：thread：线程ID （【注意】：不是指针）；retval：存储线程结束状态。

```c
struct thrd{
    int val;
    char str[256];
};
void *tfnjoin(void *arg)
{
    struct thrd *tval;

    tval = malloc(sizeof(tval));
    tval->val=100;
    strcpy(tval->str,"hello thread");

    return (void *)tval;
}
void my_pthread_join()
{
    pthread_t tid;
    struct thrd *retval;

    int ret = pthread_create(&tid,NULL,tfnjoin,NULL);
    if(ret!=0)
    {
        fprintf(stderr,
                "pthread_create error:%s\n",strerror(ret));
        exit(1);
    }
    
    //回收线程
    ret=pthread_join(tid,(void **)&retval);
    if(ret!=0)
    {
        fprintf(stderr,
                "pthread_join error:%s\n",sterror(ret));
    }
    printf("child thread exit with val=%d,str=%s\n",retval->val,retval->str);

    //退出当前线程
    pthread_exit(NULL);
    
}
```

​     **对比记忆：**

​         进程中：main返回值、exit参数-->int；等待子进程结束 wait 函数参数-->int *

​         线程中：线程主函数返回值、pthread_exit-->void *；等待线程结束 pthread_join 函数参数-->void **

### pthread_detach函数

实现线程分离

​     int pthread_detach(pthread_t thread); 成功：0；失败：错误号

​     线程分离状态：指定该状态，线程主动与主控线程断开关系。线程结束后，其退出状态不由其他线程获取，而直接自己自动释放。网络、多线程服务器常用。

​     进程若有该机制，将不会产生僵尸进程。僵尸进程的产生主要由于进程死后，大部分资源被释放，一点残留资源仍存于系统中，导致内核认为该进程仍存在。

### pthread_cancel函数

杀死(取消)线程            其作用，对应进程中 kill() 函数。

​     int pthread_cancel(pthread_t thread); 成功：0；失败：错误号

​     【注意】：线程的取消并不是实时的，而有一定的延时。需要等待线程到达某个取消点(检查点)。

​     类似于玩游戏存档，必须到达指定的场所(存档点，如：客栈、仓库、城里等)才能存储进度。杀死线程也不是立刻就能完成，必须要到达取消点。

​     取消点：是线程检查是否被取消，并按请求进行动作的一个位置。通常是一些系统调用creat，open，pause，close，read，write..... 执行命令man 7 pthreads可以查看具备这些取消点的系统调用列表。也可参阅 APUE.12.7 取消选项小节。

可粗略认为一个系统调用(进入内核)即为一个取消点。如线程中没有取消点，可以通过调用pthread_testcancel()函数自行设置一个取消点。

被取消的线程，  退出值定义在Linux的pthread库中。常数PTHREAD_CANCELED的值是-1。可在头文件pthread.h中找到它的定义：**#define PTHREAD_CANCELED ((void \*) -1)。**因此当我们对一个已经被取消的线程使用pthread_join回收时，得到的返回值为-1。

### 线程取消

总结：终止某个线程而不终止整个进程，有三种方法：

1. 从线程主函数return。这种方法对主控线程不适用，从main函数return相当于调用exit。

2. 一个线程可以调用pthread_cancel终止同一进程中的另一个线程。

3. 线程可以调用pthread_exit终止自己。

### 控制原语对比

  进程             						 					线程

  fork         					 					pthread_create

  exit       					   					pthread_exit

  wait/waitpid         						pthread_join

  kill           										pthread_cancel

  getpid      				   					pthread_self      命名空间

### 线程属性

​	线程属性主要包括如下属性：作用域（scope）、栈尺寸（stack size）、栈地址（stack address）、优先级（priority）、分离的状态（detached state）、调度策略和参数（scheduling policy and parameters）。默认的属性为非绑定、非分离、缺省的堆栈、与父进程同样级别的优先级。

#### 线程属性初始化

注意：应先初始化线程属性，再pthread_create创建线程

初始化线程属性

int pthread_attr_init(pthread_attr_t *attr); 成功：0；失败：错误号

销毁线程属性所占用的资源

int pthread_attr_destroy(pthread_attr_t *attr); 成功：0；失败：错误号

#### 线程的分离状态

线程的分离状态决定一个线程以什么样的方式来终止自己。

非分离状态：**线程的默认属性是非分离状态**，这种情况下，原有的线程等待创建的线程结束。只有当pthread_join()函数返回时，创建的线程才算终止，才能释放自己占用的系统资源。

分离状态：分离线程没有被其他的线程所等待，自己运行结束了，线程也就终止了，马上释放系统资源。应该根据自己的需要，选择适当的分离状态。

线程分离状态的函数：

设置线程属性，分离or非分离

​			int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate); 

获取程属性，分离or非分离

 			int pthread_attr_getdetachstate(pthread_attr_t *attr, int *detachstate); 

 参数：  attr：已初始化的线程属性

detachstate： PTHREAD_CREATE_DETACHED（分离线程）

PTHREAD _CREATE_JOINABLE（非分离线程）

​		这里要注意的一点是，如果设置一个线程为分离线程，而这个线程运行又非常快，它很可能在pthread_create函数返回之前就终止了，它终止以后就可能将线程号和系统资源移交给其他的线程使用，这样调用pthread_create的线程就得到了错误的线程号。要避免这种情况可以采取一定的同步措施，最简单的方法之一是可以在被创建的线程里调用pthread_cond_timedwait函数，让这个线程等待一会儿，留出足够的时间让函数pthread_create返回。设置一段等待时间，是在多线程编程里常用的方法。但是注意不要使用诸如wait()之类的函数，它们是使整个进程睡眠，并不能解决线程同步的问题。

```c
void *tfn_status(void *arg)
{
    printf("子线程：pid=%d,tid=%lu\n", getpid(), pthread_self());
    sleep(1);
    return NULL;
}

void my_pthread_status()
{
    pthread_t tid;

    //线程属性初始化
    pthread_attr_t attr;
    int ret = pthread_attr_init(&attr);
    if (ret != 0)
    {
        fprintf(stderr, "pthread_attr_init error %s\n", strerror(ret));
        exit(1);
    }

    //设置线程属性为分离属性
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
    if (ret != 0)
    {
        fprintf(stderr, "pthread_attr_setdetachstate error %s\n", strerror(ret));
        exit(1);
    }

    //创建线程
    ret = pthread_create(&tid, &attr, tfn_cancle, NULL);
    if (ret != 0)
    {
        fprintf(stderr, "pthread_create error %s\n", strerror(ret));
        exit(1);
    }

    //销毁线程属性所占用的资源
    ret = pthread_attr_destroy(&attr);
    if (ret != 0)
    {
        fprintf(stderr, "pthread_attr_destory error %s\n", strerror(ret));
        exit(1);
    }
  
    //尝试回收子线程
    ret=pthread_join(tid,NULL);
    if (ret != 0)
    {
        fprintf(stderr, "pthread_join error %s\n", strerror(ret));
        exit(1);
    }

    printf("主线程：pid=%d,tid=%lu\n", getpid(), pthread_self());
    //退出主线程
    pthread_exit(NULL);
}
```

**设置分离属性**

1.pthread_attr_t attr;								创建一个线程属性结构体变量

2.pthread_attr_init(&attr);					初始化线程属性

3.pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

​																		设置线程属性为分离态

4.pthread_create(&tid, &attr, tfn_cancle, NULL);					创建线程

5.pthread_attr_destroy(&attr)					销毁线程属性

#### 线程使用注意事项

1. 主线程退出其他线程不退出，主线程应调用pthread_exit

2. 避免僵尸线程

   pthread_join

   pthread_detach

   pthread_create指定分离属性

被join线程可能在join函数返回前就释放完自己的所有内存资源，所以不应当返回被回收线程栈中的值;

3. malloc和mmap申请的内存可以被其他线程释放

4. 应避免在多线程模型中调用fork，除非马上exec，子进程中只有调用fork的线程存在，其他线程在子进程中均pthread_exit.

5. 信号的复杂语义很难和多线程共存，应避免在多线程引入信号机制

## 线程同步

## 互斥量mutex 

Linux中提供一把互斥锁mutex（也称之为互斥量）。

------

mutex可以看成只有两种状态，0或1；初始化为1，可以加锁；当 mutex变为0时，可以解锁。

pthread_mutex_lock---->mutex--

pthread_mutex_unlock----->mutex++

------

每个线程在对资源操作前都尝试先加锁，成功加锁才能操作，操作结束解锁。

​     资源还是共享的，线程间也还是竞争的，                               

​     但通过“锁”就将资源的访问变成互斥操作，而后与时间有关的错误也不会再产生了。

![image-20201025103338259](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201025103338259.png)

   但，应注意：同一时刻，只能有一个线程持有该锁。

​     当A线程对某个全局变量加锁访问，B在访问前尝试加锁，拿不到锁，B阻塞。C线程不去加锁，而直接访问该全局变量，依然能够访问，但会出现数据混乱。

​     所以，互斥锁实质上是操作系统提供的一把“建议锁”（又称“协同锁”），建议程序中有多线程访问共享资源的时候使用该机制。但，并没有强制限定。

​     因此，即使有了mutex，如果有线程不按规则来访问数据，依然会造成数据混乱。

### 主要应用函数

#### 加锁基本流程

1. pthread_mutex_t lock														创建锁			
2. pthread_mutex_init()                                                         初始化
3. pthread_mutex_lock()                                                         加锁
4. 访问共享数据            
5. pthread_mutex_unlock()                                                     解锁
6. pthread_mutex_destory()                                                    销毁锁

```c
pthread_mutex_t mutex;                  //定义一把互斥锁

void *mutex_tfn(void *arg)
{
    srand(time(NULL));
    while(1)
    {
        pthread_mutex_lock(&mutex);                 //加锁
        printf("hello  ");
        sleep(rand()%3);                       //模拟长时间操作共享资源
        printf("world\n");
        pthread_mutex_unlock(&mutex);               //解锁
        sleep(rand()%3);
        
    }
    return NULL;
}

void my_mutex()
{
    pthread_t tid;
    srand(time(NULL));
    int ret=pthread_mutex_init(&mutex,NULL);                        //初始化互斥锁
    if(ret!=0)
    {
        fprintf(stderr,"mutex_init error：%s\n",strerror(ret));
        exit(1);
    }

    //创建线程
    pthread_create(&tid,NULL,mutex_tfn,NULL);
    while(1)
    {
        pthread_mutex_lock(&mutex);             //加锁
        printf("HELLO  ");
        sleep(rand()%3);
        printf("WORLD\n");
        pthread_mutex_unlock(&mutex);           //解锁
        sleep(rand()%3);
    }


    pthread_mutex_destroy(&mutex);                                  //销毁互斥锁
}
```

**尽量保证锁的粒度越小越好。（访问共享数据前加锁，访问共享数据结束后立即解锁 ）**

#### pthread_mutex_init函数

初始化一个互斥锁(互斥量) ---> 初值可看作1

​     int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);

​     参1：传出参数，调用时应传 &mutex   

​     restrict关键字：只用于限制指针，告诉编译器，所有修改该指针指向内存中内容的操作，只能通过本指针完成。不能通过除本指针以外的其他变量或指针修改

​     参2：互斥量属性。是一个传入参数，通常传NULL，选用默认属性(线程间共享)。 参APUE.12.4同步属性

1. 静态初始化：如果互斥锁 mutex 是静态分配的（定义在全局，或加了static关键字修饰），可以直接使用宏进行初始化。e.g.  pthead_mutex_t muetx = PTHREAD_MUTEX_INITIALIZER;

2. 动态初始化：局部变量应采用动态初始化。e.g.  pthread_mutex_init(&mutex, NULL)

#### pthread_mutex_destroy函数

销毁一个互斥锁

​     int pthread_mutex_destroy(pthread_mutex_t *mutex);

#### pthread_mutex_lock函数

加锁。可理解为将mutex--（或-1）

​     int pthread_mutex_lock(pthread_mutex_t *mutex);

#### pthread_mutex_unlock函数

解锁。可理解为将mutex ++（或+1）

​     int pthread_mutex_unlock(pthread_mutex_t *mutex);

#### pthread_mutex_trylock函数

尝试加锁：非阻塞轮询

​     int pthread_mutex_trylock(pthread_mutex_t *mutex);

加锁失败直接返回错误号（如：EBUSY），不阻塞；成功返回0；

### 加锁与解锁

#### lock与unlock：

​     lock尝试加锁，如果加锁不成功，线程阻塞，阻塞到持有该互斥量的其他线程解锁为止。

​     unlock主动解锁函数，**同时将阻塞在该锁上的所有线程全部唤醒**，至于哪个线程先被唤醒，取决于优先级、调度。默认：先阻塞、先唤醒。

​     例如：T1 T2 T3 T4 使用一把mutex锁。T1加锁成功，其他线程均阻塞，直至T1解锁。T1解锁后，T2 T3 T4均被唤醒，并自动再次尝试加锁。

​     可假想mutex锁 init成功初值为1。 lock 功能是将mutex--。   unlock将mutex++

#### lock与trylock：

​     lock加锁失败会阻塞，等待锁释放。

​     trylock加锁失败直接返回错误号（如：EBUSY），不阻塞。

## 死锁

​     		1. 线程试图对同一个互斥量A加锁两次。

     2. 线程1拥有A锁，请求获得B锁；线程2拥有B锁，请求获得A锁

## 读写锁

​		与互斥量类似，但读写锁允许更高的并行性。其特性为：**写独占，读共享**。

### 读写锁特性

1. 读写锁是“写模式加锁”时， 解锁前，所有对该锁加锁的线程都会被阻塞。

2. 读写锁是“读模式加锁”时， 如果线程以读模式对其加锁会成功；如果线程以写模式加锁会阻塞。

3. 读写锁是“读模式加锁”时， 既有试图以写模式加锁的线程，也有试图以读模式加锁的线程。那么读写锁会阻塞随后的读模式锁请求。优先满足写模式锁。**读锁、写锁并行阻塞，写锁优先级高**

​     读写锁也叫共享-独占锁。当读写锁以读模式锁住时，它是以共享模式锁住的；当它以写模式锁住时，它是以独占模式锁住的。**写独占、读共享。**

​     读写锁非常适合于对数据结构读的次数远大于写的情况。

​	相较于互斥量而言，适合读模式较多的情况。

### 读写锁函数

#### pthread_rwlock_init函数

初始化一把读写锁

​     int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);

​     参2：attr表读写锁属性，通常使用默认属性，传NULL即可。

#### pthread_rwlock_destroy函数

销毁一把读写锁

​     int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);

#### pthread_rwlock_rdlock函数

以读方式请求读写锁。（常简称为：请求读锁）

  int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);

#### pthread_rwlock_wrlock函数

以写方式请求读写锁。（常简称为：请求写锁）

  int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);

#### pthread_rwlock_unlock函数

解锁

​     int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);

#### pthread_rwlock_tryrdlock函数

非阻塞以读方式请求读写锁（非阻塞请求读锁）

int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);

#### pthread_rwlock_trywrlock函数

非阻塞以写方式请求读写锁（非阻塞请求写锁）

​     int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);

**以上7 个函数的返回值都是：成功返回0， 失败直接返回错误号。** 

​     pthread_rwlock_t类型 用于定义一个读写锁变量。

​     pthread_rwlock_t rwlock;

```c
int counter;                    //全局变量
pthread_rwlock_t rwlock;

void *th_write(void *arg)
{
    int t;
    int i = (int)arg;

    while (1) {
        t = counter;
        usleep(1000);

        pthread_rwlock_wrlock(&rwlock);
        printf("=======write %d: %lu: counter=%d ++counter=%d\n", i, pthread_self(), t, ++counter);
        pthread_rwlock_unlock(&rwlock);

        usleep(10000);
    }
    return NULL;
}

void *th_read(void *arg)
{
    int i = (int)arg;

    while (1) {
        pthread_rwlock_rdlock(&rwlock);
        printf("----------------------------read %d: %lu: %d\n", i, pthread_self(), counter);
        pthread_rwlock_unlock(&rwlock);

        usleep(900);
    }
    return NULL;
}

void my_create_pthread()
{
    int i;
    pthread_t tid[8];                       //存储线程id
    //初始化读写锁
    pthread_rwlock_init(&rwlock,NULL);

    //创建线程
    for(i=0;i<3;++i)
    {
        pthread_create(&tid[i],NULL,th_write,(void *)i);
    }

    for(i=0;i<5;++i)
    {
        pthread_create(&tid[i+3],NULL,th_read,(void *)i);
    }

    for(i=0;i<8;++i)
    {
        pthread_join(tid[i],NULL);
    }

    //销毁读写锁
    pthread_rwlock_destroy(&rwlock);
    
}
```

## 条件变量

条件变量本身不是锁！但它也可以造成线程阻塞。通常与互斥锁配合使用。给多线程提供一个会合的场所。

### 主要应用函数

#### pthread_cond_init函数

初始化一个条件变量

int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);         

参2：attr表条件变量属性，通常为默认值，传NULL即可

也可以使用静态初始化的方法，初始化条件变量：

pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

#### pthread_cond_destroy函数

销毁一个条件变量

int pthread_cond_destroy(pthread_cond_t *cond);

#### pthread_cond_wait函数

阻塞等待一个条件变量

  int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);

函数作用：

1.**阻塞**等待条件变量cond（参1）满足 

2.释放已掌握的互斥锁（**解锁互斥量**）相当于pthread_mutex_unlock(&mutex);

 **1、2两步为一个原子操作。**

3.当被唤醒，pthread_cond_wait函数返回时，**解除阻塞**并**重新申请获取互斥锁**pthread_mutex_lock(&mutex);

#### pthread_cond_timedwait函数

限时等待一个条件变量

int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime);

​     参3：   参看man sem_timedwait函数，查看struct timespec结构体。

​         struct timespec {

​             time_t tv_sec;      /* seconds */ 秒

​             long  tv_nsec;   /* nanosecondes*/ 纳秒

​         }                                   

形参abstime：绝对时间。                                         

如：time(NULL)返回的就是绝对时间。而alarm(1)是相对时间，相对当前时间定时1秒钟。  

​              struct timespec t = {1, 0};

​              pthread_cond_timedwait (&cond, &mutex, &t); 只能定时到 1970年1月1日 00:00:01秒(早已经过去) 

​          正确用法：

​              time_t cur = time(NULL); 获取当前时间。

struct timespec t;  定义timespec 结构体变量t

​              t.tv_sec = cur+1; 定时1秒

pthread_cond_timedwait (&cond, &mutex, &t); 传参              参APUE.11.6线程同步条件变量小节

​         在讲解setitimer函数时我们还提到另外一种时间类型：

​    struct timeval {

​       time_t   tv_sec; /* seconds */ 秒

​       suseconds_t tv_usec;  /* microseconds */ 微秒

​    };

#### pthread_cond_signal函数

唤醒至少一个阻塞在条件变量上的线程

int pthread_cond_signal(pthread_cond_t *cond);

#### pthread_cond_broadcast函数

唤醒全部阻塞在条件变量上的线程

  int pthread_cond_broadcast(pthread_cond_t *cond);

### 生产者消费者条件变量模型

线程同步典型的案例即为生产者消费者模型，而借助条件变量来实现这一模型，是比较常见的一种方法。假定有两个线程，一个模拟生产者行为，一个模拟消费者行为。两个线程同时操作一个共享资源（一般称之为汇聚），生产向其中添加产品，消费者从中消费掉产品。

![image-20201025170242888](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201025170242888.png)

```c
/*借助条件变量模拟 生产者-消费者 问题*/
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <stdio.h>

/*链表作为公享数据,需被互斥量保护*/
struct msg {
    struct msg *next;
    int num;
};

struct msg *head;
struct msg *mp;

/* 静态初始化 一个条件变量 和 一个互斥量*/
pthread_cond_t has_product = PTHREAD_COND_INITIALIZER;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void *consumer(void *p)
{
    for (;;) {
        pthread_mutex_lock(&lock);
        while (head == NULL) {           //头指针为空,说明没有节点    可以为if吗
            pthread_cond_wait(&has_product, &lock);		//阻塞等待条件变量,然后解锁
             //pthread_cond_wait再返回时，会重新加锁
        }
        mp = head;      
        head = mp->next;    //模拟消费掉一个产品
        pthread_mutex_unlock(&lock);

        printf("-Consume ---%d\n", mp->num);
        free(mp);
        mp = NULL;
        sleep(rand() % 5);
    }
}

void *producer(void *p)
{
    for (;;) {
        mp = malloc(sizeof(struct msg));
        mp->num = rand() % 1000 + 1;        //模拟生产一个产品
        printf("-Produce ---%d\n", mp->num);

        pthread_mutex_lock(&lock);
        mp->next = head;
        head = mp;
        pthread_mutex_unlock(&lock);

        pthread_cond_signal(&has_product);  //将等待在该条件变量上的一个线程唤醒
        sleep(rand() % 5);
    }
}

int main(int argc, char *argv[])
{
    pthread_t pid, cid;
    srand(time(NULL));

    pthread_create(&pid, NULL, producer, NULL);
    pthread_create(&cid, NULL, consumer, NULL);

    pthread_join(pid, NULL);
    pthread_join(cid, NULL);

    return 0;
}
```

一个生产者多个消费者

```c
struct msg {
    struct msg *next;
    int num;
};

struct msg *head;
struct msg *mp;

/* 静态初始化 一个条件变量 和 一个互斥量*/
pthread_cond_t has_product = PTHREAD_COND_INITIALIZER;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void *consumer(void *p)
{
    for (;;) {
        pthread_mutex_lock(&lock);
        while (head == NULL) {           //头指针为空,说明没有节点    可以为if吗。如果有多个消费者，必须用while
            pthread_cond_wait(&has_product, &lock);         //阻塞等待条件变量，然后解锁
        }
        //pthread_cond_wait再返回时，会重新加锁
        mp = head;          //消费
        head = mp->next;    //模拟消费掉一个产品
        pthread_mutex_unlock(&lock);

        printf("-Consume ---%d\n", mp->num);
        free(mp);									//消费完，需要释放
        mp = NULL;
        sleep(rand() % 5);
    }
}

void *producer(void *p)
{
    for (;;) {
        mp = malloc(sizeof(struct msg));
        mp->num = rand() % 1000 + 1;        //模拟生产一个产品
        printf("-Produce ---%d\n", mp->num);

        pthread_mutex_lock(&lock);
        mp->next = head;
        head = mp;
        pthread_mutex_unlock(&lock);

        pthread_cond_signal(&has_product);  //将等待在该条件变量上的一个线程唤醒
        sleep(rand() % 5);
    }
}

//创建线程
void my_create_consumer_product()
{
    pthread_t pid, cid[4];
    srand(time(NULL));

    pthread_create(&pid, NULL, producer, NULL);
    pthread_create(&cid[0], NULL, consumer, NULL);
    pthread_create(&cid[1], NULL, consumer, NULL);
    pthread_create(&cid[2], NULL, consumer, NULL);
    pthread_create(&cid[3], NULL, consumer, NULL);

    pthread_join(pid, NULL);
    pthread_join(cid[0], NULL);
    pthread_join(cid[1], NULL);
    pthread_join(cid[2], NULL);
    pthread_join(cid[3], NULL);

    return 0;
}
```

## 信号量

可以应用于进程、线程间同步

进化版的互斥锁（1 --> N）

​    由于互斥锁的粒度比较大，如果我们希望在多个线程间对某一对象的部分数据进行共享，使用互斥锁是没有办法实现的，只能将整个数据对象锁住。这样虽然达到了多线程操作共享数据时保证数据正确性的目的，却无形中导致线程的并发性下降。线程从并行执行，变成了串行执行。与直接使用单进程无异。

​     信号量，是相对折中的一种处理方式，既能保证同步，数据不混乱，又能提高线程并发

### 信号量基本操作：

sem_wait:    1. 信号量大于0，则信号量--       （类比pthread_mutex_lock）

​      |          2. 信号量等于0，造成线程阻塞

​     对应

​      |

​     sem_post：  将信号量++，同时唤醒阻塞在信号量上的线程    （类比pthread_mutex_unlock）

但，由于sem_t的实现对用户隐藏，所以所谓的++、--操作只能通过函数来实现，而不能直接++、--符号。

**信号量的初值，决定了占用信号量的线程的个数。**

### 主要应用函数

#### sem_init函数

初始化一个信号量

​     int sem_init(sem_t *sem, int pshared, unsigned int value);

​     参1：sem信号量 

参2：pshared取0用于线程间；取非0（一般为1）用于进程间 

参3：value指定信号量初值

#### sem_destroy函数

销毁一个信号量

​     int sem_destroy(sem_t *sem);

#### sem_wait函数

给信号量加锁 -- 

​     int sem_wait(sem_t *sem);

#### sem_post函数

给信号量解锁 ++

​      int sem_post(sem_t *sem); 

#### sem_trywait函数

尝试对信号量加锁 --  (与sem_wait的区别类比lock和trylock)

​      int sem_trywait(sem_t *sem);   

#### sem_timedwait函数

限时尝试对信号量加锁 --

​     int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);

​     参2：abs_timeout采用的是绝对时间。           

​     定时1秒：

​         time_t cur = time(NULL); 获取当前时间。

struct timespec t;  定义timespec 结构体变量t

​         t.tv_sec = cur+1; 定时1秒

​         t.tv_nsec = t.tv_sec +100; 

sem_timedwait(&sem, &t); 传参

### 生产者消费者信号量模型

```c
/*信号量实现 生产者 消费者问题*/

#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <stdio.h>
#include <semaphore.h>

#define NUM 5               

int queue[NUM];                                     //全局数组实现环形队列
sem_t blank_number, product_number;                 //空格子信号量, 产品信号量

void *producer(void *arg)
{
    int i = 0;

    while (1) {
        sem_wait(&blank_number);                    //生产者将空格子数--,为0则阻塞等待
        queue[i] = rand() % 1000 + 1;               //生产一个产品
        printf("----Produce---%d\n", queue[i]);        
        sem_post(&product_number);                  //将产品数++

        i = (i+1) % NUM;                            //借助下标实现环形
        sleep(rand()%3);
    }
}

void *consumer(void *arg)
{
    int i = 0;

    while (1) {
        sem_wait(&product_number);                  //消费者将产品数--,为0则阻塞等待
        printf("-Consume---%d\n", queue[i]);
        queue[i] = 0;                               //消费一个产品 
        sem_post(&blank_number);                    //消费掉以后,将空格子数++

        i = (i+1) % NUM;
        sleep(rand()%3);
    }
}

int main(int argc, char *argv[])
{
    pthread_t pid, cid;

    sem_init(&blank_number, 0, NUM);                //初始化空格子信号量为5
    sem_init(&product_number, 0, 0);                //产品数为0

    pthread_create(&pid, NULL, producer, NULL);
    pthread_create(&cid, NULL, consumer, NULL);

    pthread_join(pid, NULL);
    pthread_join(cid, NULL);

    sem_destroy(&blank_number);
    sem_destroy(&product_number);

    return 0;
}
```

