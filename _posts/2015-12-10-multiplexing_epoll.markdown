---
layout: post
title:  "epoll in multiplexing"
date: 2015-12-10
categories: Network
tags: C Network
---

# e-poll

## Edge Trigger 와 Level Trigger

### Level Trigger

특정 상태가 유지 되는 동안 감지  
Level Trigger 방식은 select 와 poll 처럼 감지  

### Edge Trigger

특정 상태로 변화 하는 시점에만 감지  
Edge Trigger  방식은 처음 이번트가 발생시에만 감지한다.  
처음 이벤트 발생시에만 감지 하기 때문에 이벤트가 발생하여 데이터를 읽었는데 버퍼의 데이터를 모두 읽지 않고 epoll_wait 함수를 호출하면 영원히 wait 함수에서 못 깨어난다.  

Edge Trigger 로 동작하려면 non-blocking 소켓을 사용해야 한다.  

#### 주의점

1. non-block fd 만 사용해야 한다.  
2. read 나 write 함수호추하여 errno 가 EAGAIN 인지 화인하고 epoll_wait 를 수행해야 한다.  

## API

### epoll_create

``` c++
int epoll_create( int size );
```

#### 매개변수

##### size
size 만큼의 공간을 커널안에 할당 받아야 하기 때문에 size 를 매개변수로 넘겨 준다.  
The size  is not  the  maximum size of the backing store but just a hint to the kernel about how to dimension internal structures.  

#### 반환값
성공 하였으면 - 값이 아닌 정수이다.   
이 값은 file descriptor 이다.   
실패 하였으면 -1 값이 반환되고 errno 에 오류 발생 이유가 기록 되어 있다.  

##### errno

###### EINVAL

size 값이 양수가 아니다.

###### ENFILE

현재 열려 있는 file 의 개수가 시스템 제한에 걸려 있다.

###### ENOMEM

커널안에 공간을 할당 받을수 없다.(메모리 부족)

#### 주의점

epoll_create가 성공하였으면 해당 file descriptor 를 close 를 해줘야 한다.  
커널안에 할당 받은 공간과 기타 자원들을 정리 한다.  

### epoll_wait

```c++
int epoll_wait( int                   epfd,
                struct epoll_event  * events,
                int                   maxevents,
                int                   timeout );
```
#### 매개 변수

##### epfd

epoll_create 시 생성한 file descriptor

##### events

발생한 event 에 관련된 정보가 기록되어 있는 배열

##### maxevents

처리 할수 있는 event 개수
events 배열의 크기..

##### timeout

block 될 timeout 시간    
단위는 millisecond  
0 보다 작으면 이벤트가 발생할때까지 대기  

#### 반환값

성공하였으면 event 가 발생한 file descriptor 의 개수   
timeout 시간 동안 event가 발생하지 않았으면 0 이다.   
실패 하였으면 -1 이고 자세한 오류는 errno 에 기록된다.  

#### errno

###### EBADF

epfd 가 정상적인 file descriptor 가 아니다.  

###### EFAULT

events 가 가리키고 있는 메모리 공간에 쓰기를 할수 없다.  

###### EINTR

timeout 시간 전에 signal handler 에 의하여 interrupt 발생시 발생한다.  

###### EINVAL

epfd 가 epoll file descriptor 가 아니다.  

### epoll_ctl

```c++
int epoll_ctl( int                   epfd,
               int                   op,
               int                   fd,
               struct epoll_event  * event );
```

#### 매개변수

##### epfd

epoll_create 시 생성한 file descriptor  

##### op

epoll 에서 file descriptor 에 대하여 지원하는 연산  

###### EPOLL_CTL_ADD

epfd 에 fd 를 추가하고 event 에 기록되어 있는 정보를 바탕으로 감시를 수행한다.  

###### EPOLL_CTL_MOD

epfd 에 이미 추가 되어 있는 fd 에 대한 event 를 새로운 event 정보로 갱신한다.  
감시 하는 정보가 바뀐다.  

###### EPOLL_CTL_DEL

epfd 에 추가 되어 있는 fd 에 대한 정보를 삭제 한다.  
이때 events 는 NULL 일수 있다.  
kernel 2.6.9 이전에는 NULL 로 하면 안된다.  

##### fd

감시할 fd

##### events

어떤 이벤트를 감시할지에 대한 지시자  
또한 context 정보를 넘겨 줄수 있다.  
자세한 사항은 struct epoll_event 를 확인한다.  

#### 반환값

성공하였으면 0을 반환한다.
실패 하였으면 -1을 반환하고 자세한 오류는 errno에 기록된다.  

#### errno

###### EBADF  

epfd 나 fd 가 정상적인 file descriptor 가 아니다.  

###### EEXIST  

op 가 EPOLL_CTL_ADD 이고 file descriptor fd 가 이미 epfd 에 등록되어 있다.  

###### EINVAL  

epfd 가 epoll file descriptor 가 아니다.  

###### ENOENT  

OP 가 EPOLL_CTL_MOD 나 EPOLL_CTL_DEL 이고 fd 가 epfd 에 등록되어 있지 않다.  

###### EPERM  

fd 가 epoll 를 지원하지 못한다.  

### struct epoll_event

```c++
struct epoll_event
{
    \_\_uint32_t     events;
    epoll_data_t   data;
};
```

#### EPOLL Event

##### EPOLLIN

입력(read)이벤트에 대해서 검사  

##### EPOLLOUT

출력(write)이벤트에 대해서 검사한다.  

##### EPOLLERR

파일지정자에 에러가 발생했는지를 검사  

##### EPOLLHUP

Hang up이 발생했는지 검사  

##### EPOLLPRI

파일지정자에 중요한 데이터가 발생했는지 검사  
##### EPOLLET

파일지정자에 대해서 ET 행동을 설정, 기본 값은 LT  

#### epoll_data_t

```c++
typedef union epoll_data
{
    void      * ptr;
    int         fd;
    \_\_uint32_t  u32;
    \_\_uint64_t  u64;
} epoll_data_t;
```

이벤트 발생시 사용할 context 정보  

## 장점 & 단점

### 장점
select 나 poll 에 비해 효율적이다.  
Realtime Signal에 비해 사용하기 쉽다.  

### 단점

linux 에 한정하여 사용 가능하다.  
