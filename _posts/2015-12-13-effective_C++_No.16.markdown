---
layout: post
title:  "Effective C++ No.16 new 및 delete 를 사용할때는 형태를 반드시 맞추자"
date: 2015-12-13
categories: Effective_C++
tags: C++ Effective_C++
---

```c++
std::string * stringArray = new std::string[100];

delete stringArray;
```

new 와 delete 쌍이 안 맞다.  
실제로 코드가 위와 같이 되어 있으면 프로그램은 정의되지 않은 동작(undefined behavior)을 보인다.  

new 연산은 2가지 내부 동작을 한다.  
1. 메모리를 할당 (operator new 라는 이름의 함수가 쓰임)  
2. 할당된 메모리에 대해 한개 이상의 생성자가 호출

delete 연산자를 사용할때도 2가지 내부 동작을 한다.  
1. 기존 할당된 메모리에 대해 한개 이상의 소멸자가 호출  
2. 메모리를 해제 (operator delete 라는 이름의 함수가 쓰임)

new 로 힙에 만들어진 단일 객체의 메모리 배치 구조와 객체 배열에 대한 메모리 배치구조는 다르다.  
배열을 위해 만들어지는 힙 메모리에는 대개 배열원소의 개수가 저장되어 있다.  
이때문에 delete 연산자는 소멸자가 몇번 호출될지를 알수 있다.  
그러나 단일 객체에는 이러한 정보가 없다.  
모든 컴파일러가 위와 같이 구현하지는 않았지만 거의 모든 컴파일러가 위와 같이 동작한다고 보면 된다.  

어떤 포인터에 대해 delete를 적용할때 delete 연산자로 하여금 ''배열 크기정보가 있다'라는 것을 알려주기 위하여
대괄호 쌍을 delete 뒤에 붙여 주어야 한다.  
그래야 delete 가 '포인터가 배열을 가리키고 있구나''라고 가정하게 된다.  
그렇지 않으면 단일 객체로 간주한다.  

```c++
std::string * stringPtr1 = new std::string;
std::string * stringPtr2 = new std::string[100];
...
delete stringPtr1;                               // 객체 한개를 삭제
delete[] stringPtr2                              // 객체의 배열을 삭제
```

stringPtr1에 '[]'형태를 사용하면 delete는 객체의 배열로 판단하고 엉뚱한 메모리 공간의 데이터를 읽어서 배열 크기 만큼 소멸자를 호출하려고 한다.  
이것은 엉뚱한 객체의 소멸자를 호출할수도 있고 우리가 의도 하지 않은 행동을 한다.  
stringPtr2에서 '[]'형태를 사용하지 않으면 delete는 객체 하나로 판단하고 하나의 객체만 소멸자를 호출하게 된다.    
그러면 나머지 객체에 대해서는 소멸자가 호출되지 않는다.   

new 표현식에 []을 썻으면 여기에 대응되는 delete 표현식에도 []을 써야 한다.  
new 표현식에 []을 안 썻으면 여기에 대응되는 delete 표현식에도 []을 쓰지말어야 한다.  

동적 할당된 메모리에 대한 포인터를 멤버 데이터를 가지고 있는 클래스를 만들는 중이고 이 클래스에서 제공하는 생성자도 여러개일 경우 이 규칙을 준수해야 한다.  
왜냐하면 포인터 멤버를 초기화하는 부분인 생성자에서 new 형태를 똑같이 맞춰줘야 한다.  
delete 에서는 어떤 형태의 delete 를 써야 할지 알수 없다.  

typedef로 정의된 어떤 타입의 객체를 메모리에 생성하고 new를 썼을때 나중에 어떤 형태의 delete 를 써야 할지는 typedef 작성자가 책임을 지어야 한다.  

```c++
typedef std::string AddressLines[4];

std::string * pal = new AddressLines;

delete pal;                                      // undefined behavior
delete [] pal;                                   // 정상 처리
```

배열 타입은 typedef 타입으로 만들지 않는것이 좋다.  
표준 c++ 라이브러리에는 string 이나 vector 같은 좋은 클래스 템플릿이 존재 해서 이것을 활용하면 동적 할당 배열이 필요 없을수 있다.  
위의 예제서는 AddressLines 은 string 의 vector 로 정의해도 된다.  

> new 표현식에 []을 썻으면, 대응 되는 delete 표현식에도 []을 써야 한다.    
> 마찬가지로 new 표현식에 []을 안 썻으면 대응 되는 delete 표현식에도 []을 쓰지 말어야 한다.     
