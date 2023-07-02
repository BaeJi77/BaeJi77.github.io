---
title: "About go code review comment"
description: ""

categories: 
  - dev
tags:
  - dev
  - golang
---

# 개요

해당 글을 github [golang/go CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments) 라는 golang에 code reviwe와 관련된 문서를 기반으로 요약해서 작성했습니다.

해당 내용들이 golang code를 작성하는데 정답은 아니라고 생각합니다. 그럼에도 글을 작성하는 이유는 팀내에서 해당 글을 기반으로 이야기를 전달받은 내용이 있어 정리를 하기 위해서 글을 작성하게 되었습니다.

번역을 하는 중에 의도가 잘못 전달될 수 있기에 혹시 잘못된 부분이 있다면 알려주시면 감사합니다.

# 본 글

## Context

`context.Context`는 여러 데이터를 가지고 핸들링하는 경우도 있으며 특정 요청이나 실행에 대해서 취소 시그널에서도 사용된다. Go 프로그램은 Context를 명시적으로 전달하여 수신한 RPC 및 HTTP 요청부터 외부 요청까지 전체 함수 호출 체인을 따라갑니다.

- Context를 사용하는 대부분의 함수는 첫 번째 매개변수로서 해당 Context를 받아야 합니다:

```go
func F(ctx context.Context, /* other parameters */) {}
```

- 특별히 요청을 사용하지 않은 경우 Context를 전달하는 것이 일반적이며 그런 경우 context.Background()를 사용하시는게 좋습니다.
- 구조체 안에 field에 Context를 멤버로 추가하지는 대신에 각 메서드에 ctx 매개변수를 추가하여 전달하십시오.
- Custom Context struct, interface를 만들어서 사용하지 마세요.
- Context는 변경되지 않은 객체이기 때문에 똑같이 사용되어야는 경우에 같은 객체를 사용해도 괜찮습니다.

## Copy

- 일반적으로, 포인터 유형 *T와 연관된 메서드가 있는 경우 T 유형의 값을 복사하지 마십시오.

밑에는 정확한 제가 이해한 부분에 대해서 이렇게 예시를 작성해봤습니다.

```go
type Data struct {
	Value int
}

func (d *Data) ModifyValue(newValue int) {
	d.Value = newValue
}

func main() {
	data := Data{Value: 1}

	copiedData := data
	copiedData.ModifyValue(10) // This will not modify the original data

	// Print the values
	fmt.Println("Original Data:", data.Value)
	fmt.Println("Copied Data:", copiedData.Value)

	data2 := &Data{Value: 2}

	copiedData2 := data2
	copiedData2.ModifyValue(11) // This will modify the original data

	// Print the values
	fmt.Println("Original Data 2:", data2.Value)
	fmt.Println("Copied Data 2:", copiedData2.Value)
}
---
Original Data: 1
Copied Data: 10
Original Data 2: 11
Copied Data 2: 11

```

## Declaring Empty Slices

```go
var t []string

or

t := []string{}
```

이렇게 비어있는 slice를 선언하는 경우에 첫번째에 존재하는 nil 가능성이 있는 slice 선언 방식을 선호하라고 합니다. 

작성자의 첨부) 해당 이유를 찾아봤습니다. 정확한 이유인지 모르겠지만 nil로 했을 경우와 실제로 값이 명확하게 없는 경우를 다르게 프로그래밍할 수 있는 경우가 있기 때문에 그런것 같습니다. 선언을 nil로 해서 실제로 값이 주입되지 않으면 nil로 끝나지만 길이가 zero인 slice를 만들면 이게 실제로 값이 주입한것인 아닌지 구별하기 어렵기 때문인것 같습니다.


## Goroutine Lifetimes

