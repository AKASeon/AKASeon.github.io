---
layout: post
title:  "Effective C++ No.3 낌새만 보이면 const 를 들이대 보자"
date: 2015-11-27
categories: Effective_C++
tags: C++ Effective_C++
---

# const

const 키워드가 붙은 객체는 외부 변경을 불가능하게 한다.  
소스코드 수준에서 붙인다는 점과 컴파일러가 이 제약을 단단하게 지켜준다.

## 사용 예시

```c++
char greating[] = "Hello";         

char * p = greating;               // 비 상수 포인터
                                   // 비 상수 데이터

const char * p = greating;         // 비 상수 포인터
                                   // 상수 데이터

char * const p = greating;         // 상수 포인터
                                   // 비 상수 데이터

const char * const p = greating;   // 상수 포인터
                                   // 상수 데이터
```

## 포인터가 가리키는 대상

포인터가 가리키는 대상을 상수로 만들 때 const 를 사용하는 스타일이 조금씩 다르다.  
타입앞에 const 를 붙일 수도 있고 타입 뒤쪽 * 표 의 앞에 const 를 붙일수 있다.  

```c++
void f1( const Widget * pw );

void f2( Widget const * pw );
```

## STL 반복자

STL 반복자(iterator) 는 포인터를 본뜬것이기 때문에 기본적인 동작 원리가 T * 포인터와 흡사하다.    
어떤 반복자를 const 선언하는 일은 포인터를 상수로 선언하는것(T* const 포인터)과 같다.  
반복자는 자신이 가리키는 대상이 아닌것을 가리키는 경우가 허용 되지 않지만, 반복자가 가리키는 대상 자체는 변경 가능하다.   
변경 불가능한 객체를 가리키는 반복자(const T * 포인터의 STL 대응물)가 필요하면 const_iterator 를 쓰면 된다.  

```c++
std::vector<int> vec
...
const std::vector<int>::iterator iter = vec.begin();
                                 // iter 는 T * const 처럼 동작
*iter = 10;                      // OK iter 가 가리키는 대상을 변경
++iter;                          // 에러 iter 는 상수

std::vector<int>::const_iterator cIter = vec.begin();
                                 // cIter 는 const T * 처럼 동작
*cIter = 10;                     // 에러 *cIter 가 상수
++cIter;                         // OK
```

## 함수 반환값

함수 반환 값을 상수로 정해 주면 안정성과 효율을 포기하지 않고도 사용자측 에러 돌발 상황을 줄여준다.

```c++
class Retaional
{
    ...
};

const Rational operator * ( const Relation & lhs,
                            const Relation & rhs );
```

아래와 같이 오타나 잘못 입력한 구문들에 대하여 컴파일러가 에러로 알려 줄수 있다.

```c++
Relation a, b, c;

( a * b ) = c;         

if ( a * b = c )
{
    ...
}
```

## 매개 변수 or 지역 객체

매개 변수 혹은 지역 객체를 수정 할수 없게 하는것이 목적이라면 const 로 선언하자.

# 상수 멤버 함수

멤버 함수에 붙는 const 키워드는 "해당 멤버 함수가 상수 객체에 대해 호출될 함수"라는 것을 알려주는 것이다.

1. 클래스의 인터페이스를 이해하기 좋게 한다.
2. 상수 객체를 사용할 수 있게 한다.

c++ 프로그램의 실행 성능을 높이는 핵심 기법중 하나가 객체 전달을 '상수 객체애 대한 참조자(reference-to-const)'로 진행한는 것이다.  
이 기법을 제대로 활용하기 위해서는 상수 상태로 전달된 객체를 조작할수 있는 const 멤버 함수, 즉 상수 멤버 함수가 준비되어 있어야 한다.  

## const 멤버 함수의 오버로딩

const 키워드가 있고 없고의 차이만있는 멤버 함수들은 오버로딩이 가능하다.  

```c++
class TextBlock
{
public:
    ...
    const char & operator[] ( std::size_t position ) const
    {
        return text[position];
    }

    char & operator[] ( std::size_t position )
    {
        return text[position];
    }

private:
    std::string text;

};
```

```c++
TextBlock tb( "Hello" );
std::cout << tb[0];
tb[0] = 'x'

const TextBlock ctb( "Hello" );
std::cout << ctb[0];
ctb[0] = 'x'    // 컴파일 오류
```

위에서 발생한 컴파일 오류는 순전히 operator[] 의 반환 타입 때문에 생겼다.   
operator[] 의 호출이 잘못된것은 없다. 이 에러는 const char & 타입에 대입 연산을 시도 했기 때문에 발생하였다.  
상수 멤버로 되어 있는 operator[] 의 반환 타입이 const char &  이다.  

operator[] 의 비 상수 멤버는 참조자(reference)를 반환한다.  
char 하나만 쓰면 안된다.    
기본제공 타입을 반환하는 함수의 반환 값을 수정하는 일은 절대로 있을 수 없다.  
이것이 가능하다고 하더라도 반환시 '값에 의한 반환'을 수행하는 c++ 의 성질이 있다.  
그래서 수정되는 값은 tb.text[0] 의 사본이지 tb.text[0] 이 아니다.  

## 비트 수준 상수성과 논리 수준의 상수성

### 비트 수준 상수성(bitwise constness), 물리적 상수성(physical constness)
어떤 멤버 함수가 그 객체의 어떤 데이터 멤버도 건드리지 않아야 그 멤버 함수가 const 임을 인정한다.
즉 그 객체를 구성하는 비트중 어떤것도 바뀌면 안된다.

```c++
class CTextBlock
{
public:
    ...
    char & operator [] ( std::size_t position ) const
    {
        return pText[position];
    }

private:
    char * pText;
}
```

