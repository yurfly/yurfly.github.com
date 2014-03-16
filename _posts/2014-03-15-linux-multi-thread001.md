---
layout: post
title: 创建多线程-Linux多线程001
categories: c
tag: multi-thread
date: 2014-03-15
---
Linux多线程第一篇，创建多线程。Linux系统下的多线程遵循POSIX线程接口，称为pthread。编写Linux下的多线程程序，需要使用头文件pthread.h，连接时需要 使用库libpthread.a。

<!--more-->

###目录

*   pthread_create()创建多线程
*   pthread_join()
*   pthread_exit()



--------------



####pthread_create() 用来创建一个线程

例1：

{% highlight c linenos%}
void *Thread_Function(void * parameter)
{
    int  param = *(int*)parameter;
    printf("I'am a thread, parameter is %d \n",param);
}

int main()
{ 
    int parameter;//!@ 传进线程函数的参数
    pthread_t    thread_id;//!@ 线程标识符
    int res = pthread_create(&thread_id, NULL, Thread_Function, (void*)&parameter));
    if(0 == res)
    {
        printf("Create Success !");
    }else
    {
        printf("Create Failed!");
        exit(0);
    }
    pthread_join( thread_id, NULL);
    return 0;
}
{% endhighlight %}

####参数说明:
* 第1个参数：线程标识符 的指针
* 第2个参数：用来设置线程属性，一般设为空指针，这样将生成默认属性的线程
* 第3个参数：线程运行函数的起始地址
* 第4个参数：运行函数的参数

> /\*Create a new thread, starting with execution of START-ROUTINE
   getting passed ARG. Creation attributed come from ATTR. The new
   handle is stored in *NEWTHREAD. */
   
#####函数原型 ： (/usr/include/pthread.h)

{% highlight c%}
extern int pthread_create (pthread_t *__restrict __newthread,
      __const pthread_attr_t *__restrict __attr,
      void *(*__start_routine) (void *),
      void *__restrict __arg);
{% endhighlight %}

####函数补充说明：
* 第1个参数是pthread_t型指针变量。它只在当前进程中保证是唯一的，在不同的系统中pthread_t这个类型有不同的实现，它可能是一个整数值，也可能是个结构体，地址等。在本系统(CentOS 5.5)中，pthread_t本质是无符号长整型,头文件/usr/include/bits/pthreadtypes.h中查得定义
typedef unsigned long int pthread_t;

* 第2个参数用于制定各种不同的线程属性。pthread_attr_t在头文件 /usr/include/bits/pthreadtypes.h 中定义。

{% highlight c%}
typedef union
{
  char __size[__SIZEOF_PTHREAD_ATTR_T];
  long int __align;
} pthread_attr_t;
{% endhighlight %}

属性值不能直接设置，须使用相关函数进行操作，初始化的函数为pthread_attr_init，这个函数必须在pthread_create函数之前调用。属性对象主要包括是否绑定、是否分离、堆栈地址、堆栈大小、优先级。默认的属性为非绑定、非分离、缺省1M的堆栈、与父进程同样级别的优先级

* 第3个参数，新创建的线程从这个函数的首地址开始运行

* 第4个参数，如果需要传递多个参数，可以把这些参数放到一个结构中，然后传入结构体指针。

####返回值：
* 若成功则返回0，否则返回错误代码
* 常见的错误返回代码有：

> EAGAIN    表示系统限制创建新的线程，例如线程数量超过系统限制

> EINVAL   表示第二个参数代表的线程属性值非法。

* 创建线程成功后，新创建的线程开始运行参数3和参数4确定的函数，原来的线程则继续运行下一行代码。因此可以用函数pthread_join()来等待一个线程的结束。



----------------



pthread_join() 用来等待一个线程的结束

####参数说明:
* 第1个参数： 被等待线程 的标识符
* 第2个参数： 用户定义的指针，它可以用来存储被等待线程的返回值

> /* Make calling thread wait for termination of the thread TH. The
   exit status of the thread is stored in *THREAD_RETURN, if THREAD_RETURN
   is not NULL.
   This function is a cancellation point and therefore not marked with
   __THROW. */
   
