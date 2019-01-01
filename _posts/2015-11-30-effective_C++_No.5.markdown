---
layout: post
title:  "Effective C++ No.5 C++ 가 은근슬쩍 만들어 호출해 버리는 함수에 촉각을 세우자."
date: 2015-11-30
categories: Effective_C++
tags: C++ Effective_C++
---

# 기본으로 생성해주는 함수

c++ 의 어떤 멤버 함수는 클래스안에 직접 선언해 넣지 않으면 컴파일러가 저절로 선언해준다.  
- 복사 생성자 ( copy constructor )  
- 복사 대입 생성 ( copy assignment operator )  
- 소멸자 ( destructor )  
- 생성자  
이들은 모두 public 멤버이고 inline 함수이다.  

```c++
class Empty
{
};
```

컴파일하면 아래 와 같은 코드가 자동으로 생성된다.   

```c++
class Empty
{
public:
    Empty()                                         // 기본생성자
    {
        ...
    }

    Empty ( const Empty & rhs )                    // 복사 생성자
    {
        ...
    }

    ~Empty()                                       // 소멸자
    {
        ...
    }

    Empty & operator= ( const Empty & rhs )        // 복사 대입 연산자
    {
        ...
    }
}
```
# 생성되는 조건

각 생성자나 연산자, 소멸자가 호출되는 코드가 있을 경우 컴파일러가 필요하다고 판단하여 각 코드들을 자동으로 생성한다.  

```c++
Empty e1;                    // 생성자

Empty e2( e1 );              // 복사 생성자

e2 = e1;                     // 복사 대입 연산자
```

# 복사 생성자와 복사 대입 생성자

컴파일러가 만든 복사 생성자와 복사 대입 생성자가 하는일은 원본 객체의 비 정적 데이터를 사본 객체쪽으로 그냥 복사하는것이 전부이다.  

```c++
template<typename T>
class NamedObject
{
public:
    NamedObject( const char * name,
                 const T    & value );

    NamedObject( const std::string & name,
                 const T           & value );


private:
    std::string nameValue;
    T           objectValue;
};
```

NamedObject 탬플릿안에는 생성자가 선언되어 있으므로, 컴파일러는 기본 생성자를 생성하지 않는다.    

복사 생성자나 복사 대입 연산자는 NamedObject 에 선언되어 있지 않기 때문에 이 두 함수의 기본형이 컴파일러에 의해 생성된다.  

```c++
NamedObject<int> no1( "Smallest Prime Number", 2 );

NamedObject<int> no2(no1);                   // 복사 생성자를 호출
```

컴파일러가 생성한 복사 생성자는 no1.nameValue 와 no1.objectValue 를 사용하여 no2.nameValue 와 no2.objectValue 를 각각 초기화 한다.  
nameValue 는 string 이고 표준 string 타입은 자체적으로 복사 생성자를 갖고 있으므로 no2.nameValue 의 초기화는 string 의 복사 생성자에 no1.nameValue 를 인자로 넘겨 호출하여 초기화 한다.  
int 는 기본 제공 타입이므로 no2.objectValue 의 초기화는 no1.objectValue 의 각 비트를 그대로 복사해 오는 것으로 끝난다.  

# 클래스 안에 상수와 참조자가 멤버 변수인 경우

```c++
#include <string>

template<class T>
class NamedObject
{
public:
    NamedObject( std::string  & name,
                 const T      & value ) : nameValue( name ),
                                          objectValue( value )
    {
    };

private:
        std::string & nameValue;                  // 이 멤버는 참조자이다.
        const T       objectValue;                // 이 멤버는 상수이다.
};

int main( void )
{
    std::string newDog( "N" );
    std::string oldDog( "O" );

    NamedObject<int> p( newDog, 2 );
    NamedObject<int> s( oldDog, 36 );

    p = s;
}
```

오류 메시지는 아래와 같다.

```
returns@returns ~$ g++ NamedObject.cpp
NamedObject.cpp: In member function 'NamedObject<int>& NamedObject<int>::operator=(const NamedObject<int>&)':
NamedObject.cpp:4:7: error: non-static reference member 'std::string& NamedObject<int>::nameValue', can't use default assignment operator
 class NamedObject
       ^
NamedObject.cpp:4:7: error: non-static const member 'const int NamedObject<int>::objectValue', can't use default assignment operator
NamedObject.cpp: In function 'int main()':
NamedObject.cpp:26:7: note: synthesized method 'NamedObject<int>& NamedObject<int>::operator=(const NamedObject<int>&)' first required here
     p = s;
```

참조자와 상수를 멤버로 가지는 클래스는 기본 복사 대입 연산자와 기본 복사 생성자를 만들수 없다.  
C++ 의 참조자는 원래 자신이 참조하고 있는 것과 다른 객체를 참조할 수 없다.  

참조자를 데이터 멤버로 갖고 있는 클래스에 대입 연산을 지원하려면 직접 대입 연산자를 정의해 주어야 한다.  

복사 대입 연산자를 private 로 선언한 기본 클래스로 부터 파생된 클래스의 경우, 이 클래스는 암시적으로 복사 대입 연산자를 가질수 없다.  

> 컴파일러는 경우에 따르 클래스에 대해 기본 생성자, 복사 생성자, 복사 대입 연산자, 소멸자를 암시적으로 만들어 놓을수 있다.