```c++
const CTextBlock cctb( "Hello" );

char * pc = &cctb[0];              // 상수 버젼의 operator[] 를 호출하여
                                   // cctb 의 내부 데이터에 대한 포인터를 얻음
*pc = 'J';                         // cctb 는 'Jello' 값을 가짐
```

### 논리적 상수성(logical constness)
상수 멤버 함수라고 해서 객체의 한 비트도 수정할수 없는 것이 아니라 일부 몇 비트 정도는 바꿀수 있되,
그것을 사용자측에서 알아채지 못하게 하면 상수 멤버 자격이 있다.

```c++
class CTextBlock
{
public:
    ...
    std::size_t length() const;

private:
    char         * pText;
    std::size_t    textlength;
    bool           lengthIsValid;
};

std::size_t CTextBlock::length() const
{
    if ( !lengthIsValid )
    {
        textlength = std::strlen( pText );    // 컴파일 오류, 비트 수준 상수성에 맞지 않음
        lengthIsValid = true;
    }

    return textlength;
}
```

mutable 키워드를 사용한다.  
mutable 은 비 정적 데이터 멤버를 비트수준 상수성의 족쇄에서 풀어 주는 키워드 이다.

```c++
class CTextBlock
{
public:
    ...
    std::size_t length() const;

private:
    char                 * pText;
    mutable std::size_t    textlength;
    mutable bool           lengthIsValid;
};

std::size_t CTextBlock::length() const
{
    if ( !lengthIsValid )
    {
        textlength = std::strlen( pText );    // 컴파일 성공
        lengthIsValid = true;
    }

    return textlength;
}
```

# 상수 멤버 및 비상수 멤버 함수에서 코드 중복 현상을 피하는 방법

operator[] 함수가 특정 문자의 참조자만 반환하지만 이것 말고 여러가지를 더 할수 있다.  
경계 검사 라던지 접근 정보 로깅, 내부자료 무결설 검증도 할수 있다.  
이런 코드들을 모조리 operator[] 의 상수/비상수 버젼에 넣어 버리면 아래와 같은 코드가 탄생한다.

```c++
class TextBlock
{
public:
    ...
    const char & operator[] ( std::size_t position ) const
    {
        ...                    // 경계 검사
        ...                    // 접근 데이터 로깅
        ...                    // 자료 무결성 검증

        return text[position];
    }

    const char & operator[] ( std::size_t position )
    {
        ...                    // 경계 검사
        ...                    // 접근 데이터 로깅
        ...                    // 자료 무결성 검증

        return text[position];
    }

private:
    std::string text;
};
```

위 코드에는 코드 중복이 있다. 이 코드중복은 컴파일 시간, 유지보수, 코드 크기 부풀림등 안 좋은 결과를 발생 시킨다.  

```c++
class TextBlock
{
public:
    ...
    const char & operator[] ( std::size_t position ) const
    {
        ...
        ...
        ...
        return text[position];
    }

    char & operator[] ( std::size_t position )
    {
        return const_cast<char &> (          // const 를 떼어 낸다.
            static_cast<const TextBlock&>    // * this 의 타입에 const 를 붙인다.
            (* this)[position]               // op[] 삼수 버젼을 호출
        );
    }
};
```

비 상수 operator[] 가 상수 버젼을 호출하게 하는것은 * this 의 타입 캐스팅을 통하여 가능하다.

```c++
const char & step1 = static_cast<const TextBlock&>( * this )[position];
```

상수 operator[] 의 반환값에서 const 를 떼어 내는 캐스팅을 수행하여 비 상수 operator[] 함수의 비 상수 반환 값을 생성한다.

```c++
char & step2 = const_cast<char & > step1;
```

코드중복을 피하기 위하여 상수 버젼이 비 상수 버젼을 호출하게 만드는 것은 안전하지 않다.  
상수 멤버에서 비 상수 멤버를 호출하게 되면, 수정하지 않겠다고 약속한 그 객체를 배신하고 것이고
그 객체는 변경될 가능성이 있다.  
상수 멤버 함수에서 비상수 멤버함수를 호출하는 코드를 어떻게든 컴파일하려면 const_cast 를 적용하여 * this 에 붙은 const 를 떼어내야 하는데,
이것은 재앙의 씨앗이다. 그러나 이것의 역순 호출은 안전성에 문제가 없다.  
비 상수 멤버함수 안에서는 그 객체를 바꾸든 안 바꾸든 마음대로 할수 있기 때문에 거기에서 상수 멤버 함수를 호출한다고 특별한 잘못될수 없다.  

# 마무리

const 는 포인터나 반복자에 대해서 그렇고, 포인터/반복자/참조자가 가리키는 객체에 대해서 그렇고, 함수의 매개변수 및 반환 타입에 대해서도 마찬기지이고 지역 변수는 물론이고 멤버 함수에 까지 쓸수 있으면 최대한 사용하는것이 좋다.

> 1. const 를 붙여 선언하면 컴파일러가 사용상의 에러를 잡아내는데 도움을 줍니다.
>    const 는 어떤 유효 범위에 있는 객체에도 붙을수 있으며, 함수 매개변수 및 반환 타입에도 붙을수 있으면
>    멤버 함수에도 붙을수 있습니다.
> 2. 컴파일러 쪽에서 보면 비트 수준 상수성을 지켜야 하지만, 여러분은 개념적인(논리적인) 상수성을 사용해서 프로그래밍해야 합니다.
> 3. 상수 멤버및 비 삼수 멤버 함수가 기능적으로 똑같에 구현되어 있을 경우에는 코드 중복을 피하는 것이 좋은데,
>    이때 비 상수 버젼이 상수 버젼을 호출하도록 만스세요.