- 고루틴을 생성할 때는 고루틴이 언제 종료되는지 명확하게 해야합니다.
- 고루틴은 채널로 인해서 블로킹되어서 메모리 누수가 발생할 수 있습니다. ([참고자료](https://mcauto.github.io/back-end/2019/02/27/goroutine-leaks/))
- 만약 메모리 누수가 되지 않더라도 더 이상 필요하지 않다면 종료시키는게 좋습니다. 고루틴은 패닉을 발생시키거나 데이터 경쟁 문제를 발생시킬 가능성을 가지고 있습니다.

- 동시성 코드를 작성할때 가능한 간단하게 만들어서 고루틴의 수명을 명확하게 보이도록 하세요!

## Imports

- 이름 충돌이 일어나는 것이 아니라면 import한 패키지의 이름을 바꾸지 마세요.
- imports가 black line와 함께 특정 구조화되어져서 그룹핑 되어야된다.
- golang 기본 패키지들은 항상 첫번째 그룹에 존재합니다.

```go
package main

import (
	"fmt"
	"hash/adler32"
	"os"

	"github.com/foo/bar"
	"rsc.io/goversion/version"
)
```

## In-Band Errors

C 언어에서 처럼, 에러 상황에서 `0`, `-1`, `null`, `""`를 반환하는 경우가 있습니다.

```go
// Lookup returns the value for key or "" if there is no mapping for key.
func Lookup(key string) string

// Failing to check for an in-band error value can lead to bugs:
Parse(Lookup(key))  // returns "parse failure for value" instead of "no value for key"
```

Go에서는 multiple return value를 제공하고 있습니다. client은 그것을 이용해서 명확하게 return된 값이 valid한지 알수 있습니다. return되는 값이 value와 함께 error일수도 있고, 따로 설명할 필요가 없는 경우 boolean을 사용하기도 합니다.

```go
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

```go
Parse(Lookup(key))  // compile-time error

---
value, ok := Lookup(key)
if !ok {
	return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

`nil`, `""`, `0`, `-1`과 같은 반환 값들에 대해서 호출자가 다른 값들과 다르게 처리할 필요가 없는 경우는 괜찮습니다.

## Indent Error Flow

- 코드가 진행되는 경우 최소한의 들여쓰기를 유지하기 위해서 노력하십시오. 그렇게하게 되면 코드를 더 쉽게 읽을수 있을것입니다.

예를 들어,

```go
// bad
if err != nil {
	// error handling
} else {
	// normal code
}

// good
if err != nil {
	// error handling
	return // or continue, etc.
}
// normal code
```

```go
// bod
if x, err := f(); err != nil {
	// error handling
	return
} else {
	// use x
}

// good
x, err := f()
if err != nil {
	// error handling
	return
}
// use x
```

## Interfaces

- Go 언어에서 interface는 해당 인터페이스를 사용하는 패키지에 속합니다. 인터페이스를 구현하는 패키지는 구체적인 유형을 반환해야 합니다. (interface를 반환하는 것이 아니라.) 이렇게 하면 새로운 메서드를 구현에 추가할 때 광범위한 리팩토링이 필요하지 않습니다.

-> interface를 사용하는 쪽에서 만들라고 이야기하는 것 같습니다. 그리고 해당 의존성이 있어야될 것을 외부에서 파라미터로 주입받은 형태로 사용하라고 하는것 같습니다.
-> [Accept interfaces, return structs](https://bryanftan.medium.com/accept-interfaces-return-structs-in-go-d4cab29a301b)에 대한 개념을 말하는 것처럼 느껴졌습니다.

```go
package producer  // producer.go

type MetricSender struct{ … }
func (s MetricSender) Send(data any) err { … }

// interface를 반환하는 것이 아닌 구체적인 객체를 반환함
func NewMetricSender() MetricSender { return MetricSender{ … } }

---
package consumer  // consumer.go

// 사용하는 쪽에서 해당 interface를 구현함.
type Sender interface { 
  Send(data any) err
}

// 그리고 그것에 대해서 interface를 주입해서 사용하는 식으로.
func Do(s Sender) error {
  err := s.Send()
  ...
}
---

package main

func main() {
  ms := producer.NewMetricSender()

  err := comsumer.Do(ms)
  ...
}
```

- mocking을 위해서 구현하는 쪽에서 interfaces를 구현하지 말아라.

- 여러곳에서 사용하는 것이 아니라면 interface를 먼저 만들어서 사용하지 말아라.

## Package Names

- 패키지에 대한 이름은 모든 의존성을 가지는 경우에 사용하게 되기 때문에, 너는 identifiers를 제거할 수 있습니다. 

`chubby.ChubbyFile` (chubby: package name, ChubbyFile: type name)인 경우에 type에 있는 `Chubby`를 제거하여서 `chubby.File`를 사용하세요.

- `util`, `common`, `misc`, `api`, `types`, and `interfaces`와 같은 의미없는 package name을 피하세요.

## Receiver Names

- generic names (me, this, self)와 같은 이름을 사용하지 말아라.
- 굳이 설명이 가능한 이름으로 할 필요가 없다. 한글자 혹은 두글자로도 충분하다.
- 하지만 반드시 일치성은 중요하다. 만약 `c`로 했다면 지속적으로 `c`를 사용해라. `cl`, `client`를 사용하지 말고.

```go
type Client struct {
    // Fields and methods
}

// Good example
func (c *Client) Connect() {
    // Implementation
}

// Bad example - using generic names
func (me *Client) Connect() {
    // Implementation
}

// Bad example - inconsistent receiver names
func (cl *Client) Connect() {
    // Implementation
}
```

## Receiver Type

작성자의 내용) 많은 경우 pointer receiver을 권장하는 것 같습니다. 특히 반드시 pointer receiver이어야 되는 경우도 있습니다.

- receiver이 map, func, chan인 경우에는 pointer을 **사용하지 마세요!**
- receiver에 대해서 변경을 해야되는 경우 반드시 pointer을 **사용하세요.**
- receiver가 Mutex or 동기화 과정이 있다면 복제를 방지하기 위해 반드시 pointer을 **사용하세요.**
- 동시에 호출되는 메소드로 인해서 변경이 되는 경우에 reciver는 pointer을 **사용하세요**.
- receiver가 struct인데 가지고 있는 값들이 slice, array, 그리고 변경이 되는 원소를 가지고 있다면 pointer을 사용하는 것을 **선호합니다.**
- receiver가 변경되지 않으면 포인터를 가지고 있지 않으며 간단한 타입만 가지고 있다면 value receiver가 직관적이다. 
- 가능한 모든 메소드에 대해서 receiver type을 섞어서 사용하지 말아라. 
- 마지막으로 의심스러운 경우 ponter receiver을 **사용해라.**


## Synchronous Functions

- 동기 함수를 선호하세요.
- 동기 함수는 호출 내에서 고루틴에 대한 이해와 데이터 경쟁을 피하는데 도움이 됩니다. 그리고 테스트도 쉬워지며 호출자가 데이터 결과를 기달리는 것도 처리하지 않아도 됩니다.
- 호출자가 동시성이 필요하다면 고루틴으로 함수를 호출하여 처리할 수 있도록 하세요.

```go
// good: 동기 함수 
func Sum(a, b int) int {
    return a + b
}

// bod: 비동기 함수
func SumAsync(a, b int, result chan int) {
    result <- a + b
}

// good: 동기 함수를 사용하는 호출
sum := Sum(2, 3)

// bad: 비동기 함수를 사용하는 호출
result := make(chan int)
go SumAsync(2, 3, result)
sum := <-result
```

# 소감

모든 내용들을 정리한 것은 아니지만 그럼에도 좋은 내용들이 많았다고 생각합니다. 

반드시 정답은 아니지만 그럼에도 이런 정보들이 많이 사용되고 좋은 practice처럼 사용된다면 충분히 좋은 공부가 된것 같습니다. 무엇보다 글을 정리하면서 왜 이렇게 하지라는 질문을 하다보니 더욱 받아들이기 쉬웠던 것 같습니다.

기회가 되면 `effective-go` 혹은 `ultimate-go`에 도전해보고 싶다는 생각이 들었습니다.
