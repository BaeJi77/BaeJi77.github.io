---
title: "golang에서 rate limit을 적용해보자!"
description: "golang 패키지를 이용하여 rate limit 알고리즘 이해와 token bucket 적용해보기"

categories: 
  - dev
  - golang
tags:
  - dev
  - golang
  - rate limit
---

# rate limit 이란

> 컴퓨터 네트워크에서 속도 제한은 네트워크 인터페이스 컨트롤러가 보내거나받는 요청 속도를 제어하는 ​​데 사용됩니다. DoS 공격을 방지하고 웹 스크래핑을 제한하는 데 사용할 수 있습니다. 

출처: wiki

어떤 개발을 진행할 때 remote service에 요청을 보내서 데이터를 가져오거나 데이터를 저장하는 경우가 있습니다. 하지만 외부 서비스 같은 경우 너무 갑작스러운 트래픽으로 인해서 다른 사용자들에게 안좋은 경험을 줄 수가 있습니다.

이런 경우를 대비하기 위해서 request 요청 수를 제한하는 것입니다.

## 활용 예시

1. Dos 공격 방지

Dos 공격은 서버 리소스를 모두 사용해버려서 다른 사용자가 해당 서비스를 사용하지 못하도록 만드는 공격입니다. 그런 것을 방지하기 위해서 너무 많은 트래픽이 서비스에 한번에 들어오는 것을 막을 수 있습니다.

2. 비용 절감

트래픽에 대해서 제한이 걸려 있기 때문에 특별히 더 많은 트래픽을 받기 위해서 비용을 지불하지 않다고 된다. 그 뿐만 아니라 제 3자에게 비용을 받으면서 서비스를 하는 경우 rate limit을 통해서 제한을 걸 수 있습니다.

적용할 수 있는 경우는 client에서 적용하는 경우와 server 쪽에서 적용하는 경우가 있습니다.

기본적으로 server 쪽에서 적용했을 경우 더욱 편하고 활용도가 높다는 생각은 합니다.

client 같은 경우 client library를 제공하지 않은 경우 사용자가 직접 구현을 해야되는 문제와 동일한 트래픽을 보낸다는 가정이 없기 때문에 rate limit을 어떻게 적용해야될지에 대한 기준이 어렵습니다. 

## 알고리즘

해당 feature에 대한 여러 구현이 가능합니다. 가볍게 알고리즘에 대해서 알아봅시다.

1. token bucket

알고리즘 자체가 매우 간단하며 여러 기업들일 사용하는 알고리즘입니다. 

어떤 특정 bucket이 있습니다. 해당 bucket에 capacity가 존재합니다. 그리고 다시 refill되는 rate가 존재합니다. 

요청을 실행할 때마다 token이 하나씩 사용됩니다. 해당 bucket에서는 capacity 이상 절대 토큰을 가질 수 없습니다. 그러기에 refill되기 전까지는 요청을 처리할 수 없습니다. 

refill되는 방식도 여러 방법이 있습니다. 엄청 짧은 시간 단위로 token이 생성되는 경우와 특정 시간마다 한번씩 생성되는 경우가 있습니다. (1분에 60개씩 refill이 된다고 가정했을 경우. 1초 1개씩 refill되는 경우가 있을 수 있고, 1분에 한번씩 60개가 refill될 수도 있습니다.)

> capacity가 5이며, refill이 3/min 라고 했을 경우 (1분에 한번씩 refill된다고 가정)

처음 1분동안 10개에 요청이 오더라도 token을 5개 밖에 없기 때문에 5개만 처리할 수 있습니다. 그 이후 refill이 되어서는 3개를 처리할 수 있습니다. 

그 이후에 요청이 없을 경우 계속 refill이 될 것이고 하지만 해당 bucket은 최대 5개에 token을 가지고 있습니다.

- 장점
  - 구현이 쉽다. 
  - endpoint마다 적용했을 경우 사용자 구분이 쉬움
