---
layout: post
title:  "Effective C++ No.13 자원 관리에는 객체가 그만!"
date: 2015-12-09
categories: Effective_C++
tags: C++ Effective_C++
---

```c++
class Investment
{
    ...
};

Investment * createInvestment();

void f()
{
    Investment * pInv = createInvestment();

    ...

    delete pInv;
}
```
createInvestment 함수로 얻은 객체에 대한 삭제를 하지 않을 경우가 있다.   
... 부분에서 return 를 수행하여 f 함수를 끝내 버린다던지 해서 delete pInv 가 수행되지 않을수 있다.  
 이러면 pInv 는 자원해제가 되지 않어 메모리 릭이 발생한다.  

createInvestment 함수로 얻어낸 자원이 항상 해제되도록 만들 방법은 자원을 객체에 넣고 그 자원 해제를 소멸자가 맡도록 하며,
그 소멸자는 실행 제어가 f 를 떠날때 호출되록 만드는것이다.  
자원을 객체에 넣음으로써 C++ 가 자동으로 호출해 주는 소멸자에 의해 해당 자원을 저절로 해제할수 있다.  

auto_ptr 은 포인터와 비슷하게 동작하는 객체(스마트 포인터, smart pointer)로서 가리키는 대상에 대해 소멸자가 자동으로 delete 를 호출한다.  

```c++
void f()
{
    std::auto_ptr<Investment> pInv( createInvestment() );
    ...
}
```

1. 자원을 획득한 후에 자원 관리 객체에 넘긴다.  
2. 자원 관리 객체는 자신의 소멸자를 사용하여 자원이 확실히 해제되도록한다.  

RAII( Resource Acquisition is Initialization, 자원 획득, 초기화 )  
객체의 안전한 사용을 위해 객체가 쓰이는 범위가 벗어나면 알아서 자원을 해제해 준다.  

auto_ptr 이 자신이 소멸할때 자신이 가리키고 있는 대상에 대해 자동으로 delete 를 수행하기 때문에
어떤 객체를 가리키는 auto_ptr 의 개수가 둘이상이면 절대로 안된다.  
auto_ptr 객체를 복사하면 원본 객체는 null 로 된다.  
복사하는 객체만이 그 자원의 유일한 소유권을 갖는다고 가정한다.  

```c++
std::auto_ptr<Investment> pInv1( createInvestment() );

std::auto_ptr<Investment> pInv2( pInv1 );                // pInv1 은 null

pInv1 = pInv2;                                           // pInv2 는 null
```

동적으로 할당되는 모든 자원에 대한 관리 객체로서 auto_ptr 을 사용하는것은 최선이 아니다.  
STL 컨테이너의 경우엔 원소들이 정상적인 복사 동작을 가져야 한다.  
그렇기 때문에 auto_ptr 은 이들의 원소로 사용하면 안된다.  

auto_ptr 를 사용할수 없는 상황이라면, 그 대안으로 참조 카운팅 방식의 스마트 포인터( reference-counting smart pointer, RCSP )를 사용한다.  
RCSP 는 어떤 자원을 가리키는 외부 객체의 개수를 유지하고 있다가 그 개수가 0이 되면 해당 자원을 자동으로 삭제하는 스마트 포인터이다.  

TR1 에서 제공되는 tr1::shared_ptr(No.54 참조) 이 대표적이 RCSP 이다.  

```c++
void f()
{
    ...
    std::tr1::shared_ptr<Investment> pInv( createInvestment() );
    ...
}
```

```c++
void f()
{
    ...
    std::tr1:shared_ptr<Investment> pInv1( createInvestment() );

    std::tr2::shared_ptr<Investment> pInv2( pInv1 );               // pInv1 과 pInv2 가 같은 객체를 가리키고 있다.  

    pInv1 = pInv2;
    ....
}
```

tr1::shared_ptr 은 auto_ptr 을 쓸수 없는 STL 컨테이너등의 환경에서 사용할수 있다.  

자원을 관리하는 객체를 써서 자원을 관리하는것은 중요하다.  
auto_ptr 과 tr1::shared_ptr 은 그렇게 하는 여러가지 방법중에 몇가지이다.  
tr1::shared_ptr 은 No.14, 18, 54 에서 더 살펴보자.  

auto_ptr 과 tr1::shared_ptr 은 소멸자 내부에서 delete 연산을 사용한다.  
delete [] 연산이 아니다.  ( delete 와 delete[] 의 차이는 No.16 참조)  
동적으로 할당할 배열에 대해 auto_ptr 이나 tr1::shared_ptr 을 사용하면 안된다.  
동적 배열을 사용하였을때 컴파일러 에러 조차 발생하지 않는다.  

```c++
std::auto_ptr<std::string> aps( new std::string[10] );

std::tr1::shared_ptr<int> spi( new int[1024] );
```

동적 할당된 배열을 위해 준비된 auto_ptr 이나 tr1::shared_ptr 같은 클래스가 제공되지 않는다.  
동작으로 할당된 배열은 vector 나 string 으로 거의 대체 가능 하다.  
boost 를 보면 boost:scoped_array 와 boost::shared_array 를 사용하면 동적 배열에서도 같은 기능을 사용할수 있다.  

자원해제를 일일이 직접 하다 보면(자원 관리 클래스를 사용하지 않고 delete 를 사용하여) 언젠가 잘못을 저지르고 만다는 이야기 이다.  
이럴때 auto_ptr 이나 tr1::shared_ptr 를 사용해야 한다라고 이야기를 하였는데 이런 자원 관리 클래스로도 제대로 관리 할수 없는 자원도 있다.  
이럴 경우 자원관리 클래스를 직접 만들어야 한다.  (No.14, 15)

createInvestment 함수의 반환 타입이 포인터로 되어 있는데, 이 부분때문에 문제가 생길수도 있다.  
반환된 포인터에 대한 delete 호출을 호출자 쪽에서 해야 하는데, 그것을 잊어 버리고 넘어갈수도 있다.  
이 문제는 createInvestment 인터페이스를 수정하여 해결할 수 있다.  
이것은 No18 에서 확인하자.  

> 1. 자원 누출을 막기 위해 생성자안에서 자원을 획득하고 소멸자에서 그것을 해제하는 RAII 객체를 사용하자.  
> 2. 일반적으로 널리 쓰이는 RAII 클래스는 tr1::shared_ptr 그리고 auto_ptr 이다.  
>    이 둘 가운데 tr1::shared_ptr 이 복사 시의 동작이 직관적이기 때문에 대게 더 좋다.  
>    반면 auto_ptr 은 복사 되는 객체(원본 객체)를 null 로 만든다.  
