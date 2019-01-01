---
layout: post
title:  "Effective C++ No.11 operator= 에서는 자기대입에 대한 처리가 빠지지 않도록 하자"
date: 2015-12-06
categories: Effective_C++
tags: C++ Effective_C++
---

자기 대입(self assignment)이란 어떤 객체가 자기자신에 대해 대입 연산자를 적용하는 것이다.  

```c++
class Widget
{
    ...
};

Widget w;
...
w = w;
```

```c++
class Bitmap
{
    ...
};

class Widget
{
    ...

private:
    Bitmap * pb;
};

Widget & Widget::operator= ( const Widget & rhs )
{
    delete pb;
    pb = new Bitmap( * rhs.pb );

    return * this;
}
```

operator= 내부에서  * this(대입 상대)와 rhs가 같은 객체일수 있다.  
둘이 같은 객체이면 delete 연산자가 * this 객체의 비트맵에만 적용되는 것이 아니라 rhs 객체에 까지 적용이 된다.  
즉 Bitmap 객체인 pb 데이터가 사라진다.  

전통적인 방법은 opeator= 의 첫 머리에서 일치성 검사(identity test)를 통해 자기 대입을 점검한다.  

```c++
Widget & Widget::operator= ( const Widget & rhs )
{
    if ( this == & rhs )          // 객체가 같은지 (자기 대입인지) 확인한다.  
    {                             // 객체가 같으면 (자기 대입이면) 아무것도 안한다.  
        reutrn * this;
    }

    delete pb;
    pb = new Bitmap( * rhs.pb );

    return * this;
}
```

이 operator= 는 자기대입에는 안전할지 몰라도 예외에는 아직 문제가 있다.  
new Bitmap 에서 예외가 발생하면 Widget 객체는 결국 삭제된 Bitmap 을 가리키는 포인터를 가지고 있다.  

operator= 를 예외에 안전하게 구현하면 대개 자기대입에도 안전한 코드가 만들어 진다.  
예외 안정성에만 집중하면 자기 대입 문제는 보통 무사히 넘어 갈수 있다.  

```c++
Widget & Widget::operator= ( const Widget & rhs )
{
    Bitmap * pOrig = pb;

    pb = new Bitmap( * rhs.pb );
    delete pOrig;

    return * this;
}
```

new Bitmap 에서 예외가 발생하더라도 pb 는 변경되지 않은 상태를 유지 하기 때문에 예외에 안전하다.  
원래 비트맵을 복사해 놓고, 복사해 놓은 사본을 포인터가 가리키게 만든후, 원본을 삭제 하는 순으로 처리 하기 때문에 자기 대입에도 안전하다.  

효율을 너무 신경쓴 나머지 일치성 테스트를 함수 앞단에 추가 할수도 있다.  
그렇지만 일치성 코드가 들어 가면 그 만큼 코드(소스코드, 목적 코드)가 커지는데다가 처리 흐름에 따른 분기를 만들게 되어 성능에 영향을 줄수도 있다.  
CPU 명령어 선행인출(instruction prefetch), 캐시, 파이프 라이닝 등의 효과도 떨어줄수 있다.  

복사후 바꾸기(copy and swap)기법을 사용하면 다른 방법으로 구현할수 잇다.   
이 기법은 예외 안전성과 아주 밀접한 관계가 있다.(No.29)  

```c++
class Widget
{
    ...
    void swap( Widget & rhs );
    ...
};

Widget & Widget::operator= ( const Widget & rhs )
{
    Widget temp( rhs );

    swap( temp );

    return * this;
}
```

c++ 의 특징을 살리면 조금 다른게 구현 할수도 있다.  
1. 클래스의 복사 대입 연산자는 인자를 값으로 취하도록 선언하는것이 가능하다.  
2. 값에 의한 전달을 수행하면 전달된 대상의 사본이 생긴다.(No.20)

```c++
Widget & Widget::operator= ( Widget rhs )
{
    swap( rhs );

    return * this;
}
```

명확성을 포기하였지만 객체를 복사하는 코드가 함수 본문으로 부터 매개 변수의 생성자로 옮겨졌기 때문에 컴파일러가 더 효율적인 코드를 생성할 수 있는 여지가 있다.  

> 1. operator 를 구현할때는 어떤 객체가 그 자신에 대입되는 경우를 제대로 처리하도록 만든다.  
>    원본 객체와 복사 대상 객체의 주소를 비교해도 되고 문장의 순서를 적절히 조정할수도 있으며 복사후 맞바꾸기 기법을 써도 된다.  
> 2. 두개 이상의 객체에 대해 동작하는 함수가 있다면, 이 함수에 넘겨지는 객체들이 사실 같은 객체인 경우에 정확하게 동작하는지 확인해야 한다.  
