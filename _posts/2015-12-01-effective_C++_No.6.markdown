---
layout: post
title:  "Effective C++ No.6 컴파일러가 만들어낸 함수가 필요 없으면 확실히 이들의 사용을 금해 버리자"
date: 2015-12-01
categories: Effective_C++
tags: C++ Effective_C++
---

의도적으로 복사 생성자와 복사 대입 연산자의 사용을 막아야 할 경우가 있다.    
그렇지만 복사 생성자와 복사 대입 연산자를 선언하지 않아도 컴파일러가 자동으로 생성한다.  
그렇기 때문에 복사 생성자와 복사 대입 연산자를 의도적으로 막을수 있기 선언하는 방법에 대해 소개 한다.  

# private 멤버로 선언

외부로 부터의 호출은 차단 할수 있으나 그 클래스의 멤버 함수나 프렌드 함수가 호출할수 있다.  

# 정의를 하지 않는다.

정의 되지 않는 함수를 호출하면 링크 시점에서 에러를 확인할수 있다.  

```c++
class HomeForSale
{
public:
    HomeForSale()
    {
    }

    ~HomeForSale()
    {
    }

private:
    HomeForSale ( const HomeForSale & );                  // 선언만 있다.
    HomeForSale & operator= ( const HomeForSale & );
};

int main( void )
{
    HomeForSale     h1;
    HomeForSale     h2( h1 );
    HomeForSale     h3;

    h3 = h1;

    return 0;
}
```

```
a.cpp: In function 'int main()':
a.cpp:28:5: error: 'HomeForSale::HomeForSale(const HomeForSale&)' is private
     HomeForSale ( const HomeForSale & );
     ^
a.cpp:35:28: error: within this context
     HomeForSale     h2( h1 );
                            ^
a.cpp:29:19: error: 'HomeForSale& HomeForSale::operator=(const HomeForSale&)' is private
     HomeForSale & operator= ( const HomeForSale &);
                   ^
a.cpp:38:8: error: within this context
     h3 = h1;
```

# 링크 시점이 아니라 컴파일 시점에 에러 발생하도록

복사 생성자와 복사 대입 연산자를 private 으로 선언하고 이것을 HomeForSale class 에 넣지 않고 기본 클래스에 넣는다.   
그리고 이 기본 클래스를 상속 받아 HomeForSale 클래스를 선언한다.  

```c++
class Uncopyable
{
protected:
    Uncopyable()
    {
    }
    ~Uncopyable()
    {
    }

private:
    Uncopyable( const Uncopyable & );
    Uncopyable & operator= ( const Uncopyable & );
};

class HomeForSale : private Uncopyable
{
public:
    HomeForSale()
    {
    }

    ~HomeForSale()
    {
    }
};

int main( void )
{
    HomeForSale     h1;
    HomeForSale     h2( h1 );
    HomeForSale     h3;

    h3 = h1;

    return 0;
}
```

```
returns@returns ~$ g++ a.cpp
a.cpp: In copy constructor 'HomeForSale::HomeForSale(const HomeForSale&)':
a.cpp:12:5: error: 'Uncopyable::Uncopyable(const Uncopyable&)' is private
     Uncopyable( const Uncopyable & );
     ^
a.cpp:16:7: error: within this context
 class HomeForSale : private Uncopyable
       ^
a.cpp: In function 'int main()':
a.cpp:35:28: note: synthesized method 'HomeForSale::HomeForSale(const HomeForSale&)' first required here
     HomeForSale     h2( h1 );
                            ^
a.cpp: In member function 'HomeForSale& HomeForSale::operator=(const HomeForSale&)':
a.cpp:13:18: error: 'Uncopyable& Uncopyable::operator=(const Uncopyable&)' is private
     Uncopyable & operator= ( const Uncopyable & );
                  ^
a.cpp:16:7: error: within this context
 class HomeForSale : private Uncopyable
       ^
a.cpp: In function 'int main()':
a.cpp:38:8: note: synthesized method 'HomeForSale& HomeForSale::operator=(const HomeForSale&)' first required here
     h3 = h1;
        ^
```
Uncopyable 로부터의 상속은 public 일 필요는 없다.  
Uncopyable 의 소멸자는 가상 소멸자가 아니여도 된다.  
Uncopyable 클래스는 데이터 멤버가 전혀 없기 때문에 공백 기본 클래스 최적화( empty base class optimization ) 기법이 먹혀들 여지가 있다.  
Uncopyable 클래스는 기본 클래스이기 때문에 다중 상속으로 갈 가능성이 있다. 다중 상속시에는 공백 기본 클래스 최적화 기법이 적용되지 않을수 있다.  

부스트 라이브러리를 보면 Uncopyable 과 같은 역할을 하는 noncopyable 클래스를 제공한다.  

> 컴파일러에서 자동으로 제공하는 기능을 허용치 않으려면, 대응되는 멤버 함수를 private 으로 선언한 후에 구현은 하지 않은채로 두어라.  
> Uncopyable 가 비슷한 기본 클래스를 쓰는것도 한 방법이다.   
