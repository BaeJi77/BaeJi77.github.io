---
title: "about golang - goroutine basic (1)"
description: ""

categories: 
  - dev
tags:
  - dev
  - golang
  - goroutine
---

# Introduction

프로그래밍 세계에서 동시성은 성능과 효율성을 향상시키는 데 중요한 역할을 합니다. 

Go 프로그래밍 언어의 주요 기능 중 하나는 `goroutine`으로 알려진 ``경량 동시성 메커니즘``입니다. 

이 글에서는 `goroutine`과 그 이점, Go에서 동시 프로그래밍에 기여하는 방법을 살펴봅니다.

# `goroutine`이란?

`goroutine`의 핵심은 Go 런타임에서 관리하는 경량 스레드입니다. 이를 통해 함수 또는 메서드를 독립적으로 동시에 실행할 수 있습니다. `goroutine`은 동시 시스템을 설계하는 간단하고 표현적인 방법을 제공하여 Go를 동시 프로그래밍 작업을 처리하기 위한 강력한 언어로 만듭니다.

반면 스레드는 운영 체제에서 관리합니다. `goroutine`에 비해 더 많은 메모리가 필요하고 오버헤드가 더 높습니다. 대조적으로 `goroutine`은 매우 효율적으로 설계되었으며 최소한의 비용으로 생성 및 파괴할 수 있습니다. 이를 통해 Go 프로그램은 손쉽게 확장하고 수천 개의 `goroutine`을 동시에 처리할 수 있습니다.

이 부분에 대해서는 [다음 글]()에서 구체적으로 설명하겠습니다!

# 간단한 사용법: `goroutine` 살펴보기

추가 `goroutine`을 생성하려면 `go` 키워드와 동시에 실행하려는 함수를 사용할 수 있습니다. 아래 코드를 확인해보세요!

```go
func main() {
    // Code executed by the main goroutine

    go helloWorld() // Create a new goroutine
}

func helloWorld() {
    // Code executed by the new goroutine
}
```

이 예제에서 `helloWorld` 함수는 별도의 `goroutine`으로 동시에 실행됩니다. 이를 통해 프로그램은 여러 작업을 동시에 수행할 수 있으므로 성능과 응답성이 향상됩니다.

## main goroutine

Go에서 `main goroutine`은 프로그램이 시작될 때 자동으로 생성됩니다. 프로그램 실행을 위한 진입점 역할을 합니다. `main` 함수에 작성된 모든 코드는 이 `goroutine` 내에서 실행됩니다. 

Go에서 모든 실행에 대해서 `goroutine`이 동작하고 있다고 생각하면 되지만 `main goroutine`은 일반적인 goutine과 다르게 유일하게 생성되는 `goroutine`이라는 것을 기억하셔야 됩니다.


# `goroutine`의 장점

- Lightweight and Efficient: `goroutine`은 가벼워 성능에 큰 영향을 미치지 않고 수천 개를 생성할 수 있습니다. 최소한의 메모리 오버헤드가 있으며 Go 런타임에서 효율적으로 예약하고 관리할 수 있습니다.
- Simplified Concurrency: Go는 `goroutine` 간의 통신 및 동기화를 위한 채널과 같은 내장 메커니즘을 제공합니다. 따라서 기존의 스레드 기반 프로그래밍에 비해 더 적은 수의 동기화 프리미티브로 동시 프로그램을 더 쉽게 작성할 수 있습니다.
- Scalability: 개발자는 `goroutine`을 사용하여 많은 수의 동시 작업을 처리할 수 있는 확장성이 뛰어난 시스템을 설계할 수 있습니다. 작업을 독립적으로 실행할 수 있고 명시적인 조정이 필요하지 않은 시나리오에 적합합니다.
- Responsive Applications: 개발자는 `goroutine`을 활용하여 장기 실행 작업 중에도 높은 반응성을 유지하는 반응형 애플리케이션을 구축할 수 있습니다. `goroutine`은 비동기 실행을 가능하게 하여 I/O 작업이나 비용이 많이 드는 계산을 기다리는 동안 다른 작업을 계속할 수 있도록 합니다.

# `goroutine` 사용 시 주의사항

- Resource Management: `goroutine`을 사용할 때 적절한 리소스 관리를 해야 합니다. 예를 들어, 파일 I/O나 네트워크 작업이 포함된 경우 연결을 닫고 리소스를 적절하게 해제하여 누수나 과도한 리소스 사용을 방지해야 합니다.
- Concurrency issue: 여러 `goroutine`이 동시에 공유 데이터에 접근하고 수정하는 경우 `data race`이 발생하거나 동시성을 관리하기 위해서 lock을 사용하다가 `deadlock`이 발생할 수 있습니다. 이를 피하기 위해 뮤텍스나 채널과 같은 동기화 메커니즘을 사용하여 공유 리소스에 대한 접근을 조정하고 데이터의 일관성을 보장해야 합니다.
- Goroutine Leaks: `goroutine`을 적절하게 처리하지 않으면 `goroutine` 누수가 발생할 수 있습니다. 이는 `goroutine`이 생성되었지만 제대로 종료되지 않은 상태를 의미합니다. 이로 인해 리소스의 과도한 사용이 발생하고 애플리케이션의 성능이 저하될 수 있습니다. 더 이상 필요하지 않은 경우 `goroutine`을 정상적으로 종료하는 것이 중요합니다.

