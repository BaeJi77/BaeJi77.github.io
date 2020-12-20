---
title: "Java 난수을 만드는 방법 (Random 라이브러리)"
description: "How to create random number in java?"

categories:
  - dev
  - java
tags:
  - dev
  - java
---

# Overview
우리는 가끔씩 랜덤으로 어떤 수를 만들어서 로직에 사용해야되는 경우가 있다. Java에서는 어떤 방법으로 만들 수 있는지 알아봅시다.

# `Random`
`java.util.Random`이라는 친구는 기본적으로 Java에서 제공해주는 라이브러리입니다. Random값을 얻을 수 있습니다. 
``` java
import static org.assertj.core.api.Assertions.assertThat;

import java.util.Random;

import org.junit.jupiter.api.Test;

class RandomStudyTest {

    @Test
    void randomTest() {
        Random random = new Random();
        for (int i = 0; i < 100; i++) {
            int randomNumber = Math.abs(random.nextInt()) % 100;
            assertThat(0 <= randomNumber && randomNumber < 100).isTrue();
        }

        for (int i = 0; i < 100; i++) {
            int randomNumber = random.nextInt(100);
            assertThat(0 <= randomNumber && randomNumber < 100).isTrue();
        }
    }
}
```
100보다 작은 수를 만드는 경우를 생각해봤을 때 2가지 방법이 있다고 말할 수 있습니다..

Random 메소드는 단일 쓰레드에서 동작하는 경우 큰 문제는 없습니다. 하지만 멀티 쓰레드 환경에서는 성능적인 문제를 만들 수 있습니다.

`Random`을 호출하는데 있어서 `seed`를 접근하는데 있어서 성능적으로 떨어지게 됩니다.

구체적인 이유는 실제 밑에 있는 것처럼 AtomicLong으로 만들어진 `seed` 값때문입니다. 해당 값을 접근하는데 있는데 막 접근하는게 아니고 순서를 지켜가면서 접근하기 떄문에 성능적인 문제가 있습니다. 알고리즘적으로 글로벌하게 해당 값을 접근하기 위해서 노력하고 그 과정에서 성능적으로 떨어지게 됩니다. (Random값을 얻기 위해서 기달리고 도전하는 과정이 리소스 낭비)

``` java
class Random implements java.io.Serializable {
    ...
    private final AtomicLong seed;
    ...
}
```

# `ThreadLocalRandom`
`ThreadLocalRandom`는 `ThreadLocal`과 `Random` class가 합쳐졌고 그것으로 인해서 현재 thread에 대해서 독립적으로 동작합니다. 그러기 때문에 `Random`에서 발생하는 리소스 낭비가 없습니다. 

그리고 `Random`과 다르게 seed를 설정하지 못하고 만약 seed 설정시 exception을 발생시킵니다.

``` java
ThreadLocalRandom threadLocalRandom = ThreadLocalRandom.current();
```
위와 같은 방법을 사용하게 되면 현재 thread에 대한 `ThreadLocalRandom`을 얻을 수 있습니다.

위와 같은 문제를 해결하기 위해서 JDK1.7 부터는 `ThreadLocalRandom`라는 class가 생겼습니다.

`ThreadLocalRandom`는 각 인스턴스별로 각자 seed 값을 가지고 있기 때문에 `Random`보다 속도적으로 빠릅니다. 

`Random`과 `ThreadLocalRandom` 모두 `thread-safe`하지만 속도적으로는 `ThreadLocalRandom`이 친구가 더 좋습니다.

``` java
import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.ThreadLocalRandom;

import org.junit.jupiter.api.Test;

class RandomStudyTest {

    @Test
    void threadLocalRandomTest() {
        ThreadLocalRandom threadLocalRandom = ThreadLocalRandom.current();
        for (int i = 0; i < 100; i++) {
            int randomNumber = Math.abs(threadLocalRandom.nextInt()) % 100;
            assertThat(0 <= randomNumber && randomNumber < 100).isTrue();
        }

        for (int i = 0; i < 100; i++) {
            int randomNumber = threadLocalRandom.nextInt(100);
            assertThat(0 <= randomNumber && randomNumber < 100).isTrue();
        }
    }
}
```
`Random`을 이용하는 경우와 똑같습니다.. 더 다양한 메소드를 지원하지만 기본적으로는 이렇게 사용할 수 있습니다.

# 마무리
우리는 항상 여러 thread를 사용해서 프로그램이 동작하는 것을 인식하는게 좋습니다. `Random`을 사용해도 문제는 없습니다. 하지만 혹시 `Random`을 많이 사용해야된다면, 혹은 작은 일이지만 Thread를 많이 사용하는 환경이라면 `ThreadLocalRandom`가 훨씬 좋은 선택일 수 있습니다.

# Ref
https://www.baeldung.com/java-thread-local-random

http://dveamer.github.io/backend/JavaRandom.html