####函数原型 ： (/usr/include/pthread.h)

{% highlight c%}
extern int pthread_join __P ((pthread_t __th, void **__thread_return));
{% endhighlight %}

###函数补充说明：
这个函数是一个线程阻塞的函数，调用它的函数将一直等待，直到被等待的线程结束为止，当函数返回时，被等待线程的资源被收回。
一个线程不能被多个线程等待，否则第一个接收到信号的线程成功返回，其余调用 pthread_join() 的线程则返回错误代码ESRCH。

####子线程退出方式：

####如果需要只终止某个线程而不终止整个进程，可以有三种方法：

* 从线程函数return，函数结束了，调用它的线程也就结束了。
不过，这种方法对主线程不适用，从main函数return，相当于调用exit。
(如果进程中任一线程调用了exit,_Exit或者_exit,那么整个进程就会终止。)

* 一个线程可以调用pthread_cancel()终止同一进程中的另一个线程。注意 pthread_cancel() 并不等待线程终止，它仅仅提出请求，具体被取消线程的动作跟其线程属性相关。

* 通过函数pthread_exit()来 终止自己

* pthread_exit()用来 结束 一个线程

###参数说明:

* 唯一的参数是函数的返回代码

> /* Terminate calling thread.
   The registered cleanup handlers are called via exception handling
   so we cannot mark this function with __THROW.*/
   
函数原型 ： (/usr/include/pthread.h)

{% highlight c %}
extern void pthread_exit (void *__retval) __attribute__ ((__noreturn__));
{% endhighlight %}

####函数补充说明
* 只要pthread_join()中的第2个参数thread_return不是NULL，这个值将被传递给thread_return。
* 最后要说明的是，一个线程不能被多个线程等待，否则第一个接收到信号的线程成功返回，其余调用pthread_join()的线程则返回错误代码ESRCH。

####头文件：
* pthread_create();
* pthread_join(); 
* pthread_exit(); 

三个函数都在 (/usr/include/pthread.h)中定义

使用之前都需要包含头文件 #include<pthread.h>

####其它说明：
* linux下用C开发多线程程序，Linux系统下的多线程遵循POSIX线程接口，称为pthread。
* Linux多线程函数位于libpthread共享库中，编译的时候要加上 -lpthread 选项，以便连接时调用静态链接库libpthread.a，因为pthread并非Linux系统的默认库

####线程属性集的操作和更改：

* 对线程属性集的操作和更改都必须在调用 pthread_create() 之前进行。对属性集的后续更改不会影响已创建的线程。属性集可以在多个 pthread_create() 函数中使用。

> pthread_attr_init() 初始化属性集，以便在 pthread_create()调用中使用。

> pthread_attr_destroy()  删除属性集的内容。

> pthread_attr_getdetachstate()    线程是分离状态还是可连接状态

> pthread_attr_getguardsize()      线程的守护区大小

> pthread_attr_getinheritsched()   继承或从属性对象中获得调度策略和关联属性

> pthread_attr_getprocessor_np()   将线程绑定到特定处理器

> pthread_attr_getschedparam()     设置线程关联属性对象

> pthread_attr_getschedpolicy()    设置线程的调度策略

> pthread_attr_getscope()          

> pthread_attr_getstackaddr()      设置线程的栈基址

> pthread_attr_getstacksize()      设置线程的栈大小

每个函数都有对应的set函数，比如pthread_attr_setdetachstate()等。

一般在程序开始时初始化多个属性集，以便后面使用。

####线程都和进程的联系：
* 默认情况下，创建的线程都和进程有联系，此时可以用 pthread_join() 函数等待特定的线程结束，线程结束后该函数返回，系统收回资源。
* 如果主线程不掉用 pthread_join() 等待子线程结束的话，那么线程结束时资源不会收回，直到整个进程结束资源才会收回。
* 但是如果该线程与进程分离，变为独立线程的话，就不能用 pthread_join() 函数了，否则会出错。
* 函数 pthread_detach() 可以使某个线程变为分离线程，分离线程结束时系统会自动收回资源。


