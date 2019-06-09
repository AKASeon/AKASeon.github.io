---
layout: post
title:  "Effective C++ No.4 객체를 사용하기 전에 반드시 그 객체를 초기화 하자"
date: 2015-11-29
categories: Effective_C++
tags: C++ Effective_C++
---

# 객체 초기화

비 멤버 객체에 대해서는 초기화를 손 수 하여야 한다.

```c++
int x = 0;
const char * text = 'A C-Style string';
```

c++ 의 초기화의 나머지 부분은 생성자로 귀결된다.  
생성자에서는 그 객체의 모든것을 초기화 하자.  

## 대입
```c++
class PhoneNumber
{
    ...
};

class ABEntry
{
public:
    ABEntry ( const std::string            & name,
              const std::string            & address,
              const std::list<PhoneNumber) & phones );

private:
    std::string             theName;
    std::string             theAddress;
    std::list<PhoneNumber>  thePhones;
    int                     numTimesConsulted;
};

ABEntry::ABEntry( const std::string            & name,
                  const std::string            & address,
                  const std::list<PhoneNumber) & phones )
{
    // 이것은 초기화가 아니라 대입이다.
    theName = name;
    theAddress = address;
    thePhones = phones;
    numTimesConsulted = 0;
}
```
## 멤버 초기화 리스트

C++ 규칙에 의하면 어떤 객체이든 그 객체의 데이터 멤버는 생성자의 본문이 실행되기 전에 초기화 되어야 한다.    
대입이 아니라 생성자에서 데이터 멤버를 초기화하고 싶으면 대입문 대신 멤버 초기화 리스트를 사용하면 된다.

```c++
ABEntry::ABEntry( const std::string            & name,
                  const std::string            & address,
                  const std::list<PhoneNumber) & phones ) : theName( name ),
                                                            theAddress( address ),
                                                            thePhones( phones ),
                                                            numTimesConsulted( 0 )
{
    // do nothing
}
```

대입만 사용하였을 경우 theName, theAddress, thePhones 에 대해 기본 생성자를 호출하여 초기화를 미리 해 놓은후에 생성자에서 곧바로 새로운 값을 대입하고 있다.  
먼저 호출된 기본 생성자에서 해 놓은 초기화는 아깝게도 헛짓이 되었다.  
그렇지만 멤버초기화 리스트를 사용하면 피해갈수 있다.  
초기화 리스트에 들어가는 인자는 바로 데이터 멤버에 대한 생성자의 인자로 쓰이기 때문이다.  
대부분의 데이터 타입에 대해서는, 기본 생성자 호출후에 복사 대입 연산자를 연달아 호출하는 방법보다 복사 생성자를 한번 호출하는 쪽이 더 효율적이다.  

기본 제공타입의 객체는 초기화와 대입에 걸리는 비용의 차이가 없지만, 역시 멤버 초기화 리스트에 넣어 주는 쪽이 좋다.  

매게 변수가 없는 생성자가 ABEntry 클래스가 있더라도 멤버 초기화 리스트에 매게 변수 없이 초기화하여 기본 생성자가 호출 되도록하자.  

```c++
ABEntry::ABEntry( const std::string            & name,
                  const std::string            & address,
                  const std::list<PhoneNumber) & phones ) : theName(),
                                                            theAddress(),
                                                            thePhones(),
                                                            numTimesConsulted( 0 )
{
    // do nothing
}
```

데이터 멤버 타입이 사용자 정의 타입이면, 컴파일러가 자동으로 그들 멤버에 대해 기본 생성자를 호출하게 되어 있다.  
그렇지만 기본 제공 타입 객체이면 컴파일러에 따라 초기화가 안되어 있을수 있다.  
그렇기 때문에 명시적으로 멤버 초기화 리스트에 모든 멤버 객체를 넣어 어떤 멤버가 초기화 안되있는지 확인할수 있도록한다.  
그리고 기본 제공 타입 객체가 초기화 되지 않은 상태로 동작할경우 미정의 동작(undefined-behavior)에 빠질수 있다.  

### 상수이거나 참조자인 기본 제공 타입 멤버 초기화

기본 제공 타입의 멤버를 초기화 리스트에 넣는일이 선택이 아니라 의무가 될때도 있다.  
상수 이거나 참조자로 되어 있을 경우 반드시 초기화되어야 한다.  
상수나 참조자는 대입 자체가 불가능 하다.  

### 한 클래스의 다수의 생성자

클래스등중 상당수가 여러개의 생성자를 갖고 있다.  
각 생성자마다 멤버 초기화 리스트가 붙어 있지만 데이터 멤버와 기본 클래스가 적지 않게 붙어 있다면, 생성자 마다 주렁주렁 매달려 잇는 멤버 초기화 리스트의 모습이 그리 좋지만은 않다.  
대입으로도 초기화가 가능한 데이터 멤버들은 초기화 리스트에서 빼내어 별도 함수로 만드는것도 나쁘지 않다.  
이들에 대한 대입 연산을 하나의 함수에 몰아 놓고, 모든 생성자에서 이 함수를 호출하게 하는것이다.  

