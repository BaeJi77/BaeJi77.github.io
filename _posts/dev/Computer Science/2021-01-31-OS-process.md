---
title: "Operating System 기본 개념 - 프로세스"

categories:
  - spring
  - dev
tags:
  - dev
  - spring
  - java
  - gradle
---

# 프로세스 + 스케줄링 + 동기화

### System call 실행과정

User mode에서 해당 system call을 호출한 경우 그것은 라이브러리에서 선언된 system call를 호출한 것이다. system call interface는 해당하는 system call의 넘버를 kernel에 넘긴다. Kerner에서는 system call table에서 그 number에 해당하는 system call 실행하고 그것을 system interface를 통해서 user mode로 결과를 가져온다. (인자가 적은 경우 레지스터에 저장에서 참조, 아닌 경우 memory에 block이나 table에 저장하여 그 block에 주소를 넘겨준다.)

### Program vs Process

Program는 code와 data로 된 실행파일이며, 그 파일을 실행했을 경우 메모리에 올라온 경우 Process가 되면 구성으로는 힙과 스택, data와 code 그리고 PCB로 구성된다. PCB는 Process의 모든 정보있는 자료구조이다. (PC,stack pointer,열려있는 file 등등).

### Context switch

CPU가 현재 프로세스를 멈추고 다른 프로세스를 실행시키는 것. 현재 정보를 해당 PCB에 저장하고 다른 프로세스의 PCB를 읽어서 CPU에 다른 프로세스를 올린다. 오버헤드가 매우 크다.

### Fork , exec, exit, wait

Fork 현재 프로세스를 완전히 똑같이 만든 자식을 만드는 것. Exec 현재 프로세스를 완전히 다른프로그램으로 바꾸는 것. Exit 자식이 자신의 일을 끝내고 부모에게 인자를 전달해줄때 사용, wait 부모는 자식이 끝날때까지 기달리는 것.

### Process간 communication (IPC)

첫번째로 Message box, 두번째로 shared memory. MB같은 경우 link(message box)를 만들어서 a가 link에 값을 넣으면 b는 그것을 사용.

Shared memory같은 경우 두 프로세스 사이에 공유 메모리 영역을 만들어서 그 영역을 두 프로세스가 접근 가능하게 만듬.

Message box는 많은 시스템콜이 호출되고 적은 데이터 전송일때 유용하고 shared memory는 프로그래밍이 어렵지만 많은 데이터를 서로 communication할때 유용하게 사용. 그리고 shared memory를 할 경우 동기화를 해주어야 한다.

### Process vs thread

Process는 운영체제로부터 자원을 할당받는 작업의 단위이고 thread는 process가 할당받은 자원을 이용하는 실행의 단위이다. 각 Process는 별도에 주소 공간과 자원을 가지고 있지만 각 thread는 process안에서 스택을 제외한 자원을 공유한다. 멀티 프로세스를 사용하면 되는데 굳이 thread를 사용하는 이유는 multitasking을 위해서 여러 process를 만드는 것은thread를 만드는 것보다 자원을 더 많이 소모하며 process간 통신하기 위해 설정하는 것보다 thread간 통신을 위해 소모되는 자원이 더 적기때문이다. Thread는 자원의 효율을 증가시키지만 공유 자원을 사용하면서 생기는 문제를 막기 위해서 동기화 문제에 신경써야한다.

### Multi tasking에서 thread에 개념

process를 하나 더 만들어서 multi tasking을 하면 되지만 너무 많은 자원을 소비하게 된다. 또한 IPC 또한 설정하여서 process간에 정보교환도 하게 만들어야된다. Multithreading이라는 개념을 만들어 보자.

### 멀티프로세스 vs 멀티쓰레딩

Process를 만드는 것은 thread를 만드는 것에 비해 자원에 소모가 심하며 context switch를 하는 경우 더욱 적은 자원을 소모하며 가능하다. 그리고 또한 process간 통신 방법보다 thread간에 통신방법이 훨씬 간단하다. 하지만 thread로 멀티태스킹을 하는 경우 디버그가 어려운 점과 가장 큰 문제는 thread간 통신을 하며 공유자원을 사용하면서 동기화 문제에 신경써야한다.

### Thread의 개념

