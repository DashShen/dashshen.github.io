---
title: 阻塞IO，非阻塞IO的c实现
date: 2016-10-31 18:16:07
tags:
---
一般理解阻塞IO与非阻塞IO基本都是基于文字描述，然后举一个浅显易懂的例子来解释这两个概念。这里我就使用c写两个关于阻塞IO,以及非阻塞IO的例子，会使用到pipe来创建一个管道，利用这个管道,让父进程与子进程之间进行通信，来模拟IO事件。

# 阻塞IO
阻塞IO的代码相对简单，只需要创建管道，然后fork出子进程，父进程发送消息，子进程接受消息。但是，父进程不是立即发送消息，而是等到三秒之后再发送消息，因此子进程只能到消息的到来才能往下执行。代码如下：

```c
#include <stdio.h>
#include <unistd.h>
#include <time.h>

int main() {
  int fds[2];
  char buf[1024];
  pid_t pid;
  pipe(fds);
  int r = fds[0], w = fds[1];
  pid = fork();
  if (pid > 0) {
    char msg[] = "This msg was sended from father";
    // 三秒之后再向自进程发送消息
    sleep(3);
    write(w, msg, sizeof(msg));
    close(r);
    close(w);
  } else if (pid == 0) {
    printf("No message and process is blocking\n");
    // read方法会阻塞，因为还没有收到消息
    read(r, buf, sizeof(buf));
    // 接收到消息之后，再打印输出
    printf("Get message from father: %s\n", buf);
    close(r);
    close(w);
  }
  return 0;
}
```

用于是阻塞的，所以在编译执行代码之后,会在这里卡住：

```shell
$ gcc blocking.c
$ ./a.out
No message and process is blocking                                                                                       
```

然后三秒钟之后，父进程发送消息了，子进程收到之后，将消息打印至控制台：

```shell
$ No message and process is blocking                                                                                       
$ Get message from father: This msg was sended from father                                                                 
```

# 非阻塞IO
非阻塞IO现对于阻塞IO的实现稍微复杂一点，会用到fcntl方法去修改文件描述的状态。将管道的读端的文件描述符该为非阻塞的,然后调用read()方法后，进程不会阻塞，而是会一直去询问是否有IO流到来。如果没有数据到来，则read()方法会返回-1,如果有数据，则会返回数据的长度。下面是具体代码：

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <time.h>

void set_noblocking(int fd);

int main() {
  int fds[2];
  char buf[1024];
  pid_t pid;
  pipe(fds);
  int r = fds[0], w = fds[1];
  set_noblocking(r);
  pid = fork();
  if (pid > 0) {
    char msg[] = "This msg was sended from father";
    // 五秒之后再发送消息
    sleep(5);
    write(w, msg, sizeof(msg));
    close(r);
    close(w);
  } else if (pid == 0) {
    int not_ready = -1;
    while (not_ready < 0) {
      // 每隔一秒轮询消息是否到来
      sleep(1);
      printf("Reader not ready\n");
      not_ready = read(r, buf, sizeof(buf));
      printf("Value of read() reaturned %d\n", not_ready);
    }
    printf("Get message from father: %s\n", buf);
    close(r);
    close(w);
  }
  return 0;
}

void set_noblocking(int fd) {
  // 设置成为为阻塞的fd
  int flags = fcntl(fd, F_GETFL);
  fcntl(fd, F_SETFL, flags | O_NONBLOCK);
}
```

非阻塞IO的执行结果,由于子进程会每隔一秒去询问是否有数据到来，如果没有带来，则会打印not ready。5秒之后，父进程发出了一条消息，子进程收到之后就将消息打印了出来。这样子就模拟了一次非阻塞IO的过程。

```shell
$ gcc noblocking.c
$ ./a.out
Reader not ready                                                                                                         
Value of read() reaturned -1                                                                                             
Reader not ready                                                                                                         
Value of read() reaturned -1                                                                                             
Reader not ready                                                                                                         
Value of read() reaturned -1                                                                                             
Reader not ready                                                                                                         
Value of read() reaturned -1                                                                                             
Reader not ready                                                                                                         
Value of read() reaturned 32                                                                                             
Get message from father: This msg was sended from father
```

总的来说，阻塞的IO会使当前进程一直等到数据的到来，要不然不会执行后续的代码，而非阻塞的情况正好相反。