---
title: "[Clean code] chapter 10. 클래스"

categories:
  - dev
  - book
tags:
  - dev
  - book
  - clean code
---

# Overview
우리는 지금까지 코드 행과 코드 블록을 올바로 작성하는 방법에 초점을 맞췄다. 아무리 코드와 라인에 신경쓰더라도 더 높은 단계에 대해서 알지 못하면 깨끗한 코드를 만들기 어렵다.

`클래스`에 대해서 알아보자.

## 클래스 체계
`자바` 같은 경우 변수 목록이 나온다. 정적 공개 상수, 정적 비공개 변수, 비공개 인스턴스 변수가 나온다.

변수 목록 다음에는 공개 함수가 나온다. 비공개 함수는 자신을 호출하는 공개 함수 직후에 넣는다. 추상화 단계가 순차적으로 내려간다.

### 캡슐화
변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 한다는 법칙도 없다. 때로는 protected로 선언해 테스트 코드에 접근을 허용하기도 한다.

우리에게 테스트는 매우 중요하다. 하지만 그 전에 비공개 상태를 유지할 온갖 방법을 강구한다.

## 클래스는 작아야 한다.
클래스를 만들 때 첫 번째, 두 번째, 세 번쨰 규칙 모두 작아야 한다는 것이다. 클래스를 설계할 때에도 함수와 마찬가지로 더 작게가 기본 규칙이다.

그렇다면 얼마나 작아야되는 가에 대한 질문이 있을 수 있다. 

함수는 물리적인 행 수로 크기를 측정했다. 클래스는 맡은 책임을 센다.

클래스의 크기, 담당하고 있는 메소드가 적다고 해서 크기를 측정할 수 없다. 메소드가 적다고 책임이 적은 것은 아니다.

`클래스 이름은 해당 클래스 책임을 기술해야 한다.` 그러기 때문에 모호한 이름을 사용은 하지 말아라. Processor, Manager, Super등과 같은 단어는 좋지 않다.

### 단일 책임 원칙
SRP은 클래스나 모듈을 변경할 이유가 하나, 단 하나뿐이어야 한다는 원칙이다. SRP는 `책임`이라는 개념을 정의하며 적절한 클래스 크기를 제시한다. 클래스는 책임, 즉 변경할 이유가 하나여야 한다는 의미이다.

책임, 즉 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다. 더 좋은 추상화가 더 쉽게 떠오른다.

SRP는 객체 지향 설계에서 더욱 중요한 개념이다. 또한 이해하고 지키기 수월한 개념이기도 하다.

소프트웨어 개발에서 사람들은 돌아가는 것에 초점을 맞추고 있는 깨끗하고 체계적인 것에 대해서 초점을 맞춰져 있다. 하지만 어떻게 되든 실제 작은 클래스가 많은 시스템이든 큰 클래스가 몇 개뿐인 시스템이긴 잘 돌아간다.

어느 시스템을 익힐 때는 그 양은 비슷하다. 하지만 만약 큰 클래스를 있는 경우에는 한번 수정에 대해서, 코드를 익힐 때 너무 많은 데이터를 알아야 한다. 하지만 클래스가 작으면 책임이 하나이기 때문에 변경할 이유가 하나이며 다른 시스템, 클래스를 알 필요가 없다.

### 응집도
클래스는 인스턴스 변수 수가 작아야 한다. 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다. 일반적으로 메소드가 변수를 더 많이 사용할 수록 응집도가 더 높다.

일반적으로 응집도가 높은 클래스를 선호한다. 응집도가 높다는 말은 클래스에 속한메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미이기 때문이다.

때때로 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아진다. 이는 클래스 쪼개서 응집도를 높여야된다는 신호이다.

### 응집도를 유지하면 작은 클래스 여럿이 나온다.
큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다. 

예를 들자면, 큰 함수 일부를 작은 함수로 만들려고 한다. 작은 함수에서 큰 함수의 인자 4개를 사용한다면 모두 파라미터로 받아야되는가? 인스턴스 변수로 만들면 된다. 하지만 그렇게 되면 응집력을 잃는다. 자주 사용하면 변수들을 클래스로 만들어서 관리한다면 그것으로 작은 클래스를 만들 수 있고 그것을 통해 응집력을 잃는 것을 방지할 수 있다.

