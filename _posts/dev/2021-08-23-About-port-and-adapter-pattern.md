---
title: "헥사고날 설계 (Port And Adapter Architecture)에 대해 알아보자"
description: ""
date: "2021-08-23"

categories:
  - dev
  - pattern
tags:
  - dev
  - pattern
---

# 개요

핵사고날 아키텍쳐라고 불리기도 하고 포트와 어댑터 아키텍쳐(패턴)이라고 불리는 것에 대해서 알아보도록 하겠습니다.

# Port And Adapter Pattern

## layered pattern (cake pattern)

우리가 자주 사용하는 패턴입니다. controller, service, repository, etc.. 이런 패키지들을 만들고 해당 기능들과 관련해서 구현된 코드 파일들을 해당 패키지에 넣습니다.

해당 패턴은 연관성이 없는 파일들이 해당 패키지에 들어갑니다. 당연히 해당 기능을 하는 집단을 찾기 위해서는 빠르게 찾을 수 있지만 거기에서 연관된 다른 기능 모듈들을 찾기 위해서는 찾는 과정이 필요합니다.

``` 
└── com
    └── test
        └── pattern
            ├── controller
            │   ├── User.java
            │   └── Store.java
            ├── dto
            │   ├── RegisterRequest.java
            │   ├── RegisterResponse.java
            │   ├── StoreRequest.java
            │   ├── StoredItemDto.java
            │   └── UserDto.java
            ├── entity
            │   ├── User.java
            │   ├── Item.java
            ├── persistence
            │   ├── ItemRepository.java
            │   └── UserRepository.java
            └── service
                ├── InventoryService.java
                ├── MembershipService.java
                ├── TotalRentalService.java
                └── TotalRentalServiceImpl.java
```
[출처: 라인엔지니어링 블로그](https://engineering.linecorp.com/ko/blog/port-and-adapter-architecture/)

보는 것과 같이 일반적으로 관련이 없는 항목들이 같은 패키지에 존재하게 됩니다.

## Port And Adapter pattern

모든 module을 Port와 adapter을 통해서 나눠서 개발을 진행합니다. Port는 외부 module과의 통신이 진행되는 경우 사용됩니다.

![그림1](/assets/images/2021-08-23-port-and-adapter/portandadapter-1.png)

위에 그림처럼 실제 핵심 내부 로직인 service는 port를 통해서 들어오고 port를 통해서 다른 모듈을 호출합니다. 

이렇게 하는 경우 핵심 service 로직은 외부에 변경에 대해서 많은 보호를 받게 됩니다. 

모든 Port는 interface입니다. 외부에서 해서 해당 모듈 내부로 들어오게 되는 경우에 만나는 port를 in-port라고 부르고 밖으로 나가는 port에 대해서는 out-port라고 하겠습니다.

in-port의 구현체는 내부 핵심 로직인 service입니다. 그렇게 되면 해당 in-port를 호출하는 controller (http api)는 adapter가 됩니다. 

out-port의 구현체는 외부 모듈과 직접 통신하거나 그것을 감싼 모듈일 것입니다. 예를 들어 redis에 요청을 보내는 모듈은 adapter가 되게 됩니다.


```
└── com
    └── test
        └── pattern
            ├── application
            │   ├── impl
            │   │   └── TotalRentalServiceImpl.java
            │   ├── InventoryService.java
            │   └── TotalRentalService.java
            ├── domain
            │   ├── item
            │   │   ├── Item.java
            │   │   └── ItemRepository.java
            │   └── member
            │       ├── MembershipService.java
            │       ├── User.java
            │       └── UserRepository.java
            └── interfaces
                ├── common
                │   ├── StoredItemDto.java
                │   └── UserDto.java
                ├── member
                │   ├── MembershipController.java
                │   ├── RegisterRequest.java
                │   └── RegisterResponse.java
                └── store
                    ├── StoreController.java
                    └── StoreRequest.java
```

이전 구조와는 다르게 연관성이 있는 패키지들을 한곳에 모아둠으로서 조금 더 구조를 파악하기 쉽습니다.

# 실제 사례 및 장점

## 구조

out-port가 interface이기 때문에 out-port에 어떤 구현이 바뀌더라도 service에서는 아무런 영향을 받지 않습니다. 

in-port를 통해서 해당 interface를 사용한 모듈에서는 service에 대한 변경이 있어도 큰 문제가 없습니다. (어차피 interface가 똑같기 때문에). 

모든 adapter들은 자기들만에 데이터 포맷이 존재할 수 있습니다. 그러기 때문에 변환 과정을 adapter에서 변환하는 과정이 진행됩니다. 

## 테스트

모든 port가 interface이다 보니 테스트를 진행할 때 매우 용이합니다..

직접 port에 대한 interface를 해당 테스트 안에서 구현해서 사용해서 mock과 같은 형식으로 처리할 수 있습니다.

혹은 자주 사용되는 것은 해당 port에 대한 test adapter를 만들어서 활용할수도 있을 것 같습니다.

``` java
interface FindAllUserPort {
    List<User> findAllUser();
}

class TestFindAllUserAdapter implements FindAllUserPort {
    @Override
    List<User> findAllUser() {
        return new ArrayList<User>();
    }
}

class TestUserService {
    
    FindAllUserPort = new TestFindAllUserAdapter();
    FindAllUserPort = new FindAllUserPort() {
        ...
    };
}
```

## 적용한 모습 
``` 
└── com
    └── test
        └── pattern
            ├── domain
            │   ├── port
            │   │   ├── in
            │   │   └── out
            │   ├── model
            │   │   ├── User.java
            │   │   └── Item.java
            │   └── service
            │   │   ├── ...
            │   │   └── ...
            └── application (web-application, scheduler, etc..)
                ├── adapter
                │   ├── in
                │   └── out
                └── config
```

실제 프로젝트에 적용한 모습입니다. 해당 예제는 다른 블로그에 있는 예제를 사용했던 것이기에 지금 모습과 많이 다를 수 있습니다. 하지만 기존 모습과 함께 domain 부분과 appication 부분을 분리할 수 있다고 과정하에 개발이 진행되었습니다.

패키지 순환 구조가 발생하지 않도록 설계했으면 언제든지 domain 패키지와 application 패키지가 분리해서 관리할 수 있도록 되었습니다.

# 객체지향적인 관점

## SOLID

- S: O
  - 한 기능에 대한 책임은 한곳에서 책임 지어야 한다.
  - 모든 기능이 port와 그것에 맞는 adapter를 만들기 때문에 오직 하나의 메시지에 대해서 응답하고 처리하는 클래스, 객체를 만들 수 있다.
- O: O
  - interface 기반으로 새로운 adapter 변경에 대해서는 자유롭고 수정에 대해서는 어떤 변경 없이도 구현이 가능하다.
- L: X
- I (인터페이스 분리 원칙): O
  - 클라이언트가 오직 필요로하는 인터페이스(메시지)에 대해서만 의존한다.
  - 모든 기능에 대해서 Port를 만들고 그것에 대한 adapter를 만들기 때문에 가능하다. 
- D (의존성 역전 원칙): O
  - port와 adapter 관계로 인해서 다형성이 보장이 되며 그것으로 인해서 interface 구현에 대해서 구현하는 사람, 혹은 모듈에 따라서 그것에 상위 클래스 혹은 객체들이 의존하게 됨.

## 응집도와 결합도

- 응집도
  - 오직 하나의 기능(메시지)만을 처리하도록 하기 때문에 응집도가 매우 높다.
- 결합도
  - 모든 메시지 간에 port가 존재하기 때문에 결합도가 매우 낮다.

# 참조

- [라인 엔지니어링 블로그](https://engineering.linecorp.com/ko/blog/port-and-adapter-architecture/)
