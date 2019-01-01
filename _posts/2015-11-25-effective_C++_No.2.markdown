---
layout: post
title:  "Effective C++ No.2 #define 을 쓰려거든 const, enum, inline 을 떠올리자"
date: 2015-11-25
categories: Effective_C++
tags: C++ Effective_C++
---

가급적 선행 처리자보다는 컴파일러를 더 가까이 하자
컴파일단계에서 오류를 보다 쉽게 찾을수 있게 하자.

# \#define 대신  상수 사용

```c++
#define ASPECT_RATIO 1.653

const double AspectRatio = 1.653
```

코드 상에는 ASPECT_RATIO 라는 이름이 존재 하지만 컴파일러는 알지못한다.  
ASPECT_RATIO 는 심볼릭 테이블이 들어가지 않고 소스 코드에서 ASPECT_RATIO 를 1.653  으로 치환하기 때문이다.   
그렇기 때문에 컴파일 오류 발생시 ASPECT_RATIO 가 아니라 1.653 이 에러 메시지 노출된다.  
그리고 1.653 으로 치환되면서 1.653 사본이 등장 횟수 만큼 삽입이 된다.  

AspectRatio 는 언어차원에서 지원하는 상수 타입의 데이터이기 때문에 컴파일러가 인지 하고 있고 심볼릭테이블에도 존재 한다.  
그리고 삼수 타입의 AspectRatio 는 여러번 사용되었어도 사본은 딱 한개가 생성된다.  

# \#define 을 상수로 교체시 주의점

## 상수 포인터(const pointer)를 정의하는 경우

상수 정의는 대개 헤더 파일에 넣으므로 포인터는 꼭 const 로 선언해야 한다.  
포인터는 물론 포인터가 가리키는 대상까지 const 선언하는것이 보통의 경우이다.

```c++
const char * const authorName = "Scott Meyers";
```

위 처럼 문자열 상수를 사용할 경우 const 를 두번 써야 하기 때문에 string 객체를 사용하는 편이 좋다

```c++
const std::string authorName( "Scott Meyers" );
```

## 클래스 멤버로 상수를 점의하는 경우

상수의 사본 개수가 한개를 넘지 못하게 하고 싶다면 정적(static) 멤버로 만들어야 한다.  

```c++
// 헤더 파일
class GamePlayer
{
private:
    static const int NumTurns = 5;
    int scores[NumTurns];
};
```

NumTurns 은 정의된것이 아니라 선언된것이다.   
정의가 마련되어 있어야 하는것이 보통이지만, 정적멤버로 만들어지는 정수로 타입의 클래스 내부 상수는 예외이다.  
그렇지만 몇가지 이유(컴파일러가 잘못 만들어.., 클래스 상수의 주소를 구하는..)로 정의를 제공 해야 할수도 있다.  

```c++
// 구현 파일
const int GamePlayer::NumTurns;
```

클래스 상수의 정의는 구현파일에 위치, 헤더파일이 아니다.  
이 정의에는 상수의 초기값이 있으면 안된다. 클래스 상수의 초기값은 해당 상수가 선언된 시점에 바로 주어지기 때문이다.  

클래스 상수를 #define 으로 만들지 말자. #define 은 정의되면 컴파일이 끝날때까지 유효하다.  

정적 클래스 멤버가 선언된 시점에 초기값을 주는 것이 대개 맞지 않다고 판단되기 때문에 오래된 컴파일러는 위의 문법이 받아드려지지 않을수 있다.   

```c++
// 헤더 파일
class CostEstimate
{
private:
    static const double FudgeFactor;
};

// 구현 파일
const double CostEstimate::FudgeFactor = 1.35;
```

보통의 경우 위와 같이 하면 해결 되지만 해당 클래스를 컴파일 도중에 클래스 상수가 필요할때이다.  
예를 들면 GamePlayer::scores 등의 배열 멤버를 선언할때이다. (컴파일러는 컴파일 도중 이 배열의 크기를 알아야 합니다.)  
이때 enum hack 기법을 사용할수 있다.  

### enum hack (나열자 둔갑술)

enum 타입의 값은 int 가 놓일곳에도 쓸수 있다.

#### enum hack 은 동작방식이 const 보다 #define 이 가깝다.

const 는 주소를 알아낼수 있지만, enum 의 주소를 얻어 오는것은 안된다. #define 역시 마찬가지이다.  
정수 상수를 주소를 얻어 쓰거나 참조자를 쓰는 것을 막고 싶다면 enum 을 사용하면 된다.  
컴파일러가 정수 타입에 대한 const 객체에 대해 저장공간을 마련하지 않겠지만 어떤 컴파일러는 저장공간을 차지 할수 있다.  
const 객체 에 대한 메모리를 만들지 않는 방법을 쓰고 싶다면 enum 를 사용하면 된다.  
enum 은 #define 처럼 어떤 형태의 쓸데 없는 메모리 할당을 하지 않는다.  

#### 많은 코드를이 이미 이 기법을 사용하고 있으므로 이 패턴의 코드를 발견시 쉽게 의도를 파악할수 있다.

탬플릿 메타 프로그램의 핵심 기법이기도 하다.

# 인라인 함수

기존 매크로의 효율을 그대로 유지하고 정규 함수의 모든 동작 방식 및 타입 안정성까지 완벽하게 취할수 있는 방법은 인라인 함수에 대한 템플릿을 준비하는 것이다.  

```c++
#define CALL_WITH_MAX( a, b )    f( (a) > (b) ? (a) : (b) )

template<typename T>
inline void callWithMax( const T & a, const T & b )
{
    f( a > b ? a : b );
}
```



> 단순한 상수를 쓸때는, #define 보다 const 객체 혹은 enum 을 우선 생각 합시다.
> 함수처럼 쓰이는 매크로를 만들려면, #define 매크로보다는 인라인 함수를 우선 생각 합시다.