- 단점
  - bucket capacity와 refill rate에 대한 configuration value을 잘 적용하기가 쉽지 않다.


2. leaky bucket

token bucket과 매우 비슷하지만 요청 처리율이 고정되어 있습니다. FIFO 큐를 활용해서 구혆ㅂ니다.

- 요청이 도착하면 큐가 가득 차 있는 확인하고 가득 차 있지 않은 경우 추가한다.
- 가득 차있는 경우 새 요청은 버린다.
- 지정된 시간마다 큐에서 요청을 꺼내어 처리한다.

parameter로 bucket size와 처리율을 설정합니다. 처리율은 1분에 몇개를 처리할 것인가에 값입니다.

- 장점
  - 큐 크기가 제한적이기 때문에 메모리에 이점.
  - 처리율이 고정이기 때문에 항상 어느정도 처리하고 그 이상 처리하지 않을 것이라는 확신이 있습니다.
- 단점
  - burst 트래픽이 온 경우에는 오래된 요청만 남아있고 새로운 요청은 모두 버려진다.
  - parameter에 대해서 튜닝이 어려움이 있다.


추가적으로 고정 윈도우, sliding 윈도우 알고리즘 등 다양한 알고리즘이 존재합니다.

# In golang

## 라이브러리

- [go/x/time: token-bucket rate limit algorithm](https://github.com/golang/time)

golang에서 지원하는 패키지 중에 time 확장 버전에 속한 라이브러리가 있습니다. `go get`을 통해서 패키지를 다운아서 사용해야 합니다.

내부 코드를 봤을 경우 `token-bucket` 알고리즘을 활용해서 구현했다고 나와있습니다.

- [uber: leaky-bucket rate limit algorithm](https://github.com/uber-go/ratelimit)

uber에서 go 패키지 중에 하나이며 `leaky-bucket` 알고리즘을 활용해서 구현했다고 합니다.


그 외에서 다양한 라이브러리가 존재할 것이라고 생각합니다.

## 코드

`golang.org/x/time/rate` 을 활용해서 해당 feature에 대해서 보여드리겠습니다. 

``` go
import (
	"fmt"
	"time"

	"golang.org/x/time/rate"
)

func main() {
	// 1초 1개 허용. bucket size: 10
	limiter := rate.NewLimiter(rate.Limit(1), 5)

	r1 := limiter.Allow()
	fmt.Printf("request r1: %v\n", r1)

	r2 := limiter.Allow()
	fmt.Printf("request r2: %v\n", r2)

	r3 := limiter.Allow()
	fmt.Printf("request r3: %v\n", r3)

	r4 := limiter.Allow()
	fmt.Printf("request r4: %v\n", r4)

	r5 := limiter.Allow()
	fmt.Printf("request r5: %v\n", r5)

	r6 := limiter.Allow()
	fmt.Printf("request r6: %v\n", r6)

	time.Sleep(1 * time.Second)
	r7 := limiter.Allow()
	fmt.Printf("request r7: %v\n", r7)

	r8 := limiter.Allow()
	fmt.Printf("request r8: %v\n", r8)
}
```

- 결과

``` go
request r1: true
request r2: true
request r3: true
request r4: true
request r5: true
request r6: false
request r7: true
request r8: false
```

해당 라이브러리는 단순히 가능한지에 대한 allow 메소드를 제외한 2가지에 메소드를 더 지원합니다. 우리는 token이 없을 경우에 처리할 수 있는 경우의 수가 다양할 수 있습니다. 

단순히 안되면 실패라는 메시지를 보낼 수 도 있지만 기달리다가 처리할 수도 있으면 몇초를 기달리다가 타임아웃을 줄수도 있는 것입니다. 

유즈 케이스에 따라서 적용하는게 달라질 수 있다고 생각합니다.


# ref

- [가성면접 사례로 배우는 대규모 시스템 설계 기초 책](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966263158)
- https://engineering.linecorp.com/ko/blog/high-throughput-distributed-rate-limiter/
