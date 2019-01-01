---
layout: post
title:  "Effective C++ No.8 예외가 소멸자를 떠나지 못하도록 붙들어 놓자."
date: 2015-12-03
categories: Effective_C++
tags: C++ Effective_C++
---

```c++
class Widget
{
public:
    ...
    ~Widget()
    {
        ...
    }
};

void doSomething()
{
    std::vector<Widget> v;
}
```

v 에 들어 있는 widget 이 열개 인데 첫 번째 것을 소멸시키는 도중에 예외가 발생하였다고 가장하자.  
나머지 아홉개는 여전히 소멸되어야 하므로 v 는 이들에 대해 소멸자를 호출한다.  
그런데 이 과정에서 문제가 또 발생하였다고 가정하자.  
두 번째 Widget 에 대해 호출된 소멸자에서도 예외를 던지면 어떻게 될까?  
이미 첫번째 Widget 에서 발생한 예외가 있고 두번째 Widget 이 또 예외를 발생하였다.  
이 두 예외가 동시에 발생한 조건이 어떤 미묘한 조건이냐에 따라 프로그램 실행이 종료되던지 정의되지 않은 동작(undefined-behavior)을 보인다.  


```c++
class DBConn
{
public:
    ...
    ~DBConn()
    {
        db.close();
    }

private:
    DBConnection db;
}
```
소멸자에서 close 와 같은 정리 작업을 수행하게 되면 DBConn 객체는 Connection 에 대한 정리 작업을 따로 신경 써줄 필요가 없다.  

```c++
{
    DBConn dbc( DBConnection::create() );
    ....
}
```
그렇지만 db.close() 함수는 항상 성공만 할수 없다.  
db.close() 에서 예외가 발생한다면 위에서 말한 문제점을 내재 하고 있다.  

DBConn 소멸자 내부에서 예외가 발생하면 아래 2가지중에 한가지 방안을 선택 할수 있다.

- close에서 예외가 발생하면 프로그램을 끝낸다.

```c++
DBConn::~DBConn()
{
    try
    {
        db.close();
    }
    catch ( ... )
    {
        // close 호출이 실패한 로그 작성
        std::abort();
    }
}
```

- close 를 호출한 곳에서 일어난 예외를 무시한다.

```c++
DBConn::~DBConn()
{
    try
    {
        db.conn();
    }
    catch ( ... )
    {
        // close 호출이 실패한 로그 작성        
    }
}
```

```c++
class DBConn
{
public:
    ...
    void close()
    {
        db.close();
        closed = true;
    }

    ~DBConn()
    {
        if ( !closed )
        {
            try
            {
                db.close();
            }
            catch ( ... )
            {
                // close 호출이 실패한 로그 작성
            }
        }
    }

private:
    DBConnection db;
    bool         closed;
}
```

어떤 동작이 예외를 발생시키면서 실패할 가능성이 있고 또 그예외를 처리해야할 필요가 있다면 그 예외는 소멸자가 아닌 다른 함수에서 발생해야 한다.  
예외를 일으키는 소멸자는 정의되지 않은 동작(undefined-behavior)을 발생시킬수 있다.  

> 1. 소멸자에서는 예외가 빠져나가면 안된다. 만약 소멸자 안에서 호출된 함수가 예외를 던질 가능성이 있다면 어떤 예외이던지 소멸자에서 모두 받아낸 후에 무시하던지 프로그램을 종료하던지 해야 한다.  
> 2. 어떤 클래스의 연산이 진행되다가 던진 예외에 대해 사용자가 반응할 필요가 있다면, 해당 연산을 제공하는 함수는 반드시 보통함수(소멸자가 아닌 함수)이어야 한다.  
