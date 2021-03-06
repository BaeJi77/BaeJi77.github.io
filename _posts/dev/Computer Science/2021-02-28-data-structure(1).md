---
title: "Data Structure 기본 개념 (1)"

categories:
  - cs
  - data-structure
tags:
  - dev
  - cs
  - data-structure
---

# 자료구조

정해진 규칙대로 자료를 다룬다는 것. 결국 데이터를 잘 다루기 위해서 만들어짐.

"기억해두자. 우리가 원하는 것은 방대한 자료들을 다루면서도 매우 빠른 시간 안에 원하는 결과를 낼 수 있도록 하는 것이다."

## 추상적 자료 구조 (ADT)

추상적 자료 구조에서는 **구현을 분리하여 생각**한다. 후술 할 스택 자료 구조를 정의할 때 push나 pop과 같은 **연산을 정의하고, 그 연산이 연산 복잡도 O(1)을 만족하기만 한다면 어떻게 구현하든 상관하지 않는다.** 인터페이스와 구현을 분리하여 생각하자는 것이다. 그렇기 때문에 큐를 배열로 구현된 것을 쓸 수도 있고, 연결 리스트로 구현된 것을 쓸 수도 있다.

⇒ 가장 중요한 것은 이 자료구조의 본질에 대한 것이다. 이 자료구조는 이 연산에서는 이렇게, 저 연산에서는 저렇게. 그런 것들이 정의되어 있고 실제 내부 구조에 대해서는 전혀 신경쓰지 않는 것이다. 한마디로 인터페이스만 제공해주고 그 내부 동작에 대해서는 신경쓰지 않는 것이다.

⇒ stack에서 중간에 있는 값을 볼 수 있을까요?
    아니요. stack 같은 경우 ADT가 정의되어 질 때 가장 위에 있는 값만을 보기 위해서 만들어진 자료구조이기 때문에 중간에 있는 값을 보는 것은 이 자료구조와 맞지 않는 자료구조 활용이라고 생각합니다.

## 스택과 큐

함수를 호출하면 실행 흐름이 그 함수로 넘어간다. 그 함수에서 또 다른 함수가 실행될 수도 있고, 실행을 끝내고 함수를 빠져나올 수도 있다. 함수에서 빠져나오면 마지막에 함수를 실행했던 곳으로 돌아온다. 

이렇게 작동될 수 있도록 하기 위해 프로그래밍 언어론적으로 함수 실행 시 활성 레코드(Activation Record)를 쌓는다고 말한다. 활성 레코드에는 함수를 종료한 뒤 돌아가야 할 주소와 현재 활성 레코드의 크기에 대한 정보가 포함되어 있기 때문에, 함수 스택을 정리할 때 활성 레코드를 참고하여 규칙적으로 운영할 수 있다.

메세지 큐는 다양한 곳에서 응용된다. scanf와 같은 동기 입력 함수를 실행하면 사용자의 입력을 기다리는데, 터미널에 키보드를 입력하면 입력이 수행되는 것을 볼 수 있다. 이는 OS가 키보드에서 생긴 이벤트를 프로세스의 이벤트 큐에 넣어주는 것인데, 이벤트 큐는 메세지 큐의 대표적인 예 중 하나이다. 메세지 큐 자체가 또 하나의 추상적인 개념이기 때문에 프로세스 간 통신(Inter Process Communication)을 비롯한 여러 곳에서 활용되는 개념이다.

## 링크드 리스트
연결 리스트가 이러한 상황에 유용한 구조이다. 연결 리스트의 원소는 다음 원소를 가리키는 포인터를 갖고 있기만 하기 때문에 중간 삽입이나 삭제를 할 때면 포인터만 잘 처리해주면 된다. 배열을 사용할 때처럼 한 칸씩 원소를 다 밀어내거나 당겨오는 복사 작업을 해야 할 필요가 없기 때문에 유용하다. **그러나 임의 원소 접근이 느리기 때문에 잦은 임의 원소 접근이 필요한 경우엔 적합하지 않다.**

