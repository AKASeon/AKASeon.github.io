---
layout: post
title:  "Effective C++ No.26 변수의 정의는 늦출 수 있는 데까지 늦추는 근성을 발휘하자."
date: 2016-01-07
categories: Effective_C++
tags: C++ Effective_C++
---

```c++
std::string encryptPassword( const std::string & password )
{
    using namespace std;

    string encrypted;

    if ( password.length() < MinimumPasswordLength )
    {
        throw logic_error( "Password is too short" );
    }
    ...

    return encrypted;
}
```

예외가 발생하면 encrypted 변수는 사용되지 않는다.  
사용되지 않는 변수에 대해서도 encrypted 객체의 생성과 소멸에 대한 비용이 든다.  
그렇기때문에 encrypted 변수는 사용하기전에 정의하는것이 좋다.  

```c++
std::string encryptPassword( const std::string & password )
{
    using namespace std;

    if ( password.length() < MinimumPasswordLength )
    {
        throw logic_error( "Password is too short" );
    }

    string encrypted;
    ...

    return encrypted;
}
```

encrypted 변수가 정의될 때 초기화 인자가 하나도 없다.  
그렇기 때문에 기본 생성자가 호출 될것이다.  
대부분의 경우 어떤 객체에 값을 주는 작업을 먼저 하게 된다.  
이 때 대개 대입 연산을 사용한다.  
그런데 객체를 기본 생성하고 값을 대입하는 방법은 '특정 값'으로 직접 초기화하는 방법보다 효율이 좋지 않다. (No.4 참조)

```c++
std::string encryptPassword( const std::string & password )
{
    ...
    std::string encrypted;

    encrypted = password;

    encrypt( encrypted );

    return encrypted;
}
```

```c++
std::string encryptPassword( const std::string & password )
{
    ...
    std::string encrypted( password );

    encrypt( encrypted );

    return encrypted;
}
```

어떤 변수를 사용해야 할 때가 올기 전까지 그 변수의 정의를 늦추는 것은 기본이고, 초기화 인자를 얻을수 있기 전까지 정의를 늦출수 있는지 확인하여야 한다.  
이렇게 해야 사용하지 않는 객체가 만들어졌다 없어 지는 일이 없다.  

어떤 변수가 loop 안에서만 사용되면 이 변수는 loop 밖에서 정의하는것이 좋을까? 아님 loop 안에서 정의하는것이 좋을까?

A 방법

```c++
Widget w;

for ( int i = 0; i < n; ++i )
{
    w = i 에 따라 달라지는 값
    ...
}
```

B 방법

```c++
for ( int i = 0; i < n; ++i )
{
    Widget w( i 에 따라 달라지는 값 );
    ...
}
```

- A 방법 : 생성자 1번 + 소멸자 1번 + 대입 n 번
- B 방법 : 생성자 n 번 + 소멸자 n 번

클래스중에 대입에 들어가는 비용이 생성자, 소멸자의 비용보다 적게 나오는 경우가 있다.  
이럴경우 A 방법이 효율이 좋다. 그렇지 않으면 B 방법이 효율이 좋다.  
그렇지만 A 방법은 w 의 유효범위가 B 방법보다 넓어지므로 유지보수성이 안 좋아 질수 있다.  

1. 대입이 생성자-소멸자 쌍보다 비용이 덜들고
2. 전체 코드에서 수행성능에 민감한 부분을 수정하지 않는다면  
B 방법이 보통 더 좋다.  

> 변수의 정의는 늦출수 있을때 까지 늦추자.  
> 프로그램이 더 깔끔해지며 효율도 좋아집니다.  
