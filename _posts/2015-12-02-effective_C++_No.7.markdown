---
layout: post
title:  "Effective C++ No.7 다형성을 가진 기본 클래스에서는 소멸자를 반드시 가상 소멸자로 선언하자"
date: 2015-12-02
categories: Effective_C++
tags: C++ Effective_C++
---

# 가상 소멸자

TimeKeeper 정도의 이름을 가진 클래스를 기본 클래스로 생성하고 이 기본 클래스를 상속 받아 적절한 용도에 맞게 설계한다.  

```c++
class TimeKeeper
{
public:
    TimeKeeper();
    ~TimeKeeper();
    ...
};

class AtomicClock : public TimeKeeper
{
    ...
};

class WaterClock : public TimeKeeper
{
    ...
};

class WristWatch : public TimeKeeper
{
    ...
};
```

```c++
TimeKeeper * getTimeKeeper();   // TimeKeeper 로 부터 파생된 클래스를 통해
                                // 동적으로 할당된 객체의 포인터를 반환
```

```c++
TimeKeeper * ptk = getTimeKeeper();
...
delete ptk;
```

기본 클래스를 통하여 파생 클래스가 삭제 될때 그 기본 클래스에 비 가상 소멸자가 들어 있으면 프로그램 동작은 정의되지 않는 동작(undefined-behavior)이다.   
대개 그 객체의 파생 클래스 부분이 소멸되지 않는다.  

```c++
#include <iostream>

class BaseClass
{
public:
    BaseClass()
    {
    }

    ~BaseClass()
    {
        std::cout << __FUNCTION__ << std::endl;
    }
};

class TestClass : private BaseClass
{
public:
    TestClass()
    {
    }

    ~TestClass()
    {
        std::cout << __FUNCTION__ << std::endl;
    }

    BaseClass * getBaseClass()
    {
        return this;
    }
};

int main( void )
{
    TestClass  * test = new TestClass();;
    BaseClass  * base;

    base = test->getBaseClass();

    delete base;

    return 0;
}
```

```
returns@returns ~$ ./a.out
~BaseClass
```

파생 클래스 객체를 기본 클래스 포인터로 삭제 할때는 기본 클래스의 소멸자 앞에 virtual 를 붙여 준다.  
이러면 객체 전부가 소멸된다.  

```c++
class TimeKeeper
{
public:
    TimeKeeper();
    virtual ~TimeKeeper();
    ...
};

TimeKeeper * ptk = getTimeKeeper();
..
delete ptkl;
```

```c++
#include <iostream>

class BaseClass
{
public:
    BaseClass()
    {
    }

    virtual ~BaseClass()
    {
        std::cout << __FUNCTION__ << std::endl;
    }
};

class TestClass : private BaseClass
{
public:
    TestClass()
    {
    }

    ~TestClass()
    {
        std::cout << __FUNCTION__ << std::endl;
    }

    BaseClass * getBaseClass()
    {
        return this;
    }
};

int main( void )
{
    TestClass  * test = new TestClass();;
    BaseClass  * base;

    base = test->getBaseClass();

    delete base;

    return 0;
}
```

```
returns@returns ~$ ./a.out
~TestClass
~BaseClass
```

가상 소멸자를 갖고 있는 않은 클래스는 기본 클래스로 사용하지 않아야 한다.

기본 클래스로 사용되지 않을 클래스의 소멸자를 가상으로 선언하는것은 좋지 않다.  

# 가상 함수 테이블 포인터, 가상 함수 테이블

가상 함수를 c++ 에서 구현하려면 클래스에 별도 자료 구조가 하나 들어 간다.  
이 자료구조는 프로그램 실행중에 주어진 객체에 대해 어떤 가상 함수를 호출해야 하는지를 결정하는데 쓰이는 정보이다.  
실제로는 포인터 형태를 취하고, 대개 vptr[가상함수 테이블포인터(virtual table pointer)]라는 이름으로 불린다.  
vptr은 가상함수의 주소, 즉 포인트들의 배열을 가리키고 있으면 가상 함수 포인터 배열은 vtbl[가상 함수 테이블(virtual table)]이라고 불린다.  
가상함수를 하나라도 갖고 있는 클래스는 반드시 그와 관련된 vtbl을 가지고 있다.  

가상 함수 호출이 어떻게 구현 되었는가에 대한것 보다 클래스에 가상함수가 들어 가면 객체의 크기가 커진다.  
이것은 vptr 과 vtbl 정보를 갖고 있기 때문이다.  