Go에서 `goroutine`은 정말로 강력한 도구입니다. 하지만 위에 있는 주의사항을 신경쓰지 않는다면 프로그램이 정상적으로 동작하기 어려우며 신뢰성이 떨어질 수 있습니다.

## 주의사항 극복

### Concurrency issue

동시성 프로그래밍을 하는 경우에 반드시 `data races` 상황은 발생할 수 있는 문제입니다. 데이터 경합 상태는 여러 스레드 또는 고루틴이 적절한 동기화 없이 동시에 공유 데이터에 액세스하고 수정할 때 동시 프로그래밍에서 발생하는 일반적인 문제입니다. 동일한 메모리 위치에 대한 두 개 이상의 동시 작업이 적절하게 순서가 지정되지 않아 프로그램의 예측할 수 없고 잘못된 동작이 발생할 때 발생할 수 있습니다.

이것을 해결하기 위해서 `Mutex`, `RWMutex`, `Channel`를 이용해서 해결합니다.

- `Mutex`, `RWMutex`: race condition이 발생하는 코드 영역에 대해서 하나의 gorotine만 접근하도록 막습니다. `RWMutex`는 read와 write에 대해서 mutex를 다르게 설정하여서 read에 있어서 성능적으로 우수하게 처리하겠다는 전략으로 만들어졌습니다.

```go
var mu &sync.Mutex

func RaceConditionMethoe() {
  mu.Lock()
  defer mu.Unlock()

  ...
}
```

```go
var mu &sync.Mutex

func RaceConditionMethoe() {
  mu.Lock()
  // start raceo condition code section
  ...
  // end raceo condition code section
  mu.Unlock()
}
```

위에 코드에서와 같이 특정 코드 영역에 접근할 수 있는 goroutine이 하나로 제한함으로서 문제가 생기지 않도록 막는것입니다.

그 외에서 golang에서 자체적으로 제공해주는 [sync](https://pkg.go.dev/sync) 패키지를 참고해보세요!

최근에 관심있게 보고 있는 패키지로는 [conc](https://github.com/sourcegraph/conc)라는 입니다. 기본으로 제공해주는 `sync`에 발생하는 지속적인 문제를 간단하게 해결해주는 패키지라고 생각하면 됩니다.

### Goroutine Leaks

Goroutine 같은 경우는 만드는 것에 대해서 막지 못합니다. 단순히 `go` 키워드와 함께 메소드를 실행하게 되면 바로 gorutine을 생성할 수 있습니다. 그리고 특별히 개별적인 gorotine에 대해서 관리하거나 트래킹하기가 어렵습니다.

그렇기 때문에 잘못 gorotine을 만들었을 경우 누수되기 매우 쉽습니다. 해당 경우를 막기 위해서는 간단하게 방법을 소개하겠습니다.

1. gorotine에 종료 조건을 명확하게 하기. 특정 목적으로 실행했을 경우 그 실행 이후에는 반드시 종료되도록 조건을 만들어두는게 좋습니다. gorotine은 실행 이후 특별히 컨트롤할 수 없지만 `context`같은 데이터를 전달하여서 goroutine을 컨트롤할 수 있습니다.
2. golang profile: [pprof](https://github.com/google/pprof)는 동작하고 있는 go 프로그램에 대한 정보를 보여줍니다. 해당 정보들을 통해서 gorotine에 대해서 분석할 수 있습니다.
3. go 정적 분석 도구: `go vet`는 golang에서 지원하는 정적 분석을 도와줍니다.

# FAQ

Q: 하나의 함수 내에서 여러 개의 `goroutine`을 생성할 수 있나요?
A: 예, 하나의 함수 내에서 각각의 `goroutine`을 생성하기 위해 `go` 키워드를 사용할 수 있습니다. 많은 데이터를 병렬적으로 빠르게 처리하고 싶을때 `for` statement를 이용해서 가능합니다.

Q: `goroutine`은 한 개의 CPU에서만 실행되나요?
A: 아니요, `goroutine`은 여러 개의 CPU에서 동시에 실행될 수 있도록 설계되었습니다. 이를 통해 사용 가능한 리소스를 효율적으로 활용할 수 있습니다.

Q: 실행 중인 `goroutine`을 취소할 수 있나요?
A: Go는 `goroutine`에 대한 직접적인 취소 메커니즘을 제공하지 않습니다. 그러나 `context 취소` 또는 `채널 기반의 신호 전달`과 같은 기법을 사용하여 `goroutine`을 graceful하게 종료할 수 있습니다.

# 결론

Go의 `goroutine`은 동시성 실행을 가능하게 해주는 핵심 기능으로, 성능과 확장성 면에서 상당한 이점을 제공합니다. 

`goroutine`을 사용하면 개발자는 효율적이고 성능이 매우 좋은 동시성 프로그래밍을 쉽게 처리할 수 있습니다. 그러나 리소스 관리, 데이터 경쟁, 동시성 버그에 대한 주의사항을 고려하는 것이 중요하다고 생각합니다.

추후 글에서 goroutine이 왜 효율적인지 그리고 어떤 부분을 조심해야 되는지에 대해서 작성해보겠습니다. 
