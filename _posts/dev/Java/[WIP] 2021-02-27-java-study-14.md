---
title: "14장. 제네릭"
description: "자바의 제네릭에 대해 학습하세요."

categories:
  - dev
  - java
tags:
  - dev
  - java
---

# 목표
자바의 제네릭에 대해 학습하세요. [issue](https://github.com/whiteship/live-study/issues/14)

# 학습할 것

# 라이브 방송
## 책 추천


## 방송 중
- 제네릭은 어떤 타입인지 모름. 그래서 `.getId()` 이런 것을 사용하지 못함.
- 그런 것을 사용하기 위해서 dao에 대해서 제네릭으로 하나의 구현체만 만들고 모든 변수에 대해서 받을 수 있도록
- 그 jpa repository가 제대로 만들어지지 않았을 떄 이런 형식으로 제네릭을 써서 만들었다. 

- dao를 만들지 않고도 레포지토리를 만들 수 있다.
  - appleDao, bananaDao 필요가 없다.
  - 우리가 리스트를 사용하는데 어떤 클래스를 만드나요? 
  - 전혀 그렇지 않다.
  - 그럼에도 불구하고 이렇게 클래스를 만드는 이유는?
    - 특화된 기능 기능을 구현하고 싶은 경우. 도메인에 특화된 레포지토리에 대해서 기능이 필요한 경우가 있음.
    - banana repository가 아닌 어떤 도메인에 대한 특별한 기능이 필요할 수 있다.
  - 우리는 나중에 빈을 등록해야 된다. 
    - @Repository 이렇게 쓰면 되는 건데 클래스를 사용하지 않는다면
    - @Configuration과 그 안에 메소드를 만들어서 
      ``` java
      @Bean
      public GenericDao<Apple, Integer> appleDao(){
        return GenericDao()<>;
      }
      ```
    - 위와 같은 방식으로 사용할 수 있다. 그냥 빈 등록해버리기

- 브릿지 메소드
  - 제네릭 클래스를 상속받은 경우에 특정 상황에서 메소드를 만들어주는 경우가 있다.
  - 상속 받았지만 그 구체적인 상속으로 인해서 똑같은 메소드이름에 파라미터가 달라지기 때문에 함수 오버로딩이 된다. 
    - 그러기 때문에 완벽한 상속은 아니게 된다.
  - 그런 요소에서 컴파일가 해당 부분을 커버하기 위해서 중간 다리 메소드 하나를 만들게 된다.

- 런타임 중에 제네릭된 타입에 대한 정보를 알아낼 수 있냐?
  - 리플렉션 사용?
  - 아니면 타입을 매번 선언때마다 직접 넣는 방법. (생성자에다가 해당 클래스 정보를 넣음.)
  - 메타데이터로 남는다. 그래서 리플렉션으로 가져오면 된다.
  - 그 파라미터라이즈타입을 사용하여서 값을 가져올 수 있다. 
  - 근데 이게 바로 선언한 경우 ``List<Apple> appleList = ...`` 이런 경우에는 상위 클래스가 오브젝트이기때문에 가져올 수 없고 특정 클래스를 사용속받은 경우에는 가능하다.

## 베스트 글 모음