---
title: "golang with common mistake (3)"
description: "6장 함수와 메소드, 7장 에러 관리에 대한 요약 및 추가적인 설명"

categories: 
  - dev
tags:
  - dev
  - golang
---

# 이전 글

- [2장 코드와 프로젝트 구성, 3장 데이터 타입](https://baeji77.github.io/dev/golang-with-common-mistake-1/)
- [4장 제어 구문, 5장 스트링](https://baeji77.github.io/dev/golang-with-common-mistake-2/)

# 개요

해당 내용은 [100 Go Mistakes Go 100가지 실수 패턴과 솔루션](https://product.kyobobook.co.kr/detail/S000211704725) 라는 책에 있는 내용 중 직접 많이 경험하고 도움이 되었던 내용들을 작성하는 것입니다.

더 다양하고 실제 책에 내용이 궁금하다면 해당 책을 읽어보시는 것을 추천드립니다.

# 100 go mistakes

## 6장. 함수와 메소드

### #42 적절한 리시버 타입을 결정하라

메소드를 선언할때 리시버 타입을 선언해서 사용해야 됩니다. 그냥 값 리시버를 사용할지, 아니면 포인터 타입을 사용할지. 해당 글에서는 명확하게 케이스를 분리해서 이야기해주었습니다.

- * 리시버가 반드시 포인터여야 하는 경우

• 메서드에서 리시버의 값을 변경해야 할 경우: 리시버가 슬라이스고, 메서드에서 원소를 추 가하는 경우도 마찬가지로 적용된다.

```
type slice []int

func (s *slice) add(element int) {
  *s = append (*s, element)
}
```

• 메서드 리시버에 복제할 수 없는 필드가 있는 경우: 예를 들어, sync 패키지의 타입을 들 수 있다(#74 sync 타입을 복제하지 마라"에서 자세히 설명한다).


- * 포인터 리시버가 바람직한 경우

• 리시버가 큰 오브젝트인 경우: 이럴 때는 포인터를 사용하면 복제 작업으로 인한 성능 저하 를 피할 수 있어서 호출 효율을 높일 수 있다. 크기의 기준은 벤치마킹을 통해 알아낼 수 있 다. 수많은 요인에 영향을 받기 때문에 구체적으로 어느 정도여야 크다고 볼 수 있는지를 딱 잘라서 말할 수는 없다.

- * 리시버가 반드시 값이어야 하는 경우

• 리시버의 불변성을 보장해야 하는 경우

• 리시버가 맵이나 함수, 채널인 경우: 나머지 경우는 컴파일 에러가 발생한다.

- * 값 리시버가 바람직한 경우

• 값이 변하지 않아도 되는 슬라이스로 된 경우

• 리시버가 작은 배열이나 구조체라서 가변 필드(예 time.Time)가 없는 값 타입인 경우

• 리시버가 int, float64, string 같은 기본 타입인 경우

그리고 마지막으로 혼합해서 사용하는 경우가 있기는 합니다. 예로는 time.Time같은 경우 encoding과 관련해서 메소드를 구현할때 포인터가 들어가도록 만들어야되지만 그런 경우는 보통 흔하지 않으면 권장되지 않다고 말하고 있습니다.

### #43 기명 결과 매겨변수를 적절히 사용하라

기명 결과 매개변수를 먼저 말하자면 밑에 예제이 있는 `b` 입니다. 바로 반환 값에 대한 변수를 미리 선언하고 그것을 메소드 내에서 사용하는 것입니다.

```golang
func f(a int) (b int) {
  b = 5
  return
}
```

위에 코드를 보게 되면 b를 따로 선언하지 않았지만 b를 반환값에 변수로 판단하여서 바로 사용합니다. 그리고 또한 반환할 때 특별할 값을 적지도 않아도 괜찮습니다.

적절한 균형점을 찾아야됩니다. 만약 특정 메소드가 엄청나게 많은 반환값을 가지고 있으면 그것 모두 타입이 비슷하다면 기명결과 매겨변수를 사용하는 것이 코드의 가독성을 올리는데 도움이 될 수 있습니다.

하지만 많은 반환 결과를 가지고 있지 않거나 명확하게 타입으로 구분이 되는 경우에는 도움이 되기 보다는 오히려 코드를 해석하기 어렵게 만들 수 있습니다.


### #45 nil 리시버 리턴 관련 실수

### #47 defer 인수와 리시버 평가 방식을 무시하지 마라

`defer` 키워드는는 종료하기 전까지 기달렸다가 추후에 실행됩니다. 하지만 defer를 사용한다면 잘못된 동작을 만들 수도 있습니다.

```golang
func f() error {
  var ststus string
  defer notify(status)
  
  status = "hello"
  ...
}
```

nofity에는 언제나 empty string이 전달됩니다. 그 이유는 defer은 언제나 인수를 `즉시 평가`합니다. 해당 메소드가 종료되어서 반환될때 평가되는 것이 아닙니다. 그러기 때문에 defer를 사용할때 그때 그 변수값을 그대로 사용하는 것입니다. 추후에 값을 변경해도 달라지지 않습니다.

그것을 해결하기 위한 방법으로 포인터값 전달, 클로저 함수 사용입니다.

- 포인터값 전달

```golang
func f() error {
  var ststus string
  defer notify(&status)
  
  status = "hello"
  ...
}
```

이렇게하면 status의 주소를 전달하는 것입니다. 실제로 주소는 변경되지 않았고 status의 값은 바뀌었기 때문에 변경된 값을 전달할 수 있습니다.

- 함수 클로저 사용

```golang
func f() error {
  var ststus string
  defer func() {
    notify(status)
  } 
  
  status = "hello"
  ...
}
```

이렇게하면 defer가 실행되어야 될때 함수가 실행됩니다. 그렇다면 status가 제대로 갱신된 값을 전달하기 때문에 원하는대로 동작하는 것을 확인할 수 있습니다.

## 7장. 에러 관리

### #48 패닉 발생 관련 실수

`panic`은 golang에서 지원하는 문제 상황을 전파하기 위해서 만든 키워드입니다. panic를 호출하면 그것을 recover하기 전까지 콜스택을 계속 거슬러 올라갑니다.

```golang
func main() {
  fmt.Println("a")
  panic("foo")
  fmt.Println("b")
}

=====
a
panic: foo

goroutine 1 [running]:
main.main()
	/tmp/sandbox3578583717/prog.go:9 +0x5f
```

위에 예시처럼 `b`가 출력되기 전에 panic이 있었고 그것을 recover하지 않았기 때문에 프로그램이 종료가 되었습니다.

```golang
func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("recover: ", r)
		}
	}()

	f()
}

func f() {
	fmt.Println("a")
	panic("foo")
	fmt.Println("b")
}

===

a
recover:  foo
```

이번 예시는 recover를 통해서 프로그램 종료까지는 막았습니다. recover가 defer를 같이 이용하면 panic을 매우 유용하게 핸들링할 수 있습니다.

golang에서는 panin를 자주 사용하지 않습니다. 그렇다면 언제 panic을 사용해야 될까요?

1. 프로그래밍 에러를 알려줘야되는 경우
  - 프로그래밍적으로 실수를 했을때 그것에 대해서 바로 알수있도록 하기 위해서. ex) http code 범위 잘못 설정할 경우
2. 애플리케이션에 꼭 필요한 의존성을 만족하지 못하는 경우
  - db 드라이브 설정을 할때 정상적이지 않으면 panic을 발생


### #52 에러를 두 번 처리하지 마라

에러를 처리할때는 로그를 남기고 거기서 끝내던지 혹은 그것을 반환하는 것입니다. 두 개 모두 한다면 그것은 디버깅할때 어려움을 줄 수 있습니다.

만약 에러를 반환한다면 단순히 error만 반환하는게 아니고 에러에 대한 설명과 함께 에러를 wrap해서 반환하면 좋습니다.

```golang
func main() {
	f()
}

func f() {
	if err := erorrNum(1); err != nil {
		log.Printf("error errorA method. err=%v", err)
	}
}

func erorrNum(num int) error {
	err := fmt.Errorf("invalid number: %v", num)
	return fmt.Errorf("error for handle number: %w", err)
}
===

2009/11/10 23:00:00 error errorA method. err=error for handle number: invalid number: 1
```

단순히 에러만 반환하는게 아니고 에러를 wrap해서 반환하게 되면 `errors.As` 혹은 `errors.Is`로도 판단할 수 있고 error 상황에 대한 detail한 내용을 반환하고 log로 남기기 때문에 디버깅에 용이하니다.

### #54 defer에서 발생한 에러를 처리하라

golang에서 defer를 이용해서 특정 메소드를 실행하면 코드를 간결하게 짤수 있습니다. 하지만 만약 defer를 이용한 메소드가 error를 반환한다면 어떻게 처리해야될까요?

아무것도 하지 않는 것이 가장 좋지 않습니다. 왜냐하면 나중에 개발자(본임 포함)가 해당 코드를 읽었을때 error을 반환한다는 것을 알지 못할 확률이 매우 높습니다.

```golang
defer a.Close()
```

그렇다면 인자를 받지만 핸들링하지 않는것은 어떨까요? 당연히 이전보다 훨씬 괜찮습니다. 하지만 문제가 있습니다. 해당 에러를 그냥 무시해도 괜찮을까요?

```golang
defer func() { _ = a.Close()}()
```

만약 에러를 핸들링하고 싶다면 어떻게 하는 것이 좋을까요? 단순히 로그를 남기는 것도 괜찮지만 해당 에러를 실제 본 함수와 같이 사용하고 싶다면요?

```golang
func f() (err error) {
  con, err := db.connection(...)
  defer func() { 
    err = con.Close()
  }()

  err = con.request(...)
  ...
}
```

위와 같이 핸들링하게 되면 `con.Close()`에서 발생하는 에러를 핸들링할 수 있는지만 `con.request()`에서 발생하는 error는 없어지게 됩니다. 그러니 최종적으로 이렇게 하시는 걸 추천드립니다.

```golang
func f() (err error) {
  con, err := db.connection(...)
  defer func() { 
    closeErr = con.Close()
    if err != nil {
      if closeErr != nil {
        log.Printf("failed to close: %v", err)
      }
      return
    }
    err = closeErr
  }()

  err = con.request(...)
  ...
}
```

# 마무리

이번에는 `함수와 메소드`, 그리고 `에러 관리`에 대해서 알아봤습니다. 가장 크게 느꼈던 부분은 메소드 리시버 선택에 대한 내용과 에러 관리에 대한 전반적인 내용이었습니다. 다른 언어들은 대부분 메소드를 만들때 저런것들을 신경쓰지 않지만 golang을 고려해야되기 때문에 처음에 매우 곤란했는데 뭔가 딱 정해진 케이스들에 대해서 이야기해주니 너무 좋았습니다. 그리고 또 다른 언어와 다른 특징일 수 있는 글로벌하게 erorr를 관리하지 않고 위에서 계속 관리를 해줘야되는 경우가 많은 그런 다양한 케이스에 대해서 알 수 있어서 좋았습니다.

다음에는 또 찾아뵙겠습니다.
