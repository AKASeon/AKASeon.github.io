---
layout: post
title: No.31 파일 사이의 컴파일 의존성을 최대한 줄이자
date: 2019-01-06
categories: Effective_C++
tags: C++ Effective_C++
---

c++ 은 인터페이스와 구현을 깔끔하게 분리하는 일에 별로 일가견이 없음
c++ 의 클래스 정의는 클래스 인터페이스만 지정하는 것이 아니라 구현 세부사항까지 사당히 많이 지정하고 있음 
```c++
class Person {
public:
    Person( const std::string & name,
            const Date &        birthday,
            const Address &     addr );
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::string theName;
    Date        theBirthDate;
    Address     theAddress;
};
```
위 코드만으로 Person 클래스는 컴파일이 불가능
string, Date, Address 가 어떻게 정의되어 있는 모름
\#include 지시자를 통하여 해당 클래스가 정의된 파일을 로드 

```c++
#include <string>
#include "date.h"
#include "address.h"
```

\#include 문은 Person을 정의한 파일과 위의 헤더 파일 사이에 컴파일 의존성(compliation dependency)이 생김
위 헤더파일 셋중 하나라도 바뀌거나 이들과 또 엮여 있는 헤더파일들이 바뀌기만 해도 Person클래스를 정의한 파일은 컴파일러에 의해 다시 컴파일 됨

```c++
namesapce std {
    class string;       // 전방 선언
}                       // 틀린 부분

class Date;             // 전방 선언
class Address;          // 전방 선언

class Person {
public:
    Person( const std::string & name,
            const Date &        birthday,
            const Address &     addr );
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
};
```

위 코드의 문제점
1. string 은 클래스가 아니라 typedef로 정의된 타입 동의어
    - basic_string<char> 를 typedef
2. 컴파일러가 컴파일 도중에 객체의 크기를 전부 알아야 함

클래스를 둘로
- 인터페이스만 제공하는 클래스
    - 인터페이스를 제공하는 클르새 이름을 Person  
- 인터페이스를 구현한 클래스
    - 구현을 맡은 클래스의 이름을 PersonImpl   

```c++
#include <string>
#include <memory>

class PersonImpl;

class Date;
class Address;

class Person {
public:
    Person( const std::string & name,
            const Date &        birthday,
            const Address &     addr );
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::tr1::shared_ptr<PersonImpl> pImpl;
};
```

'정의부에 대한 의존성(dependencies on definitions)'을 '선언부에 대한 의존성(dependencies on declarations)'으로 변경하는것이 키
- 컴파일러 의존성을 최소화 하는 핵심 원리
- 헤더파일을 만들때 실용적으로 의미를 갖는 한 자체 조달(self-sufficient) 형태로
- 다른 파일에 대한 의존성을 갖도록 하되 정의부가 아닌 선언부에 대해 의존선을 갖도록


- 객체 참조자 및 포인터로 충분한 경우에는 객체를 직접 쓰지 않는다.
    - 어떤 타입에 대한 참조자 및 포인터를 정의 할때는 그 타입의 선언부만 필요
    - 어떤 타압의 객체를 정의 할때는 그 타입의 정의가 필요
- 할 수 있으면 클래스 정의 대신 클래스 선언에 최대한 의존적으로 만든다.
    - 어떤 클래스를 선언할 때는 그 클래스의 정의를 가져오지 않아도 됨
    - 클래스 객체를 값으로 전달하거나 반환하더라도 클래스 정의가 필요 없음
```c++
class Date;                         // 클래스 선언
Date today();                       // Date 클래스의 정의를 가져올 필요 없음
void clear Appointments( Date d );  
```

        - '값에 의한 전달' 방식은 좋은 방법이라고 보기 여렵다.
        - 그렇지만 불필요한 컴파일 의존성을 끌고 들어 오지 않을수 있다.
- 선언부와 정의부에 대해 별도의 헤더 파일을 제공한다.
    - "클래스를 둘로 쪼개자" 라는 지침을 사용하려면 헤더 파일이 짝으로 있어야 함
        - 선언부를 위한 헤더파일
        - 정의부를 위한 헤더 파일
    - 이 두 파일은 관리도 짝으로 해야 함
        - 한쪽에서 어떤 선언이 변경되면 다른쪽도 똑같이 변경
        
핸들 클래스(handle class)
- pimpl 관용구를 사용하는 Person 같은 클래스
- 핸들 클래스에서 어떤 함수를 호출하게 되어 있다면, 핸들 클래스에 대응되는 구현 클래스쪽으로 그 함수 호출을 전달
    - 구현 클래스가 실제 작업
