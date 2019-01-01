---
layout: post
title:  "Effective C++ No.9 객체 생성 및 소멸 과정중에는 절대로 가상 함수를 호출하지 말자"
date: 2015-12-04
categories: Effective_C++
tags: C++ Effective_C++
---

객체 생성 및 소멸 과정중에는 가상 함수를 호출하면 절대 안된다.  
이유는 우선 호출 결과가 원하는 결과가 아니다.

```c++
class Transaction
{
public:
    Transaction();

    virtual void logTransaction() const = 0;
};

Transaction::Transaction()
{
    ...
    logTransaction();              // 생성자에서 가상 함수를 호출
}

class BuyTransaction : public Transaction
{
public:
    virtual void logTransaction() const;
    ...
};

class SellTransaction : public Transaction
{
public:
    virtual void logTransaction() const;
};
```

```c++
BuyTransaction b;
```

BuyTransaction 생성자가 호출되는 것은 맞다.  
그러나 우선은 Transaction 생성자가 호출되어야 한다.   
파생 클래스 객체가 생성될때 그 객체의 기본 클래스 부분이 파생 클래스 부분 보다 먼저 호출되는것은 정석이다.   
Transaction 생성자에서 마지막 부분에 logTransaction 함수를 호출한다.  
여기서 호출되는 logTransaction 함수는 BuyTransaction 의 logTransaction 이 아니라 Transaction 이 가지고 있는 함수 이다.  

기본 클래스의 생성자가 호출되는 동안에는, 가상함수는 절대로 파생 클래스 쪽으로 내려가지 않는다.  
그 대신, 객체 자신이 기본 클래스 타입인 것 처럼 동작한다.  
기본 클래스 생성 과정에서는 가상함수가 먹히지 않는다.    

이렇게 동작하느 이유는 기본 클래스 생성자가 파생 클래스 생성자 보다 앞서 실행되기 때문에,
기본 클래스 생성자가 돌아가고 있는 시점에서는 파생 클래스의 데이터 멤버는 아직 초기화된 상태가 아니다.  
기본 클래스 생성자에서 호출된 가상함수가 파생 클래스 쪽으로 내려간다면 파생 클래스 만의 멤버를 건드릴 것이다.  
파생 클래스 멤버는 아직 초기화되지 않았다.   
초기화 되지 않는 멤버를 건들이기 때문에 이것은 정의되지 않는 동작(undefined-behavior)이다.   

파생 클래스 객체의 기본 클래스 부분은 생성되는 동안은 그 객체의 타입은 바로 기본 클래스 이다.  
호출 되는 가상함수는 모두 기본 클래스의 것으로 결정될 뿐만 아니라 런타임 타입 정보를
사용하는 언어요소를 사용한다고 해도 이순간엔 모두 기본 클래스 타입으로 취급한다.    

객체가 소멸될(소멸자가 호출될)때도 동일하다.   
파생 클래스의 소멸자가 일단 호출되고 나면 파생 클래스만의 데이터 멤버는 정의되지 않은 값으로 가정한다.   
기본 클래스 소멸자에 진입할 당시의 객체는 기본 클래스 객체가 되며, 모든 C++ 기능들 역시 기본 클래스 자격으로 처리된다.   

```c++
class Transaction
{
public:
    Transaction()
    {
        init();
    }

    virtual void logTransaction() const = 0;
    ...

private:
    void init()
    {
        ...
        logTransaction();        // 비 가상 함수에서 가상 함수를 호출
    }
};
```

이 코드가 실행 되면 logTransaction 은 Transaction 클래스 안에서 순수 가상 함수이기때문에,
대부분의 시스템에서 바로 abort된다.  
logTransaction 함수가 보통 가상함수 이며 Transaction 멤버 버젼이 구현되어 잇을 경우엔
Transaction 의 버젼이 호출된다.  

이런 문제를 피하기 위해서 생성 중이거나 소멸중인 객체에 대해 생성자나 소멸자에서 가상 함수를 호출하는 코드를 생성하지 않고
생성자와 소멸자가 호출하는 모든 함수들이 똑같은 제약을 따르도록해야 한다.  

위와 같은 문제를 해결하기 위해서 logTransaction 을 Transaction 클래스의 비 가상 멤버 함수로 바꾼다.  
그리고 파생 클래스의 생성자들로 하여금 필요한 로그 정보를 Transaction 의 생성자로 넘겨야 하는 규칙을 만든다.  
logTransaction 이 비 가상 함수 이기때문에 Transaction 의 생성자는 이 함수를 안전하게 호출할 수 있다.  

```c++
class Transaction
{
public:
    explicit Transaction( const std::string & logInfo );
    void logTransaction( const std::string & logInfo ) const ; // 이제는 비 가상 함수
    ...
};

Transaction::Transaction( const std::string & logInfo )
{
    ...
    logTransaction( logInfo );
}

class BuyTransaction : public Transaction
{
public:
    BuyTransaction( parameters ) : Transaction( createLogString( parameters ) ) // 로그 정보를 기본 클래스로 넘김
    {
        ...
    }

private:
    static std::string createLogString( parameters );
};
```

createLogString 함수는 기본 클래스 생성자로 넘길 값을 생성하는 용도로 사용되는 도우미 함수 이다.  
정적 멤버로 되어 있기 때문에 생성이 채 끝나지 않은 BuyTransaction 객체의 미 초기화된 데이터 멤버를 자칫 실수로 건드릴 위험도 없다.  

> 생성자 혹은 소멸자 안에서 가상 함수를 호출하지 말자. 가상 함수라도 해도 지금 실행중인 생성자나 소멸자에 해당되는 클래스의 파생 클래스쪽으로 내려가지 않는다.
