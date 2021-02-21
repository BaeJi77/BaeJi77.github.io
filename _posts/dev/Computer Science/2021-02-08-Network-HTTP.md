---
title: "Network 기본 개념 - HTTP"

categories:
  - cs
  - network
  - http
tags:
  - dev
  - cs
  - network
  - http
---

# HTTP
## 버전별 비교
- HTTP
    - **Connectless**하게 통신하기 때문에 한번 요청 후에 다시 요청을 하려면 다시 연결을 해야 됩니다. (http/1)
    - **상태정보 유지 안함 (Stateless)** -> 연결을 끊는 순간 서로 상태 정보를 유지하지 않음. 서로의 정보를 모름
    - 위와 같은 성질때문에 **모든 요청은 독립적임.**
    - 전에 요청했던 결과를 관리하기 위하여 세션과 쿠키으로 처리 가능
    - 웹 브라우저와 웹 서버 간의 서버 통신을 위하여 만들어진 프로토콜입니다.
    - 기본적으로 메소드가 존재하고 그것에 따라 의미가 달라지거나 보내는 형식이 달라질 수 있습니다.
- HTTP1.1
    - HTTP가 느린 이유
        - request를 날릴 때마다 connection을 새로 만들어야 함. 그래서 지속 가능한 keep-alive 가능하도록 함
    - HTTP pipelining - request 를 여러개 서버에 보낼 수 있는 기능이 생김 (but, response를 순서대로 받아야 하는 문제가 발생 - HOLB (Head of line blocking) problem - 첫번째 보낸 요청에 문제가 있다면 두번째, 세번째 요청도 기달려야 함 - why? TCP는 무조건 데이터를 순서대로 처리해야 되기 때문에)
    - 도메인 샤딩
- HTTPS
    - HTTP/1, 1.1 버젼에서 보안적이 요소를 추가하기 위하여 만들었습니다.
    - 통신을 할때 보내는 부분이 암호화 되어 있지 않은 경우 모두 내용을 확인 할 수 있기에 그것에 대한 대응책입니다. (헤더부터 바디까지 모두 암호화)
    - 그렇게 되더라도 실제 어디로 보내고 받는지 정도 확실히 알기 때문에 받는 쪽에서 그 데이터를 받아서 해석하면 상관없음. 모든 부분 암호화.
    - 공인 인증 방식으로 CA라는 공인 기관이 중간에서 신뢰성을 보장해줍니다. 그 이후에 공개키 암호화 방식이나 대칭키 암호화 방식으로 서로 데이터를 주고 받을 수 있습니다.
    - 암호화하기 때문에 HTTP/1 보다는 속도가 조금 느릴 수도 있습니다.
- HTTP/2
    - 위에 같은 HTTP/1의 문제를 해결해줌
    - Multiplexing 개념
        - 하나의 TCP연결로 하나의 클라이언트의 요청을 모두 처리하는 방법
        - 이전에는 하나의 리소스마다 TCP 연결을 만들고 요청하였다.
    - Server push: html을 받아서 요청하는 내용들을 서버에서 한번에 같이 보내준다. (그 전에는 파일을 로딩하다가 필요하면 요청하는 방식이였음)
    - 헤더 압축
        - 이전에 해본 결과가 있으면 테이블의 인덱스만 전송해서 보냄
        - header을 huffman 인코딩을 통해서 보냄
- HTTP/3
    - UDP로 통신
    - QUIC위에서 돌아가는 HTTP (Quick UDP Internet Connection)
    - TCP가 결국 느린 것은 통신 연결을 진행하는 과정과 끊어버리는 과정이 존재함.
    - HOLB문제에서 벗어 나지 못함.
        - 패킷이 순서대로 되어야지만 정확한 데이터를 만들어 낼 수 있음. 왜냐하면 바이트의 흐름이기 때문에
        - 허지만 UDP는 하나의 패킷 (Datagram)으로 보내기 때문에 애초에 독립적인 패킷임. 서로 종속성이 없음
    - 원래부터 연결 설정이 없었기에 QUIC은 1RTT로 해결함.
    - 멀티플렉싱 지원
    - 보안적으로 문제 X
    - IP 주소가 바뀌는 것에 대한 문제 없음
    - 흐름 제어는 Stop and wait ARQ가 아닌 패킷을 보낼 때 다른 데이터도 같이 보내서 패리티 비트로 원본 데이터를 복구 가능함

# SSL / TLS

- ssl과 tls는 암호화 프로토콜이다.
- 현재 ssl은 사용이 권장되지 않고 tls는 ssl를 조금 더 발전시킨것이다.
- tls handshake 과정이 존재하고 이 과정을 통해 사용자가 내가 원하는 곳인지 확인가능하며 서로 대칭키를 교환함으로써 암호화 통신을 할 수 있도록 한다.
- 자세한 과정은 위에 내용 숙지 바람.
- 과정
  - 신뢰받은 기관은 서버의 정보를 hash한 값을 기관의 RSA 개인키로 암호화하여 서버의 정보와 암호화한 값을 모두 보내줌
  - 사용자는 서명한 기관의 공개키로 서명을 복호화한다. (기관의 공개키는 브라우저에서 가지고 있는 경우가 많다.)
  - 본문의 내용을 hash한 결과와 hash되어서 보내진 값을 비교한다.
  - 이것을 통해 변조를 확인한다.
  - 서버와 클라이언트 간에 검증 절차에 따라 서버는 클라이언트에 데이터를 보내주는데 기관에 저장된 데이터를 hash해서 보내서 비교한다던지에 대한 방법을 선택한다.
  - 그리고 필요하다면 둘이 DH 키 교환 방식을 위해 자신의 키 값들을 서로 보낸다.
  - 이 과정을 진행하면서 클라이언트에서도 자신의 키와 함께 서로를 검증할 수 있는 데이터를 보내고 서버는 그것을 통해 정상적인 사용자인지 확인한다.
  - 이 과정 이후 똑같은 키를 가지고 있기에 대칭키 암호화 방식을 통해 서로 데이터를 주고 받을 수 있게 된다.

