# 1、chmod

## 数字设定法

chmod [+ | - | =] mode 文件名

+：添加

-：减去

=：覆盖

mode:	八进制的0777

​	r：4	读

​	w: 2	写

​	x:  1	执行

​	-：0	没有任何权限

eg: 0756

​	-0:八进制

​	-7：rwx	文件所有者

​	-5：r-x	文件所属组

​	-6：rw-	其他人

题目：

​	--xrwx--x

​	将所有者和同组用户的权限设置为-wx

​		wx:3

​		chmod =331 filename

​	给所有者和所属组减去r

​	chmod -440 filename

# 2、chowm-修改文件所有者或所属组

​	chown 新的文件所有者 文件名

​	chown 新的所有者：新的组 文件名

# 3、chgrp-修改文件所属组

​	chgrp 新的组 文件名

# 4、文件的查找和检索

## 	1) 根据文件属性查找 -find

​		 **a.根据文件名查找**

​				find 查找的目录 -name "查找的文件名"					//如果不用引号，在使用通配符时可能会报错

​		 **b.根据文件类型查找**

​				find 查找的目录  -type 文件类型

​				普通文件：f

​				目录：d

​				符号链接：l

​				管道：p

​				套接字：s

​				字符设备：c

​				块设备：b

​				查找当前目录的普通文件：find . -type f

​		**c.文件大小**

​			find 查找目录  -size -10M

​			+:大于

​			-：小于

​			等于10k:10k

​			大于10k小于100k

​			find 当前目录 -size +10k  -size -100k

​		**d.按日期**

​			**创建的日期**：-ctime -n/+n

​				-n:n天以内

​				+n:n天以外

​			**修改日期**:   -<font color='red'>mtime</font> -n/+n

​			**访问日期**：-<font color='red'>atime</font> -n/+n

​		**e.按深度**

​			find查找目录 -<font color='red'>maxdepth</font> num						//查找的最大深度为num

​			find 查找目录 -<font color='red'>mindepth</font> num						//查找的最小深度为num

​		**f.高级查找**

​			eg:查找指定目录，并列出该目录中文件详细信息

​			<font color='red'>find 查找目录 查找类型 -exec shell 命令{}\;</font>

<font color='red'>			find 查找目录 查找类型 -ok shell 命令{}\;</font>

​			find ./ -type d -exec ls -l {} \;

​			另一种：OK

​			ok :比exec安全；原因，会向用户确认是否执行该操作

​			效率高的操作：管道

​			find ./ -type d | **xargs** ls -l(shell 命令)

------

## 2）grep搜索文件内容

- ​	**grep -r(递归查找子目录) “查找的内容” 搜索的路径**

  - eg:搜索家目录中带helloworld字符串的文件

    grep -r "helloworld" ~

## 3)总结

- ​	find 搜索的路径  参数   搜索的内容
- ​    grep 参数  搜索内容    搜索路径

# 5、压缩包的管理

## 1.linux下常见压缩格式：

- .gz   --gzip
- .bz2 -bzip2

## 2.常用压缩命令

​	**tar - 打包**

​		**参数：**

​		c -创建压缩文件

​		x -释放压缩文件

​		v -打印提示信息

​		**f -指定压缩包的名字**

​		z -使用gzip压缩文件		-xxx.tar.gz

​		j -使用bzip2的方式压缩文件		-xxx.tar.bz2

​		**压缩：**

​		tar  参数  压缩包的名字（需要指定后缀）  原材料<font color='red'> -C 指定目录</font>

​		如果是想压缩到当前目录可以不用 -C

​		**解压缩：**

​		tar zxvf  压缩包的名字 -C解压路径

​	**rar** 

​		**rar需要安装**

​		**压缩：**（会自动添加.rar文件后缀）

​		rar a 压缩包名（不用指定后缀）压缩内容

​		压缩目录加参数 -r

​		**解压缩：**

​		rar x 压缩包名 解压路径（不需要指定参数）

​	**zip/unzip**

​		**压缩：**

​		zip 参数 压缩包名 原材料

​		如果有目录： -r

​		**解压缩：**

​		unzip 压缩包的名字 -d 指定解压目录

​		unzip不需要解压参数，但是解压路径需要参数

# 6、linux的目录结构

/bin:binary，二进制文件，可执行程序，shell命令

/dev:device,  <font color='red'>在linux下一切皆文件</font>    设备目录

<font color='red'>/lib</font>: linux运行的时候需要加载的一些动态库

/mnt:手动的挂载目录

/media:外设自动的挂载目录

<font color='red'>/roo</font>t:linux的超级用户的家目录

/usr:unix system resourse 当前linux的资源目录

​		头文件：stdio.h stdlib.h

​		用户安装的程序：/usr/local

/etc:存放的配置文件

​		/etc/passwd:存放的当前系统的用户的信息

​		/etc/group:存放的用户组信息

