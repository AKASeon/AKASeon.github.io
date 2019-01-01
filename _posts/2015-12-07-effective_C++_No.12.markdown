---
layout: post
title:  "Effective C++ No.12 객체의 모든 부분을 빠짐 없이 복사하자"
date: 2015-12-07
categories: Effective_C++
tags: C++ Effective_C++
---

복사 함수(copying function)  
- 복사 생성자  
- 복사 대입 연산자  

```c++
void logCall( const std::string & rhs );

class Customer
{
public:
    ...
    Customer ( const Customer & rhs );
    Customer & operator= ( const Customer & rhs );
    ...

private:
    std::string name;
};

Customer::Customer( const Customer & rhs ) : name( rhs.name )
{
    logCall( "Customer copy constructor" );
}

Customer & Customer::operator= ( const Customer & rhs )
{
    logCall( "Customer copy assignment operator ");

    name = rhs.name;

    return * this;
}
```

여기 까지는 문제 없이 정상적으로 복사 함수가 동작한다.  

```c++
class Date
{
    ...
};

class Customer
{
public:
    ..

private:
    std::string name;
    Date        lastTransaction;
};
```

class Data 를 추가하면 복사 함수의 동작은 완전 복사가 아니라 부분 복사(partial copy)가 이루어 진다.  
name 은 복사가 되지만 lastTransaction 은 복사가 되지 않는다.  
컴파일러는 여기에 대해서 하나의 warning 조차 알려주지 않는다.  
컴파일러가 자동으로 생성해 주는 복사 함수를 사용하지 않고 직접 복사 함수를 만들었으면
그 복사 함수 안에서 모든 멤버 변수에 대하여 모든 멤버 변수를 복사를 수행해야 한다.  
복사 함수 만이 아니라 생성자에서도 모두 처리해 줘야 한다.  

```c++
class PriorityCustomer : public Customer
{
public:
    ...
    PriorityCustomer ( const PriorityCustomer & rhs );
    PriorityCustomer & operator= ( const PriorityCustomer & rhs );
    ...

private:
    int priority;
};

PriorityCustomer::PriorityCustomer ( const PriorityCustomer & rhs ) : priority( rhs.priority )
{
    logCall( "PriorityCustomer copy constructor" );
}

PriorityCustomer & PriorityCustomer::operator= ( const PriorityCustomer & rhs )
{
    logCall( "PriorityCustomer copy assignment operator" );

    priority = rhs.priority;

    return * this;
}
```

PriorityCustomer 클래스의 복사 함수는 PriorityCustomer의 모든 것을 복사 하고 있는 것 처럼 보이지만,
Customer 로 부터 상속한 데이터 멤버들의 사본은 복사하지 않는다.  

PriorityCustomer 의 복사 생성자에는 기본 클래스 생성자에 넘길 인자들도 명시되어 있지 않아서
PriorityCustomer 객체의 Customer 부분의 멤버 변수는 인자 없이 실행되는 Customer 생성자에 의해 초기화 된다.  
생성자에서는 기본적인 초기화만 수행한다.  

PriorityCustomer 의 복사 대입 연산자는 기본 클래스의 데이터 멤버를 건드릴 시도도 하지 않기 때문에 기본 클래스의 데이터 멤버는 변경되지 않는다.  

파생 클래스에 대한 복사 함수를 생성하기로 하였으면 기본 클래스 부분을 빠지지 않도록 하여야 한다.  
기본 클래스 부분이 private 멤버일 가능성이 높기 때문에 이를 직접 건들기는 어렵다.  
대신 파생 클래스의 복사 함수 안에서 기본 클래스의 복사 함수를 호출하도록 한다.  

```c++
PriorityCustomer::PriorityCustomer ( const PriorityCustomer & rhs ) : Customer( rhs ),
                                                                      priority( rhs.priority )
{
    logCall( "PriorityCustomer copy constructor" );
}

PriorityCustomer & PriorityCustomer::operator= ( const PriorityCustomer & rhs )
{
    logCall( "PriorityCustomer copy assignment operator" );

    Customer::operator= ( rhs );
    priority = rhs.priority;

    return * this;
}
```

객체의 복사 함수를 작성할때는 2가지를 꼭 확인해야 한다.  
1. 해당 클래스의 데이터 멤버를 모두 복사  
2. 이 클래스가 상속한 기본 클래스의 복사 함수도 꼬박 꼬박 호출  

복사 생성자와 복사 대입 연산자의 본문이 비슷하게 생성되는 경우가 자주 있어서, 한쪽에서 다른 쪽을 호출하게 만들어서 코드 중복을 피하고 싶을수도 있다.  
복사 대입 연산자에서 복사 생성자를 호출하는 것은 말이 안된다.   
이미 만들어져 있는 객체를 생성하고 있다.   
복사 생성자에서 복사 대입 연산자를 호출하는 것도 말이 안된다.   
생성자의 역할은 새로 만들어진 객체를 초기화 하는것이지만 대입 연산자의 역할은 이미 초기화가 끝난 객체에 값을 주는것이다. 즉 초기화된 객체에만 적용된다.  

복사 생성자와 복사 대입 연산자의 코드 본문이 비슷 하면 양쪽이 겹치는 부분을 별도의 멤버 함수로 분리 하는 방법도 있다.  
안전할 뿐만 아니라 검증된 방법으로 복사 생성자와 복사 대입 연산자에 나타나는 코드 중복을 제거 하는 방법을 고려해 보아야 한다.  

> 1. 객체 복사 함수는 주어진 객체의 모든 데이터 멤버 및 모든 기본 클래스 부분을 빠뜨리지 말고 복사해야 한다.  
> 2. 클래스의 복사 함수 두개를 구현 할때 한쪽을 이용해서 다른쪽을 구현하려는 시도는 절대로 하지 말자.    
>    그대신 공통된 동작을 제 3의 함수에다 분리해 놓고 양쪽에서 이것을 호출하게 만들어서 해결하자.  
