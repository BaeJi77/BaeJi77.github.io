---
title: "13장. I/O"
description: "자바의 I/O에 대해 학습하세요."

categories:
  - dev
  - java
tags:
  - dev
  - java
---

# 목표
자바의 I/O에 대해 학습하세요. [issue](https://github.com/whiteship/live-study/issues/13)

# 학습할 것
## I/O
I/O는 기본적으로 input / output을 줄여서 표기할 때 사용한다. 

입력같은 경우는 키보드나 마우스와 같이 특정 매체를 통해서 컴퓨터로 데이터를 주입하는 경우에 속하며 

출력같은 경우는 모니터, 파일들과 같이 프로그램에서 데이터를 다른 곳으로 보여줄 때 출력이라고 표현한다.

## 스트림 (Stream) / 버퍼 (Buffer) / 채널 (Channel) 기반의 I/O
### 스트림
흐름이라는 뜻을 가진다. I/O에서 스트림은 기본적으로 한 방향으로 흐른다고 파악하면 될 것이다.

- 인풋스트림 -> 입력이 흐르는 통로
- 아웃풋스트림 -> 출력이 흐르는 통로

### 버퍼
buffer는 메모리 공간에 특정 공간을 활용하여서 한번에 보내는 방식을 의미한다.

스트림과 다른 개념이 아닌 스트림을 구현하는 하나의 방식으로 생각하면 된다.

### 채널
스트림과 다르게 채널은 양방향 통신을 베이스로 하고 있다.

## InputStream과 OutputStream
스트림같은 경우 자바에서 제공하는 바이트 기반으로 입출력이 장치에서 추상클래스의 상위 단계이기 때문에 구체적인 구현에 대한 모든 애들을 해당 타입으로 해서 사용할 수 있다.

- InputStream
``` java
public int read()
public int read(byte[] b)
public int read(byte[] b, int off, int len)
```
위와 같은 메소드를 제공해주는데 실제 구현이 되어진 클래스도 모두 해당 메소드를 가지고 있기 때문에 그 밑에 타입에서도 바로 사용가능하다.

- OutputStream
``` java
public void write(int b);
public void write(byte[] b);
public void write(byte[] b, int off, int len);
```
아웃풋스트림도 비슷하다.

## Byte와 Character 스트림
자바의 I/O와 관련해서 크게 2가지 종류가 있다.

### Byte 스트림
1Byte는 8bit를 의미하는 단위이다. 1Byte를 가지고 여러 값을 표현할 수 있지만 그렇지 못하는 경우도 있다. Int, Long과 같은 경우 4Byte, 8Byte가 필요하기 때문에 제대로 된 값을 표현하지 못한다.

여기서 Byte를 이용하는 것은 해당 값들을 짤라서 보내는 것이 아닌 데이터를 단순히 Byte로만 취급하여서 나눠서 보내겠다는 것이다.

``` java
InputStream is = new ByteArrayInputStream(...);
OutputStream ops = new ByteArrayOutputStream(...);

while(1) {
    int read = is.read();
    if(read == 0) {
        break;
    }

    ops.write(read);
}
```
데이터를 읽고 해당 데이터가 없는 경우에 read() 메소드는 0을 리턴한다. 데이터를 끝까지 읽고 해당 프로세스를 끝낸다.


### Character 스트림
바이트 기반 스트림과 다른 것은 2byte 단위로 처리한다. 여러 종류의 인코딩과 자바에서 사용하는 유니코드 간의 변환을 자동으로 처리한다.

``` java
StringReader input = new StringReader(inputData);
StringWriter output = new StringWriter();
int data;

while ((data = input.read()) != -1) {
    output.write(data);
}
```

## 표준 스트림 (System.in, System.out, System.err)
- 콘솔로 부터의 데이터 입력을 뜻한다.
- 자바 어플리케이션이 실행과 동시에 생성하는 코드를 제공하기 때문에 별도의 스트림을 만들 필요가 없다.

``` java
public static final InputStream in = null;
public static final PrintStream out = null;
public static final PrintStream err = null;
```
인풋같은 경우는 `InputStream`으로 되어있고, 아웃풋 같은 경우는 `PrintStream`으로 되어 있다.

### 표준 스트림 변경 방법
``` java
public static void setIn(InputStream in) {
     checkIO();
     setIn0(in);
}
    
public static void setOut(PrintStream out) {
     checkIO();
     setOut0(out);
}
    
public static void setErr(PrintStream err) {
     checkIO();
     setErr0(err);
}
```

## 파일 읽고 쓰기
``` java
private static void pipe(File source, File destination) throws IOException {
    long now = System.currentTimeMillis();
    try (InputStream inputStream = new BufferedInputStream(new FileInputStream(source));
          OutputStream outputStream = new BufferedOutputStream(new FileOutputStream(destination))) {

        int readCnt = 0;
        while ((readCnt = inputStream.read()) > -1) {
            outputStream.write(readCnt);
        }
    }
    System.out.println("bufferedStream - Performance time: " + (System.currentTimeMillis() - now));
}
```
`try-with-resource`를 이용해서 input, output stream을 선언했으면 어떤 선언도 하지 않고 사용했다.

buffered같은 경우 처음에 버퍼크기를 선언할 수 있다는 것도 알아두면 좋다. 적게하면 메모리를 처음 적게 사용하는 부분이 있지만 그만큼 IO연산이 많아진다.


# ref
[klom27님의 블로그](https://b-programmer.tistory.com/268)

# 라이브 방송

## 베스트 글 모음
https://www.notion.so/I-O-af9b3036338c43a8bf9fa6a521cda242

https://b-programmer.tistory.com/268

https://github.com/kyu9/WS_study/blob/master/week13.md

https://blog.naver.com/swoh1227/222237603565

https://blog.naver.com/swoh1227/222244309304

https://alkhwa-113.tistory.com/entry/IO

https://alkhwa-113.tistory.com/entry/NIO

https://github.com/mongzza/java-study/blob/main/study/13%EC%A3%BC%EC%B0%A8.md

