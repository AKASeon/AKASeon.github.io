---
layout: post
title:  "Effective C++ No.28 내부에서 사용하는 객체에 대한 '핸들'을 반환하는 코드는 되도록 피하자'"
date: 2016-03-06
categories: Effective_C++
tags: C++ Effective_C++
---

```c++
class point
{
    Point( int x, int y );
    ...
    void setX( int newVal );
    void setY( int newVal );
    ...
};

struct RectData
{
    Point ulhc;
    Point lrhc;
};

class Rectangle
{
    ...
private:
    std::tr1::sharted_ptr<RectData> pData;
    // sharted_ptr 은 No.13 으로
};
```

사용자 정의 타입을 전달 할때 값에 의한 전달 보다는 참조에 의한 전달방식이 효율적이라고 예전에 정리 하였다.  
그래서 (스마트) 포인터로 Point 객체에 대한 참조자를 반환하는 형태로 함수를 생성하였다.  

```c++
class Rectangle
{
public:
    ...
    Point & uppperLeft() const { return pData->ulhc };
    Point & lowerRight() const { return pData->lrhc };
    ...
};
```

컴파일은 잘되지만 이것은 잘 못 된 코드이다. 자기모순적인 코드이다.  
uperLeft 함수와 lowerRight 함수는 상수 멤버 함수이다.  
원래 꼭지점 정보만 알아낼수 있도록 사용되고 Rectangle 객체를 수정할수 있는 일을 하지 않는 함수이다.  

```c++
const Rectangle rec( coord1, coord2 );  // rec 은 (0,0) 부터 (100,100) 영역에 있는 삼수 Rectangle 객체

rec.upperLeft().setX( 50 );             // 이제 부터는 rec 은 (50, 0) 부터 (100, 100)의 영역
```

1. 클래스 데이터 멤버는 아무리 숨겨봤자 그 멤버의 참조자를 반환하는 함수들의 최대 접근도에 따라 캡슐화 정도가 정해진다.  
2. 어떤 객체에서 호출한 상수 멤버 함수의 참조자 반환 값이 실제 데이터가 그 객체의 바깥에 저장되어 있다면,
   이 함수의 호출부에서 그 데이터의 수정이 가능하다.  

지금은 참조자를 반환하는 멤버 함수만 이야기 하였지만 이들이 포인터나 반복자를 반환하도록 되어 있어도 같다.
참조자, 포인터 및 반복자는 어쨋든 모두 핸들이고 어떤 객체의 내부요소에 대한 핸들을 반환하게 만들면 언제든지   
그 객체의 캡슐화를 무너뜨리는 위험을 무릅쓸 수 밖에 없다.  

어떤 객체의 '내부요소라고' 하면 데이터 멤버만 생각하는데 일반적인 수단으로 접근 불가능한 멤버 함수도 객체의 내부 요소에 들어 간다.  
외부 공개가 차단된 멤버 함수에 대해, 이들의 포인터를 반환하는 함수를 만들면 안된다.  

```c++
class Rectangle
{
        ...
public:
    const Point & upperLeft() const { return pData->ulhc };
    const Point & lowerRight() const { return pData->lrhc };
};
```

이렇게 설계하면 사용자는 사각형을 정의하는 꼭지점 쌍을 읽을 수는 있지만 쓸수는 없다.  

upperLeft 함수와 lowerRight 함수를 보면 내부 데이터에 대한 핸들을 반환하고 있다.  
이것을 남겨두면 다른쪽에서 문제가 될수 있다.  
가장 큰 문제가 무효참조 핸들로서 핸들이 있기는 하지만 그 핸들을 따라 갔을때 실제 객체의 데이터가 없는 경우이다.  

```c++
class GUIObject { ... };

const Rectangle boundingBox( const GUIObject & obj );

GUIObject * pgo;
..
const Point * pUppperLeft = &(boundingBox(*pgo).upperLeft());
```

boundingBox 함수를 호출하면 Rectangle 에 대한 임시 객체가 만들어 진다.  
이 객체는 겉으로 들어나지 않고 temp 라고 부르기로 한다.  
그리고 이 temp 에 대해 upperLeft 가 호출되는데 이 호출로 인하여 temp 의 내부 데이터가 참조자로 나온다.  
마지막으로 이 참조자에 대한 & 연산자의 결과 값, 주소가 pUppperLeft 포인터에 저장된다.  
그리고 boundingBox 함수의 반환값, 임시객체인 temp 가 소멸된다.
temp 가 소멸되기 때문에 upperLeft 호출한 결과 값인 포인터 주소가 가리키는 객체는 사라지게 된다.  

위와 같은 이유로 객체의 내부에 대한 핸들을 반환하는 함수는 위험하다.  
포인터, 참조자, 반복자도 위험하다. 핸들에 const 를 붙여 사용해도 위험하다.  
핸들을 반환하는 함수 그 자체만으로 위험하다. 그 핸들이 참조하는 객체보다 오래 동안 살아 있을수 있다.  

그렇다고 해서 핸들을 반환하는 멤버함수를 절대로 두지 말라는 이야기는 아니다.  
operator[] 연산자는 string 이나 vector 등의 클래스에서 개개의 원소를 참조할수 있게 만드는 용도로 제공되고 있다.  
실제로 이 연산자는 내부적으로 해당 컨테이너에 들어 있는 개개의 원소 데이터에 대한 참조자를 반환하는 식으로 동작한다.  

> 어떤 객체의 내부요소에 대한 핸들을 반환하는 것은 되도록 피하자.   
> 캡슐화 정도를 높이고 삼수 멤버 함수가 객체의 삼수성을 유지한채로 동작할수 있도록 하며 무효 참조 핸들이 생기는 경우를 최소화 할수 있다.  
