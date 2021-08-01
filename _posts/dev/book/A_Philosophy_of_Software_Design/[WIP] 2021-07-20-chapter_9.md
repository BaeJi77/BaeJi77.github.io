---
title: "[A Philosophy of Software Design] chapter 9. Better Together or Better Apart"

categories:
  - dev
  - book
tags:
  - dev
  - book
  - A Philosophy of Software Design
---

# chapter 9. Better Together or Better Apart

## 9.1

## 9.2

## 9.3


## 9.3 같이 사용하는 것. 중복을 방지한다.

반복적으로 발생하는 코드들에 대해서 제거할 수 있는지 확인해보아라. 이것을 해결하는 방법으로 반복되는 코드 패턴을 하나의 메소드로 만들고 그것을 호출하도록 한다. 단순 한줄, 두줄을 만드는 것을 큰 이득을 얻을 수 없다. 만약 snippet이 많은 변수를 가지고 있다면 대체되는 메소드 또한 복잡한 시그니처를 가져야 될 것이다.

또 다른 방법으로는 특별한 코드를 한곳에서만 사용하도록 리팩토링하는 것이다. 만약 반복적으로 에러를 발생하는 코드가 있다고 가정하고 그것을 처리하는 것이 똑같다고 했을 때. 만약 언어적으로 `goto` 문을 허용한다면 우리는 동일한 코드에서 에러를 처리할 수 있도록 한다. `goto`는 악용했을 경우 굉장히 코드 복잡성을 올리지만 겹쳐져 있는 코드에서 벗어야 되는 경우에서 충분히 효과적으로 사용할 수 있다.

## 9.4 분리하라. 일반적인 목적 코드와 특별 목적 코드를.

일반적인 목적으로 사용되는 매커지니즘에 모듈은 오직 범용적인 목적인 매커니즘만 제공해야 한다. 범용 목적 매커니즘과 연관되어 있는 특별한 목적 코드는 다른 모듈로 구별되어야 한다.

> Red Flag: Repetition
> If the same piece of code (or code that is almost the same) appears over and over again, that’s a red flag that you haven’t found the right abstractions.

일반적으로 low 레벨 레이어 시스템에서 범용 목적인 경우가 많으며 높은 레이어 일수록 더욱 특별한 목적으로 만든 코드이다. 예를 들어 어플리케이션의 가장 높은 레이어는 해당 어플리케이션에 완전히 특별한 기능으로 구성되어져 있다. 

범용 목적코드와 특별 목적 코드를 나누기 위해서는 특별 목적 코드를 위쪽 레이어로 움직이게 만든다. 만약 너가 범용 목적 코드와 특별 목적 코드가 하나의 추상화 레벨에 있는 것을 본다면, 범용 목적 기능과 해당 레이어보다 높은 레이어에서 특별 목적 기능을 제공할 수 있는지 확인해봅시다.

## 9.5 예시: insertion cursor and selection

> Red Flag: Special-General Mixture
> This red flag occurs when a general-purpose mechanism also contains code specialized for a particular use of that mechanism. This makes the mechanism more complicated and creates information leakage between the mechanism and the particular use case: future modifications to the use case are likely to require changes to the underlying mechanism as well.

## 9.6 예시: logging class 분리

``` java
try {
   rpcConn = connectionPool.getConnection(dest);
} catch (IOException e) {
   NetworkErrorLogger.logRpcOpenError(req, dest, e);
   return null;
}
```
``` java
private static class NetworkErrorLogger {
   /**
   *  Output information relevant to an error that occurs when
      trying
   *  to open a connection to send an RPC.
   *
   *  @param req
   *       The RPC request that would have been sent through the
            connection
   *  @param dest
   *       The destination of the RPC
   *  @param e
   *       The caught error
   */
   public static void logRpcOpenError(RpcRequest req, AddrPortTuple
              dest, Exception e) {
     logger.log(Level.WARNING, "Cannot send message: " + req + ".
             \n" + "Unable to find or open connection to " + dest +
             " :" + e);
   } ...
}
```

`NetworkErrorLogger`는 여러 매소드들을 가지고 있는 다양한 에러 상황과 관련해서.

이런 분리는 어떤 이득 없이 복잡성만 올린다. 대부분의 코드는 한줄로 되어 있지만 많은 문서를 요구한다. 그리고 해당 메소드들은 오직 한곳에서만 사용된다. 그뿐만 아니라 해당 로깅 메소드는 그 호출에 대해서 높은 의존성이 있다. 사람들은 해당 로그 정보를 보기 위해서 해당 로그 클래스를 보거나 해당 메소드를 이해하기 위해서 로그 클래스를 보는 경우가 있다. 

위에 예제는 로깅 메소드를 지우고 해당 에러가 발생하는 곳에 메소드 내용을 넣는 것이 좋다. 그것이 더욱 코드를 이해하기 쉽게 할 거이다.

## 9.7 예시: edior undo mechanism


## 9.8 Splitting and joining method

> Red Flag: Conjoined Methods
> It should be possible to understand each method independently. If you can’t understand the implementation of one method without also understanding the implementation of another, that’s a red flag. This red flag can occur in other contexts as well: if two pieces of code are physically separated, but each can only be understood by looking at the other, that is a red flag.
> 만약 메소드를 분리했는데 분리된 메소드들을 같이 봐야지만 이해가 되도록 분리했다면 그것은 좋지 않다.

## 9.9 Conclusion

# 느낀 점
