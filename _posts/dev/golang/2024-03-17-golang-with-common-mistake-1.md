---
title: "golang with common mistake (1)"
description: ""

categories: 
  - dev
tags:
  - dev
  - golang
---

# 개요

해당 내용은 [100 Go Mistakes Go 100가지 실수 패턴과 솔루션](https://product.kyobobook.co.kr/detail/S000211704725) 라는 책에 있는 내용 중 직접 많이 경험하고 도움이 되었던 내용들을 작성하는 것입니다.

더 다양하고 실제 책에 내용이 궁금하다면 해당 책을 읽어보시는 것을 추천드립니다.

# 100 go mistakes

## 코드와 프로젝트 구성

### #1 의도하지 않은 변수 가림을 조심하라.

```go
var err error
if flag {
  v, err := doSometing()
  if err != nil {...}
} else {
  v, err := doSometing2()
  if err != nil {...}
}

if err == nil {
  // always
}
```

위에 코드에서 첫번째 라인에 있는 err는 계속 nil 상태입니다. if문 안에서 `:=`을 이용해서 값을 받는 경우 새로운 스콥이기 때문에 새로운 `err`에 값이 주입됩니다.

그런 경우 원하는 동작보다 똑같은 결과만 나오게 됩니다.

### #3 init 함수를 잘못 사용하지 마라

init 함수는 기본적으로 3가지 문제를 가질 수 있습니다.

1. 에러 관리 능력이 부족하다. <- 에러가 났을 경우 핸들링할 수 없다.
2. 테스트 코드를 작성하기 힘들다. <- 나의 호출로 인해서 동작하는 것이 아니다보니 컨트롤하기 어렵다.
3. 초기화 과정에서 상태를 설정해야 할 경우 글로벌 변수로 처리해야 한다.

위에 3가지 경우에 대해서 어느정도 대답을 할 수 있는 경우라면 init함수를 사용하는 것도 괜찮다고 합니다. 그 예로 책에서는 go blog에 있는 정적 HTTP 설정을 말했습니다.

### #6 제공자 측에 인터페이스를 두지 마라

해당 내용은 이전에 작성한 [google golang comment 정리](https://baeji77.github.io/dev/go-code-review-comment/) 글에서도 나왔던 내용입니다.

개인적으로 golang의 interface는 구현체는 실제 interface의 존재를 알지 못하더라도 interface가 요구하는 메소드를 구현하면 실제 해당 interface의 구현체로 판단합니다.

그러기 때문에 실제 구현체가 있는 곳에서 interface를 만들 필요가 없습니다. 실제로 인터페이스를 사용하는 곳에서 인터페이스를 만들어서 정의하면 됩니다.

- provider.go

```go

var _ DoInterface = &Provider{}

type Provider {
  ...
}

func (p *Provider) Do() error {
  ...
}

func (p *Provider) Other1() error {
  ...
}

func (p *Provider) Other2() error {
  ...
}

```

- comsumer.go

```go
interface DoInterface {
  Do() error
}

type Comsumer {
  do DoInterface
  ...
}

func NewComsumer(do DoInterface) Comsumer {

}
```

위와 같이 Comsumer에서 실제 구현체와 별개로 Comsumer에서 사용할 수 있게 됩니다. 가장 큰 장점은 소비자 입장에서는 구현체 전체를 알 필요가 없고 다른 기능을 건드릴 수 없게 됩니다. 

책에서는 여러 SOLID 원칙과 연결하여서 소개하고 있습니다. 저는 추가적으로 때때로 아닐수도 있지만 의존성이 조금 간결해지는 효과를 보는것 같습니다.

### #7 인터페이스를 리턴하지마라

위에 이야기 했던 `#6`와 조금 이어지는 이야기 일수도 있습니다. 제공자 측에서 interface를 제공할 때 밑에와 같이 반환할 수 있습니다.

```go
var _ DoInterface = &Provider{}

interface DoInterface {
  Do() error
}

type Provider {
  ...
}

func NewDoInterface() DoInterface {
  return &Provider{...}
}

func (p *Provider) Do() error {
  ...
}
```

이렇게 `DoInterface` 반환하게 되면 실질적으로 해당 Provider은 캡슐화되었다고 생각하지만 해당 컴포넌트에서 가지고 있는 추가적인 기능을 만들때 마다 DoInterface는 커질 수 밖에 없습니다.

처음에는 Do() 라는 메소드만 있을 수 있지만 지속적으로 커지는 경우 불필요한 Interface가 커지고 역할이 분리되어지게 됩니다.

그리고 실질적으로 그것을 받아서 사용하는 Comsumer 입장에서는 알 필요가 없는 메소드들까지 알고 사용하게 됩니다.

결국 #6과 같은 결론으로 단순히 Provider는 해당 기능을 하고 실제 사용하는 곳에서 interface를 정의해서 제공해주면 됩니다. 그렇게 되면 Comsumer쪽 코드 변경은 없으면 Provider만 다른 곳에서 계속 사용할 수 있습니다.

### #8 any는 아무것도 알려주지 않는다

any는 golang에서 interface{}를 따로 alias한 타입입니다. interface{}는 모든 값을 저장할 수 있습니다.

그렇기 때문에 실제 golang에서 가지고 있는 정적 타입 컴파일러의 장점인 컴파일 단계에서 에러를 핸들링할 수 있는 장점을 활용할 수 없을수도 있습니다.

그래서 실질적으로 값을 표현할때는 any를 사용하기 보다는 특정 타입을 사용하는 것을 추천하고 있습니다.

any를 사용하는 케이스는 두가지라고 합니다.

1. `encoding/json` 패키지를 사용할 때.
2. `database/sql` 패키지를 사용할 때.

### #11 함수형 옵션 패턴을 사용하라

어떤 component를 만들 때 실제 configuration 값을 어떻게 전달하고 그 전달한 값을 가지고 componenet(instance)를 만들지 고민하는 경우가 많습니다.

어떤 특정 값은 0일수도 있고 어떤 특정 값은 validate를 따라야되는 경우가 있습니다.

`Config 구조체`를 사용하거나 `빌더 패턴`을 사용하거나 합니다.

하지만 golang에서는 `함수형 옵션 패턴`을 많이 사용한다고 합니다.

한마디로 Option값을 가지고 있는 함수를 전달해서 component를 선언할때 적용한다고 생각하시면 됩니다.

```go
type ClientManager{
  port int
  endpoint string
}

type Option func(*ClientManager) error

func WithPort(port int) ClientManager {
	return func(m *ClientManager) {
    if port < 0 {
      ...
    }

		m.port = port
	}
}


func NewClientManager(options ...Option) ClientManager {
  cm := ClinetManager{}

  for _, opt := range options {
		opt(cm)
	}

  return cm
}
```

실제로 위와 같이 사용할 수 있습니다. 해당 메소드를 사용하는데 여러 값들을 변경하면서 사용하는 경우 매우 유용하게 사용되고 실제 golang 오픈소스에서 많이 발견됩니다.

### #12 프로젝트를 제대로 구성하라

golang은 정형화된 코드 구조가 딱 정해져있지는 않습니다. 자바를 사용하게 되면 기본적으로 어떤 구조를 많이 사용하고 어떤 디렉토리를 구조를 사용한다고 하는데 그런 것들이 딱 정해져 있는 느낌은 아니었습니다.

하지만 권장사항은 있는 것 같습니다. 모든 golang 프로젝트가 해당 사항을 따르는 것은 아니지만 해당 사항을 따랐을때 다른 코드를 읽을때와 다른 사람이 나의 프로젝트를 읽을때 도움이 될것으로 예상합니다.

[go project-layout](https://github.com/golang-standards/project-layout)를 참고 부탁드립니다.

### #16 린터를 활용하라

linter는 굉장히 개발자가 할 수 있는 많은 실수를 잡아줍니다. #1에서 나왔던 문제도 잡아줄 수도 있으면 기타 여러 옵션들을 통해서 추후 발생할 수 있는 문제들을 잡을 수 있습니다.

golang 자체적으로 `govet`이라는 것을 제공합니다. 하지만 더 많고 다양한 lint를 사용하고 싶으시다면 + formatter을 사용하고 싶다면 `golangci-lint`를 꼭 찾아서 적용해보시기를 바랍니다.

## 데이터 타입

### #20 슬라이스의 길이와 용량을 정확하게 이해하라

array와 비슷하지만 vector라는 데이터 타입이 존재합니다. 현재 용량보다 큰 데이터가 더 들어오게 되면 배열 용량을 두배로 증가하는 로직이 들어가 있는 다이나믹 array라고 말할 수 있습니다.

golang에서는 slice는 vector로 구현되어 있습니다. 그래서 실제로 2가지 개념이 존재합니다. 실제 elements의 수인 길이와 slice가 현재 받을 수 있는 elements의 수를 나타내는 용량이라는 개념이 있습니다.

slice에 append를 하다가 용량보다 길이가 커지게 되면 현재 용량보다 큰 새로운 slice를 만들고 현재 elements를 복사해서 옮기는 작업을 합니다. (1024개 이하면 2배 큰 slcie를 만들고 그 이상이면 25% 큰 slice를 만듭니다.) 그 작업으로 지속적으로 데이터를 받을 수 있게 되는 것입니다.

해당 개념을 헤깔리게 되면 실제 slice를 선안할때부터 큰 메모리 낭비를 할수도 있으면 원하지 않은 동작들을 할수도 있습니다. 그리고 성능적으로 문제가 있을 수 있습니다.

이런 개념에 대해서 반드시 알아야 됩니다.

### #22 nil과 빈 슬라이스를 혼동하지 마라

slice를 선언할 수 있는 방법은 매우 다양합니다. 책에서는 4가지가 나왔습니다. 그 경우마다 장단점이 있기 때문에 그것을 고려해서 선언하라고 합니다.

```go
var s []string // empty=true, nil=true

s = []string(nil) // empty=true, nil=true

s = []string{} // empty=true, nil=false

s = make([]string, 0) // empty=true, nil=false
```

추가적으로 책에서는 이렇게 권장하고 있습니다.

```go
var s[]string: 최종 길이를 모르고 슬라이스가 빌 수도 있는 경우
[]string(nil): nil 슬라이스이면서 empty슬라이스를 생성하는 편의 구문
make([]strign, length): 최종 길이를 아는 경우. <- 이 경우는 실제 성능적으로 slice 길이를 조절하지 않아도 되기 때문에 성능이 좋은 편입니다.
```

### #23 슬라이스가 비었는지 제대로 확인하라

#22과 연결되는 내용입니다.

모든 slice은 선언과 동시에 nil이라도 길이가 0입니다. 그래서 실제 slice가 nil이라도 len(slice)를 통해서 검증하게 되면 원하지 않는 동작을 할 수 있습니다.

결국 모든 slice를 길이로만 검증하려고 한다면 자신이 원하지 않은 결과가 나올 수도 있습니다.

### #26 슬라이스와 메모리 누구 관련 실수

- 용량 누수

slice에 대해서 golang 자체적으로 제공해주는 slicing하는 방법이 있습니다. 예를 들면 `slice[:5]`와 같은 방식으로 slice를 짤라서 다른 slice로 사용할 수 있습니다.

하지만 이렇게 되었을 경우에는 길이는 실제로 slicing한 만큼이지만 용량은 그대로입니다.

한마디로 길이도 1억, 용량이 1억인 slice를 `slice[:5]`이렇게 해서 처음 5개 데이터만 가지고 오는 slice를 새로 만들었을때, 길이는 5, 용량은 1억인 slice가 만들어지는 것입니다.

한마디로 사용하지 않는 용량이 큰 slice를 만들 수 있다는 것입니다. 

해결 방법은 간단하게 하나하나 복제하는 것입니다.

- 슬라이스와 포인터

slice을 포인터를 가진 slice에 대해서 slicing한 이후에 접근하지 못하더라도 gc 이후에도 지속적으로 메모리에 남아있는 문제가 있습니다.

해결방법으로는 slicing을 원하는 slice만큼 복제를 진행하거나 명시적으로 필요하지 않은 엘레멘트에 대해서는 명시적으로 nil로 값을 넣는것입니다.

### #27 비효율적인 맵 초기화 관련 실수

golang의 map은 hash table 기반으로 bucket을 넣어서 데이터를 저장합니다.

배열이 있고 그 배열에는 버킷이 있는데 키-값 으로 배칭된 bucket이 있습니다. 만약 데이터를 계속 넣을때 똑같은 배열 값, 똑같은 버켓이 계속 데이터가 들어간다면 `O(n)`에 데이터 처리될 수 있습니다.

만약 map의 크기를 알 수 있다면 해당 문제를 최대한 최적화할 수 있습니다.

# 마무리

실제 개발을 하면서 발생했던 문제들도 있지만 추후에 많이 신경 쓸 수 있는 것들도 많이 배운 것 같습니다. 실제로 더 다양한 내용이 있어서 더 많이 공부하고 싶다는 생각이 듭니다.

다음에 더 딥한 내용으로 더 작성해보겠습니다.
