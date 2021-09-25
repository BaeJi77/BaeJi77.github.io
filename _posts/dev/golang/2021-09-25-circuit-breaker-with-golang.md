---
title: "golang으로 circuit breaker 구현해보기"

categories:
  - dev
  - golang
tags:
  - dev
  - golang
  - circuit breaker
---

[마틴 파울러 아저씨의 글](http://martinfowler.com/bliki/CircuitBreaker.html)을 기반으로 작성했습니다.

# circuit breaker

요즘 여러 서비스에서 MSA를 적용하면서 모든 기능이 하나의 서버에 존재하기 어렵다. 그 뿐만 아니라 다양한 내가 관리할 수 없는 외부 서비스가 크게 존재할 수도 있다. 그러면서 자연스럽게 RPC (remote procedure call)를 하는 경우가 많아졌다. 

RPC가 진행되는 과정에서 내가 요청하는 remote 서비스가 실패로 인해서 연쇄적인 작용이 일어날 수도 있다.

```
# 정상
A -> B -> C

# C 서비스 불능
1. A -> B -x> C
2. A -x> B-x> C
```

자연스럽게 B 요청이 실패하게 되면 연쇄적으로 A도 해당 RPC가 실패하게 됨을써 에러가 전파되는 문제가 발생한다. 단순히 실패가 문제는 아니다. 

실패가 진행되었을 때 만약 retry가 진행되는 로직이 있거나 특정한 회복 로직이 존재하지 않다면 B 서비스에 대한 또 다른 위험 요소가 되기도 한다. (많은 요청이 retry를 하면서 많은 메모리와 thread를 가지게되면서 문제가 발생할수도 있다. 혹은 C 서비스가 정상적으로 돌아올 때 까지 기달릴 수도 있다.)

그런 상황을 방지하기 위해서 만들어 진것인 `circuit breaker` 이라는 개념이다. 

> Circuit breaker is a design pattern used in software development. It is used to detect failures and encapsulates the logic of preventing a failure from constantly recurring, during maintenance, temporary external system failure or unexpected system difficulties.

`circuit breaker`는 회로 차단기라는 의미가 있으며 특정 선정한 임계치가 넘게 되면 요청을 하지 못하도록 하는 패턴이다.

상태로는 CLOSE, OPEN, HALF-OPEN 가 존재한다. CLOSE는 RPC가 지속적으로 진행해도 되는 상태이며, OPEN은 RPC를 막은 상태, HALF-OPEN은 OPEN에서 상태를 회복하기 위해서 특정 조건에 따라서 RPC를 실행하는 상태이다. 

# Implementation with golang

마틴 파울러 아저씨 글에서는 다른 언어로 만들어진 코드가 파편적으로 존재한다. 해당 코드들과 개념들을 이용해서 golang을 통해 구현해보겠다.

## algorithm

`circuit breaker`에는 임계치라는 개념이 존재한다. 특정 임계치를 어떻게 설정하느냐에 따라서 알고리즘이 달라진다. 

내가 적용할 알고리즘은 연속적으로 실패하는 RPC 수를 기반으로 만들었다. 만약 OPEN 상태에서 지정한 특정 시간이 지난 이후에 한번씩 HALF-OPEN으로 RPC를 실행하게 되며 그것을 기반으로 CLOSE가 될지 계속 OPEN이 될지 결정하게 된다.

## 실행 결과

[sample code](https://github.com/BaeJi77/blog-code/tree/main/2021-09/go-circuit-breaker)

``` go
type CircuitBreaker interface {
	Reset()
	RecordFailure()
	IsAvailable() bool
}

type FailureCountThresholdBreaker struct {
	mutex                  sync.Mutex
	name                   string
	state                  State
	failureThreshold       int32
	continuousFailureCount int32
	lastFailureTime        time.Time
	resilienceWaitingTime  time.Duration
}

// 몇번 연속을 실패하는지에 대한 값. OPEN이 된 상태 이후로 얼마 이후 다시 요청을 할 것인지에 대한 시간 값을 받음
func NewFailureCountThresholdBreaker(threshold int32, resilienceWaitingTime time.Duration) CircuitBreaker {
	return &FailureCountThresholdBreaker{
		mutex:                  sync.Mutex{},
		state:                  Close,
		failureThreshold:       threshold,
		continuousFailureCount: 0,
		lastFailureTime:        time.Now(),
		resilienceWaitingTime:  resilienceWaitingTime,
	}
}
```

``` go
func main() {
	cb := NewFailureCountThresholdBreaker(1, 3*time.Second)

	log.Printf("cb.IsAvailable()=%t\n", cb.IsAvailable())

	cb.RecordFailure()
	log.Printf("cb.IsAvailable()=%t\n", cb.IsAvailable())

	cb.RecordFailure()
	log.Printf("cb.IsAvailable()=%t\n", cb.IsAvailable())

	time.Sleep(5 * time.Second)
	log.Printf("cb.IsAvailable()=%t\n", cb.IsAvailable())

	cb.Reset()
	log.Printf("cb.IsAvailable()=%t\n", cb.IsAvailable())
}
---

2021/09/25 17:02:58 cb.IsAvailable()=true       (1)
2021/09/25 17:02:58 cb.IsAvailable()=false      (2)
2021/09/25 17:02:58 cb.IsAvailable()=false      (3)
2021/09/25 17:03:03 cb.IsAvailable()=true       (4)
2021/09/25 17:03:03 cb.IsAvailable()=true       (5)
```

main에 있는 간단한 코드에서 1번 실패했을 시 circuit breaker 상태를 OPEN을 만들도록 했다.

> (1) 은 당연히 괜찮다.
> (2) 한번 실패 이후에 가능한지 확인하니 안된다고 한다.
> (3) 특정 시간이 지나지 않았기 때문에 당연히 안된다.
> (4) 3초 이후부터는 HALF-OPEN 상태로 요청을 보낼 수 있는 상태이다.
> (5) 만약 RESET를 한다면 CLOSE상태로 되기 때문에 요청이 가능한 상태로 바뀐다.

# 다양한 library
직접 구현이 가능하지만 임계치를 정하는 알고리즘이 매우 다양하다. 그것과 관련해서 다양한 라이브러리들이 존재한다.

혹시 구현을 하지 않고 직접 구현된 라이브러리를 쓰고 싶다면 밑에 있는 것들을 확인해보고 도입해보는게 좋을 것 같다.

- https://github.com/sony/gobreaker
- https://github.com/rubyist/circuitbreaker
- https://github.com/afex/hystrix-go
- https://github.com/eapache/go-resiliency


# ref
- http://martinfowler.com/bliki/CircuitBreaker.html
- https://engineering.linecorp.com/ko/blog/circuit-breakers-for-distributed-services/

