---
title: 关于UNIX的文件描述符
tags:
  - UNIX
  - Linux
  - 文件描述符
  - 进程
  - 线程
  - IO
permalink: file-descriptors-for-unix
date: 2016-05-12 19:56:23
---

## 文件描述符表与打开文件表
UNIX中每个进程拥有一个**描述符表**，通过文件描述符可以索引到打开文件表的表项，如图所示
![UNIX文件的表示](http://7xtzty.com1.z0.glb.clouddn.com/e9f7c37aed39b22eb040927bc10d32c9.png)
这里的文件表项（即图中的file结构）记录了文件状态标志、当前文件偏移量、V节点指针、引用数等信息，可以看出，表项实际上是描述了当前进程对文件的访问情况。
## 多进程中的情况
- 若是独立进程各自打开同一文件，它们会拥有各自独立的文件描述符表和打开文件表，所以它们访问同一文件也是互不影响的。
- 子进程会继承父进程的文件描述符表和打开文件表，此时父进程的每个文件表项中的`f_count`项会+1（这点类似于内存管理里的引用计数，只有当`f_count`的值为零的时候该表项才会被系统回收，对一个文件描述符执行`close`操作就是将`f_count`减一）。

由于以上特点，一个多进程的服务器模型是这样的：
```C++
listenfd = socket(...);
bind(listenfd,...);
for( ; ; )
{
    connfd = accept(listenfd,...);
    if((pid = fork()) == 0)     //child
    {
        close(listenfd);
        doit(connfd);   //do something
        close(connfd);
        exit(0);
    }
    close(connfd);      //parent
}
```
上述模型的基本思路是`fork`一个子进程处理与客户端的业务，而父进程继续等待下一个客户的连接，从而完成并发。
注意到子进程返回后第一件事就是`close(listenfd)`，因为`close`只是将引用数`f_count`减一，并不会导致`listenfd`直接关闭。

## 多线程中的情况
一个进程中的多个线程共享一个文件描述符表和打开文件表，所以一个多线程服务器的模型可能是这样的：
```C++
listenfd = socket(...);
bind(listenfd,...);
for( ; ; )
{
    connfdp = malloc(sizeof(int));
    *connfdp = accept(listenfd,...);
    pthread_create(...,thread,connfdp);
}
//Thread routine
void *thread(void* vargp)
{
    int connfd = *((int*)vargp);
    pthread_detach(pthread_self());
    free(vargp);
    doit(connfd);   //do something
    close(connfd);
    return NULL;
}
```
子线程中没有`close(listenfd)`，因为子线程和父线程共享一个打开文件表，不会导致`f_count`增加（注意“继承”与“共享”的差别）。
