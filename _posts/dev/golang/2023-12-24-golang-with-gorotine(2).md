---
title: "about golang - goroutine basic (2)"
description: "goroutine에 대해서 구체적으로 "

categories: 
  - dev
tags:
  - dev
  - golang
  - goroutine
---

해당 아티클에 존재하는 코드는 [여기](https://github.com/BaeJi77/blog-code/blob/main/2023-12/go-goroutine-2/without-m/main.go)에서 확인해보세요.

# 개요

[이전 글](https://baeji77.github.io/dev/golang-with-goroutine/)에는 간단한 `goroutine` 설명과 장단점, 그리고 사용법. 그리고 그 사용하면서 자주 문제가 되는 부분에 대한 주의사항과 해결책에 대해서 말해봤습니다.

이번에는 실제 `goroutine`이 어떤 형태로 되고 있고 어떻게 동작하는지에 대해서 구체적으로 설명해보겠습니다.

# `goroutine`과 `therad`

고루틴과 쓰레드. 어찌보면 프로그래밍 언어에서 동시성(병렬성x)을 제공하기 위해서 사용하는 개념들입니다. 하지만 비슷하면서 다른 부분들이 너무 많습니다.

## new와 free

리소스를 생성하고 릴리즈하는데 있어서 차이가 존재합니다. 고루틴같은 경우 실제 os에서 관리되는 리소스가 아니다보니 os에 영역에 접근할 필요가 없습니다. 그리고 os쓰레드에 비해서 훨씬 적은 사이즈로 만들 수 있습니다. 

많은 블로그 예시를 가지고 온다면 java thread 같은 경우 만드는 경우에 256kb~2mb 사이즈로 쓰레드를 만들 수 있습니다. 만약 가장 큰 2mb로 설정하여서 쓰레드를 만든다면 1GB 메모리에 512개 쓰레드밖에 만들지 못합니다. 그리고 java thread같은 경우 고정된 메모리 크기를 가집니다. 처음 할당하면 그 크기가 더 커지거나 더 작아질 수 없습니다. 그 의미는 해당 메모리보다 더 크게 사용하게 되면 `OutOfMemoryError`와 같은 에러를 만나게 되는 것이고 적게 사용하게 되면 페이징으로 사용하지 않지만 할당되어서 다른 쓰레드들이 사용하지 못하는 메모리 공간을 계속 차지고 있는 문제가 발생합니다.

그것과 반대로 고루틴 같은 경우 2kb 라는 작은 크기로 고루틴을 만들 수 있습니다. 2mb에 비하면 약 1000배 작은 공간으로 만들수 있으며 그 의미는 1000배 더 많이 많이 만들 수 있다는 것입니다. 그리고 2kb는 가장 작은 사이즈이면 동적으로 사이즈가 늘어날 수 있습니다. 그 의미는 페이징 이슈를 어느정도 해결하면서 `OutOfMemoryError` 어느정도 대처할 수 있습니다.

## Context switching cost

동작하던 쓰레드가 blocking이 된다면 해당 쓰레드는 동작하고 있던 정보를 (다음에 동작했을 경우를 대비해서) 저장하고 새로 동작할 쓰레드는 자신의 정보를 CPU에 입력(로드)합니다. 이 과정에서 많은 데이터가 저장되고 로드됩니다. 

하지만 고루틴 같은 경우 실질적으로 os에서 동작하는 context switching이 아닌 user level에서 context switching이 일어나기 때문에 훨씬 더 적은 데이터를 저장하고 로드하게 됩니다. 실제로 적은 데이터를 관리하고 그것과 함께 os 영역과 연관성이 줄어들기에 일단적인 thread에 비해서 빠르게 context switching을 할 수 있습니다.

## os thread와 mapping 방식

UNIX like 운영체제에서 user level thead는 kernel level 쓰레드와 매핑되어야지 실제 cpu에서 할당받고 동작할 수 있습니다. 여기서 3가지 방식이 존재합니다.

- 1:1 model: kernel thread 1:1 user thread
  - 멀티 테스킹시에 이점이 있습니다. (user thread가 모두 kernel thread와 연결되었기 때문에 동등하게 커널 자원을 받을수 있음)
  - 동작을 기달리는 쓰레드를 탐색해야되는 시간이 필요합니다.

- 1:N model: kernel thread 1:N user thread
  - kernel thread에 연결된 여러 user thread를 선택해서 바로 실행가능.
  - 특정 user thread가 동작할 때 다른 thread들이 동작할 수 없습니다. (동작하는 쓰레드가 blocking된 경우 kernel 쓰레드가 blocking되고 그 kernel thread와 연결된 다른 user thread까지 아무것도 못함.)

- N:M model: kernel thread N:M user thread
  - 최대한 많은 user thread가 동작할 수 있으며 빠르게 다른 user thread를 선택할 수 있습니다.
  - 구현이 복잡합니다.

Java 같은 경우 1:1 model을 사용하고 있으며 golang같은 경우 N:M 방식으로 고루틴을 만들어서 동작시키고 있습니다. golang은 언어적 차원에서 N:M thread model을 구현했습니다.

위에 이야기한 것처럼 우리가 많이 기존에 사용하고 있던 프로그래밍 언어(native thread를 사용하는 프로그래밍 언어)에서 가지고 있는 쓰레드 문제점들이 있었습니다. 그 문제점들을 golang에서는 고루틴을 활용하여 해결했습니다.

# goroutine details

고루틴이 실제로 내부적으로 어떻게 동작하는지 알아보겠습니다.

고루틴은 내부적으로 스케줄러가 동작시킵니다. 그 핵심으로 [G-M-P model](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit)을 활용합니다.

## G(goroutine)

스택, 명령 포인터 및 필요한 다른 상태를 포함하는 고루틴을 나타냅니다.

## M(machine)

코드를 실행할 수 있는 OS 스레드를 나타냅니다. M은 Go 코드를 실행하기 위해 P와 연결되어 있어야 하지만, P 없이 차단되거나 시스템 호출 중일 수 있습니다.

## P(Processor)

P는 스케줄링 컨텍스트 또는 "프로세서"를 나타냅니다. P는 고루틴을 실행하기 위한 자원을 관리하며, 고루틴의 LRQ(로컬 실행 큐)를 유지합니다. P의 수는 `GOMAXPROCS` 환경 변수로 설정되며, 일반적으로 사용 가능한 논리 CPU 코어의 수와 같습니다.

P라는 개념은 딱 와닿는 개념이 아니기 때문에 추가적으로 설명한다면,

- 고루틴 실행 큐 관리: P는 실행을 기다리고 있는 고루틴들의 로컬 큐를 관리합니다. 이 큐에는 아직 M에 의해 실행되지 않은 고루틴들이 들어 있습니다.
- 고루틴 스케줄링: P는 자신의 로컬 큐에 있는 고루틴을 M에 할당하여 실행합니다. P는 또한 다른 P의 큐에서 고루틴을 훔쳐오는 작업 훔치기(work stealing)를 통해 균형을 맞춥니다.
- 자원 할당: P는 고루틴이 실행되기 위해 필요한 자원(예: 메모리 할당)을 관리합니다.

## G-M-P와 scheduling

### 용어

- LRQ(Local Run Queue)
  - P에 내부적으로 존재하는 실행해야될 goroutine을 저장하는 로컬 실행 큐입니다. P와 연결된 M은 해당 LRQ에 있는 goroutine을 가져와 실행합니다.
- GRQ(Global Run Queue)
  - 이름대로 P에 내부에 있는 LRQ에 속하지 않는 실행 큐입니다. 소속되지 않은 이유는 다양하지만 그 예로는 실행 중에 blocking이 된 이후에 다시 실행될 수 있는 상태가 되었을 때 LRQ로 돌아가지 않고 GRQ로 들어가게 됩니다. P는 자신의 LRQ가 비게되면 GRQ에 있는 goroutine을 자신의 LRQ에 넣습니다.

### G-M-P와 LRQ
![goroutine gmp 1](/assets/images/2023-12-24-golang-with-goroutine/gmp_1.png)

위에 그림처럼 P는 자기에게 연결된 LRQ를 가지고 있습니다. 그리고 P는 M와 연결되어서 자신이 가지고 있는 G(goroutine)을 할당시키고 실행시킵니다. 

### G-M-P와 GRQ

![goroutine gmp 2-1](/assets/images/2023-12-24-golang-with-goroutine/gmp_2-1.png)

![goroutine gmp 2-2](/assets/images/2023-12-24-golang-with-goroutine/gmp_2-2.png)

P는 자신의 LRQ에 확인하고 비어있을 경우에는 GRQ를 확인해서 큐에 goroutine이 있는 경우 자신의 LRQ에 넣습니다.

추가적인 내용으로 `Work Stealing`이라는 개념이 있습니다. 만약 GRQ에서 G가 없다면 다른 P가 가지고 있는 LRQ에서 G를 가져(훔쳐)옵니다. 해당 내용은 [링크](https://rakyll.org/scheduler/)에 있는 글을 추천드립니다.

### GMP와 blocking goroutine

![goroutine gmp 3-1](/assets/images/2023-12-24-golang-with-goroutine/gmp_3-1.png)

동작하고 G(goroutine)이 blocking operation을 해야되었을 경우 M과 함께 blocking이 되는 경우가 있습니다. 그랬을 경우 M은 연결된 P와는 따로 떨어져서 M와 P는 동시에 같이 동작하게 됩니다. 

![goroutine gmp 3-2](/assets/images/2023-12-24-golang-with-goroutine/gmp_3-2.png)

하지만 G(goroutine)같은 M과 함께 blocking되기도 하지만 혼자 blocking이 되는 경우가 있습니다. 한마디로 M가 같이가 아닌 혼자 blocking이 되기 때문에 M은 다른 goroutine을 처리할 수 있습니다. 이 경우에 대해서는 밑에서 더 자세하게 설명드리겠습니다.

### 만약 P가 없다면

여러 글을 읽으면서 근본적으로 P는 왜 존재하는 것인가라는 생각을 많이 하게 되었습니다. 그러면서 제가 내린 결론을 한번 내려봤습니다. 해당 내용은 저의 뇌피셜이 들어간 내용이니 검증이 필요합니다. 일단 P는 N:M 쓰레드 모델을 활용하기 위한 방법이락 생각했습니다. 만약 P가 없다면 M은 여러 G와 연결되어야했을 것입니다. 그렇게 된다면 1:N 쓰레드 모델이 되는 것입니다. 하나의 M이 blocking이 되면 그것과 연결된 G는 모두 blocking이 되고 동작하지 못했을 것입니다. 이렇게 되면 많은 G를 만들더라도 동시성을 최대한으로 올리지 못했을 것이라고 생각합니다. 한마디로 최대한 M과 G를 활용할 수 있는 도구라는 결론을 내렸습니다.

그리고 추가적으로 여러 글을 찾아본 봐로는 locality를 지원하기 위해서 P가 존재한다는 이야기가 있는 것 같습니다.

## 동기식과 비동기식 system call

동기식 system call이 호출하게 되면 결국 G와 M은 동시에 같이 blocking될 수 밖에 없습니다. 하지만 그것을 계속 기달리게 된다면 해당 P에서는 지속적으로 실행해야될 G들이 존재합니다. 그렇기 때문에 해당 M-G set을 P와 연결을 끊고 다른 M와 자신이 가지고 있는 G와 연결해서 계속 동작시킵니다. (새로운 M(커널 쓰레드)를 만드는지 아니면 연결된 M과 진행하는지는 확실하지 않습니다.)

하지만 비동기식 system call이 호출되면 G는 M가 떨어져서 혼자 blocking이 됩니다. 명확하게는 비동기식으로 동작할 수 있는 system call인 경우입니다. IO 멀티플렉싱이나 네트워크 IO와 같은 비동기적으로 처리할 수 있는 방법들이 존재합니다. golang같은 경우 standand library를 사용하는 경우 자동으로 비동기적 system call을 실행합니다. 그러기 때문에 M(커널 쓰레드)를 만들지 않고 G 혼자 blokcing하게 됩니다.

해당 내용과 관련해서 [아티클](https://ykarma1996.tistory.com/188)에 마지막 부분에 있는 예시를 보시면 더욱 도움이 될 것 같습니다.

### 예시

```go
func goroutineSleep(wg *sync.WaitGroup) {
	defer wg.Done()
	time.Sleep(100 * time.Second)
}

func main() {
	wg := &sync.WaitGroup{}
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go goroutineSleep(wg)
	}

	time.Sleep(time.Second)

	fmt.Printf("goroutines: %d, get M number %d\n", runtime.NumGoroutine(), runtime.GOMAXPROCS(0))

	wg.Wait()
}
---

# 실행 결과
goroutines: 1001, get M number 10

# 정확한 검사(GODEBUG=schedtrace를 이용한 정보)
SCHED 1008ms: gomaxprocs=10 idleprocs=10 threads=9 spinningthreads=0 needspinning=0 idlethreads=7 runqueue=0 [0 0 0 0 0 0 0 0 0 0]
```

### 비동기를 지원하는 상황

- System I/O(File I/O, Network I/O, etc.)
- `time.Sleep()`
- channel waiting
- ...

# 마무리

이번에는 golang에 goroutine이 어떻게 동작하는지 구체적으로 알아봤습니다. 아직도 정확하게 제가 잘 알고 있다는 생각은 안합니다. 글을 읽다보니 goroutine이 주기적으로 GRQ로 LRQ에 있는 정보를 보내는 것부터해서 GC와 함께 동작하기 위해서 10ms만 최대한 실행한다는 내용들도 들어있었지만 구체적인 글을 찾기 어려워서 더 알아내는 것을 포기했습니다.

그럼에도 이번에 블로그에 정리하면서 제가 매일 사용하고 goroutine에 대해서 잘 알 수 있게 되었습니다. 

# ref
- https://tech.ssut.me/goroutine-vs-threads/
- https://stonzeteam.github.io/How-Goroutines-Work
- https://rakyll.org/scheduler/
- https://medium.com/a-journey-with-go/go-goroutine-os-thread-and-cpu-management-2f5a5eaf518a
- https://medium.com/curg/%EA%B3%A0%EB%A3%A8%ED%8B%B4-go-%EC%96%B8%EC%96%B4%EC%9D%98-%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AA%A8%EB%8D%B8-1045986cc001
- https://ssup2.github.io/theory_analysis/Golang_Goroutine_Scheduling/
- https://ykarma1996.tistory.com/188
- https://www.mimul.com/blog/go-vs-java-thread/