/opt:安装第三方应用程序

/home:linux操作系统所有用户的家目录

/tmp:存放临时文件

​	当重启服务器是，该文件夹会清空

# 7、Makefile

**c++**

```makefile
INC_DIR:= ./
SRC_DIR:= ./
SRCS:=$(wildcard *.cc)
OBJS:= $(patsubst %.cc, %.o, $(SRCS))
LIBS:= -lpthread

CXX:=g++

CXXFLAGS:= -w -g  $(addprefix -I, $(INC_DIR)) $(LIBS) 

EXE:=./server.exe

$(EXE):$(OBJS)
	$(CXX) -o $(EXE) $(OBJS) $(CXXFLAGS)

clean:
	rm -rf $(EXE)
	rm -rf $(OBJS)
```

**c**

```makefile
CC := gcc
RM :=rm -f

out := server
srcs :=$(wildcard *.c)
$(out):  $(srcs)
	$(CC) $^  -o $@ -g -pthread -lcrypt -I /usr/include/mysql/ -lmysqlclient 
.PHONY:clean rebuild 
rebuild:clean $(out)
clean:
	$(RM)  $(out)
```

# 8、open/close

**头文件**

```c
1. #include <sys/types.h>
2. #include <sys/stat.h>
3. #include <fcntl.h>
```

**函数原型**

​	int open（const char * pathname,int flags）

​	int open（const char * pathname,int flags,mode_t mode）	

**返回值**

​	open函数的返回值如果操作成功，它将返回一个文件描述符，如果操作失败，它将返回-1

**flags**

​	必选项 O_RDONLY：只读模式 , 

​				O_WEONLY：只写模式,

​				O_RDWR：可读可写模式

​	可选项

​		O_APPEND： 表示追加，如果原来文件里面有内容，则这次写入会写在文件的最末尾。
​		O_CREAT： 表示如果指定文件不存在，则创建这个文件

**mode**

​	mode参数表示设置文件访问权限的初始值，和用户掩码umask有关，比如0644表示-rw-r–r–。

​	mode只有在flags中有O_CERAT时才作用，如果没有，则mode可以忽略。

**不同点**

​	open函数是Unix下系统调用函数，操作成功返回的是文件描述符，操作失败返回的是-1,

​	fopen是ANSIC标准中C语言库函数，所以在不同的系统中调用不同的内核的API，返回的是一个指向文件结构的指针。

​	同时open函数没有缓冲，fopen函数有缓冲，open函数一般和write配合使用，fopen函数一般和fwrite配合使用。

**文件描述符**	

​	STDIN_FILENO:标准输入-0

​	STDOUT_FILENO:标准输出-1

​	STDERR_FILENO:标准错误-2

# 9、read/write

**作业：**有一个很大的文件，将数据读出来，写进另一个文件中。

**read:**

​	**函数原型：**ssize_t read(int fd,void *buf,size_t count);

​	**参数：**

​		fd:open 函数的返回值

​		buf:缓冲区，存储要读取的数据

​		count:缓冲区能存储的最大字节数sizeof(buf)

​	**返回值：**

​		失败返回-1。

​		成功：

​		>0:读出的字节数

​		=0:文件读完了

​	**write:**

​		函数原型：size_t write(int fd,const void * buf,size_t count);

​		**参数：**

​			fd:open函数返回值

​			buf:要写到文件的数据

​			count:strlen（buf）						//注意不是sizeof(buf)

​		**返回值：**

​			-1：失败。

​			>0：写入到文件的字节数。

# 10、读取一个文件的内容，并写入另一个文件

```c
 	//打开一个文件englis,将该文件内容写入另外一个文件
    int fd=open("/root/project/testCplus/english.txt",O_RDWR);
    printf("fd=%d\n",fd);                      //fd文件描述符

    //打开另一个文件，如果不指定mode,则每次穿件文件时权限可能不一样（不指定会随机生成权限）
    int fd1=open("/root/project/testCplus/temp",O_WRONLY|O_CREAT,0664);
    printf("fd1=%d\n",fd1);

    //read
    char buf[1024];
    int len = read(fd,buf,sizeof(buf));
    printf("%d\n",len);
    while(len>0){

        //数据写入文件中
        //此处需要用len,不能用strlen
        //strlen 是一个函数，它用来计算指定字符串 str 的长度，但不包括结束字符
        int ret=write(fd1,buf,len);
        printf("ret=%d\n",ret);

        //read
        len=read(fd,buf,sizeof(buf));
    }
    //循环结束，代表文件读完
    close(fd);
    close(fd1);
```

# 11、perror和errno

​	

```c
	//打开一个文件englis,将该文件内容写入另外一个文件
    int fd=open("/root/project/testCplus/english.txt",O_RDWR);
    printf("fd=%d\n",fd);                      //fd文件描述符
    if(fd==-1)
        perror("open1");                     //打印错误信息
```