1개의 process에서 여러개의 thread가 존재가능. Data, code, heap영역은 공유하여서 사용하고 stack과 register는 thread별로 가지고 있는다. Process보다 context switch가 빠르며(register와 stack만 바꾸면 되니까), 생성시에도 빠른 속도가 있다. 왜 각 thread는 스택영역을 따로 가지고 있을까? (생각해보기)
⇒ 스택과 레지스터는 프로세스에서 어떤 부분을 실행하고 어떤 함수를 실행하고 있었는지에 대한 정보를 저장합니다. 그런데 그런 정보를 공유하고 있다고 한다면 3개의 쓰레드를 생각해봤을 때. A, B, C. A에서 B로 넘어가면서 콜 스택이 쌓이고 그것을 다시 A로 갔을 때 어떤 부분이 자신의 것인지 파악하기 어렵고. PC같은 것도 A를 가르키고 있다고 CS가 일어나서 B로 넘어갔다고 하면 기존 A의 PC는 어디에 저장해야되는 문제들이 존재한다고 생각합니다. 그래서 따로 저장해야 되는 부분이라고 생각합니다.

### Thread pool

무제한으로 thread를 만드는 것은 자원이 너무 많이 소비된다. 제한 수의 thread를 만들고 요청이 왔을 경우 thread를 할당해준다. 장점으로는 조금 더 빠르게 요청에 응답하고 thread가 프로그램 내에서 제한된 수로 고정된다.

### 스케줄러란?

Process 상태가 ready인 task 중에 하나를 선택하여 cpu를 할당해주는 것. Waiting은 신경쓰지않고 오직 ready인 task만을 선택한다. 선점형과 비선점형이 있다. 비선점형은 어떤 task를 실행 중에 어떤 중요한 task가 들어와도 기존에 실행하던 task를 수행하고 그 task가 끝나면 다른 task를 하는 것이고, 선점형은 task를 수행 중에 다른 task로 context switch가능하다.

### Dispatch란?

Ready에 있는 task를 running으로 바꾸면서 그 task를 cpu에 올리는 것을 말하며 dispatch latency는 dispath하여서 running이 될때까지 걸리는 시간이다.

### 스케줄러 알고리즘란

1) FCFS(먼저 들어온 task에게 줌) (=FIFO)

2) SJF : 남은 CPU time이 가장 짧게 남은 task에게 준다.

=> 가정 최적이지만 불가능하다.

3)Priority : 고정된 우선순위와 변동가능한 우선순위에 따라 결정

4)Round-robin : 현재 ready 상태에 있는 모든 task에게 동등하게 실행시키게 해준다. (time quantum만큼)5) Multilevel Queue : task를 여러 queue에 넣고 queue간에 스케줄링을하고 queue안에서도 스케줄링을 한다. (Feedback Queue인 경우 starvation을 막기 위해 task가 queue간에 이동을한다.)

### Multiple-Processor 스케줄러란?

여러개에 프로세서(CPU)가 존재할 때 스케줄러는 어떻게 되는것인가? 
SMP(Symmetric Multiprocessing), ASMP(Asymmetric Multiprocessing). 

이슈) 1. Affinity(친밀감) : 최근에 실행한 CPU에 cache가 존재하여 조금 더 빠르게 수행가능
2. Load balancing(일을 분배하는 것) : 일이 몰린곳이 있는 경우 여러 CPU로 일을 분배하는 것.

### LINUX 스케줄링

0~139에 우선순위 스케줄링.

1) real-time class : 0~99 우선순위를 가지고 있음.

=>SCHED_FIFO : 동일한 우선순위를 가지고 있는 경우 FIFO

=>SCHED_RR : 동일한 우선순위를 가지고 있는 경우 RR

2) conventional class : 100~139 우선순위를 가지고 있음.

=>CFS(Completely Fair Sharing) : 언제 어디서든 자신의 비중만큼 실행되게 만든다(우선순위가 높을수록 비중이 큼).

### 동기화와 비동기화

일반적으로 동기와 비동기의 차이는 메소드를 실행시킴과 동시에 반환 값이 기대되는 경우를 **동기** 라고 표현하고 그렇지 않은 경우에 대해서 **비동기** 라고 표현한다. 동시에라는 말은 실행되었을 때 값이 반환되기 전까지는 blocking되어 있다는 것을 의미한다. 비동기의 경우, blocking되지 않고 이벤트 큐에 넣거나 백그라운드 스레드에게 해당 task 를 위임하고 바로 다음 코드를 실행하기 때문에 기대되는 값이 바로 반환되지 않는다.

