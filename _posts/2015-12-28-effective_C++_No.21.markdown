---
layout: post
title:  "Effective C++ No.21 함수에서 객체를 반환해야 할 경우에 참조자를 반환하려고 들지 말자"
date: 2015-12-28
categories: Effective_C++
tags: C++ Effective_C++
---

```c++
class Rational
{
public:
    Rational( int number = 0,
              int denominator = 1 );
    ...
private:
    int n, d;

    friend const Rational operator * ( const Rational & lhs,
                                       const Rational & rhs );
};
```

이 클래스의 operator * 는 곱셈 결과를 값으로 반환하도록 선언되어 있다.  
Rational 객체의 생성과 소멸에 들어 가는 비용을 생각해야 한다.  

값이 아닌 참조자를 반환할수 있으면 비용 부담은 없어 진다.  
그렇지만 반환되는 것이 참조자이기 때문에 반환되는 객체가 어느곳에는 존재해야 한다.  
함수 수준에서 새로운 객체를 만드는 방법은 두가지 뿐이다.  
하나는 스택에 만드는것이고 또 하나는 힙에 만드는 것이다.  

```c++
const Rational & operator * ( const Rational & lhs,
                              const Rational & rhs )
{
    Rational result( lhs.n * rhs.n, lhs.d * rhs.d )

    return result;
}
```

생성자가 호출되는것이 싫어서 참조자로 반환을 하였지만 함수 안에서도 역시 생성자를 호출하고 있다.  
그리고 또 다른 문제가 존재 한다.  
이 연산자 함수는 result에 대한 참조자를 반환하는데 result 는 지역객체이다.  
지역객체는 함수가 끝날때 같이 소멸된다.  
그렇기 때문에 위 연산자는 Rational 객체에 대한 참조자를 반환하지 않는다.  
이 함수 안에서 예전에 Rational 객체로 사용되었던 곳을 참조자로 반환한다.   
이미 소멸자가 호출되어 버린 공간이기 때문에 이 함수를 호출한곳에서 반환 받은 객체를 사용하려고 하면 정의되지 않는 동작을 일으킨다.  
(지역 객체의 포인터 역시 동일하게 문제가 발생한다.)  

함수가 반환할 객체를 힙에 생성하였다가 참조자로 반환하는것을 확인해 보자.  

```c++
const Rational & operator * ( const Rational & lhs,
                              const Rational & rhs )
{
    Rational * result = new Rational( lhs.n * rhs.n, lhs.d * rhs.d );

    return * result;
}
```

여전히 생성자가 한번 호출되는것은 같다.  
new 로 할당된 메모리를 초기화 할때 생성자가 호출된다.  
그러나 여기에는 new 로 할당한 객체를 누가 delete 로 삭제를 해주냐하는 문제가 존재 한다.  

```c++
Rational w, x, y, z;

w = x * y * z;                                   // operator * ( operator * ( x, y ),  z ) 와 같다.
```

여기 한문장에서 operator * 호출이 두번 일어나고 있기 때문에 new 에 짝을 맞추어 delete 도 역시 호출되어야 한다.  
그런데 한문장으로 사용하였을 경우에 delete 를 호출할수 있는 방법이 존재 하지 않는다.  
operator * 로 부터 반환되는 참조자 뒤에 숨겨진 포인터에 대해서는 사용자가 접근할 방법이 존재 하지 않는다.  

정적 지역 객체를 이용하여 반환하는 방법에 대해 확인해 보자.  
이 방법은 스택이나 힙에서 발생할수 있는 문제가 존재 하지 않는다.  

```c++
const Rational & operator * ( const Rational & lhs,
                              const Rational & rhs )
{
    static Rational result;

    result = ...

    return result;    
}
```
```c++
bool operator == ( const Rational & lhs,
                   const Rational & rhs );


Rational a, b, c, d;

if ( ( a * b ) == ( c * d ) )
{
    ....
}
else
{
    ...
}
```

((a*b) == (c*d)) 표현식은 항상 true 값을 반환한다.  
a, b, c, d 에 어느 값이 있더라도 마찬가지이다.  

```c++
if ( operator == ( operator * (a, b), operator * (c, d)) )
```

operator == 함수가 호출될때 두개의 operator * 함수 호출이 된다.  
이 함수들은 operator * 안에 정의된 정적 객체 Rational 객체의 참조자가 반환될것이다.  
operator == 이 비교하는 피 연산자는 operator * 안의 정적 Rational 객체의 값, 그리고 operator * 안의 정적 Rational 객체의 값이다.  
이 두 값은 항상 같을 수 밖에 없다.  

정적 배열을 사용하면 해결할수 있을 것 같다는 생각을 할수도 있다.  
정적 배열을 사용하려면 배열의 크기인 n 부터 정해야 한다.  
이 n 값은 작아도 문제이고, 커도 문제이다.  
n 값이 너무 작으면 반환값을 저장할 공간이 부족할것이고, 너무 크면 프로그램의 수행 성능이 떨어진다.  
함수가 처을 실행될때 배열안의 모든 객체를 생성한다.  
n 번의 생성자가 호출될 것이다.  

새로운 객체를 반환해야 하는 함수를 작성하는 방법에는 정도가 있다.  
새로운 객체를 반환하게 만들면 된다.  

```c++
inline const Rational operator * ( const Rational & lhs,
                                   const Rational & rhs )
{
    return Rational( lhs.n * rhs.n, lhs.d * rhs.d );
}
```

이 코드에서 반환값을 생성하고 소멸시키는 비용이 여전히 존재하긴 한다.  
그러나 이 비용은 올바른 동작에 지불되는 작은 비용이다.  

참조자를 반환할것인가 아니면 객체를 반환할 것인가를 결정할때는 어떤 선택을 하던지 올바른 동작이 이루어 져야 한다.  

> 지역 스택 객체에 대한 포인터나 참조자를 반환하는 일, 혹은 힙에 할당된 객체에 대한 참조자를 반환하는 일
> 또는 지역 정적 객체에 대한 포인터나 참조자를 반환하는 일은 그런 객체가 두개 이상 필요해질 가능성이 있다면 절대로 하면 안된다.  
> (No.4 를 보면 지역 정적 객체에 대한 참조자를 반환하도록 설계된 올바른 코드 예제를 찾을수 있습니다. 최소한 단일 쓰레드 환경에서 통한다.)
