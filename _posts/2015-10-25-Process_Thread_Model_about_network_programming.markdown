---
layout: post
title:  "Process, Thread Model about network Programming"
date:   2015-10-25
categories: Network
tags: C Network
---
# 개요
Server 는 여러 Client 의 요청을 처리 할수 있어야 한다.  
또한 Server 는 Client 요청을 처리 하는 동안에도 다른 Client 의 요청도 처리 할수 있어야 한다.  
그러나 Server 가 하나의 Process, Thread 에서 Client 의 요청을 처리 하는 도중에 다른 Client 에 대한 요청을 처리할수는 없다.   
하나의  Client 의 요청에 대한 처리를 완료 하고 다른 Client 의 요청을 처리 하게 된다.   
이럴경우 오랜 시간이 걸리는 Client 요청을 처리 하고 있을때 다른 Client 는 아무런 응답을 받을수 없다.  
그렇기 때문에 여러 Client 에 보내온 요청을 처리 할수 있는 방법중 새로운 Process나 Thread 에서 Client 에 대한 요청을 처리 한다.  
그리고 Client 의 요청을 기다리는 Process 나 Thread 에서는 Client 의 요청이 들어 왔을때 Network Event 에 대한 처리만 하고   
Client 의 요청에 대한 처리는 새로운 Process 나 Thread 가 처리 하도록 한다.  

# Process Model
Server 는 한 Client 의 요청에 대하여 처리 할수 있는 Process 를 생성하고 그에 대한 응답을 Client 에게 준다.  

## fork
```c++
#include <unistd.h>

pid_t    fork( void )
```

fork 시스템 콜을 사용하여 새로운 Process 를 생성하고 자식 Process 에서 Client 에 대한 요청을 처리 하도록한다.  
fork 를 수행 하였을때 자식 Process 는 0 를 결과로 받는다.    
그리고 부모 Process 는 자식 Process 의 PID 를 결과로 받는다.  
fork 시스템 콜은 현재 Process 와 동일한 이미지로 동작하는 Process 를 새로 생성한다.  

```c++
sProcessID = fork();
if ( sProcessID < 0 )
{
    printf( "fork Function is failed\n" );
}
else if ( sProcessID == 0 )
{
    // do something in child process
    (void)TCPServerHandleClient( sClientSocket );

    close( sClientSocket );
    sClientSocket = 0;
    break;
}
else
{
  // do somehting in parent proces
}
```

# Thread Model
Server 는 한 Client 의 요청에 대하여 처리 할수 있는 Thread 를 생성하고 그에 대한 응답을 Client 에게 준다.  
새로운 Thread 를 만들고 그 Thread 에서 응답을 주기 위하여 pthread 를 이용한다.  

## pthread_create
```c++
#include <pthread.h>

int pthread_create( pthread_t               * restrict thread,
                    const pthread_attr_t    * restrict attr,
                    void                    * ( * start_routine )( void * ),
                    void                    * restrict arg );
```
pthread_create 를 사용하여 thread 를 새로 생성한다.  
생성한 이후에 새로운 thread 들은 client 에 대한 처리를 할수 있도록 한다.  
pthread_create 의 3번째 인자에 작업할 Function 을 주어서 Client 에 대한 처리를 할수 있다.  

```c++
if ( pthread_create( &sThread,
                     NULL,
                     TCPServerHandleEvent,
                     (void*)&sClientSocket )
     != 0 )
{
    printf( "pthread_create is failed\n" );
}
```
```c++
void * TCPServerHandleEvent( void * aData )
{
    int        sClientSocket = 0;

    sClientSocket = * (( int * )aData);

    TCPServerHandleClient( sClientSocket );

    close( sClientSocket );
    sClientSocket = 0;

    return NULL;
}
```

# 결론
하나의 Client 에 대한 요청을 하나의 Process 및 Thread 가 처리 하는 방법에 대하여 이야기해 보았다.  
한 Client 의 요청이 들어 올때마나 Process 및 Thread 를 새로 생성할 경우 생성하는 비용에 대한 오버헤드가 발생한다.   
(물론 Thread 의 경우 Thread Pool 기술을 사용하면 이런 비용은 줄일수 있다.)  
그리고 Server 는 이런 Process 및 Thread 를 따로 관리를 해줘야 한다.  
이에 따른 관리 비용이 추가적으로 발생하게 된다.  
또 하나의 Process 나 Thread 가 하나의 Client 밖에 처리 못하기 때문에 많은 Client 가 Server 에 접속 하였을 경우 수많은 Context Switching 이 발생하게 된다.  
위와 같은 방법의 Model 이 잘못되었다고는 볼수 없지만 많은 Client 에 대한 요청이 있고 동시에 많은 처리 하는 Server 에는 적합하지 않다고 볼수 있다.  
물론 많은 Client 에 대해 고민할 필요가 없고 latency 가 중요한 Server 에는 이런 모델을 채용하는것도 나쁘지 않다.  