## 데이터 초기화 순서

1. 기본 클래스는 파생 클래스보다 먼저 초기화
2. 클래스 데이터 멤버는 그들의 선언된 순서대로 초기화
멤버 초기화 리스트에 넣는 멤버들의 순서도 클래스에 선언한 순서와 동일하게 맞춰주도록 하자.

## 비 지역 정적 객체의 초기화 순서는 개별 번역 단위에서 정해짐

### 정적 객체

정적 객체는 자신이 생성된 시점 부터 프로그램이 끝날때까지 살아 있는 객체를 일컫는다.  
정적 객체  
1. 전역 객체  
2. 네임스페이스 요효범위에 적의된 객체  
3. 클래스 안에 static 으로 선언된 객체  
4. 함수안에서 static 으로 선언된 객체  
5. 파일 유효범위에서 static 으로 정의된 객체  

함수 안에 있는 정적객체를 지역 정적 객체(local static object)라고 한다.  
나머지는 비지역 정적객체(non-local static object)라고 한다.  
모든 정적 객체는 프로그램이 끝날때 자동으로 소멸한다. (main 함수의 실행이 끝날때)  

### 번역단위

번역 단위(translation unit)는 컴파일을 통해 하나의 목적 파일을 만드는 바탕이 되는 소스코드를 말한다.  
기본적으로 소스파일 하나가 번역 단위 인데 그 파일이 #include 하는 파일까지 합쳐서 하나의 번역 단위이다.  

한쪽 번역 단위에 있는 비 정적 객체의 초기화가 진행되면서 다른쪽 번역 단위에 있는 비 지역 정적 객체가 사용되는데,
불행히도 이 객체가 초기화 되지 않을지도 모른다.  
별개의 번역 단위에서 정의된 비 지역 정적 객체들의 초기화 순서는 '정해져 있지 않다'라는 사실때문에 그렇다.  

```c++
class FileSystem
{
public:
    ...
    std::size_t numDisks() const;
    ...
};

extern FileSystem tfs;
```

```c++
class Directory
{
public:
    Directory( params );
    ...
};

Directory::Directory( params )
{
    ...
    std::size_t disks = tfs.numDisks();
    ...
}
```

```c++
Directory tempDir( params );
```

정적 객체의 초기화 순서 때문에 문제가 심각해 질수도 있는 상황이 있다.  
tfs 가 tempDir 보다 먼저 초기화 되어 있지 않으면 tempDir 생성자는 tfs 가 초기화 되지 않았는데 tfs 를 사용하려 한다.  
그러나 tfs 와 tempDir 은 제작자도 다르고 만들어진 시기도 다른다 그리고 소스파일 위치까지 다르다.  
이들은 다른 번역 단위 안에서 정의된 비 지역 객체이다.  

서로 다른 번역 단위에 정의된 비지역 정적 객체들 사이의 상대적인 초기화 순서는 정해져 있지 않다.  
비 지역 정적 객체들의 초기화 대해 '적절한' 순서를 결정하기는 어렵다.   

## 싱글톤 패턴(singletom pattern)

비 지역 정적 객체를 하나씩 맡는 함수를 준비하고 이 안에서 각 객체를 넣는것이다.  
함수 속에서도 이들은 정적 객체로 선언하고, 그 함수안에서는 이들에 대한 참조자를 반환하게 만든다.  
사용자쪽에서는 비 지역 정적객체를 직접 참조하지 않고 함수 호출로 대신한다.  
'비지역 정적 객체'가 '지역 정적 객체'로 바뀌엿다.  

```c++
class FileSystem
{
    ...
};

FileSystem & tfs()
{
    static FileSystem fs;

    return fs;
}

class Directory
{
    ...
};

Directory::Directory( params )
{
    ...
    std::size_t disk = tfs().numDisks();
    ...
}

Directory & tempDir()
{
    static Directory td;

    return td;
}
```

# 정리

1. 멤버가 아닌 기본제공 타입 객체는 직접 초기화하자.  
2. 객체의 모든 부분에 대한 초기화에는 멤버 초기화 리스트를 사용하자.  
3. 별개의 번역 단위에 정의된 비지역 정적 객체에 영향을 끼치는 불확실한 초기화 순서를 염두에 두고 이런한 불확실성을 피하자.  

> 1. 기본 제공 타입의 객체는 직접 손으로 초기화 한다. 경우에 따라서 저절로 되기도 하고 안되기도 한다.
> 2. 생성자에서는, 데이터 멤버에 대한 대입문을 생성자 본문 내부에 넣는 방법으로 멤버를 초기화 하지 말고 멤버 초기화 리스트를 즐겨 사용하자.
>    그리고 초기화 리스트에 데이터를 나열할때는 클래스에 각 데이터 멤버가 선언된 순서와 똑같이 나열하자.
> 3. 여러 번역 단위에 있는 비지역 정적 객체들의 초기화 순서 문제는 피해서 설계해야 한다.
>    비 지역 정적 객체를 지역 정적 객체로 바꾸면 된다.
