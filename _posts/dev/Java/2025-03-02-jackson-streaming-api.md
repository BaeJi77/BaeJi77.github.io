---
title: "Spring에서 대용량 데이터를 Jackson을 통해서 처리하는 방법 (feat. streaming api)"
description: "Jackson streaming api의 장단점과 실제로 얼마나 큰 이점이 있을까?"

categories:
  - java
  - spring
  - dev
tags:
  - dev
  - java
  - jackson
---

해당 글에서 사용한 코드는 [해당 링크](https://github.com/BaeJi77/blog-code/tree/main/2025-03/jackson-streaming-api)에서 확인할 수 있습니다.

# 개요

안녕하세요! 이번 글에서는 `Jackson Streaming API`에 대해 알아보겠습니다. java/kotlin을 사용하는 많은 분들이 JSON 데이터를 다룰 때 Jackson을 사용하지만, 대용량 데이터를 처리할 때는 메모리 문제로 고민하신 적이 있을 거예요. 이런 문제를 해결하는 방법 중 하나가 바로 Jackson Streaming API입니다. 🏃‍♂️💨

이 글에서는 Jackson의 기본 개념부터 Streaming API의 장단점, 그리고 실제 실험을 통해 성능 차이를 분석해볼 예정이니, JSON 처리에 관심 있는 분이라면 끝까지 읽어보세요! 😉

# TL;DR

1. Jackson Streaming API는 `대용량 JSON 데이터`를 메모리 효율적으로 처리할 수 있는 방법이다.
2. Spring Boot 기본 ObjectMapper와 비교했을 때 Jackson Streaming API는 메모리 사용량이 낮고, 처리 속도가 최대 2배 이상 빠르다.
3. 단, 코드가 다소 복잡해지고 유지보수가 어려울 수 있어 사용 목적과 데이터 구조를 고려해 적용하는 것이 좋다.

# 배경

서버와 클라이언트 간 데이터를 주고받을 때, 우리는 특정 포맷을 사용합니다. 사람들이 가장 널리 알고 쓰이는 포맷이 바로 JSON입니다. Java/Kotlin 환경에서 JSON을 다룰 때 가장 많이 사용하는 라이브러리가 Jackson이에요. 특히, Spring Boot에서는 아무런 설정 없이도 기본적으로 Jackson을 사용합니다.

Jackson은 굉장히 강력한 라이브러리지만, 대용량 데이터를 다룰 때는 메모리 부담이 커지는 단점이 있습니다. 이 문제를 해결하기 위해 Jackson Streaming API를 사용하면 보다 효율적으로 데이터를 처리할 수 있습니다. 🚀

## Jackson이란

##  Jackson 라이브러리 개요

`Jackson`은 Java에서 JSON 데이터를 직렬화(Serialization) 및 역직렬화(Deserialization)하는 데 널리 사용되는 라이브러리입니다. 

## JSON 처리 방식

Java에서 JSON을 다루는 방법에는 여러 가지가 있지만, Jackson은 다음과 같은 주요 방식으로 JSON을 처리할 수 있습니다.

- **Data Binding**: `ObjectMapper`를 사용하여 Java 객체와 JSON 간 변환을 수행
- **Tree Model**: `JsonNode`를 사용하여 JSON 구조를 트리 형태로 다루는 방식
- **Streaming API**: JSON 데이터를 순차적으로 읽고 쓰는 방식

기본적으로 가장 많이 사용하는 방식은 `data binding`방식으로 `ObjectMapper`를 이용하는 방식입니다. `data binding`을 풀어서 말하자면 string이나 byte array로 된 데이터를 Java object로 변환하거나 Java object를 string이나 byte array를 도와주는 라이브러리입니다.

# Jackson Streaming api란

`Jackson Streaming API`는 JSON 데이터를 대용량으로 처리해야 하는 경우 혹은 메모리 사용량을 최소화해야 하는 경우에 사용됩니다. 

일반적으로 ObjectMapper 방식은 전체 JSON을 한 번에 메모리에 올려 처리하기 때문에 메모리 소비가 크지만 코드가 간결합니다. 반면, Streaming API는 JSON을 토큰 단위로 읽고 쓰므로 메모리 사용량이 낮고 성능이 뛰어남이 특징입니다.

## 장점.

✅ 대용량 JSON 데이터 처리 가능
- 1GB 이상의 JSON 데이터를 메모리에 로드하면 `OutOfMemoryError` 발생 가능성 높음
- Streaming API는 JSON을 순차적으로 처리하여 대용량 데이터도 안정적으로 처리 가능

✅ 메모리 사용 최적화
- ObjectMapper는 JSON을 Java 객체로 변환할 때 전체 JSON을 메모리에 올려야 함
- Streaming API는 필요한 부분만 읽고 쓰므로 메모리 사용량이 적음

✅ 성능 향상
- ObjectMapper보다 30~40% 성능 향상 가능
- 필요 없는 필드를 스킵하면서 빠르게 JSON을 읽고 쓸 수 있음

## 실제 실험.

- 실험 환경

```
Server: Macbook Pro (M1 Pro) memory 16GB
Spring boot: 3.4.3
JVM: 21
```

> 최대한 독립적 환경으로 실험하기 위해서 요청하기 전 spring boot 재시작을 하고 요청했습니다.

### very small data (10개 - 1K)

- spring boot default objectampper

```
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Request URI: /persons/object/data-10.json
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Streaming API 성능 측정:
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 실행 시간: 71 ms
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 메모리 사용량: 6819224 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 전송된 데이터 크기: 1103 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 처리량: 15.171104753521126 KB/s
```

- jackson streaming api

```
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Request URI: /persons/streaming/data-10.json
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Streaming API 성능 측정:
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 실행 시간: 27 ms
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 메모리 사용량: 2792536 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 전송된 데이터 크기: 1103 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 처리량: 39.894386574074076 KB/s
```


### small data (1000개 - 121KB)

- spring boot default objectampper

```
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Request URI: /persons/object/data-1K.json
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Streaming API 성능 측정:
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 실행 시간: 75 ms
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 메모리 사용량: 7532824 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 전송된 데이터 크기: 121293 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 처리량: 1579.3359375 KB/s
```

- jackson streaming api

```
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Request URI: /persons/streaming/data-1K.json
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Streaming API 성능 측정:
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 실행 시간: 33 ms
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 메모리 사용량: 3004000 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 전송된 데이터 크기: 121293 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 처리량: 3589.3998579545455 KB/s
```

### large data (100,000개 - 13.5MB)

- spring boot default objectampper

```
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Request URI: /persons/object/data-100K.json
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Streaming API 성능 측정:
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 실행 시간: 1265 ms
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 메모리 사용량: 40709760 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 전송된 데이터 크기: 13522333 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 처리량: 10439.054008152174 KB/s
```

- jackson streaming api

```
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Request URI: /persons/streaming/data-100K.json
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Streaming API 성능 측정:
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 실행 시간: 161 ms
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 메모리 사용량: 3063056 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 전송된 데이터 크기: 13522333 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 처리량: 82021.13863548136 KB/s
```

### very large data (10,000,000개 - 1.5GB)

- spring boot default objectampper

```
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Request URI: /persons/object/data-10M.json
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Streaming API 성능 측정:
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 실행 시간: 27616 ms
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 메모리 사용량: 3967265520 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 전송된 데이터 크기: 1492222373 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 처리량: 52768.2651771731 KB/s
```

- jackson streaming api

```
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Request URI: /persons/streaming/data-10M.json
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : Streaming API 성능 측정:
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 실행 시간: 13166 ms
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 메모리 사용량: 13350344 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 전송된 데이터 크기: 1492222373 bytes
[nio-8080-exec-1] c.c.b.b.j.c.StreamingPerformanceFilter   : 처리량: 110682.69870369228 KB/s
```

### 실험 결과

| 데이터 크기     | API 유형                 | 실행 시간 (ms) | 메모리 사용량 (bytes) | 전송된 데이터 크기 (bytes) | 처리량 (KB/s) |
| --------------- | ------------------------ | -------------- | --------------------- | -------------------------- | ------------- |
| Very Small Data | Spring Boot ObjectMapper | 71             | 6,819,224             | 1,103                      | 15.171        |
|                 | Jackson Streaming API    | 27             | 2,792,536             | 1,103                      | 39.894        |
| Small Data      | Spring Boot ObjectMapper | 75             | 7,532,824             | 121,293                    | 1,579.336     |
|                 | Jackson Streaming API    | 33             | 3,004,000             | 121,293                    | 3,589.400     |
| Large Data      | Spring Boot ObjectMapper | 1,265(1.2s)    | 40,709,760(40M)       | 13,522,333                 | 10,439.054    |
|                 | Jackson Streaming API    | 161(0.1s)      | 3,063,056(3M)         | 13,522,333                 | 82，021.139   |
| Very Large Data | Spring Boot ObjectMapper | 27，616(27s)   | 3，967，265，520(3G)  | 1，492，222，373           | 52，768.265   |
|                 | Jackson Streaming API    | 13，166(13s)   | 13，350，344(13M)     | 1，492，222，373           | 110、682。699 |

- 결론

위의 데이터를 통해 다양한 크기의 데이터 세트를 처리할 때 **Jackson Streaming API**가 **Spring Boot Default ObjectMapper**에 비해 전반적으로 더 효율적이라는 것을 알 수 있습니다.

가장 크게 보이는 차이점은 `10만개 데이터`에 대해서 처리할때 **objectampper**은 약 **8배 더 느리게 동작했고 메모리는 약 13배 더 사용**했습니다. `1억개 데이터`를 처리할때는 **objectmapper**는 **2배 더 느리고 실행되었고 메모리는 300배 더 사용**했습니다.

결과적으로 `매우 큰 데이터를 다룰 때 특히` Jackson Streaming API가 성능(특히 메모리 사용량) 면에서 상당한 이점을 제공합니다.

# 팁

- 큰 데이터를 처리할 때 더 큰 이점이 있음.

위에 실험을 기반으로 데이터가 10MB가 넘는 데이터에 대해서 명확한 처리 시간 및 메모리 사용량에 대한 이점이 있는것이 확인됩니다. 작은 데이터를 처리할때는 특별한 처리없이 spring boot objectmapper을 사용하는게 생산성면에서 더 좋을것이라고 생각합니다.

- 활용 가능성이 높음.

이번 테스트 과정에서는 파일을 읽어서 처리했지만 외부 요청을 통한 값을 input stream으로 받아서 처리할수도 있고 db 요청에 대한 결과도 lazy하게 처리해서 다양한 면에서도 실제 메모리 사용량 및 처리 속도를 높일수도 있습니다. (db 요청 같은 경우 transaction을 처리해야될 가능성이 높습니다.)

# 단점 및 주의사항

- 코드를 짜기가 어렵기 때문에 스키마 변경에 예민하게 처리해야되는 로직에는 추천할 수는 없을것 같습니다.
- 추가적으로 매우 복잡한 스키마에 대해서 처리하기가 매우 어렵고 귀찮을수도 있습니다. 이 주의사항은 objectmapper을 직접 선언해서 사용하면 조금 편하게 streaming api를 사용할 수 있습니다.
- 기존 메모리에 있는 object 데이터들를 jackson streaming api를 통해서 데이터를 처리(reqeust에 대한 반환과 file 쓰기 등등)해줘도 충분히 이점이 있지만 데이터 받아서 처리하는 것이 아닌 lazy하게 데이터를 처리하고 streaming 했을때 더 큰 이점이 있다고 생각합니다.

# 요약

Jackson Streaming API는 JSON 데이터를 한 번에 메모리에 올리는 방식이 아니라, 스트림 기반으로 순차적으로 읽고 쓰는 방식을 제공합니다. 이를 통해 대용량 데이터를 다룰 때 메모리 사용량을 최적화하고 성능을 향상시킬 수 있습니다.

실험 결과를 통해 확인된 주요 장점:

- 메모리 절약: Spring Boot의 기본 ObjectMapper보다 메모리 사용량이 현저히 낮음
- 빠른 속도: 데이터 크기가 커질수록 실행 시간이 단축됨
- 대용량 데이터에 적합: 1GB 이상의 JSON 파일을 안정적으로 처리 가능

그러나 Streaming API는 코딩 난이도가 높고, 유지보수가 어려울 수 있으며, 복잡한 JSON 구조를 다루기가 까다롭다는 점을 고려해야 합니다.

따라서, Jackson Streaming API는 특히 대용량 데이터를 다룰 때 강력한 도구이지만, 모든 상황에서 반드시 필요한 것은 아니므로 적절한 사용처를 고민해야 합니다.

# 마무리

이번에는 jackson에서 low level api인 streaming api에 대해서 알아봤습니다. 특히 큰 용량 데이터를 처리할때 장점이 있으며 성능적으로 충분히 좋다는 것을 실험을 통해서 알게 되었습니다. 

하지만 그만큼 코딩을 할때나 유지보수시에 고민해봐야될 수 있어서 도입할때 충분한 고민이 필요한 것으로 보입니다.

# 참고

- https://www.baeldung.com/jackson-streaming-api
- https://cassiomolin.com/programming/combining-jackson-streaming-api-with-objectmapper-for-parsing-json/
