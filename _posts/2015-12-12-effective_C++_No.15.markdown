---
layout: post
title:  "Effective C++ No.15 자원 관리 클래스에서 관리되는 자원은 외부에서 접근 하도록 하자"
date: 2015-12-12
categories: Effective_C++
tags: C++ Effective_C++
---

자원 관리 클래스는 실수로 발생할수 있는 자원 누출을 막아 주는 보호막 같은 역할을 한다.  
수많은 API들이 자원을 직접 참조하도록 만들어져 있어서 자원 관리 클래스가 아니라 직접 자원들을 넘겨 주어야 한다.  

No13. 에서 createInvestment 함수 같은 팩토리 함수를 호출한 결과를 포인터에 담기 위해 auto_ptr 이나 tr1::shared_ptr 와 같은 스마트 포인터를 사용하였다.  

```c++
int daysHeld( const Investment * pi );

{
    std::tr1::shared_ptr<Investment> pInv( createInvestment() );

    int days = daysHeld( pInv );                 // 컴파일 에러
}
```

daysHeld 함수는 Investment * 타입의 실제 포인터를 원하는데 pInv 는 std::tr1::shared_ptr<Investment> 객체이다.   
RAII 클래스(std::tr1::shared_ptr) 의 객체가 그 객체가 감싸고 있는 실제 자원으로 변환할 필요가 있다.   
이 방법에는 명시적 변환(explicit conversion)과 암시적 변환(implicit conversion)이 있다.  

tr1::shared_ptr 및 auto_ptr 은 명시적으로 변환을 수행하는 get 이라는 멤버 함수를 제공한다.  
이 함수를 통하여 스마트 포인터 객체에 들어 있는 실제 포인터를 얻을수 있다.  

```c++
int days = daysHeld( pInv.get() );
```

제대로 만들어진 스마트 포인터 클래스라면 거의 모두다 그렇듯 tr1::shared_ptr 과 auto_ptr 은 역참조 연산자(operator-> , operator*)도 오버로딩 하고 있다.   

```c++
class Investment
{
public:
    bool isTexFree() const;
    ...
};

Investment * createInvestment();

{

    str::tr1::shared_ptr<Investment>  pi1( createInvestment() );
    bool                              taxable1 = !( pi1->isTexFree() );        // operator-> 를 써서 자원에 접근
    ...
    std::auto_ptr<Investment> pi2( createInvestment() );
    bool                      texable2 = !( (* pi2).isTexFree() );             // operator * 를 써서 자원에 접근

}
```

RAII 객체 안에 들어 있는 실제 자원을 얻어낼 필요가 있기 때문에 RAII 클래스는
설계자중에는 암시적으로 변환함수를 제공하여 자원 접근을 매끄럽게 할수 있도록 만드는 분도 있다.   

```c++
FontHandle getFont();

void releaseFont( FontHandle fh );

class FontHandle
{
public:
    explicit Font( FontHandle fh ) : f( fh )
    {
    }

    ~Font()
    {
        releaseFont( f );
    }

private:
    FontHandle f;
};
```

Font 클래스에 명시적 변환 함수 get 를 제공한다.  

```c++
class Font
{
public:
    ...
    FontHandle get() const
    {
        return f;
    }
    ...
}
```

```c++
void changeFontSize( FontHandle f, int newSize );

{
    Font F( getFont() );
    int newFontSize;
    ...
    changeFontSize( f.get(), newFontSize );      // Font 에서 명시적으로 FontHandle 를 변환후 넘긴다.
}
```

변환할때 마다 무슨 함수를 호출해주어야 한다는 점이 불편할수도 있다.
그래서 FontHandle 로의 암시적 변환 함수를 Font 에 제공하도록 할수도 있다.

```c++
class Font
{
public:
    ...
    operator FontHandle() const                  // 암시적 변환 함수
    {
        return f;
    }
    ...
};
```

```c++
Font f( getFont() );
int newFontSize;
...
changeFontSize( f, newFontSize );                // Font 에서 FontHandle 로 암시적 변환
```

암시적 변환을 할 경우 의도치 않은 동작이 있다.  

```c++
Font f1( getFont() );
...
FontHandle f2 = f1;                              // Font 객체를 복사하기를 원하였지만
                                                 // f1 이 FontHandle 로 변환한 다음에 FontHandle 이 복사 되었다.  
```

RAII 클래스를 실제 자원으로 바꾸는 방법으로 명시적 변환을 제공할지 아니면 암시적 변환을 제공할지는
그 RAII 클래스만의 특정한 용도와 사용 환경에 따라 달라 집니다.  
No.18 에 따라 맞게 쓰기는 쉽게, 틀리게 쓰기는 어렵게 만들어야 한다.  
늘 그런것은 아니지만 암시적 변환보다 get 같은 명시적 변환 함수를 제공하는 것이 좋을때가 많다.  
이것은 원하지 않은 타입 변환이 일어날 여지를 줄여 준다.  
그렇지만 암시적 타입 변환에서 생기는 사용시의 자연 스러움이 좋을수도 있다.  

RAII 클래스에서 자원 접근 함수를 열어주는 설계가 캡슐화에 위배되는것은 아닌가라는 의문이 있을수 있다.  
캡슐화에 위배되기는 하지만 잘못된 설계는 아니다.  
RAII 클래스는 애초부터 데이터 은닉이 목적이 아니라 자원해제를 실수없이 수행하는것이 목적이다.  
자원 해제라는 기본 기능 위에 캡슐화 기능을 얹을수 있지만 꼭 필요한 기능은 아니다.  
이미 만들어진 RAII 클래스중에 이미 자원의 엄격한 캡슐화와 느슨한 캡슐화를 동시에 지원하는것이 꽤 있다.  
shared_ptr 클래스는 참조 카운팅 매커니즘에 필요한 장치들은 모두 캡슐화하고 있지만 그와 동시에 자신이 관리하는 포인터는 쉽게 접근 할수 있다.  
꼼꼼히 제대로된 설계된 클래스가 그렇듯 사용자가 볼 필요 없는 데이터는 가리고 접근해야 하는 데이터가 있으면 열어 주는 것이다.  

> 1. 실제 자원을 직접 접근해야 하는 기본 API들도 많기 때문에 RAII 클래스를 만들때는 그 클래스가 관리하는 자원을 얻울수 있는 방법을 열어 주어야 한다.  
> 2. 자원 접근은 명시적 변환 혹은 암시적 변환을 통해 가능하다.  
>    안전성만 따지면 명시적 변환이 대체적으로 낫지만 편의성을 놓고 보면 암시적 변환이 괜찮다.  
