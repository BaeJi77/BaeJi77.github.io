
====

- `goroutine`은 무엇인가?
-> 여기서 나오는 개념들에 대해서 가볍게 설명할 필요 있을듯.
쓰레드는 무엇인지, 동시성 (Concurrent computing) 이란 무엇인지

- 간단한 사용법
  - main goroutine
  - 키워드와 간단한 코드.

- `goroutine`의 장점.
  - `goroutine`의 장점들~~~
  - `goroutine`은 어떻게 가벼운가?
  - `goroutine`이 다른 쓰레드 개념과 무엇이 다른가.

- `goroutine`의 단점.
  - 코딩을 잘못했을 떄 `goroutine`이 무한으로 생길 수 있는 문제. (나의 경험담)


===================

추가적으로 작성할만한 내용. 스케쥴링에 대해서. MPG 모델.

`goroutine`을 사용할때 동시성을 어떻게 지킬 수 있을지. 통신은 어떻게 할 수 있는지.

`goroutine` 활용에 대해서는 두번째 글에서

그린 쓰레드: 
Goroutines are also often called green threads. Green threads are maintained and scheduled by the language runtime instead of the operating systems. The cost of memory consumption and context switching, of a goroutine is much lesser than an OS thread. So, it is not a problem for a Go program to maintain tens of thousands goroutines at the same time, as long as the system memory is sufficient.

Go doesn't support the creation of system threads in user code. So, using goroutines is the only way to do (program scope) concurrent programming in Go.

When the main goroutine exits, the whole program also exits, even if there are still some other goroutines which have not exited yet.


