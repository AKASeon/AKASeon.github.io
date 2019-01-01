---
layout: post
title:  "Effective C++ No.25 예외를 던지지 않는 swap 에 대한 지원도 생각해보자"
date: 2016-01-06
categories: Effective_C++
tags: C++ Effective_C++
---

swap 함수는 초창기부터 STL에 포함된 이래로 예외 안전성 프로그래밍(No.29) 에 없어서는 안되는 감초역할로서, 자기 대입 현상(No.11)의 가능성에 대처하기 위한 대표적인 매커니즘이다.  

```c++
namespace std
{
    template<typename T>
    void swap( T & a, T & b )
    {
        T temp(a);
        a = b;
        b = temp;
    }
}
```

표준에서 기본적으로 제공하는 swap 의 구현코드는 복사만 제대로 지원하는(복사 생성자 및 복사 대입 생성자를 통해) 타입이기만 하면 어떤 타입의 객체이든 맞바꾸기 동작을 수행한다.  

하지만 표준 swap 의 동작은 한 번 호출하여 복사가 세번 일어난다.  

복사하면
```c++
class WidgetImpl
{
public:
    ...
private:
    int a, b, c;
    std::vector<double> v;                       // 많은 데이터가 저장되어 있을 수 있다.
                                                 // 이것을 복사하면 비용이 많이 든다.  
    ...
};

class WidgetImpl
{
public:
    Widget( const Widget & rhs );

    Widget & operator = ( const Widget & rhs )
    {
        ...
        * pImpl = * (rhs.pImpl);
        ...
    }
    ...
private:
    WidgetImpl * pImpl;
};
```

이렇게 만들어진 Widget 객체를 직접 맞 바꾼다면 pImpl 포인터만 바꿀것이다.  
그러나 표준 swap 함수는 이렇게 처리되지 않는다.  
표준 swap 함수를 사용하면 Widget 객체를 세 개를 복사하고, WidgetImpl 객체도 세 개를 복사할 것이다.  

std::sawp 에다가 내부 pImpl 포인터만 변경하도록하자.  

```c++
namespace std
{
    // 아직 컴파일되지 않는다.  
    template<>
    void swap<Widget> ( widget & a,              // T 가 widget 의 경우 std:swap 을 특수화한것이다.
                        widget & b )
    {
        swap( a.mImpl, b.mImpl );
    }
}
```

template<> 는 이 함수가 std::swap 의 완전 템플릿 특수화(total template specialization) 함수라는 것을 컴파일러에게 알려주는 부분이다.  
그리고 함수 뒤에 있는 <Widget> 은 T 가 Widget 일 경우에 대한 특수화라는 사실을 알려주는 부분이다.  

이 함수는 컴파일되지 않는다.  
a 와 b 에 들어 있는 pImpl 포인터에 접근하려고 하는데 이들 포인터가 private 멤버이다.  
특수화 함수를 프렌드로 선언할수도 있지만 이것은 표준 템플릿에 쓰인 규칙과 어긋난다.  
그래서 Widget 안에 swap 이라는 public 멤버함수를 선언하고 그 함수가 실제 맞바꾸기를 수행하도록한다.  

```c++
class Widget
{
public:
    ...
    void swap( Widget & other )
    {
        using std::swap;

        swap( pImpl, other.pImpl );
    }
    ...
};

namespace std
{
    template<>
    void swap<Widget> ( Widget & a,
                        Widget & b )
    {
        a.swap( b );
    }
}
```

public 멤버 함수 버젼의 swap 과 이 멤버 함수를 호출하는 std::swap 의 특수화 함수 모두 지원하고 있다.  

그러나 Widget 과 WidgetImpl 이 클래스가 아니라 클래스 템플릿으로 되어 있어서 WidgetImpl 에 저장된 데이터 타입을 매개변수로 바꿀수 있다면 어떻게 될까?  

```c++
template<typename T>
class WidgetImpl { ... };

template<typename T>
class Widget { ... };
```

std::swap 을 특수화 하는데 문제가 있다.  

```c++
namespace std
{
    template<typename T>
    void swap< Widget<T> > ( Widget<T> & a,      // 적합하지 않는 코드
                             Widget<T> & b )
}
```

함수 템플릿(std::swap) 을 부분적으로 특수화해 달라고 컴파일러에게 요청하였다.  
그러나 C++ 은 클래스 템플릿에 대해서는 부분 특수화(partial specialization) 을 허용하지만 함수 템플릿에 대해서는 허용하지 않는다.  
그래서 컴파일이 되지 않는다.  

함수 템플릿을 '부분적으로 특수화'하고 싶을때 취하는 방법은 오버로드 버전을 하나 추가 하는것이다.  

```c++
namespace std
{
    template<typename T>
    void swap( Widget<T> & a,
               Widget<T> & b )
    {
        a.swap( b );                             // 이코드는 아직 유효화 하지 않다.  
    }
}
```

일반적으로 함수 탬플릿은 오버로딩해은 해도 별 문제 없지만, std 는 조금 특별한 namespace 이기 때문에 이 namespace 에 대한 규칙이 다소 특별하다.  
std 내의 템플릿에 대한 완전 특수화는 OK 이지만 std 에 새로운 템플릿을 추가하는것은 OK 가 아니다.  
std 의 영역을 침범하더라도 컴파일까지 되고 실행은 된다.  
그러나 실행되는 결과가 정의되지 않는 행동이다.  
그렇기 때문에 std 에 아무것도 추가하면 안된다.  

