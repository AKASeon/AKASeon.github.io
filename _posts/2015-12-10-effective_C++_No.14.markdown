---
layout: post
title:  "Effective C++ No.14 자원 관리 클래스의 복사 동작에 대해 진지하게 고찰하자"
date: 2015-12-10
categories: Effective_C++
tags: C++ Effective_C++
---

힙에 생성되지 않는 자원은 auto_ptr 이나 tr1::shared_ptr 등의 스마트 포인터로 처리해주기에는 맞지 않다.   

```c++
void lock( Mutex * pm );
void unlock( Mutex * pm );

class Lock
{
public:
    explict Lock ( Mutex * pm ) : mutexPtr( pm )
    {
        lock( mutexPtr );
    }

    ~Lock()
    {
        unlock( mutexPtr );
    }

private:
    Mutex * mutexPtr;
}
```

```c++
Mutex m;

{
    Lock m1( &m );       // Lock 생성자에서 lock 를 획득 한다.
}                        // lock 소멸자에서 unlock 을 수행한다.

{
    Lock m11( &m );
    Lock m12( m11 );     // m12 에 m11 를 복사..
                         // 어떤 동작을 하는것이 올바른가?
}
```

RAII 객체가 복사될때 이루어져야 하는 동작은 다음중에 하나를 선택해야 한다.     
1. 복사를 금한다.   
2. 관리하고 있는 자원에 대해 참조 카운팅을 수행한다.    
3. 관리하고 있는 자원을 진짜로 복사  
4. 관리 하고 있는 자원의 소유권을 옮김  

# 복사를 금한다.    
실제로 RAII 객체가 복사되로록 놔두는거 자체가 말이 안된다.   
복사하면 안되는 RAII 클래스에 대해서는 반드시 복사가 되지 않도록 막아야 한다.   
복사를 막는 방법은 복사 연산 함수를 private 로 변경한다.  
자세한 사항은 No.6 을 참조 하자.   

```c++
class Lock : private Uncopyable
{
public:
    ...
}
```

# 관리하고 있는 자원에 대해 참조 카운팅을 수행    
해당 자원을 참조하는 객체의 개수에 대한 카운트를 증가시키는 식으로 RAII 객체의 복사 동작을 만들어야 한다.   
참고로 이런 방식은 현재 tr1::shared_ptr 에서 사용하고 있다.   

tr1::shared_ptr 은 삭제자를 지정을 허용한다.  
삭제자란 tr1::shared_ptr 이 유지하는 참조 카운트가 0이 되었을때 호출되는 함수나 함수 객체를 말한다.  
tr1::shared_ptr 생성자의 두번째 매개변수로 선택적으로 넣어 줄수 있다.  

```c++
class Lock
{
public:
    explict Lock( Mutex * pm ) : mutexPtr( pm, unlock )
    {
        lock( mutexPtr.get() );
    }

private:
    std::tr1::shared_ptr<Mutex>  mutexPtr;
}
```

여기서 Lock 의 소멸자를 선언하지 않았다.   
No.5 에서 클래스의 소멸자는 비 정적 데이터 멤버의 소멸자를 자동으로 호출한다.""라고 되어 있다.   
여기서 비정적 데이터 멤버는 mutexPtr 이다.   
mutexPtr의 소멸자에서 mutex 참조 카운트가 0이 될때 자동으로 tr1::shared_ptr 의 삭제자가 호출된다.   
여기서 삭제자는 unlock 이다.  

# 관리하고 있는 자원을 진짜로 복사  
자원을 원하는대로 복사 할수도 있다.  
이때는 자원을 다 사용하였을때 각각의 사본을 확실히 해제하는것이 자원 관리 클래스가 가지는 유일한 작업이다.  
자원 관리 객체를 복사하면 그 객체가 둘러 싸고 있는 자원까지 복사되어야 한다.  
즉 깊은 복사(deep copy) 를 수행해야 한다.   

# 관리하고 있는 자원의 소유권을 이동    
그 자원을 실제로 참조하는 RAII 객체는 딱 하나만 존재 하도록 만들고 싶다면 그 RAII 객체가 복사될때 그 자원의 소유권을 사본쪽으로 옮겨야 한다.   
이런 스타일의 복사는 No.13 에서 확인하였다. auto_ptr 의 복사 동작이다.   

객체 복사 함수는 컴파일러에 의해 생성될 여지가 있기 때문에, 컴파일러가 생성한 버젼의 동작이 원하는 동작이 아니면 직접 생성해야 한다.  
No. 5 에서 확인 가능 하다.   
이들 함수의 일반화 버젼도 지원하고 싶을수도 있는데 어떤 버전인지는 No.45 에서 확인 가능 하다.  

> 1. RAII 객체의 복사는 그 객체가 관리하는 자원의 복사 문제를 안고 가기 때문에, 그자원을 어떻게 복사하느냐에 따라 RAII 객체의 복사 동작이 결정된다.   
> 2. RAII 클래스에 구현하는 일반적인 복사 동작은 복사를 금지하거나 참조 카운팅을 해주는 선으로 마무리한다.   
>    하지만 이 외의 방법들도 가능하니 참고해 두자.  
