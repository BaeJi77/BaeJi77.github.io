---
title: "Network 기본 개념 - OSI 7-layer"

categories:
  - cs
  - network
tags:
  - dev
  - cs
  - network
---

# OSI 7-layer
## 네트워크 끼리 서로 통신을 하기 위해서 만든 7계층의 프로토콜

- 7-layer
  - layer 1 - physical - bit
  - layer 2 - data link - frame → 실제 어디로 물리적으로 어디로 이동해야 되는지 결정
    - ⇒ hop to hop - MAC address
  - layer 3 - network - datagram → 최종 목적지까지 이동을 담당. 다음 라우터로 이동해야되는지 경로 결정
    - ⇒ source to destination - IP Address
  - layer 4 - tranport - segment → 어떤 어플리케이션과 연결되어지는지에 대한 정보인 port번호와 통신상 발생하는 에러에 대하여 처리해준다.
    - ⇒ process to process - port number
  - layer 5 - session
  - layer 6 - presentation
  - layer 7 - application

## Circuit switching vs Packet switching

1. Circuit switching
    - 중앙에서 경로를 선정
    - 연결이 끝날 때 까지 처음 경로를 유지
    - 사용하지 않더라도 지속적으로 연결을 유지하고 있음
    - 처음 하나의 패킷이 이동해서 길을 만든 후 거기를 전용선으로 사용
    - 속도를 보장하지만 낭비가 매우 심함
    - 네트워크를 담당하는 곳이 부셔지는 경우 통신 불가능
2. Packet switching
    - 분산으로 경로를 선택해서 packet을 전송
    - packet들이 서로 다른 경로를 선택할 수 있음
    - 데이터를 일정 크기로 짤라서 보냄
    - 각 계층이 모듈화되어 있음
    - 혼잡이 심함 → 누군가 통제하는 것이 아니기 때문에
    - 속도나 제대로 도착하는 것에 대하여 보장되지는 않지만 각 패킷에 대하여 경로가 독립적이라 패킷이 잘 이동하는 것 뿐만 아니라 낭비가 심하지 않음.
    - 그리고 하나의 라우터가 고장난다고 해서 문제가 되지 않음. ⇒ 경로가 여러개가 존재하니

## TCP/IP

오늘날 인터넷 통신의 대부분은 패킷통신을 기본으로 하고 있습니다. TCP/IP는 이러한 패킷 통신을 위한 인터넷의 규약입니다. **IP는 데이터의 조각들을 최대한 빨리 목적지로 보내는 역할을 합니다.** 조각들의 순서가 뒤바뀌거나 일부가 누락되더라도 크게 상관하지 않고 보내는 데 집중을 합니다. **TCP는** IP보다 느리지만 꼼꼼한 방식을 사용합니다. **도착한 조각을 점검하여 줄을 세우고 망가졌거나 빠진 조각을 다시 요청합니다**. 두 방식의 조합을 통하여 인터넷 데이터 통신을 하는 것을 묶어 TCP/IP라고 부르는 것입니다.

## Ref
[03. 개알못을 위한 TCP/IP의 개념](https://brunch.co.kr/@wangho/6)
