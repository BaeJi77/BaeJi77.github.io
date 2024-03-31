---
title: "golang with common mistake (2)"
description: "4장 제어 구문, 5장 스트링에 대한 요약 및 추가적인 설명"

categories: 
  - dev
tags:
  - dev
  - golang
---

# 이전 글

- [2장 코드와 프로젝트 구성, 3장 데이터 타입](https://baeji77.github.io/dev/golang-with-common-mistake-1/)

# 개요

해당 내용은 [100 Go Mistakes Go 100가지 실수 패턴과 솔루션](https://product.kyobobook.co.kr/detail/S000211704725) 라는 책에 있는 내용 중 직접 많이 경험하고 도움이 되었던 내용들을 작성하는 것입니다.

더 다양하고 실제 책에 내용이 궁금하다면 해당 책을 읽어보시는 것을 추천드립니다.

# 100 go mistakes

## 4장. 제어 구문

### #30 range 루프에서 원소가 복제되는 특성을 정확하게 이해하라

`range`라는 키워드는 for문에서 반복을 편하게 해줍니다. 사용할 수 있는 데이터 구조에는 string, array, array pointer, slice, map, receving channel이 있습니다.

```go
s := []string{"a", "b", "c"}

for i := range s {
  // i는 index. index만 사용하는 경우. 
}

for i, v := range s {
  // i는 index. v는 실제 값.
}

for _, v := range s {
  // v는 실제 값. v만 사용하는 경우.
}
```

위에 있는 예시처럼 index와 값을 간단하게 사용할 수 있는 반복문을 만들 수 있습니다.

여기서 조심해야되는 것은 range가 동작할 때 v값은 언제나 `복제본`이다. 그러기 때문에 struct의 값에 대해서 동작을 할때 문제가 생길 수 있습니다.

```go
type nums struct {
  num int
}

for _, n := nums {
  n += 1000
}
```

위와 같은 실제로 값을 넣었을때 업데이트가 안됩니다. 그러기 때문에 명시적으로 인덱스를 선택해서 해당 값을 업데이트해야 됩니다. 밑에는 위와 같은 문제를 해결하는 방법입니다.

```go
// solution 1
for i, n := nums {
  nums[i].n += 1000
}

// solution 2
for i := 0, i < len(nums), i++{
  nums[i].n += 1000
}

// solution 3
nums가 모두 포인터로 만들어진 경우에는 그냥 값을 업데이트해도 포인트이기 때문에 실제로 값이 주입됩니다.
```

### #31 range 루프에서 인수를 평가하는 방법을 정확하게 이해하라

range는 인수 평가를 딱 선언한 이후 그것을 복제하여서 사용합니다. 한마디로 중간에 range에 들어간 값을 업데이트하더라도 실제로 변경이 일어나지 않습니다.

```go
s := []int{1, 2, 3}
for range s {
  s = append(s, 10)
}
---

s := []int{1, 2, 3}
for i := 0 ; i < len(s), i++ {
  s = append(s, 10)
}
```

첫번째와 두번째 차이는 첫번째는 끝난다는 것이고 두번째는 끝나지 않습니다. 왜냐하면 두번쨰는 `i < len(s)`가 매번 루프를 돌때마다 평가하기 때문에 끝나지 않습니다. 

하지만 첫번째는 처음 실행할때 s를 복제하여서 어떤 처리를 해도, 값을 추가해도, 혹은 그 안에 있는 값을 변경해도 이전에 있던 것으로 처리됩니다. 당연히 끝나고 나서 모두 체크해보면 정상적으로 append나 값이 변경됩니다.

### #33 맵 반복 과정에서의 잘못된 가정

#### 순서

map은 값이 들어갈때 순서가 보장되지 않는다. 이전 post에서 이야기했던 듯이 map은 해쉬테이블이 있고 그 테이블에 array이 있는 체인 구조를 가지고 있다. 그러기 때문에 출력을 했을때 절대로 값을 추가한 순서대로 값이 나오지 않습니다. 

명시적으로는 무작위(random)이 아니라 정해져있지 않다(sunspecified)라고 golang 언어공식 규격에 대해서 말한다고 합니다.

#### 반복 과정에서 맵 업데이트

고언어 규격에서는 다음과 같은 규격이 있습니다.

> 루프를 도는 동안 맵 항목에 생성됐다면 반복과정에서 추가될 수도 있고, 건너뛸 수도 있다. 구체적인 결과는 생성되는 항목마다 또는 반복 회차에 따라 달라질 수 있다.

해당 문제를 해결하기 위해서는 read map와 write map을 구분하는 것입니다.

```go
m := map[int]bool {0: true, 1: false, 2: true}

m2 := copyMap(m)

for k, v := range m {
  m2[k] = v
  if v {
    m2[10 + k] = true
  }
}

fmt.Println(m2)
```

원래 `m` 하나를 이용해서 처리했을 경우 실제 결과값이 달랐는데 위와 같이 처리하면 항상 똑같은 결과가 나옵니다.

### #34 break문 작동 방식을 정확하게 이해하라

`break`는 특정 statment를 끝내는 키워드입니다. for, selete, switch와 같은 곳에 사용됩니다. 그런데 golang에서는 생각보다 해당 키워드를 값이 사용합니다. 그러기 때문에 명시적으로 break할 구분을 지정해야 될수도 있습니다.

```go
for i := 0; i < 5 ; i++ {
  switch i {
    default:
    case 2:
      break
  }
}
---

loop:
  for i := 0; i < 5 ; i++ {
    switch i {
      default:
      case 2:
        break loop
    }
  }
```

위에 코드를 보면 첫번째 같은 경우 break를 해도 for를 끝나지 않습니다. 왜냐하면 switch에 break로 판단합니다. 그래서 두번쨰와 같이 명시적으로 `loop`것을 끝내겠다고 해야됩니다. 

`go-to`문과 비슷하지만 해당 방법은 golang에서는 표준 라이브러리에서도 사용하고 있습니다.

## 5장. 스트링

### #36 룬(rune) 개념을 정확하게 이해하라

- 문자 집합: 문자로 구성된 집합. 유니코드 차셋이 있다.
- 인코딩: 문자를 바이너리로 변환하는 것. UTF-8은 모든 유니코드 문자를 1~4바이트 숫자로 표현하는 인코딩 표준이다.

간단하게 코딩으로 `"지훈"`라는 글자를 작성할 수 있습니다. 하지만 해당 문자를 컴퓨터는 사실 `"지훈"`으로 받아들일 수 없습니다. 심지어 알파벳도 `abc`로 그대로 받아들이지 못합니다. 해당 문자를 컴퓨터가 받아들일 수 있도록 바이터리 바꿔야 됩니다.

한마디로 예를들면 `a`는 `0x00` 이라고 명시적으로 받아들일 수 있도록 약속을 하는 것입니다. 그리고 그런 과정에서 인코딩 과정이 일어나서 컴퓨터가 읽을 수 있도록 되는 것입니다.

`rune`은 go언어에서 유니코드 코드 포인트입니다. 다시 말해서 유니코드에서 어떤 값을 가지는지 표시한다고 할 수 있습니다.

그래서 `rune`은 1-4바이트, 즉 최대 32비트까지 표현할 수 있습니다. 밑에는 실제 코드입니다.

```go
type rune = int32
```

한국 개발자분들 대부분은 아니지만 많은 경우 경험해봤을 수 있습니다. 한글은 기본적으로 1바이트로 표현되지 않습니다. 대부분 여러 바이트를 이용해서 만들어집니다. 

```go
test := "지"
fmt.Println(len(test))
---

3
```

위에 있는 코드를 보더라도 `지`라는 글자는 3개의 rune을 사용해서 만든 것입니다. 

명시적으로 알아야될 것은 golang은 단순히 모든 string이 하나의 바이트를 사용하는 것이 아니고 rune 이라는 개념을 사용합니다. 그리고 string 안에 있는 모두 하나하나가 따로따로 캐릭터로 표현되어야 합니다. 

밑에 있는 케이스와 같이 len을 했을때 각각 하나 바이트이면 바이트가 보여서 결국 하나의 캐릭터가 되기도 합니다.

```go
s := "hello" // => len 5
test := "지" // => len 3
```

### #37 부정확한 스트링 반복 관련 실수

```go
s := "헬low"
for i := range s {
  fmt.Printf("position %d, %c\n", i, s[i])
}

fmt.Printf("len=%d\n", len(s))
---
// 출력
position 0, í
position 3, l
position 4, o
position 5, w
len=6
```

위에서 이상한 점은 3가지 입니다. 

1. 첫번째 글자는 `í`이 아니라 `헬` 이라는 것. 
2. 우리가 보기에는 명확하게는 rune은 4라는 것. 
3. index가 갑자기 jump한 것.

`"헬"`은 이전에 말했듯이 1바이트로 구성된 것이 아닙니다. 현재 상황을 보면 3바이트를 사용합니다. 하지만 직접 `s[0]`을 했을 경우 여러개로 `"헬"`을 나타는 3바이트 중에 첫번째 바이트를 나타내었다고 판단할 수 있습니다.

위와 같은 문제를 처리하기 위한 2가지 방법에 대해서 소개하겠습니다.

```go
// range 값 직접 접근
s := "헬low"
for i, v := range s {
  fmt.Printf("position %d, %c\n", i, v)
}
---
position 0, 헬
position 3, l
position 4, o
position 5, w
```

```go
s := "헬low"
runes := []rune(s)
for i, v := range runes {
  fmt.Printf("position %d, %c\n", i, v)
}
---
position 0, 헬
position 3, l
position 4, o
position 5, w
```

### #39 최적화가 덜 된 스트링을 결합하지 마라

golang의 스트링의 핵심 속성 중에는 `불변성`이라는 것이 있습니다. 스트링을 만들때 마다 새로운 스트링을 할당하는 것입니다. 우리가 기대하는 것은 `s += "a"`를 생각하면 s에 계속 a 붙을 것 같다고 생각하지만 새로운 string이 생성된다고 생각하면 됩니다.

이런 상황에서는 `strings.Builder{}`를 사용하면 굉장히 좋은 성능을 발휘할 수 있습니다. 

```go
func concat(values []string) string {
  s := ""
  for _, value := range values {
    s += value
  }
  return s
}

func concat(values []string) string {
  total := 0
  for i := 0 ; i < len(values) ; i++ {
    total += len(values[i])
  }

  sb := strings.Bulder{}
  sb.Grow(total)
  for _, value := range values {
    _, _ = sb.WriteString(value)
  }

  return sb.String()
}
```

첫번째는 최적화를 하지 않은 상황입니다. 지속적으로 새로운 string을 만들고 있습니다. 두번째는 `strings.Builder`와 조금 더 최적화를 했습니다.

`strings.Builder` 내부적으로 바이트 slice를 가지고 있습니다. 그러기 때문에 동시에 append를 할때 경쟁 상태가 발생할 수 있습니다. 그리고 slice는 특정 크기가 되면 자동으로 slice를 키우는 작업을 진행합니다. 그러기 때문에 위에 코드에서는 크기를 계산해서 `Grow` 메소드를 통해서 slice를 키우는 작업을 줄여서 `append` 했습니다.

책에서 나오는 내용으로 보면 첫번째보다 두번째가 성능적으로 99% 정도 빠르다고 합니다. 단순히 `strigns.Builder`를 하는 코드보다는 78% 빠르다고 합니다.

# 마무리

이번에는 제어구문과 스트링에 대해서 알아봤습니다. 제어 구문은 우리가 매우 많이 사용하기 때문에 더 실수가 많이 발생하는데 해당 내용을 통해서 조금 더 조심해야될 것들이 생긴 것같고 스트링 같은 경우 다른 언어에서는 다르게 표현할 수 있지만 `rune`이라는 개념에 대해서 알 수 있어서 매우 좋았던 것 같습니다.
