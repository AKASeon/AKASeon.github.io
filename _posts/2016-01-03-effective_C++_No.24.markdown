---
layout: post
title:  "Effective C++ No.24 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 멤버 함수를 선언하자."
date: 2016-01-03
categories: Effective_C++
tags: C++ Effective_C++
---

"클래스에서 암시적 타입 변환을 지원하는것은 잘못된 생각이다" 라고 이야기 하였었다.  
그러나 이 규칙에도 예외가 존재 한다.  
가장 흔한 예외 중 하나가 숫자 타입을 만들때 이다.  

```c++
class Rational
{
public:
    Rational( int numerator = 0,
              int denominator = 1 );

    int numerator() const;
    int denominator() const;

private:
    ...
}
```

유리수의 곱셉은 Rational 클래스 자체와 관련이 있으니 operator * 는 Rational 클래스 안에 구현하는것이 자연스러울것 같다.  
No.23 에서 어떤 클래스와 관련된 함수를 그 클래스의 멤버로 두는 것은 객체 지향 원칙에 어긋난다라고 말하였지만 일단 operator * 를 Rational 의 멤버함수로 구현해 보자.  

```c++
class Rational
{
public:
    ...
    const Rational operator * ( const Rational & rhs ) const;
}
```

위와 같이 설계하면 유리수 곱셈을 아주 쉽게 할 수 있다.  

```c++
Rational oneEighth( 1, 8 );
Rational oneHalf( 1, 2 );

Rational result = oneHalf * oneEighth;

result = result * oneEighth;
```

그러나 혼합형 수치 연산을 수행하게 되면 문제가 발생한다.  

```c++
result = oneHalf * 2;                            // 정상

result = 2 * oneHalf;                            // 에러
```

곱셈은 기본적으로 교환법칙이 성릭해야 하지만 위에서는 교환 법칙이 성립되지 않는다.  

```c++
result = oneHalf.operator * ( 2 );

rseult = 2.operator * ( result );
```

첫번째 줄에서 oneHalf 객체는 operator * 함수를 멤버로 갖고 있는 클래스의 인스턴스이므로 컴파일러는 이 함수를 호출한다.  
그러나 두번째 줄에서 정수 2에는 클래스 같은 것이 연관되어 있지 않기 때문에 operator * 멤버 함수가 존재하지 않는다.  
컴파일러는 아래 처럼 호출할 수 있는비 멤버 버젼의 operator * 도 찾아 본다. (네임스페이스 혹은 전역 유효범위에 있는 operator * )

```c++
result = operator * ( 2, oneHalf );              // 에러
```

그러나 int 와 Rational 을 취하는 operator * 가 존재하지 않기  때문에 컴파일 에러가 나게 된다.  

위에서 성공한 함수 호출문을 보면 두번째 매개변수가 정수 2인데, Rational 객체를 받도록 되어 있습니다.  
2는 암시적 타입 변환(implicit type conversion) 을 통하여 컴파일러는 이 함수에 int 를 넘겼지만 Rational 생성자에 주어 호출하면 Rational로 변환이 된다.  

```c++
const Rational temp( 2 );                        // 2로 부터 임시 Rational 객체를 생성

result = oneHalf * temp;                         // oneHalf.operator * (temp) 와 같음
```

컴파일러가 이렇게 동작하는것은 명시호출(explicit) 로 선언되지 않은 생성자가 있기 때문이다.  
Rational 생성자가 명시호출(explicit) 생성자였으면 다음중 어느 쪽도 컴파일되지 않는다.  

```c++
result = oneHalf * 2;                  // 컴파일 성공

result = 2 * oneHalf;                  // 에러
```

이렇게 하면 혼합형 수치 연산에 대한 지원은 되지 않지만 최소환 두 문장의 동작이 일관되게 동작 한다.  

동작도 일관되게 유지하고 혼합성 수치 연산도 제대로 지원하게 하기 위해서는 앞에서 본 두 문장이 전부 컴파일되는 설계를 해야 한다.  

암시적 변환에 대해 매개변수가 먹혀 들려면 매개변수 리스트에 들어 있어야 한다.  
이는 호출되는 멤버 함수를 갖고 있는 객체에 해당하는 암시적 매개변수에는 암시적 변환이 되지 않는다.  
그러니깐 호출되는 멤버 함수(쉽게 말해 this 가 가리키는)를 갖고 있는 객체에 해당하는 암시적 매개변수에는 암시적 변환이 먹히지 않는다.  

operator * 를 비 멤버 함수로 만들어서 컴파일러 쪽에서 모든 인자에 대해 암시적 타입 변환을 수행하도록 내버려 두면 된다.  

```c++
class Rational
{
    ...
};

const Rational operator * ( const Rational & lhs,
                            const Rational & rhs )
{
    return Rational( lhs.numerator() * rhs.numerator(),
                     lhs.denominator() * rhs.denominator() );
}                     


Rational oneFourth(1, 4);
Rational result;

result = oneFourth * 2;
result = 2 * oneFourth;       
```

operator * 함수는 Rational 클래스의 프렌드 함수로 두는것은 지금 예제에서는 안된다.  
operator *  는 완전히 Rational 의 public 인터페이스 만을 써서 구현할수 있기 때문이다.  
"멤버 함수의 반대는 프린드 함수가 아니라 비멤버 함수이다." 라는 결론을 알수 잇다.  
어떤 클래스와 연관 관계를 맺어 놓고 싶은데 멤버 함수이면 안되는 함수에 대해, 이런것들은 프렌드로 만들어 버리면 안된다.  
프렌드 함수는 피할 수 있으면 피해야 한다.  
프렌드들 때문에 문제 생길 여지가 고마워할 여지 보다 더 많다.  
물론 어떤 경우에서는 프렌드 관계를 꼭 맺을 필요도 있다.  
그러나 '멤버 함수가 안되니까 반드시 프렌드 함수이어야 한다'라는 생각은 잘못 된것이다.  

Rational 을 클래스가 아닌 클래스 템플릿으로 만들다 보면 고민해야 할 문제점이 자금것과 다르고 이것을 해결하는 방법 또한 다르다.  
이들 문제점, 해결방법, 관련 사항들은 No.46 에서 확인해볼수 있다.  

> 어떤 함수에 들어가는 모든 매개변수에 대해 타입변환을 해 줄 필요가 있다면, 그 함수는 비 멤버여야 한다.  