파일 시스템의 **가용 디스크 블록**을 연결 리스트로 관리하는 것이 연결 리스트를 활용한 예 중 하나이다. 디스크의 물리 디스크는 블록이라는 단위로 파일 시스템에 의해 관리되는데 파일이 더 많은 공간을 필요로 하게 될 경우 파일 시스템에게 추가적인 디스크 블록을 요청하여 할당받는다. 파일 시스템은 가용 디스크 블록 리스트에서 하나를 빼내어 파일에게 할당해준다.

데이터베이스에서 없어서는 안 될 자료 구조인 **B-tree**에서도 단말 노드 간의 연결을 위해 연결 리스트가 사용된다. **단말 노드를 순차적으로 순회하기 위해서는 단말 노드의 연결 리스트를 차례로 순회하면 된다.**

## 트리

트리는 마치 나무가 자라는 것처럼 생긴 자료 구조이다. 이름도 그래서 트리다. 아마 직관적으로 의미를 느끼기 어려워지기 시작하는 자료 구조일 것이다. 트리는 connected component이면서 cycle이 존재하지 않는 특수한 그래프라고 볼 수 있다. 문제를 그래프 개념으로 탐색을 적용할 때도 트리 순회 개념이 적용된다

트리를 자료 구조로서 활용하는 대표적인 예가 검색일 것이다. 익히 알고 있듯이 이진 검색은 O(logN)의 시간 복잡도를 보인다. 이는 검색을 해나갈 때 고려 대상을 1/2씩 줄여나갈 수 있는 방법이기 때문이다. 이 개념을 좀 더 확장하여 정확히 값이 같지 않더라도 비교가 가능한 점을 이용한 것이 데이터베이스의 B-tree 구조이다.

검색을 빠르게 수행하려면 필수적인 것이 트리의 깊이를 최대한 키우지 않는 것이다. **깊이가 깊어지면 깊어질수록 비교를 수행해야 하는 횟수가 늘어난다.** 트리 전체가 메모리에 올라와있는 경우에는 큰 문제가 되지 않지만 디스크에 저장된 트리(데이터베이스의 인덱스 트리)는 비교를 수행할 때마다 디스크에 접근해야 하기 때문에 깊이가 1만 증가해도 매우 느려진다. 그렇기 때문에 균형 트리(Balanced Tree)를 유지하는 것이 중요한 문제가 되며, AVL 트리를 배우는 것은 자가 균형 트리를 유지하는 방법 중 하나를 알아보기 위함이다. 

⇒ 균형 트리를 유지 하여야지만 트리의 높이가 매우 커지지 않는다. 깊이가 깊지 않아야지만 최대한 적은 비교로 내가 원하는 목적지에 도달할 수 있다. 비교 한 번 당 많이 없애버리면 좋으니까~!

더 읽으면 좋은 자료: [트리의 종류와 이해](http://www.secmem.org/blog/2019/05/09/%ED%8A%B8%EB%A6%AC%EC%9D%98-%EC%A2%85%EB%A5%98%EC%99%80-%EC%9D%B4%ED%95%B4/)

## 그래프

그래프는 일련의 [꼭짓점](https://ko.wikipedia.org/wiki/%EA%BC%AD%EC%A7%93%EC%A0%90)(Vertex)들과 그 사이를 잇는 변(Edge)들로 구성된 [조합론](https://ko.wikipedia.org/wiki/%EC%A1%B0%ED%95%A9%EB%A1%A0)적 구조이다. 정의가 간단한만큼 세부 종류도 다양하다. 꼭짓점은 노드(Node)라고 부르기도 한다. 변이 방향성을 가지는지에 따라 유향 그래프 또는 무향 그래프로 분류하고 꼭짓점을 이분할 수 있는지에 따른 이분 그래프, 변과 변이 교차하지 않을 수 있는지에 따른 평면 그래프 등 다양한 그래프가 존재한다.

# ref
[정주홍님의 블로그](https://brunch.co.kr/@toughrogrammer/12)
