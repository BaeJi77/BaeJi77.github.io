---
title: golang에서 error처리를 어떻게 해야지 잘했다고 소문이 날까?
description: golang에서 처리하는 방법.
categories:
  - dev
  - golang
tags:
  - dev
  - golang
  - error
---

해당 블로그에서 사용된 코드는 [해당 링크](https://github.com/BaeJi77/blog-code/tree/main/2025-03/golang-error)에서 보실수 있습니다.

# 개요

golang을 사용하다보면 error을 어떻게 사용해야되는지에 대해서 많은 고민이 있습니다.

golang 개발을 해보면서 생각나는 여러가지 상황과 그 상황에서 어떻게 처리하는게 저에게는 좋았는지에 대해서 이야기해보겠습니다.

# error를 만드는 방법

golang에서는 error라는 타입이 존재합니다. 그 타입을 golang 자체적으로 존재하는 것이기 때문에 어떤 것으로 처리할수 없지만 기본적으로 만드는 방법에 대해서 이야기해보겠습니다.

1. fmt 혹은 error를 통해서 만들수 있다.

```Go
err1 := errors.New("new error from errors")
err2 := fmt.Errorf("new error from fmt")
```

fmt은 모두 알고 있듯이 입출력을 담당하는 라이브러리이고 errors은 golang에서 공식하는 error 관련 라이브러리입니다. 

두 방법으로 아주 간단하게 만들수 있습니다.

2. custom error은 Error()라는 메소드를 구현하면 된다.

```Go
func main() {
	err3 := &NewError{}
	var err4 error

	err4 = err3
}

type NewError struct{}

func (e *NewError) Error() string {
	return "new error from errors"
}
``` 

Error()라는 메소드를 구현만 하게 되면 error type이 될수 있습니다. 만약 custom하게 error을 관리하고 싶다면 이 방법을 사용할 수 있습니다.

# error을 처리하는 방법

해당 내용은 반드시 이렇게 해야된다는 지침이라기보다 기호의 차이가 존재할 수 있음을 알려드립니다.

1. early return이 편하다.

early return 이란 어떤 에러상황이나 특정 조건이 만족되지 않아서 다음 코드 실행이 의미가 없는 경우 빠르게 return를 하는 code 스타일이라고 할 수 있습니다.

```Go
func validate(param1 string, param2 int) (bool, error) {
	if len(strings.TrimSpace(param1)) == 0 {
		return false, fmt.Errorf("param1 is empty")
	}

	if param2 == 0 {
		return false, fmt.Errorf("param2 is zero")
	}

	return true, nil
}
```

위에 있는 코드는 특정 상황에 대한 조건을 체크하는 코드를 예시를 만들었습니다. 해당 코드 스타일로 코드를 짜게되면 가독성과 실제 동작 코드에 대해서 빠르게 파악 할 수 있습니다.

Java나 Kotlin 같은 경우 error라는 개념을 언어차원에서 지원해주지 않고 multi return을 지원해주지 않기 때문에 error처리가 까다로운 편인데 golang은 모두 지원해주기 때문에 조금 쉽게 코드를 짤 수가 있습니다.

2. 여러가지 error을 한번에 반환할수 있다.

error 값을 한번에 모아서 반환하는 케이스가 있습니다. 특히 여러 시도를 했지만 모두 실패한 경우 모든 실패 케이스에 대한 error값을 반환해야됩니다.

```Go
func validateWithMultiErrors(param1 string, param2 int) (bool, error) {
	var errs error
	if len(strings.TrimSpace(param1)) == 0 {
		errs = errors.Join(errs, fmt.Errorf("param1 is empty"))
	}

	if param2 == 0 {
		errs = errors.Join(errs, fmt.Errorf("param2 is zero"))
	}

	return true, nil
}


func executeMultipleTry() (bool, error) {
	var errs error
	if err := firstExecute(); err != nil {
		errs = errors.Join(errs, err)
	}

	if err := secondExecute(); err != nil {
		errs = errors.Join(errs, err)
	}

	if errs != nil {
		return false, errs
	}

	return true, nil
}

func firstExecute() error {
	return errors.New("first execute error")
}

func secondExecute() error {
	return errors.New("second execute error")
}
```

errors.Join() (go 1.20부터 지원) 이라는 메소드를 이용하면 여러 error들을 한 error로 합칠수가 있습니다.

3. error는 지속적으로 wrapping 될 수 있다.

여러 method가 중첩으로 호출되는 경우에서 error을 반환했을때 erorr가 wrapping되어서 높은 위치까지 전달될 수 있습니다.

```Go
func main() {
	wrapErrorTestErrors := firstError()
	fmt.Println(wrapErrorTestErrors) // second execute error: error on secondError(). err=wrap error
}

var wrapError = &WrapError{}

type WrapError struct{}

func (w *WrapError) Error() string {
	return "wrap error"
}

func firstError() error {
	if err := secondError(); err != nil {
		return fmt.Errorf("second execute error: %w", err)
	}

	return nil
}

func secondError() error {
	return fmt.Errorf("error on secondError(). err=%w", &WrapError{})
}
```

wrapping을 하기 위해서는 `fmt.Errorf(".... %w")` `%w` 라는 키워드를 통해서 wrapping할 수 있습니다. `%v`나 `%s`를 사용할수는 있으나 그렇게 하는 경우 errors의 추가기능을 사용하지 못합니다. (밑에서 설명예정)

보시는 것과 같이 `error on secondError(). err=%w` 에서 `WrapError`를 붙였습니다. 그러니까 `error on secondError(). err=wrap error`이 되었습니다.

그리고 마지막에 해당 error을 반환할때 wrapping을 하니 마지막에서는 ``second execute error: error on secondError(). err=wrap error``이 되었습니다.

4. error를 비교하고 싶을때 errors.Is() 혹은 errors.As()를 사용하면 된다.

error를 비교하는 방법은 여러가지일수 있지만 가장 대표적인 방법은 `errors.Is()`를 통해서 error를 가지고 있는지 판단하는 것과 `errors.As()`를 통해서 특정 error type을 가지고 있는지 판단하는 방법이 있습니다.

3번에서 말했듯이 error은 wrapping될수 있습니다. 그러기 때문에 errors.Is()와 errors.As()도 그냥 단순히 방금 반환된 error에 대해서만 판단하는게 아니라 wrapping된 모든 error들에 대해서 판단하게 됩니다.

```Go
for true {
    err := errors.Unwrap(wrapErrorTestErrors)
    if err == nil {
        break
    }
    fmt.Println(err.Error())

    wrapErrorTestErrors = err
}
===
error on secondError(). err=wrap error
wrap error
```

```Go
func main() {
	wrapErrorTestErrors := firstError()
	fmt.Println(wrapErrorTestErrors)
	if errors.As(wrapErrorTestErrors, &wrapError) {
		fmt.Printf("wrapErrorTestErrors have wrapError type. wrapErrorTestErrors=%v\n", wrapErrorTestErrors)
	}

	if errors.Is(wrapErrorTestErrors, wrapError) {
		fmt.Printf("wrapErrorTestErrors have wrapError error. wrapErrorTestErrors=%v\n", wrapErrorTestErrors)
	}
}
=======
second execute error: error on secondError(). err=wrap error
wrapErrorTestErrors have wrapError type. wrapErrorTestErrors=second execute error: error on secondError(). err=wrap error
wrapErrorTestErrors have wrapError error. wrapErrorTestErrors=second execute error: error on secondError(). err=wrap error
```

바로 위에 있는 실험처럼 가장 밑에 있는 wrapError라는 값에 대해서 모두 찾은 것을 확인할 수 있습니다.

# 여러가지 상황

## panic은 어떻게 하나요?

꼭 정답은 없지만 저 같은 경우 panic은 recover를 통해서 catch할 수 있습니다. 하지만 상황에 따라서 catch에서 에러만 남기기도하고 어떤 상황에서는 로그만 남기고 프로그램을 종료할수도 있을것 같습니다.

상황에 따라 다르지만 일단 panic은 잡아야됩니다. 그리고 많은 경우 recover() 메소드를 defer를 통해서 잡도록 하는게 가장 쉽게 처리하는 방법이라고 생각합니다.

```Go
func main() {
	defer recoverMethod()
	panicMethod()
}

func panicMethod() error {
	panic("panic occurred!!!!!!!!!!!!!!!")
}

func recoverMethod() {
	v := recover()
	fmt.Println(v)
}
====
panic occurred!!!!!!!!!!!!!!!
```

# 마무리

이번에는 golang에서 에러처리를 어떻게하는지에 대해서 제가 가지고 있는 간단한 지식을 이야기해봤습니다.

error은 어찌보면 정말 편한것 같지만 관리를 잘하지 못했을 경우 매우 귀찮은 요소가 될수도 있습니다. 이번 기회에 여러 지식을 알고 도움이 되었으면 좋겠습니다.

감사합니다.
