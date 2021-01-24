---
title: "[Clean code] chapter 5. 형식 맞추기"

categories:
  - dev
  - book
tags:
  - dev
  - book
  - clean code
---

# Overview
소프트웨어는 혼자 작성하는 것이 아니다. 그러기 때문에 하나의 소스 파일에도 여러 코드 스타일이 존재할 수 있다. 우리는 어떤 형식, 코드 스타일이 가독성을 올릴 수 있는지에 대해서 이야기 나눌 것이다.

## 형식을 맞추는 목적
코드를 형식에 맞게 작성하는 것은 중요하다. 왜냐하면 코드의 일관성은 코드를 읽는 사람에게 코드를 예상하게 만들고 코드를 파악하는데 도움이 된다. 만약 그렇지 않은 경우 코드에 대한 신뢰가 떨어지게 된다.

우리가 지금 작성한 코드가 다음 버전에서는 완전히 바뀔수도 있다. 하지만 나의 코드 가독성으로 인해서 앞으로 바뀔 코드의 품질에 큰 영향을 줄 것이다. 우리가 작성한 코드의 규율과 스타일이 시간이 지나고 여러 업데이트가 진행되더라도 계속 있을 가능성이 높다. 

## 적절한 행 길이를 유지하라
적은 길이의 행 길이로도 충분히 큰 프로젝트를 만들 수 있다. 실제 책에서는 여러 프로젝트에 파일별 코드 길이를 통계를 냈는데 작게 만드는 경우도 있고 크게 만드는 경우도 있었다. 이 내용은 작은 길이의 소스 파일이라도 충분히 큰 프로젝트를 만들 수 있는 증거가 된다.

## 신문 기사처럼 작성하라
좋은 네이밍을 가지도록 만들게 되면 어떤 내용에 대한 소스 코드인지 확실히 알려줄 수 있다. 시간을 줄일 수 있다.

## 개념은 빈 행으로 분리하라
**중요** 빈 행을 넣는 이유는 반드시 존재한다. 단순 코드가 잘 보이는 것 뿐만 아니라 의미 전달에도 큰 도움이 된다.

나 같은 경우 항상 이런 것을 신경쓰려고 한 것은 아니지만 매번 빈 행을 의무적으로 많이 넣었다. 하지만 이런 명확한 룰은 나중에 코드 작성에 도움이 될 것 같다. 하지만 모든 사람이 이런 생각을 가지고 코드를 읽으면 위험할수도 있지 않을까? ㅎ.ㅎ

## 세로 밀집도
위에 있는 내용과 비슷. 같은 개념은 세로 위치적으로 가깝게 있어야 된다.

## 수직 거리
우리는 이런 경험을 했을 것이다. 함수를 오가면서 파일을 위아래로 뺑뺑이를 돌고, 상속 관계에 있는 파일을 돌아디니면서 코드를 이해하려고 노력하는 경험. (현재 하는중) 굉장히 좋지 않은 경험일 것이다.

서로 밀접한 개념은 세로로 가까워야 한다. 한마디로 세로 밀집도가 높아야 한다. 

### 변수 선언
함수는 사용하는 위치에 최대한 가까이 선언한다. 함수가 매우 길지 않은 경우에는 함수에 맨 처음에 선언한다.

### 인스턴스 변수
클래스 맨 처음에 선언한다. 클래스 메소드가 해당 변수를 사용하기 때문에 여러번 사용될 수 있고 여러 메소드와 연관성이 있을 수 있기 때문에 클래스 맨 처음에 선언한다. 

여러 논쟁이 있을 수 있지만 특정 한 공간에 존재하도록만 한다면 큰 문제는 없다.

### 종속 함수
한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다. 또한 가능하다면 호출하는 함수를 호출 당하는 함수보다 먼저 배치한다. 해당 규칙을 일관적으로 정의해서 사용한다면 독자들은 호출한 함수가에 대한 순서에 대해서 예측할 수 있을 것이다.

### 개념적 유사성
한 함수가 다른 함수를 호출하는 직접적인 종속성, 변수와 그 변수를 사용하는 함수 이런 것들도 개념적으로 유사성이 있다. 그것 외에도 비슷한 동작을 수행하는 함수들도 좋은 예이다.

## 세로 함수
이 부분은 계속 이야기하는 것 같다. 그것만큼 중요하다고 생각한다. 함수 호출 순서에 따른 순서를 잘 만들자!!

## 가로 형식 맞추기
세로에서 볼 수 있는 여러 형태 파일과는 다르게 가로에 제한과 같은 경우는 확실히 길지 않다. 길면 100자에서 120자까지 갈 수 있지만 그것을 넘기는 것은 좋지 않다고 생각한다.

옛날에는 오른쪽으로 스크롤할 필요없이 코드를 짰다. 현재는 당연히 환경이 달라졌지만 가급적 너무 길게는 짜지 않는 것이 좋다.

## 가로 공백과 밀집도
코드를 작성하면서 가로 공백를 통해서 개념적으로 밀접함과 느슨함을 표현할 수 있다. 

``` java
private void veasureLine(String line) {
    lineCount++;
    int lineSize = line.length();
    totalChars += lineSize;
    line.addLine(lineSize, lineCount);
    recordLine(lineSize);
}

---

public double root(int a, int b, int c) {
    double determinant = determinant(a, b, c);
    return (-b - Math.sqrt(determinant)) / (2*a);
}
```

