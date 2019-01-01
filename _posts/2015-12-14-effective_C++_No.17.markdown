---
layout: post
title:  "Effective C++ No.17 new 로 생성한 객체를 스마트 포인터에 저장하는 코드는 별도의 한 문장으로 만들자."
date: 2015-12-14
categories: Effective_C++
tags: C++ Effective_C++
---

```c++
int priority();

void processWidget( std::tr1::shared_ptr<Widget> pw,
                    int                          priority );
```

자원 관리에는 객체를 사용하는 것이 좋다는것을 No.13 에서 확인하였다.  
그래서 processWidget 함수는 동적할당된 Widget 객체에 대해 스마트 포인터(tr1::shared_ptr) 사용하도록 하였다.  

```c++
processWidget( new Widget, priority() );
```

tr1::shared_ptr 의 생성자는 explicit 로 선언되어 있기 때문에 new Widget 표현식에 의해 만들어진 포인터가 tr1::shared_ptr 타입의 객체로 바꾸는 암시적인 변환이 되지 않는다.  
그렇기 때문에 위 코드는 컴파일되지 않는다.  

```c++
processWidget( std::tr1::shared_ptr<Widget>( new Widget ),
               priority() );
```

자원관리 객체를 쓰고 있지만 위의 std::tr1::shared_ptr<Widget>( new Widget ) 에서 자원 누수가 생길수도 있다.  

컴파일러는 processWidget 호출 코드를 만들기 전에 우선 이 함수의 매개변수로 넘겨지는 인자를 평가하는 순서를 가진다.  
여기서 두번째 인자는 priority 함수의 호출밖에 없지만 첫번째 인자 std::tr1::shared_ptr<Widget>( new Widget ) 은 두 부분으로 나누어져 있다.   
1. new Widget 표현식을 실행  
2. tr1::shared_ptr 생성자 호출  

processWidget 함수 호출이 이루어지기 전에 컴파일러는 다음의 세가지 연산을 위한 코드를 생성한다.  
- priority 함수 호출  
- new Widget 표현식을 실행  
- tr1::shared_ptr 생성자를 호출  

이 각각의 연산이 실행되는 순서는 컴파일러마다 다르다는게 문제이다.  
new Widget 표현식은 tr1::shared_ptr 생성자가 실행될수 있기 전에 호출되어야 한다.   
왜냐하면 tr1::shared_ptr 생성자의 인자로 넘기기 때문이다.  
new Widget 표현식과 tr1::shared_ptr 생성자 실행 순서는 보장되지만 priority 함수의 호출은 어디에 있어도 상관없다.  

1. new Widget 표현식을 실행  
2. priority 함수 호출  
3. tr1::shared_ptr 생성자를 호출  

여기서 priority 함수 호출부분에서 예외가 발생하였으면 new Widget으로 만들어진 포인터가 유실될수 있다.  
자원 누수를 막기 위해 준비한 tr1::shared_ptr 를 사용하기도 전에 예외가 발생하였기 때문에 자원 누수가 발생한다.  

위와 같은 문제를 피하기 위해서는 Widget을 생성해서 스마트 포인터에 저장하는 코드를 별도의 문장으로 만들고 그 스마트 포인터를 processWidget 에 넘긴다.  

```c++
std::tr1::shared_ptr<Widget> pw( new Widget );

processWidget( pw, priority() );
```

한 문장 안에 있는 연산들보다 문장과 문장 사이에 있는 연산들이 컴파일러에 의해 변경될 가능성이 적기 때문에 자원 누수의 가능성이 없다.  

> new 로 생성한 객체를 스마트 포인터로 놓는 코드는 별도의 한 문장으로 만들자.  
> 이것이 안 되어 있으면, 예외가 발생될 때 디버깅하기 힘든 자원 누수가 발생할수 있다.  
