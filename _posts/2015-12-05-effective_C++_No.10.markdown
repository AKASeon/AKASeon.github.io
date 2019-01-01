---
layout: post
title:  "Effective C++ No.10 대입 연산자는 * this 의 참조자를 반환하게 하자"
date: 2015-12-05
categories: Effective_C++
tags: C++ Effective_C++
---

```c++
int x, y, z;

x = y = z = 15;

x = ( y = ( z = 15 ) );
```

대입 연산은 여러개 가 사슬처럼 엮일 수 있는 재미 있는 성질을 갖고 있다.  
또한 우측 연관(right-associative) 연산이다.  
대입 연산이 사슬 처럼 엮이려면 대입 연산자가 좌별 인자에 대한 참조자를 반환하도록 구현되어야 한다.   
이것은 일종의 관례이다.  

```c++
class Widget
{
public:
    ...
    Widget & operator= ( const Widget & this )
    {
        ...

        return * this;
    }
    ...
};
```

"좌변 객체의 참조자를 반환하게 만들자"라는 규칙은 단순 대입 연산자 말고도 모든 형태의 대입 연산자에서 지켜져야 한다.  

```c++
class Widget
{
public:
    ...
    Widget & operator+= ( const Widget & rhs )
    {
        ...
        return * this;
    }

    Widget & operator= ( int rhs )
    {
        ...
        return * this;
    }
};
```

> 대입 연산자는 * this 의 참조자를 반환하도록 하자.