그래서 효율 좋은 '템플릿 전용 버전'을 사용하기 위해서는 멤버 swap 을 호출하는 비 멤버 swap을 선언하고 이 비멤버 함수를 std::swap 의 특수화 버전이나 오버로딩 버전으로 선하지만 않으면 된다.  

```c++
namespace WidgetStuff
{
    ...
    template<typename T>
    class Widget{ ... };
    ...

    template<typename T>
    void swap( Widget<T> & a,
               Widget<T> & b )
    {
        a.swap( b );
    }
}
```

이제는 어떤 코드가 두 Widget 객체에 대해 swap 을 호출하더라도 컴파일러는 C++ 의 이름 탐색 규칙에 의해 WidgetStuff namespace 안에서 Widget 의 특수화 버전을 찾는다.  

이 방법은 클래스 템플릿뿐만 아니라 클래스에 대해서도 잘 통하므로 이 방법을 사용하면 된다.  
그러나 클래스에 대해 std::swap 을 특수화해야 하는 이유가 생기기 때문에 효율좋은 클래스 타입 전용의 swap 이 되도록 많은 곳에허 호출되도록 만들고 싶으면 그 클래스와 동일한 namespace 안에 비멤버 버전의 swap 을 만들고 std::swap 의 특수화 버전도 준비해야 한다.  

```c++
template<typename T>
void doSomething( T & obj1,
                  T & obj2 )
{
    ...
    swap( obj1, obj2 );
    ...
}
```

```c++
template<typename T>
void doSomething( T & obj1,
                  T & obj2 )
{
    using std::swap;
    ...
    swap( obj1, obj2 );
    ...
}
```

컴파일러가 위의 swap 을 만났을때 하는 일은 현재의 상황에 맞는 swap 을 찾는 것이다.  
C++ 의 이름 탐색 규칙에 따라, 우선 전역 유효범위 혹은 타입 T 와 동일한 namespace 안에 T 전용의 swap 이 있는지 찾는다.  
T 전용 swap 이 없으면 컴파일러는 그 다음 조건을 찾는다.  
이 함수가 using 선언이 함수 앞부분에 있기 때문에 std 의 swap 을 사용할수도 있다.  
그러나 이런 상황이 되더다도 컴파일러는 std::swap 의 T 전용버전을 일반형 템플릿보다 더 우선적으로 선택하도록 되어 있다.  
그렇기 때문에 T 에 대한 std::swap 의 특수화 버전이 이미 준비되어 있으면 그 특수화 버전을 사용한다.  

원하는 swap 이 호출되도록 만드는 작업은 별로 어렵지 않다.  
그러나 호출문에 한정자를 잘못 붙이면 안된다.  
한정자가 붙게 되면 C++가 호출될 함수를 결정하는 매커니즘에 영향이 있다.  

첫째. 표준에서 제공하는 swap 이 여러분의 클래스 및 클래스 템플릿에 대해 납득할 만한 효율을 보이면 그냥 아무것도 하지 말자.  
둘째. 표준 swap 의 효율이 기대한 만큼 충분하지 못한다면 다음과 같이 수정하자.
1. swap 이라는 이름의 함수를 만들고 이것을 public 으로 선언한다. 이 함수는 예외를 던지면 안된다.  
2. 클래스 혹은 템플릿이 들어 있는 namespace 와 같은 namespace 에 비 멤버 swap 을 만든다. 그리고 1번에서 만든 멤버 함수를 이 비멤버 함수가 호출하도록한다.  
3. 새로운 클래스를 만들고 있다면 그 클래스에 대한 std::swap 의 특수화 버전을 준비하자. 그리고 이 특수화 버전에서도 swap 함수를 호출하도록하자.  
셋째. 사용자 입장에서 swap 을 호출할때 swap 을 호출하는 함수가 std::swap 을 볼수 있도록 using 선언을 반드시 포함하자. 그 다음에 swap 을 호출하되 namespace 한정자를 붙이지 말자.  

멤버 버전의 swap 은 절대로 예외를 던지지 않도록 만들어야 한다.  
그 이유는 swap 을 진짜 쓸모있게 응용하는 방법들중에 클래스가 강력한 예외 안전성 보장(strong exception-safety guarantee)을 제공하도록 도움을 주는 방법이 있기 때문이다.  
그 방법은 No.29 에서 확인할수 있다.  

> 1. std::swap 이 여러운의 타입에 대해 느리게 동작할 여지가 있다면 swap 멤버 함수를 제공하지. 이때 이 함수는 예외를 던지면 안된다.  
> 2. 멤버 swap 을 제공하였으면 이 멤버를 호출하는 비 멤버 swap 도 제공하자. 클래스(템플릿이 아닌)에 대해서는 std::swap 도 특수화 하자.  
> 3. 사용자 입장에서 swap 을 호출할 때는 std::swap 에 대한 using 선언을 넣어준 후에 namespace 한정 없이 swap 을 사용하자.  
> 4. 사용자 정의 타입에 대한 std 템플릿을 완전 특수화하는것은 가능하다.  그러나 std에 어떠 것도 새로 추가히지 말자.  