## Cipher Suite

TLS에서는 암호화하는 방법을 표준으로 특정하지 않고, server와 client가 합의해서 결정한다. 서로 합의해야 하는 알고리즘은 4가지인데, **대칭키 전달 방식, 인증서 서명 방식, 대칭키 알고리즘, HMAC 알고리즘**이다. TLS에서는 서버와 클라이언트가 이 4가지 알고리즘을 세트로 합의하고, 합의된 알고리즘으로 application layer의 내용을 암호화해서 전송한다.

`cipher suite`는 `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`와 같이 표기한다. 이 `cipher suite`를 읽어보면, 다음과 같이 해석할 수 있다.

- **대칭키 전달 방식**은 ECDHE(Elliptic Curve Diffie Hellman Ephermeral)을 사용하며,
- **인증서 서명 방식**은 RSA로 서명된 인증서로 상호간의 신원을 확인하고,
- **대칭키 암호화 알고리즘**은 AES 256bit와 GCM를 채택하고,
- **HMAC 알고리즘**으로는 SHA 384를 사용해서 메시지의 무결성을 확인한다.

`cipher suite`는 여러가지 있다. 그중에서 client가 지원하면서 취약하지 않은 `cipher suite`의 리스트를 서버에게 알려주면, 서버는 그중에 지원하는 `cipher suite`를 선택해서 보내 합의한다.

## TLS의 동작

TLS는 application protocol의 내용을 암호화하는데에 목적을 둔다. 암호화 전에, `TLS Handshake protocol`은 TLS 버전과 `cipher suite`를 합의하고, 인증서를 교환한다. 그리고 필요하면 `Diffie-Hellman` 키도 공유한다.

```
  Client                                               Server

  ClientHello                  -------->
                                                  ServerHello
                                                 Certificate*
                                           ServerKeyExchange*
                                          CertificateRequest*
                               <--------      ServerHelloDone
  Certificate*
  ClientKeyExchange
  CertificateVerify*
  [ChangeCipherSpec]
  Finished                     -------->
                                           [ChangeCipherSpec]
                               <--------             Finished
  Application Data             <------->     Application Data

         Figure 1.  Message flow for a full handshake

* Indicates optional or situation-dependent messages that are not
always sent.
```

1. 클라이언트 쪽에서 진행가능한 cipher suite를 서버에게 보낸다.
2. 서버는 보낸 cipher suite를 보내면 가능한 경우가 없으면 연결 끝이고 가능한 경우가 있으면 그것에 해당하는 키 교환 데이터를 보낸다. 여기서 Certificate 메시지는 서버의 인증서와 그 인증서를 확인한 CA들의 인증서인를 보낸다. 여기서 DH 키 교환방식이면 ServerKeyExchange 메시지를 통해 서버의 키를 보낸다.
3. 클라이언트는 서버쪽에서 보낸 ServerHelloDone을 받고 Certificate 메시지를 확인해서 정상적인 사용자인지 서명을 확인한다.
4. 클라이언트에서는 서버쪽에서 진행하는 키 교환 방식에 대하여 확인하고 DH 인 경우 클라이언트 키를 보낸다.
    - 그리고 그 과정에서 확인을 위해서 finished라고 보내며 내가 만든 키에 대한 결과값의 해쉬한 결과값을 보내준다.
5. 서버는 finished를 받기 전 [ChangeCipherSpec]를 통해서 키를 교환하고 finished를 받으면서 클라이언트에서 온 키의 대한 해시값과 내가 가지고 있는 키의 해시값을 비교해서 맞는지 확인한다. 맞다면 클라이언트에게 Finished를 전송
6. 그 이후로는 서로 가지고 있는 대칭키를 가지고 열심히 암호화 데이터를 보낸다.

# 추가적으로 알면 좋을 내용
[About http/3](https://evan-moon.github.io/2019/10/08/what-is-http3/)


# Ref
[HTTP mozilla](https://developer.mozilla.org/ko/docs/Web/HTTP/Overview)

[About https](https://parksb.github.io/article/24.html)

## SSL/TLS
[SSL-TLS HTTPS 란? 암호화 통신방법 설명](https://soul0.tistory.com/510)

[SSL vs. TLS - 차이점은 무엇인가?](https://smartits.tistory.com/209)

[HTTPS 와 SSL(TLS)](https://futurecreator.github.io/2018/07/12/https-and-ssl-tls/)

[Luavis' Dev Story - 알아두면 쓸데없는 신비한 TLS](https://b.luavis.kr/server/tls-101)