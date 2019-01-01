---
layout: post
title:  "select in multiplexing"
date: 2015-11-22
categories: Network
tags: C Network
---

# select
select 는 여러개의 file descriptor 를 감시할 수 있다.  
입출력을 감시하고자 하는 file descriptor 를 fd_set 이라는 file bit 배열에 set 을 한 이후에 이 배열의 값이 변했는지 확인한다.  
fd_set 배열은 일반적으로 1024 개이다.  

## function

```c++
int select( int              nfds,       // 감시해야 되는 FD 중 가장 큰값 보다 하나 큰 값
            fd_set         * readfds,    // 입력
            fd_set         * writefds,   // 출력
            fd_set         * exceptfds,  // 예외
            struct timeval * timeout );  // Timeout
```

select 의 반환 값은 성공시 등록된 File descriptor 의 개수를 반환한다.  
Timeout 으로 지정된 시간동안 File descriptor 의 변화가 없으면 0 을 반환 한다.  
select 함수의 내부 에러의 경우 -1 를 반환하고 errno 에 보다 정확한 오류 번호가 기록 되어 있다.  

```c++
void FD_CLR( int fd, fd_set * set );
```

fd 를 fd_set 배열에서 제거 한다.

```c++
int  FD_ISSET( int fd, fd_set * set );
```

fd_set 배열에 fd 에 대응 되는 비트가 설정되어 있는지 확인한다.  
설정되어 있으면 해당 fd(file descriptor) 의 상태는 변화된것이다.

```c++
void FD_SET( int fd, fd_set * set );
```

fd_set 배열에 fd 에 대응 되는 비트를 설정 한다.

```c++
void FD_ZERO( fd_set * set );
```

fd_set 배열을 초기화 한다.

## 문제점
select 는 입출력 상태의 변화를 감지하여 bit 배열인 fd_set 배열에 값을 설정한다.  
그래서 특정 file descriptor 의 변화를 확인하기 위하여 등록된 모든 file descriptor 를 확인하여야 한다.  
예를들면 fd_set 배열에 100개의 file descriptor 가 등록되어있다고 가정하자.  
이때 하나의 file descriptor 에서 입력이 들어 왔다.  
select 는 입력이 들어 왔기때문에 blocking 모드에서 나온다.  
그러면 개발자는 어느 file descriptor 에서 입력이 들어 왔는지 확인하기 위하여 100개의 file descriptor 를 하나하나 확인하는 코드를 작성하여야 한다.  
지금은 100개의 file descriptor 밖에 없지만 1000개 10000개 file descriptor 가 등록 되면 loop 를 1000번 10000번 돌면서 어디에서 입출력 변화가 있는지 확인하여야 한다.  
물론 이 동안은 입출력 상태의 변화를 감지할 수 없다.  