> 만약 큰 함수를 작은 함수로 나누고 클래스를 나누는 과정이 진행된다면?

그렇게 되는 경우 보통 프로그램이 길어진다. 그 이유는 세가지라고 말할 수 있다. 첫째, 리팩토링한 프로그램은 좀 더 길고 서술적인 변수 이름을 사용한다. 둘째, 코드에 주석을 추가하는 수단으로 함수 선언과 클래스 선은을 활용한다. 셋째, 가독성을 높이고자 공백을 추가하고 형식을 맞추었다.

하지만 이렇게 나눔으로써 각 책임을 각 클래스가 가지고 되는 만약 요구사항이 변경되고 내부 로직이 변경되는 경우 해당 클래스만 바꾸면 되는 엄청난 장점을 얻게 된다.

## 변경하기 쉬운 클래스
대다수 시스템은 지속적인 변경이 가해진다. 그리고 뭔가 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다. 깨긋한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다.

어떤 상황이 오면 복잡하든 간단하든 클래스에 손을 되야 한다. 하지만 그런 수정이 다른 코드를 망가뜨릴 위험이 존해한다.

> 예시로 복잡한 SQL class를 단순한 클래스 집합으로 만듬

경험에 의하면 클래스 일부에서만 사용되는 비공개 메서드는 코드를 개선할 잠재적인 여지를 시사한다. 그리고 그것은 당장이 아닌 실제 해당 클래스를 수정해야되는 설계를 개선하려는 고민과 시도가 필요하다.

### 변경으로부터 격리
`요구사항은 변하기 마련이다. 따라서 코드도 변하기 마련이다.`

우리는 객체지향 프로그래밍에 구체적인 클래스와 추상 클래스가 있다고 배웠다. 구체적인 클래는 구현이 바뀌면 위험에 빠진다. 그래서 우리는 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.

상세한 구현에 의존하는 코드는 테스트가 어렵다. 예를 들어, Portfolio 클래스를 만든다고 해보자. 해당 클래스는 외부 API를 사용하기 때문에 테스트 코드는 시세 변화에 영향을 받는다. 하지만 해당 API를 직접 호출하는 대신 StockExchange라는 인터페이스를 생성한 후 메서드를 하나 선언한다.
``` java
public interface StockExchange{
   Money currentPrice(String symbol);
}
---

public Portfolio{
   private StockExchange exchange;
   public Portfolio(StockExchange exchange) {
      this.exchange = exchange;
   }
}
```
이렇게 되면 StockExchange 실제 구현체가 변경되더라도 변경에 대해서 문제가 없으면 실제 테스트를 하는데에도 StockExchange를 테스트 용으로 구현하여서 주입하게 되면 테스트에도 큰 무리가 없어진다.

이와 같이 설계하게 되면 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다. 결합도가 낮다는 소리는 각 시스템 요소가 다른 요소로 뿌터 그리고 변경으로부터 잘 격리되어 있다는 의미이다.

## 결론
클래스 설계에 있어서도 클린 코드는 존재한다. 

특정 변수를 많이 사용하면 그것을 이용하여 하나의 작은 클래스를 만들 수 있다. 그것을 통해 서로의 책임을 잘 분리할 수 있다.

# key point
- 책임. 클래스를 분리하는 기준는 책임이다. 
- 작게 만들어라. 책임을 서로 최대한 분리하다보면 결합도가 올라간다.
- 추상화를 통해 클래스 간에 응집도를 낮춘다.

# 마무리
단순한 몇 페이지에서 객체지향의 원칙이 거의 모두 나온 것 같다. 그 뿐만 아니라 소프트웨어 개발에 기본이 되는 응집도와 결합도 까지. 굉장히 중요한 이야기라고 생각한다.

무엇보다 마지막 부분에 추상화로 인한 결합도를 낮추게 되면 테스트에도 용이하다는 내용은 참 유익한 내용 같다.
