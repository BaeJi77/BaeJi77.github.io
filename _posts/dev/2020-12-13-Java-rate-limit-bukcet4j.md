---
title: "Rate limit를 통해서 특정 행동을 제약해보자!!"

categories:
  - dev
  - java
tags:
  - dev
  - java
  - rate limit
  - bucket4j
---

# Overview
우리는 가끔씩 해당 메소드에 사용 제한을 하고 싶을 수 있다. 혹은 API를 공개했는데 특정 유저들에게는 제한을 하고 싶다. 이랬을 경우 우리는 해당 유저를 판단하고 사용 제한을 하도록 해야 한다. 그 방법에 대해서 고민하고 어떻게 해야되는지 이야기해보자.

# 이것을 왜 사용했나요?
Overview에서 이야기했 듯이 우리는 어떤 경우에는 제한적으로 서비스를 제한해야되는 경우가 존재한다. 성능의 한계, 동시성 문제, 혹은 스케줄링을 통한 특정 시간에만 동작하도록 하고 싶을 수 있다. 당연히 해당 문제에 대해서는 모두 각각 솔루션이 있을 것이다. 우리는 `Rate limit`을 통해서 특정 시간안에 총 몇번을 사용할 수 있는지에 대한 제한을 할 수 있다.

# Token bucket
해당 요소와 관련해서 여러 알고리즘이 존재할 수 있다. 나는 Token bucket과 관련해서 소개해주겠다. [위키 주소](https://en.wikipedia.org/wiki/Token_bucket)

`Token bucket`이란 양동이안에 토큰이 존재하고 그것이 존재하는 경우에만 사용이 가능한 경우를 말한다. 

``` text
1. 1분에 100번만 요청이 가능하도록 메소드를 만들었다. 초기에 버켓에 120개의 token이 존재
2. 해당 메소드를 1번 안에 120번 요청한 이후에는 해당 메소드를 요청할 수 없다. 버켓에 남는 token이 없으면 메소드 요청 불가. -> 모든 요청이 손실
3. 1분 후에 다시 120개의 token이 생성되면 다시 요청 가능
``` 
구현에 따라 다르지만 (1)중간 중간 토큰을 생성할 수도 있으면 (2)정확히 1분 마다 100개씩 만드는 경우도 있다. 
(1)같은 경우 위에 예시와 다르게 1초 마다 2개의 토큰이 버켓에 추가된다.
(2)같은 경우 위에 예시와 똑같다.

여기서 중요한 것은 bucket안에 token이 없는 경우에는 모든 트래픽을 버리는 것이다. (leaky bucket, couter semaphore와 다른 부분)


# 다른 라이브러리
## Guava - Rate limiter
구글에서 만든 자바 라이브러리인 Guava에 Rate limiter라는 친구가 존재합니다. 

기능적으로 분당으로 bucket을 관리하는 기능이 있다. 

## 세마포어
보통 세마포어를 생각하면 mutex를 생각하게 되는데 couter semaphore라는 개념이 존재한다. 세마포어라는 개념 자체에 대해서 잘 이해하시고 계신다면 아마 아실 것이라고 생각합니다.

세마포어 자체는 int, long과 같은 숫자 변수를 활용하는 자료구조라고 말할 수 있을 것 같은데요. 

# Bucket4j
[공식 Github](https://github.com/vladimir-bukhtoyarov/bucket4j)에 존재하는 사용 예시를 보면 이해가 참 빨리 될 수 있다.

공식 문서를 보게 되면 밑에 존재하는 3개의 객체가 가장 핵심적인 역할을 한다. 

## Refill
bucket에 token을 얼마주기로, 얼마만큼 채울것인지에 대해서 설정하는 클래스이다.

``` java
Refill refill = Refill.greedy(10, Duration.ofSeconds(1));

Refill refill = Refill.intervally(10, Duration.ofSeconds(1));
```
위에 두가지 방법이 존재한다. (다른 것도 있지만 이 2가지가 대표적이다.)

첫번째에 존재하는 것은 위에 설명한 (1)번 케이스이다. 중간 중간에 지속적으로 토큰을 리필한다.

두번째는 (2)번 케이스이다. 중간 중간에 절대 채우지 않고 딱 그 시간 만을 지켜서 한번에 채워준다.

## Bandwidth
Bandwidth는 bucket의 총 크기가 어느정도 이냐?를 설정하는 클래스이다. 한마디로 버킷 사이즈가 아무리 커도 refill이 작으면 별로 안들어오고 refill이 아무리 잘되도 bucket 사이즈 (bandwidth 설정)에 따라서 원하지 않은 동작을 할 수 있다.

``` java
Bandwidth bandwidth = Bandwidth.classic(10, Refill.greedy(10, Duration.ofSeconds(1)));

Bandwidth bandwidth = Bandwidth.simple(10, Duration.ofSeconds(1));

```
위에 두가지 방식으로 설정할 수 있다.

첫번째 같은 경우는 우리가 직접 Refill을 선언해서 넣어준다. 해당 부분은 세밀하게 셋업이 가능하다고 할 수 있다.

두번째 같은 경우 Duration만 지정해주면 되는데 내부 구조를 보게 되면 `Refill.greedy()`로 구현되어 있다.

## Bucket
`Bucket`이라는 친구는 모든 설정을 통해서 만들어진 우리의 최종 완성본이다. 우리가 만든 configuration을 builder을 통해서 같이 넣어줘서 만들어주면 된다.

``` java
Bandwidth bandwidth = Bandwidth.simple(100, Duration.ofMinutes(1));
Bandwidth bandwidth2 = Bandwidth.simple(10, Duration.ofSeconds(1));
Bucket bucket =  Bucket4j.builder()
                        .addLimit(bandwidth)
                        .addLimit(bandwidth2)
                        .build();
```
단순히 설정은 하나만 넣는게 아니고 여러개도 넣을 수 있다. 위에 설정은 1분에는 100개 사용을 허락해주지만 1초에는 10개까지 사용할 수 있도록 설정한 것이다.

``` java
bucket.tryConsume(1);
bucket.getAvailableTokens();
bucket.replaceConfiguration(bucketConfiguration);
```
위와 같은 설정으로 만들어진 `Bucket` 객체를 이용해서 위와 같은 메소드를 사용할 수가 있다. `tryConsume(1)`와 같은 경우 해당 토큰을 사용하겠다는 내용인데 저기 `1`이 다른 숫자로 사용할 수 있다. 한번에 여러개를 사용할 수 있는 것이다.

두번째에 존재하는 `getAvailableTokens()`와 같은 경우 현재 남아 있는 token 수를 얻을 수 있다.

세번째 메소드같은 경우 현재 bucket에 대한 configuration을 바꿀 수 있다. 

내가 설명한 메소드 말고도 스케줄링과 관련되도록 가능하며 다양한 메소드를 지원해주니 공식 홈페이지를 참조하기 바란다. 

# 주의 사항
1. capacity를 0으로 설정할 수 없다. 당연히 0으로 설정하는 것은 bucket을 사용하지 않는 것과 같지만 나는 사용하고 싶지만 못한다... 막혀져 있다...
2. `replaceConfiguration()` 메소드가 친절하지 않다. `Bandwidth`의 사이즈를 줄였지만 실제 token 수가 줄어들지 않는다. 버그인 것인지 설계의도인지 모르겠다. [github issue](https://github.com/vladimir-bukhtoyarov/bucket4j/issues/135)

# 마무리
여러 경우에 사용될 수 있는 개념이지만 여러 기능을 가지고 있는 라이브러리가 많지 않다. 해당 라이브러리와 관련해서 문서화가 생각보다 잘 되어 있고 기능이 매우 간단해서 사용하기 편하다.

그 뿐만 아니라 API와 관련해서 추가적인 기능을 제공하는데 여러 방면으로 활용할 수 있다고 생각한다.


# Ref
[Bucket4j github](https://github.com/vladimir-bukhtoyarov/bucket4j)
