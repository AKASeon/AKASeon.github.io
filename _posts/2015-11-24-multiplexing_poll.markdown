---
layout: post
title:  "poll in multiplexing"
date: 2015-11-24
categories: Network
tags: C Network
---
# poll

## function

```c++
int poll( struct poolfd *  ufds,
          unsigned int     nfds,
          int              timeout );
```

## pollfd
```c++
struct pollfd
{
	int fd;           // 감시할 fd
	short events;     // 감시할 이벤트
	short revents;    // 발생한 이벤트
};
```

events 에 감시하고 싶은 이벤트 를 설정한 이후 pollfd 를 첫 인자로 넘겨 주면
events 에 설정한 이벤트중 하나가 발생하였을시 poll 에서 block 이 해제되고
어느 pollfd 에서 이벤트가 발생여부를 판단하고 처리 하면 된다.

## Event

sys/poll.h 파일을 확인해 보면 events 에 설정할수 있는 flag 에 대한 설명이 있다.  
man 으로 확인해 봐도 된다.  

```c++
/*
 * Requestable events.  If poll(2) finds any of these set, they are
 * copied to revents on return.
 */
#define POLLIN      0x0001      /* any readable data available */
#define POLLPRI     0x0002      /* OOB/Urgent readable data */
#define POLLOUT     0x0004      /* file descriptor is writeable */
#define POLLRDNORM  0x0040      /* non-OOB/URG data available */
#define POLLWRNORM  POLLOUT     /* no write type differentiation */
#define POLLRDBAND  0x0080      /* OOB/Urgent readable data */
#define POLLWRBAND  0x0100      /* OOB/Urgent data can be written */

/*
 * FreeBSD extensions: polling on a regular file might return one
 * of these events (currently only supported on local filesystems).
 */
#define POLLEXTEND  0x0200      /* file may have been extended */
#define POLLATTRIB  0x0400      /* file attributes may have changed */
#define POLLNLINK   0x0800      /* (un)link/rename may have happened */
#define POLLWRITE   0x1000      /* file's contents may have changed */

/*
 * These events are set if they occur regardless of whether they were
 * requested.
 */
#define POLLERR     0x0008      /* some poll error occurred */
#define POLLHUP     0x0010      /* file descriptor was "hung up" */
#define POLLNVAL    0x0020      /* requested events "invalid" */

```

## 감시할 이벤트 설정


```c++
void setObserveEvent( pollfd *  aPollfd,
                      int       aFd,
                      Short     aEvent )
{
    aPollfd->fd = aFD;
    aPollfd->events = aEvent;
}
```

## 이벤트 확인

등록된 pollfd 를 하나씩 확인해가며 무슨 이벤트가 발생하였는지 확인하여야 한다.  
어떤 pollfd 에 이벤트가 발생하지 않았어도 그것을 확인 할 수 없기 때문에 등록된 모든 pollfd를 확인하여야 한다.  

```c++
void handleEvent( pollfd  * aPollfd,
                  int       aMaxEventFd )
{
    int     i = 0;

    for ( i = 0; i < aMaxEventFd; i++ )
    {
        if ( ( aPollfd[i].revents & POLLIN ) == POLLIN )
        {
            // handle Input event
        }
        else if ( ( aPollfd[i].revents & POLLERR ) == POLLERR )
        {
            // handle error event
        }
    }    
}
```

## 문제점
select 보다 개선되기는 했지만 poll 역시 감시하고 있는 모든 fd 를 확인하며 어디에서 어떤 이벤트가 발생하였는지 확인을 한다.  
select 처럼 모든 fd 에 대하여 확인할 필요는 없지만 poll 역시 감시 하고 있는 fd 배열을 돌면 확인을 한다.  
적은 fd 수를 유지가 되면 상관이 없지만 감시해야 되는 fd 가 많으면 select 만큼은 아니지만 poll 역시 성능 저하를 피할수 없다.    