```c++
#include "Person.h"
#include "PersonImpl.h"
.
Person::Person( const std::string & name,
                const Date &        birthday,
                const Address &     addr ) 
        : pImpl( new PersonImpl( name,
                                 birthday,
                                 addr ) )
{ } 
.
std::string Person::name() const
{
    return pImpl->name();
}
```

인터페이스 클래스(Interface class)
- 인터페이스 클래스를 추상 기본 클래스를 통해 마련해 놓고, 이 클래스로부터 파생 클래스를 만들 수 있게 할수도 있음.
    - 데이터 멤버도 없고, 생성자도 없음
    - 하나의 가상 소멸자와 인터페이스를 구성하는 순수 가상 함수만 존재

```c++
class Person {
public:
    virtual ~Person();
    
    virtual std::string name() const = 0;
    virtual std::string birthDate() const = 0;
    virtual std::string address() const = 0;
};
```
- 인터페이스 클래스를 사용하기 위해서 객체 생성 수단이 최소한 하나는 있어야 함
    - 파생 클래스의 생성자 역할을 대신하는 함수를 만들어 놓고 이것을 호출하므로 해결
        - 팩토리 함수, 가상 생성자(virtual constructor)
            - 주어진 인터페이스 클래스의 인터페이스를 지원하는 객체를 동적으로 할당한후, 그객체의 포인터를 반환
            - 인터페이스 내부에 정적 멤버로 선언 되는 경우가 많음
- 팩토리 함수

```c++
class Person {
public:
    ...
    static std::tr1::shared_ptr<Person> create ( const std::string & name,
                                                 const Date &        birthday,
                                                 const Address &     addr );
    ...
};
```
- 팩토리 함수 사용
```c++
std::string name;
Date dateOfBirth;
Address address;
...
std::tr1::shared_ptr<Person> pp( Person::create( name, dateOfBirth, address ) );
...
std::count << pp->name()
           << " was born on "
           << pp->birthDate()
           << " and now lives at "
           << pp->address();
...
```

```c++
class RealPerson: public Person {
public:
    RealPerson( const std::string & name,
                const Date &        birthday,
                const Address &     addr )
        : theName( name ), theBirthDate( birthday ), theAddress( addr )
    {
    }
    
    virtual ~RealPerson() { }
    
    std::string name() const;
    std::string birthDate() cont;
    std::string address() const;
    
private:
    std::string theName;
    Date theBirthDate;
    Address theAddress;
};
```

```c++
std::tr1::shared_ptr<Person> Person::create( const std::string & name,
                                             const Date        & birthday,
                                             const Address     & addr )
{
    return std::tr1::shared_ptr<Person>( new RealPerson( name,
                                                         birthday,
                                                         addr ) );
}
```

인터페이스 클래스를 구현하는 용도로 가장 많이 쓰이는 매커니즘
1. 인터페이스 클래스로 부터 인터페이스 명세를 물려 받게 만든후에 그 인터페이스에 들어 있는 함수(가상함수)를 구현하는 것
2. 다중 상속을 이용 

핸들 클래스와 인터페이스 클래스는 구현부로 부터 인터페이스를 뚝 떼어 놓어 놓음으로써 파일들 사이의 컴파일 의존성을 완화 시키는 효과를 가져옴

단점
1. 핸들 클래스
    - 핸들 클래스의 멤버 함수를 호출하면 알맹이 객체의 데이터까지 가기 위해 포인터를 타야 함
        - 한번 접근할때 마다 요구되는 간접화 연산이 한단계 증가
    - 객체 하나씩을 저장하는데 필요한 메모리 크기에 구현부 포인터의 크기가 더해진다.
    - 구현부 포인터가 동적 할당된 구현부 객체를 가리키도록 어디선가 그 구현부 메모리의 초기화가 필요
        - 동적 메모리 할당에 따르는 연산 오버헤드, bad_alloc 예외가 발생
2. 인터페이스 클래스 
    - 호출 되는 함수가 모두 가상함수
        - 함수 호출이 발생할때마다 가상 테이블 점프에 따라는 비용이 소모
        - 인터페이스 클래스에서 파생된 객체는 모두다 가상 테이블 포인터를 가지고 있어야 함
3. 핸들 클래스, 인터페이스 클래스
    - 인라인 함수의 도움을 받을수 없음

> 1. 컴파일 의존성을 최소화하는 작업의 배경이 되는 가장 기본적인 아이디어는 '정의' 대신에 '선언'에 의존하게 만들자. 이 아이디어에 기반한 두가지 접근 방법은 핸들 클래스와 인터페이스 클래스 이다.
2. 라이브러리 헤더는 그 자체로 모든 것을 갖추어야 하며 선언부만 갖고 있는 형태여야 한다. 이 규칙은 텝플릿이 쓰이거나 쓰이지 않거나 동일하게 적용해야 한다.
