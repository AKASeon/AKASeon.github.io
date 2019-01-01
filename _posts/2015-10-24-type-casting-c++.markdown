---
layout: post
title:  "Type Casting in c++"
date:   2015-10-24
categories: C++
tags: C++
---
# static\_cast
일반적으로 C/C++ 언어에서는 지정한 데이터형으로 강제 형 변환이 가능하다.  
하지만 강제 형 변환을 할 경우 원하지 않는 결과가 발생할 수 있으므로,  
이런 문제를 미연에 방지하기 하기 위해 static\_cast 를 사용하며,  
static\_cast 는 의미 없는 포인터의 형 변환을 막아준다.  
명시적으로 형변환하겠다는 의미이다.  
상호호환 관계가 아니면 컴파일시 에러를 에러를 발생시킨다.  

```c++
#include <stdio.h>
#include <typeinfo>

class T1
{
public:
    virtual void print1() { puts( "t1p1" ); }
};

class T2 : public T1
{
public:
    virtual void print1() { puts( "t2p1" ); }
    virtual void print2() { puts( "t2p2" ); }
};

int main( void )
{
    T1 *t1 = new T1;
    T2 *t2 = new T2;
    T1 *t;
    t = (T1*)t1;                  // C 언어 스타일의 형변환(casting), 컴파일 에러 없음
    t  = static_cast<T1*>(t1);    // C++ 스타일의 형변환(casting), 컴파일 에러 없음
    t  = static_cast<T1*>(t2);    // Up-casting, 컴파일 에러 없음
    t2 = static_cast<T2*>(t1);    // Down-casting, 컴파일 에러 없음

    int i, *pi;
    pi = (int*)&i;                // C 언어 스타일의 형변환, 컴파일 에러 없음
    pi = static_cast<int*>(&i);   // C++ 스타일의 형변환, 컴파일 에러 없음

    pi = (int*)t;                 // C 스타일, 컴파일 에러 없음
    pi = static_cast<int*>(t);    // C++ 스타일, 컴파일 에러 발생

    return 0;
}

```

# const\_cast
상수형 포인터를 비상수형 포인터로 변경하고자 할때 사용한다.  
불가피한 경우 아니면 사용하지 않는게 좋다.

```c++
#include <stdio.h>
#include <string.h>

int main( void )
{
    char *pc;
    const char msg[] = "Hello!";
    pc = msg;                     // 컴파일 에러 발생
    pc = const_cast<char*>(msg);  // 정상적으로 변환
    strcpy( pc, "hello" );
    puts( pc );

    return 0;
}
```

# reinterpret\_cast
이 연산자는 강제 형 변환을 하기 위해 사용한다.  
참고로 강제 형 변환은 c 언어 형태를 사용해도 무방하다  
불가피한 경우 아니면 사용하지 않는게 좋다.  

```c++
#include <stdio.h>
#include <string.h>

int main( void )
{
   struct node
   {
       char a, b, c, d;
   };

   int  ttt = 0x41424344;
   node* p1 = (node*)ttt;                    // C 언어 스타일 형변환
   node* p2 = reinterpret_cast<node*>(ttt);  // C++ 스타일 형변환

   return 0;
}
```

# dynamic\_cast
상속 관계 안에서 포인터나 참조자의 타입을 기본 클래스에서 파생 클래스로의 다운 태스팅과 다중 상속에서 기본 클래스간의 안전한 타입 캐스팅에 사용한다.  
안전한 타입 캐스팅이랑 런타임에 타입 검사를 한다는 것이다.  
런타임시 실제 인스턴스 타입을 검사해서 캐스팅이 가능하면 그 타입의 포인터를 리턴하고 실패하면 NULL을 리턴한다.  
런타임시 타입을 검사 하기 때문에 오버헤드가 크다.  
