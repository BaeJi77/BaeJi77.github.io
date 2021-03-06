---
title: "11장. Enum"
description: "자바의 Enum에 대해 학습하세요."

categories:
  - dev
  - java
tags:
  - dev
  - java
---

# 목표
자바의 Enum에 대해 학습하세요. [issue](https://github.com/whiteship/live-study/issues/11)

# 학습할 것
## 열거형 (Enum)이란?
- 서로 관련된 상수를 편리하게 선언하기 위한 것.
- 자바의 열거형은 `타입에 안전한 열거형` 이라서 실제 값이 같더라고 타입이 다르면 다르다는 결과를 얻는다.

## 이전에 상수를 처리하던 방법
``` java
public class Number {
    static final int ONE = 1;
    static final int TWO = 2;
}

public class Number2 {
    static final int ONE = 1;
    static final int TWO = 2;
}

main {
    if (Number.ONE == Number2.TWO) {
        // 이런 식이 가능
    }
}

```
위에 내용은 사실 전혀 같지 않다는 것을 알 수 있지만 타입이 같기 때문에 비교를 한다. 

상수를 이용하는데 똑같은 타입을 사용하는데 있어서 실수가 존재할 수 있다. -> 전혀 다른 상수의 집합을 표현하고 있다고 하더라도 전혀 컴파일 단계에서 걸러내지 못함.

## enum 정의하는 방법
``` java
enum 열거형이름 {상수명1 , 상수명2, 상수명3, ...}
---
```

``` java
enum Fruit {
    APPLE(5000), BERRY(1000), ...

    Fruit(int price) {
        this.price = price;
    }

    public int getPrice() {
        return price;
    }
}

---

main() {
    Fruit fruit = Fruit.APPLE;
    sout(fruit.getPrice);
}

---

5000

```
위와 같이 정의해서 사용할 수 있다.

우리는 상수만 정의해서 사용하는 것이 아닌 상수에 대한 추가적인 정보까지 작성해서 묶음으로 관리할 수 있다.

위에 예제를 보게 되면 과일과 관련해서 가격까지 넣을 수 있다.


### Enum type별 처리 방법
#### switch문을 활용한 방법
``` java
enum Fruit {
    APPLE(5000), BERRY(1000), ...

    Fruit(int price) {
        this.price = price;
    }

    public int getPrice() {
        return price;
    }
}

---

main() {
    Fruit fruit = Fruit.APPLE;
    switch(fruit) {
        case Fruit.APPLE:
            sout("이것은 애플");
        case Fruit.BERRY:
            sout("이것은 베리");
    }
}
---

이것은 애플
```

#### 메소드를 추가하는 방법
``` java
enum Fruit {
    APPLE(5000) {
        @Override
        void print() {
            sout("이것은 애플");
        }
    },
    
    
    BERRY(1000) {
        @Override
        void print() {
            sout("이것은 베리");
        }
    }, 
    
    ...

    Fruit(int price) {
        this.price = price;
    }

    public int getPrice() {
        return price;
    }

    abstract void print();
}
---

main() {
    Fruit fruit = Fruit.APPLE;
    fruit.print();
}

---

이것은 애플
```

## enum이 제공하는 메소드 (values()와 valueOf())
### values()
``` java
enum Fruit {
    APPLE, BERRY, ..
}
---

main() {
    for (Fruit f : Fruit.values()) {
        sout(f.name());
    }
}
---

APPLE
BERRY
```
- Enum이 가지고 있는 모든 벨류를 보여준다.

### valueOf()
``` java
enum Fruit {
    APPLE, BERRY, ..
}
---

main() {
    Fruit fruit = Fruit.valueOf("APPLE");
    sout(fruit);
}
---

APPLE
```
- 상수 값을 이용해서 Enum 인스턴스를 만드는 방법이다. 
- 여기서 중요한 것은 값은 것이 있어야지 된다는 것이다.
- 소문자를 받고 싶다고 조금 커스텀해서 적용할 수도 있다.

``` java
enum Fruit {
    APPLE, BERRY, ..

    public Fruit toValue(String fruit) {
        if (fruit == null) {
            return null;
        }

        return valueOf(fruit.toUpperCase());
    }
}
```

- java.lang.Enum
Enum 클래스를 만들게 되면 기본적으로 해당 `java.lang.Enum` 를 상속받게 된다. 

그렇기 때문에 우리가 현재 기본적으로 사용하고 있는 `values()`, `name()`, `valueOf()`와 같은 것을 사용하고 있는 것이다.

대부분의 메소드가 final로 정의되어 있기 때문에 오버라이딩은 불가능하다.

## EnumSet
``` java
enum Fruit{
	APPLE
}
---

main(String[] args){
	EnumSet<Fruit> enumSet = EnumSet.allOf(Fruit.class);
}
```
- allOf: 모든 Enum 값을 넣음.
- noneOf: 특정 것들을 제거한다.
- of: 원하는 것을 선택해서 넣음
- range: 범위에 있는 값을 모두 넣음


# Ref
[wisdom님의 블로그](https://wisdom-and-record.tistory.com/52)

[study내 블로그](https://www.notion.so/Enum-6ffa87530c424d8ab7a1b585bfb26fa2)


# 라이브 방송
## 책 추천

## 방송 중


## 베스트 글 모음
https://wisdom-and-record.tistory.com/52

https://b-programmer.tistory.com/262

https://www.notion.so/Enum-6ffa87530c424d8ab7a1b585bfb26fa2

https://parkadd.tistory.com/50

https://velog.io/@kwj1270/Enum

https://velog.io/@ljs0429777/11%EC%A3%BC%EC%B0%A8-%EA%B3%BC%EC%A0%9C-Enum

https://yadon079.github.io/2021/java%20study%20halle/week-11

https://www.notion.so/11-Enum-ccbba2bf2b7746ed8a12d2dee09aa833

https://blog.naver.com/hsm622/222218251749