# 가상함수

어떤 경우를 막론하고 소멸자를 전부 virtual 로 선언해서는 안된다.  
가상 소멸자를 선언하는 것은 그 클래스에 가상함수가 하나라도 있는 경우로 한정하자.  

가상 함수가 없지만 비 가상 소멸자 때문에 문제가 발생할수도 있다.

```c++
class SpecialString : public std::string
{
  ...  
};

SpecialString * pss = new SpecialString( "Impending Doom" );

std::string   * ps;

ps = pss;

delete ps;           // 정의 되지 않는 동작이 발생
                     // 실제로는 *ps 의 SpecialString 부분에 있는 자원이 누수
                     // 그 이유는 SpecialString 의 소멸자가 호출되지 않는다.
```

비 가상 소멸자를 가진 표준 컨테이너들의 클래스를 써서 나만의 클래스를 만들지 말어야 한다.

# 순수(pure) 가상 소멸자

순수 가상 함수는 해당 클래스를 추상 클래스(abstract class)로 만든다.  
추상 클래스는 그 자체로는 인스턴스를 못 만다는 클래스이다.  

어떤 클래스는 본래 기본 클래스로 쓰일 목적으로 만들었는데 넣을 만한 순수 가상 함수가 없을 경우에 순수 가상 소멸자를 사용한다.

```c++
class AWOV
{
public:
    virtual ~AWOV() = 0;
};

AWOV::~AWOV()  // 순수 가상 소멸자의 정의
{

}
```

만약 순수 가상 소멸자를 정의 안하면 링크 에러가 발생한다.

```c++
#include <iostream>

class BaseClass
{
public:
    BaseClass()
    {
    }

    virtual ~BaseClass() = 0;
};

class TestClass : private BaseClass
{
public:
    TestClass()
    {
    }

    ~TestClass()
    {
    }

    BaseClass * getBaseClass()
    {
        return this;
    }
};

int main( void )
{
    TestClass    test;
    BaseClass  * base;

    base = test.getBaseClass();

    return 0;
}
```

```
returns@returns ~$ g++ a.cpp
/tmp/ccFRM4wb.o: In function `TestClass::~TestClass()':
a.cpp:(.text._ZN9TestClassD2Ev[_ZN9TestClassD5Ev]+0x1f): undefined reference to `BaseClass::~BaseClass()'
collect2: error: ld returned 1 exit status
```

# 소멸자의 동작 순서

상속 계통 구조에서 가장 말단에 있는 파생 클래스의 소멸자가 먼저 호출되는 것을 시작으로,  
기본 클래스 쪽으로 올라가면서 각 기본 클래스의 소멸자가 하나씩 호출된다.  

```c++
#include <iostream>

class BaseClass
{
public:
    BaseClass()
    {
    }
    ~BaseClass()
    {
        std::cout << __FUNCTION__ << std::endl;
    }
};

class TestClass : private BaseClass
{
public:
    TestClass()
    {
    }

    ~TestClass()
    {
        std::cout << __FUNCTION__ << std::endl;
    }
};

int main( void )
{
    TestClass    test;

    return 0;
}
```  

```
returns@returns ~$ ./a.out
~TestClass
~BaseClass
```
# 정리

기본 클래스에 가상 소멸자를 추가 할수 있는 규칙은 다형성(polymorphic) 을 가진 클래스 이다.  
그러니깐 기본 클래스 인터페이스를 통해 파생 클래스 타입을 조작을 허용하도록 설계된 기본 클래스에만 적용해야 한다.

모든 기본 클래스가 다형성을 갖도로고 설계된것은 아니다.  
기본 클래스로 쓰일 수도 있지만 다형성은 갖지 않도록 설계된 클래스도 있다.  
이런 클래스는 기본 클래스의 인터페이스를 통한 파생 클래스의 객체의 조작이 허용되지 않는다.  

> 1. 다형성을 가진 기본 클래스에서는 반드시 가상 소멸자를 선언해야 한다.
>    즉 어떤 클래스가 가상 함수를 하나라도 갖고 있으면, 이 클래스의 소멸자도 가상 소멸자이어야 한다.
> 2. 기본 클래스로 설계되지 않았거나 다형을 갖도록 설계되지 않은 클래스에는 가상 소멸자를 선언하지 말어야 한다.