**输出：**

![image-20201011214211101](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201011214211101.png)

全局变量：errno

​	不同的值，对应不同的错误信息

perror(string str):会首先打印字符串str,然后再打印错误信息

# 12、lseek

​	**函数原型：**off_t lseek (int fd,off_t offset,int whence)

​	offset:偏移量

​	whence:

​		SEEK_SET：头部

​		SEEK_CUR：当前位置

​		SEEK_END：尾部

​	**使用：**

​		a. 文件指针移动到头部：

​			lseek(fd,0,SEEK_SET)；

​		b.获取文件指针当前的位置：

​			int len=lseek(fd,0,SEEK_CUR);

​		c.获取文件长度：

​			int len=lseek(fd,0,SEEK_END)；

​	**文件拓展：**

​		文件原大小100k,拓展为1100k

​		lseek(fd,1000,SEEK_END)；

​		write(fd," ",1);											//需要进行一次写操作之后，文件才能拓展成功，写什么内容都可以

# 13、阻塞和非阻塞

​	默认bash是前台程序，./a-启动了一个程序，前台程序变成了./a,bash变成了后台程序，bash变成了后台程序。

./等待用户输入10个字符串，用户实际输入大于10个字符串，剩下的还在缓冲区。

read解除阻塞读缓冲区数据，write,程序结束。

bash从后台变成前台，检测到了缓冲区数据，将数据作为shell命令去执行。

# 14、stat/lstat函数

​	stat命令：stat filename——获取文件信息

​	**头文件:**

​	#include<sys/types.h>

​	#include<sys/stat.h>

​	#include<unistd.h>

​	**文件属性：**

```C++
struct stat  
{   
    dev_t       st_dev;     /* ID of device containing file -文件所在设备的ID*/  
    ino_t       st_ino;     /* inode number -inode节点号*/    
    mode_t      st_mode;    /* protection -保护模式         --重要*/    
    nlink_t     st_nlink;   /* number of hard links -链向此文件的连接数(硬连接)*/    
    uid_t       st_uid;     /* user ID of owner -user id*/    用户id
    gid_t       st_gid;     /* group ID of owner - group id*/    
    dev_t       st_rdev;    /* device ID (if special file) -设备号，针对设备文件*/    
    off_t       st_size;    /* total size, in bytes -文件大小，字节为单位*/    
    blksize_t   st_blksize; /* blocksize for filesystem I/O -系统块的大小*/    
    blkcnt_t    st_blocks;  /* number of blocks allocated -文件所占块数*/    
    time_t      st_atime;   /* time of last access -最近存取时间*/    
    time_t      st_mtime;   /* time of last modification -最近修改时间*/    
    time_t      st_ctime;   /* time of last status change - *///最后一次改变时间（指属性）  
};  
```

**函数原型：**int stat (const *pathname,struct stat * buf);

使用stat获取文件大小

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<string.h>
int main(){
    struct stat st;
    int ret=stat("english.txt",&st);
    if(ret==-1){
        perror("error:");
        exit(1);
    }
    printf("fileSize=%d\n",(int)st.st_size);
    return 0;
}
```

**lstat:**操作链接文件

int lstat (const *pathname,struct stat * buf);

lstat读取的链接文件本身的属性

stat读取的是链接文件指向的文件的属性

# 15、文件属性相关的函数

1.access

​	**函数功能：**测试当前用户指定文件是否具有某种属性

​	**函数原型：**int access (const char *pathname,int mode);

​	**参数：**

​	pathname:文件名

​	mode:4种权限

​		R_OK--读

​		W_OK--写

​		X_OK--执行

​		F_OK--文件是否存在

​	**返回值：**

​		0-有某种权限，或者文件存在

​		-1-没有，或者文件不存在

2、chmod——修改文件权限

​		int chmod(const char*filename,int mode)

​		mode：文件权限，八进制数

3、truncate——修改文件大小

​	int truncate(const char *path,off_t length)

**参数：**

path:文件名

length:文件最终大小

​	1.比原来小，删掉后面的部分

​	2.比原来大，向后拓展

# 16、dup/dup2——复制文件描述符

​	**头文件：**

​		#include<unistd.h>

​	**函数原型：**

​		int dup(int oldfd);		//创建一个新的文件描述符，并返回；该新的描述符和原有的文件描述符指向相同的文件、管道和网络连接。并且dup返回的文件描述符总是取系统当前可用的最小整数值。

​		int dup2(int oldfd,int newfd);	

//假设newfd已经指向一个文件，首先断开与那个文件的连接；然后再指向oldfd指向的文件。			

//假设newfd没有被占用，newfd指向oldfd指向的文件

//返回第一个不小于newfd的整数值



```c
//将stdout的文件描述符指向ps.out文件，这样stdout的东西就能输出到ps.out文件中
    dup2(fd, STDOUT_FILENO);
```