### Process의 동기화가 필요한 이유?

Multi tasking이 가능해지면서 같은 자원을 사용하는데 있어서 서로 자원을 일치시키기 위해서 필요하다. 그런데 같은 자원을 접근하다가 문제가 발생할 가능성이 있으며 그것을 막기 위해서 여러 동기화 방법이 존재하는데 thread나 process에서 같은 자원을 접근하는데 사용되는 방법은 세마포어, 뮤텍스이다. 또한 다른 process자원을 받기 위해서 message box를 이용하여 값을 기달리는 경우도 있다.

### 세마포어 vs 뮤텍스

세마포어는 여러 task가 동시에 같은 자원을 접근할 때 사용하는 것으로 task간 동기화를 위해 사용된다. 여러개의 process가 접근이 가능한 경우 counter을 지정해 놓는 카운터 세마포어와 오직 하나의 process만 접근이 가능하게 설정한 binary 세마포어가 존재한다. 소유라는 개념이 없고 다른 process에 의해 자원이 릴리즈 될 수 있다.

뮤텍스는 여러 thread가 공유변수에 접근할때 오직 하나의 thread만이 접근을 허용하며 어떤 thread가 사용할때 다른 thread가 공유변수에 접근을 요청한 경우 wait하여야 한다. 그리고 다 사용한 경우에 기달리고 있는 thread를 깨우고 접근을 허용한다. 다른 thread에 의해서 릴리즈되지 않으며 자원을 소유하고 있는 thread만이 자원을 릴리즈할 수 있다.

### 카운터 세마포어 vs 바이너리 세마포어

카운터는 한 자원을 여러 task가 접근 가능한데 접근하여도 값을 변경하지 않는 경우 카운터 세마포어로 선언하여도 문제되지 않는다. 하지만 접근하여서 값을 변경 위험이 있는 경우에는 반드시 바이너리 세마포어로 공유 자원를 보호해주어야한다.

### Critical-section(임계영역)

동시에 접근해서는 안되는 공유자원을 접근하는 코드에 일부분을 말한다. 오직 하나의 task만이 공유변수에 접근이 가능하며 다른 task가 임계영역에 있을때 접근한 경우 wait하여 기달려야한다.

### 동기화를 위한 3가지 조건

1. Mutual Exclusion(상호 배제) : 현재 thread가 사용중에는 다른 thread는 critical section에 접근이 불가능하다.
2. Progress : 현재 아무도 공유자원에 접근한 상태가 아니라면 어느 한 process(thread)는 critical section에 들어가야 한다.
3. Bounded Waiting : 어느정도 기달리면 사용이 가능하다는 보장이 있어야한다.
- Peteson의 솔루션 -> 구현과 자원을 너무 많이 소모
- Semaphore -> 바이너리(하나의 task만 접근), 카운터(여러 task가 접근가능) -> deadlock이라는 문제점이 존재.

### 공유변수를 보호하기 위해서 사용할 수 있는 방법(Linux)

1. spin-lock
2. Semaphore
3. Interrupt disabling
4. Preemption disabling
5. Atomic operation API

### 모니터

process의 동기화에서 세마포어보다 조금 더 높은 레벨에 추상화된 메커니즘. 세마포어와 같이 Mutex(상호배제)를 지원하며 condition value를 가지고 있다. Condition value는 wait()과 signal()을 가지고 있으며 만약 ‘A’ 자원을 ‘가’ task가 접근해서 wait한 경우 ‘나’ task가 접근하여서 signal(java에서는 notify)하지 않으면 ‘가’ task는 계속 기달린다.

### Deadlock

둘 이상에 프로세스가 어떤 자원을 소유한 상태에서 다른 자원을 요청했을 경우(hold and wait) Critical section 진입을 무한정 기달리는 경우. 무한정 blocking된 상태를 말한다. 

- **Deadlock 조건**
    - 1)Mutual exclusion
    - 2)Hold and wait
    - 3)No preemption
    - 4)Circular wait

### Deadlock Avoidance

항상 안전한 상태(demand <= aviliable) 일때만 접근한다. 안전한 상태를 유지하는 안전한 순서를 유지하면서 process의 순서를 정한다. 안전한 순서를 찾는 방법은 2가지고 있고 첫번째로 single instance인 경우 Resource allocation graph 그리기, 두번째로 여러개의 instance를 가지고 있는 경우 banker’s algorithm사용.
