---
title: "Git rebase와 친해지기 (git conflict를 해결하는 방법 & upstream에서 rebase하기)"
description: "How to resolve git conflict by using git rebase"

categories:
  - dev
  - git
  - etc
tags:
  - dev
  - git
  - rebase
  - merge
  - etc
  - version control
  - vcs
---

# Overview
`git`을 이용한 코드를 합치는 방법은 여러가지가 있다. 이번 시간에는 `merge`를 이용한 방법이 아닌 `rebase`를 이용한 코드를 합치는 방법에 대해 알아보자!!

# 계기
프로젝트를 진행하면서 코드를 합치기 위해서서는 기존 코드에 대한 비교를 통해서 합치는 과정이 진행된다. 코드 비교 과정에서 문제가 발생하는 경우 `conflict`이 났다고 하면서 코드를 합치지 못하게 한다. 당연히 conflict이 나도록 서로 개발한 것은 문제가 될 수 있지만 그럴 수도 왜 자꾸 나를 괴롭히는 것인가!! 자주 `merge`를 통해서 해당 문제를 해결했다. 

## merge
많은 사람들이 이 방법을 알 것이라고 생각한다. 특정 `branch`에서 다른 `branch`에 있는 소스 코드를 가져와서 합치는 과정을 진행한다. 합쳐지는 방법은 `3-way-merge`을 통해서 진행되는 자세한 내용은 제가 소개하는 것보다 다른 훌륭한 블로그 글을 확인해보세요!!

`merge`을 통해서 진행하는 것은 정말로 간단하다. `merge`를 진행하게되면 그냥 소스 코드를 가져와서 합쳐지기 때문에 직접 `conflict` 난 부분을 해결하고 다시 `commit`하고 `push`하면 된다.
하지만 항상 아쉬운 것은 `merge`와 관련된 `commit`이 남는 것이다. 한개 commit이 merge되는 경우는 특별하게 남지 않지만 여러 개의 commit이 합쳐지는 경우 밑에 상황처럼 남게 된다.

![merge commit log](/assets/images/2020-11-15-git-rebase/git-rebase-merge-log-1.png)

# git rebase
기본적으로 `rebase`는 git에서 적용되는 `base`를 새롭게(`re`) 한다는 의미이다. 일단 `base`라는 친구는 현재 있는 `branch`가 나오기 있는 전에 존재했던 `commit`이라고 생각하면 편할 것 같습니다. 

## 동작 방식
(기본적으로 설명은 main에서 나온 브랜치에서 main으로 합쳐지는 과정이라고 설명하겠다.)

그러면 해당 `base`를 어떻게 다시 수정하는 것이냐? 이런 내용들은 다른 블로그에서 매우매우 그림과 함께 잘 설명을 해주시기 때문에 제가 자세하는 설명하지 않겠습니다. 간단하게 말씀드리면 현재 `main` 브랜치에서 나온 브랜치가 지속적으로 날린 `commit`에 대한 diff를 모두 현재 `rebase`하고 싶은 브랜치와 비교하게 됩니다. 그러는 과정에서 모든 commit을 비교하는 과정에서 `conflict`이 날 수 있습니다. 당연히 그냥 합쳐질 수도 있습니다.

이 과정을 중에 rebase는 기존에 존재하는 commit을 이용하는게 아니고 새로운 commit을 만들어서 지속적으로 main 뒤에다가 붙이는 작업을 진행한다. (자세한 설명은 훌륭한 블로그 글을 참고하세요... ㅠ.ㅠ) 그 결과는 main 뒤에는 합치고 싶어하던 내용들이 모두 새로운 commit이 만들어져서합쳐집니다. 새로운 commit 이라는 것은 절대 내용이 새롭다는 것이 아닌 commit hash가 달라지기 때문에 새로운 commit이라고 말하는 것입니다.

이 과정을 통해서 외부에 나온 브랜치 commit의 내용을 다른 브랜치로 합칠 수 있습니다. 

## 주의할 점
이 내용은 main 브랜치를 기반으로 만들어진 브랜치를 합칠 때 발생할 수 있는 상황을 묘사했습니다.

여기서 주의할 것은 rebase를 진행하게 되면 새로운 commit을 만든다는 것입니다. 만약 main에서 나온 A라는 브랜치가 작업을 하고 있었습니다. 거기에서 B라는 브랜치가 A라는 커밋을 기반으로 새로운 브랜치를 만들었습니다. 만약 여기서 A라는 브랜치는 merge방식이 아닌 rebase 방식으로 하는 경우 B는 자신의 base가 (A가 가지고 있는 commit)이 main으로 합쳐지지 않고 그것으로 인해서 엄청난 충돌이 발생할 수 있습니다.