함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않는다. 함수를 호출하는 코드에서 괄호 안 인수는 공백으로 분리했다. 쉼표를 강조해 인수가 별개라는 사실을 보여주기 위해서이다. 

또한 연산자 우선순위를 위해서 공백을 사용한다. 위에 예제 처럼 부호와 연산자를 확실하게 구분하기 위해서 `-`를 다르게 사용하는 것도 보이며 공백으로 인해서 연산에 대해 명확하게 볼 수 있다. 하지만 이 부분은 ide에서 지원하지 않기 때문에 이렇게 작성하고 자동 라인 정렬을 하게 되면 깨질 가능성이 있다.

## 가로 정렬
``` java
public class Test {
    private String        name;
    private TestTool      testTool;
    private TestUtilTool  testUtilTool;
}
```
위와 같이 변수에 대해서 명확하게 한 줄로 볼 수 있도록 정렬하는 경우가 있다. 이런 경우 일단 타입보다는 변수에 더 집중하게 되는 효과가 있고 이렇게 만들었어도 ide에서 자동정렬로 깨지게 된다.

## 들여쓰기
들여쓰기는 우리가 소스코드를 읽을 수 있게 만드는 요소이다. 들여쓰기를 통해서 scope을 명확하게 볼 수 있으며 그것을 통해서 읽어야되는 코드와 읽지 않아도 되는 코드를 구분할 수 있게 된다. 들여쓰기는 매우 중요하며 매우 강조해고 모두들 중요하게 생각하고 있다.

### 들여쓰기 무시하기
가끔씩 들여쓰기를 무시하고 한줄로 쓰는 것에 대해서 고민하는 적이 있다. (나 또한 그렇게 코딩을 많이 했었다.) 하지만 들여쓰기를 추가하는 것을 권장한다고 한다.

## `팀 규칙`
프로그래머는 각자가 선호하는 규칙이 존재한다. 하지만 팀에 속해있다면 팀 규칙을 선호해야 한다. 이렇게 모든 팀이 하나의 팀 규칙을 가지고 있어야지만 소프트웨어가 일관적인 스타일을 보인다.

좋은 소프트웨어 시스템은 읽시 쉬운 문서로 이뤄진다는 사실을 기억하기 바란다. 한 소스 파일에서 봤던 형식이 다른 소스 파일에 존재해야지 소스 파일에 대한 신뢰가 올라간다. 그러니 꼭 팀 규칙을 만들고 지키고 이행하자!


## 실습 `intellij에 code style 적용하기 `
![code-style](/assets/images/2020-12-01-book-clean-code-4/code-style.png)

intellij에서 이런 code style 우리가 바로 커스텀할 수 있다. 탭 크기 뿐만 아니라 다른 탭을 누르게 되면 스페이스를 하게 되는 경우 이런 것들도 모두 커스텀할 수 있다.

1. `cmd + ,`를 같이 누르게 되면 환경설정 창이 나오게 된다. 검색창에 `code style`을 검색한다. 여기서 `default`가 되어 있는 것에 톱니바퀴를 누르면 `import`와 관련된 옵션이 나온다.
![code-style](/assets/images/2020-12-01-book-clean-code-4/code-style-import-1.png)

2. code style이 저장되어 있는 `.xml` 파일을 선택한다.
![code-style](/assets/images/2020-12-01-book-clean-code-4/code-style-import-2.png)

3. 그 이후에 `OK`을 누른다.
![code-style](/assets/images/2020-12-01-book-clean-code-4/code-style-import-3.png)

4. 잘 적용된 모습을 보고 `apply`을 누르게 되면 된다.
![code-style](/assets/images/2020-12-01-book-clean-code-4/code-style-import-4.png)

## SonarQube를 통한 코드 품질 검사
[SonarQube](https://www.sonarqube.org/) 라는 툴이 존재한다.

코드 품질을 검사해주는 툴이다. CI와 연동해서 코드 품질과 관련해서 바로 Github pr 코멘트로 받을 수 있는 옵션도 존재한다.


## 의견
책에 나와있는 [코드 컨벤션](https://www.oracle.com/java/technologies/javase/codeconventions-statements.html)과 다른 내용이 있습니다. 당연히 팀마다 사람마다 다르겠지만 공통적으로 사용하는 룰을 알면 그래도 개발하는데 도움이 될 것 같다는 생각을 한다. 최근 3항 연산자에 대한 코드 컨벤션을 적용해보기도 했다.

# key point
- 무엇을 이야기하려고 하는가?
  - 코드의 형식은 코드를 읽는 사람에게 충분한 가이드를 만들 수 있다. 그러기에 코드 가독성을 올릴 수 있는 코드 형식을 만들고 적용하자.
- 가장 중요하다고 생각하는 것은 무엇인가?
  - 팀 규칙. 결국에는 여러 사람이서 같이 개발을 진행한다. 그러기에 혼자가 선호하는 방식이 아닌 룰을 잘 지키자!

## 궁금한 점
- 다들 팀 규칙을 가지고 있나요?
- 얼마나 컨벤션에 대해서 생각하고 조율하나요?

# 마무리
사실 많은 부분이 뻔한 내용 같지만 뻔한 내용을 명확하게 말해주는 것도 큰 도움이 된다고 생각한다. 어떤 정보와 생각을 인지하기 전에는 그것을 안다고 할 수 없다고 생각한다. 나는 오늘 뻔하지만 새로운 것을 알았다고 생각한다.