# rebase를 통한 conflict 해결 방법
conflict이 나는 경우는 똑같은 파일을 합치려고 하는 브랜치와 합쳐지는 브랜치 모두 수정이 된 경우에 해당 됩니다. 해당 부분에 대한 자세한 설명은 [3-way-merge](https://en.wikipedia.org/wiki/Merge_(version_control))을 한번 참고해주세요!

## 원리

우리가 위에서 설명한 것과 다르게 우리가 하는 것은 main에서 우리의 코드를 받는 것이 아니라 우리의 코드와 main에 있는 코드를 싱크를 맞추는 작업니다.

commit은 기본적으로 diff를 저장하게 된다. diff의 결과를 모두 합쳐지게 되면 현재의 코드가 만들어지게 되는데 그 commit를 기반으로 브랜치를 따게 되면 그 시점이 base가 되고 그 base를 기준으로 우리는 소스 코드를 합치는 과정에서 비교를 하게 된다. 

그러면 우리가 합치고 싶은 지점으로 base를 바꾸게 되면 자연스럽게 conflict은 나지 않을 것이다. 왜냐하면 base + 현재 브랜치의 head + 합치려고 하는 브랜치의 head 이렇게 3개를 비교하게 되는데 base를 이동시키게 되면 현재 브랜치의 head + base (= 합치려고 하는 브랜치의 head)가 될테니 conflict이 해결된다. (당연히 rebase 과정에서 conflict은 직접 해결해야 한다.)

commit은 diff의 저장이라고 했는데 rebase를 통해서 현재 코드 위에 새로운 code (diff)가 추가되고 그것 위에 나의 브랜치의 commit이 올라가는 것이기 때문에 해결된다고 할 수 있다. 

## 과정
실제 예시를 보여주면서 진행하겠습니다. 현재 제가 임의로 git을 조금 꼬이게 만들었습니다. 그리고 `pull request`를 날리니 이런 모습으로 나오게 되는 군요.
![before github pr](/assets/images/2020-11-15-git-rebase/before-github-pr.png)

그러면 이젠 터미널을 켜서 한번 진행해 봅시다.
1. `$ git fetch [합치기를 원하는 브랜치]`

upstream이 존재하는 경우 이렇게 하면 된다.
```bash
$ git fetch upstream/main
```

2. `$ git rebase [합치기를 원하는 브랜치]` 

![rebase](/assets/images/2020-11-15-git-rebase/git-rebase-1.png)
```bash
$ git rebase upstream/main
```
이렇게 치게 되면 밑에와 같이 어떤 부분에서 conflict이 났는지 나오게 됩니다. 우리는 그것을 해결해야만 합니다!! 반드시!! 
그리고 `git add [해당 파일 명]` or `git add .` (여기서 `.`은 모든 path 기준으로 밑에 있는 모든 파일을 추가하겠다는 것입니다.)을 진행하고 다시 새로운 commit을 만듭니다. (`git commit -m "Resolve commit"`)

![status](/assets/images/2020-11-15-git-rebase/git-rebase-git-status-1.png)

![conflict-1](/assets/images/2020-11-15-git-rebase/git-rebase-conflict-1.png)

![conflict-2](/assets/images/2020-11-15-git-rebase/git-rebase-conflict-2.png)

![conflict-resolve](/assets/images/2020-11-15-git-rebase/git-rebase-conflict-resolve-1.png)

3. `git rebase --continue` or `git rebase --skip`을 통한 rebase 과정 진행합니다.
지속적으로 발생하는 `conflict`을 해결하고 나게 되면 끝까지 rebase 과정이 진행되고 끝마치게 된다.

4. 그 이후 `git push origin head --force` or `git push origin head --force-with-lease`을 통해서 강제로 push를 진행합니다.
여기서 강제로 넣는 이유는 위에서 언급했던 것처럼 `rebase`는 새로운 commit을 만들게 된다. 그러기 때문에 기존에 origin에 존재하는 commit과 다르게 됩니다. (당연히 rebase을 진행하는 중에 code 수정을 하지 않은 경우 내용을 완전히 동일합니다.) 그러기 때문에 그냥 하게 되는 경우 `pull`을 하라고 하게 되는게 그렇게 하면 완전히 다시 돌아가기 때문에 강제로 `push --force`를 사용해야 됩니다.

```
--force             -- allow refs that are not ancestors to be updated
--force-with-lease  -- allow refs that are not ancestors to be updated if current ref matches expected value
```

참고로 `--force`에는 2가지 옵션이 있는데 `--force-with-lease`를 사용하게 되는 약간 안전할 수 있는게 만약 새로운 commit이 존재하는 경우에 강제로 push 하는 것을 진행하지 않는다. 

![push](/assets/images/2020-11-15-git-rebase/git-rebase-push-force-1.png)

## 결과
![after github pr](/assets/images/2020-11-15-git-rebase/before-github-pr.png)
위에 사진과 같이 rebase를 진행하게 되면 자연스럽게 해당 문제가 사라지게 된다.

## 추가적으로
당연히 folk를 통해서 작업을 진행하고 `upstream`으로 데이터 싱크를 맞추는 경우도 똑같이 진행하면 된다. 저 또한 folk를 통해서 발생하는 conflict 문제를 이렇게 해결했습니다. 동일 레포지토리에서 작업하더라도 이 같은 방식으로 rebase하면 문제를 해결할 수 있습니다.

# 마무리
어떤 방식이 best practice인지 알기 어렵다. 어떤 경우에는 rebase가 좋고 어떤 경우는 merge가 좋고. 하지만 명확한 것은 독립적인 브랜치를 사용하는 경우에는 rebase에 대한 위험이 크지 않다. 하지만 만약 그렇지 못한 경우에서 rebase를 사용하는 경우 큰 문제를 야기할 수 있다는 것을 알아 두어야 한다.

# ref
[git rebase](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0)

[HomoEfficio님의 블로그](https://homoefficio.github.io/2017/04/16/Git-%EA%B3%BC%EA%B1%B0%EC%9D%98-%ED%8A%B9%EC%A0%95-%EC%BB%A4%EB%B0%8B-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0/)
