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
`git`을 이용해서 소스 코드를 관리하다보면, 그리고 협업을 진행하다보면 `conflict`가 나는 경우가 존재한다. 그것을 `merge`가 아닌 `rebase`를 통해서 어떻게 해결하는지 한번 알아보자!!

# 계기
프로젝트를 진행하면서 코드를 합치기 위해서서는 기존 코드에 대한 비교를 통해서 합치는 과정이 진행된다. 코드 비교 과정에서 문제가 발생하는 경우 `conflict`이 났다고 하면서 코드를 합치지 못하게 한다. 당연히 `conflict`이 나도록 서로 개발한 것은 문제가 될 수 있지만 그럴 수도 왜 자꾸 나를 괴롭히는 것인가!! 나는 그럴 때 마다 자주 `merge`를 통해서 해당 문제를 해결했다. 

## merge
많은 사람들이 이 방법을 알 것이라고 생각한다. 특정 `branch`에서 다른 `branch`에 있는 소스 코드를 가져와서 합치는 과정을 진행한다. 합쳐지는 방법은 `3-way-merge`을 통해서 진행되는 자세한 내용은 제가 소개하는 것보다 다른 훌륭한 블로그 글을 확인해보세요!!

`merge`을 통해서 진행하는 것은 정말로 간단하다. `merge`를 진행하게되면 그냥 소스 코드를 가져와서 합쳐지기 때문에 직접 `conflict` 난 부분을 해결하고 다시 `commit`하고 `push`하면 된다.
하지만 항상 아쉬운 것은 `merge`와 관련된 `commit`이 남는 것이다. 한개 `commit`이 `merge`되는 경우는 특별하게 남지 않지만 여러 개의 `commit`이 합쳐지는 경우 밑에 상황처럼 남게 된다.

![merge commit log](/assets/images/2020-11-15-git-rebase/git-rebase-merge-log-1.png)

# git rebase
기본적으로 `rebase`는 git에서 적용되는 `base`를 새롭게(`re`) 한다는 의미이다. 일단 `base`라는 친구는 현재 있는 `branch`가 나오기 있는 전에 존재했던 `commit`이라고 생각하면 편할 것 같습니다. 

## 동작 방식
(밑에 설명은 `main`에서 나온 브랜치에서 `main`으로 합쳐지는 과정이라고 설명하겠다.)

그러면 해당 `base`를 어떻게 다시 수정하는 것이냐? 어떻게 다시 set하는 것이냐? 이런 내용들은 다른 블로그에서 매우매우 그림과 함께 잘 설명을 해주시기 때문에 제가 자세하게는 설명하지 않겠습니다.(사실 저보다 더 설명을 잘하기 때문에...) 간단하게 말씀드리면 현재 `main` 브랜치에서 나온 브랜치가 지속적으로 기록한 `commit`에 대한 `diff`를 모두 현재 `rebase`하고 싶은 브랜치와 비교하게 됩니다. (그러는 과정에서 `conflict`이 날 수 있습니다. 당연히 그냥 합쳐질 수도 있습니다.)

이 과정을 중에 rebase는 기존에 존재하는 commit을 이용하는게 아니고 새로운 commit을 만들어서 지속적으로 `main` 브랜치 뒤에다가 붙이는 작업을 진행합니다. 모든 commit에 대해서 `rebase` 과정이 끝나게 되면 `main` 브랜치 뒤에는 `rebase merge`를 요청했던 브랜치에 `commit`이 모두 새로운 `commit`이 되어서 만들어져서합쳐집니다. 새로운 `commit` 이라는 것은 절대 내용이 새롭다는 것이 아닌 `commit hash`가 달라지기 때문에 새로운 `commit`이라고 말하는 것입니다. (실제 내부에 존재하는 `diff`는 동일합니다.)

이 과정을 통해서 외부에 나온 브랜치 commit의 내용을 main 브랜치로 합칠 수 있습니다.

## 주의할 점
여기서 주의할 것은 `rebase`를 진행하게 되면 새로운 `commit`을 만든다는 것입니다. 만약 `main`에서 나온 A라는 브랜치가 작업을 하고 있었습니다. 거기에서 `B`라는 브랜치가 `A`라는 커밋을 기반으로 새로운 브랜치를 만들었습니다. 만약 여기서 `A`라는 브랜치는 merge방식이 아닌 `rebase` 방식으로 하는 경우 B는 자신의 `base`가 (A가 가지고 있는 `commit`)이 `main`으로 합쳐지지 않고 그것으로 인해서 엄청난 충돌이 발생할 수 있습니다.

# rebase를 통한 conflict 해결 방법
우리가 위에서 설명한 내용은 `rebase`를 통해서 소스코드가 합쳐지는 것을 말씀 드렸습니다. 이젠 제가 말씀드리고 싶은 `우리의 코드와 main 브랜치(upstream)에 있는 코드와 싱크를 맞추는 작업`, 그리고 `그것을 통한 conflict 문제 해결`에 대해서 설명드리겠습니다!!

git에 대해서 간단하게 알고 있다는 전재로 말씀드리며 간단하게 말씀드리면 `conflict`이 나는 경우는 똑같은 파일을 합치려고 하는 브랜치와 합쳐지는 브랜치 모두 수정이 된 경우에 해당에 발생됩니다. 해당 부분에 대한 자세한 설명은 [3-way-merge](https://en.wikipedia.org/wiki/Merge_(version_control))을 한번 참고해주세요!

## 원리

`commit`은 기본적으로 `diff`를 저장하게 된다. `diff`의 결과를 모두 합쳐지게 되면 현재의 코드가 만들어지게 되는데 그 `commit`를 기반으로 브랜치를 따게 되면 그 시점이 `base`가 되고 그 `base`를 기준으로 우리는 소스 코드를 합치는 과정(merge 과정)에서 비교를 하게 된다. 위에서 언급한 `3-way-merge`에서는 `base + 합치려는 브랜치의 최신 버전 + 합쳐지는 브랜치의 위치`의 현재 상태를 비교하게 됩니다.

그러면 우리가 합치고 싶은 지점으로 `base`를 바꾸게 되면 자연스럽게 `conflict`은 나지 않을 것입니다. 왜냐하면 위에서 언급한 대로 `base + 현재 브랜치의 내용 + 합치려고 하는 브랜치의 내용` 이렇게 3개를 비교하게 되는데 `base`를 이동시키게 되면 `현재 브랜치의 내용 + base (= 합치려고 하는 브랜치의 내용)`가 될테니 `conflict`이 해결될 것입니다. (당연히 `rebase` 과정에서 `conflict`은 직접 해결해야 한다.)

`commit`은 `diff`의 저장이라고 말씀 드렸습니다. `rebase`를 통해서 base가 가지고 있는 commit 위에 `새로운 code (diff)`가 추가됩니다. (합치려고 하는 브랜치의 내용이 추가됨) 이 과정을 통해서 내가 합치려고 하는 부분에서 발생할 수 있는 파일에 대한 차이를 해결할 수 있습니다. (conflict = 파일에 대한 내용 수정 차이) 그리고 나의 브랜치의 `commit`이 올라가기 때문에 문제 없이 코드를 합칠 수 있는 조건이 되는 것입니다.

## 과정
실제 예시를 보여주면서 진행하겠습니다. 현재 제가 임의로 파일에 대한 수정하면서 `git`을 조금 꼬이게 만들었습니다. 그리고 `pull request`를 날리니 이런 모습으로 나오게 되는 군요.
![before github pr](/assets/images/2020-11-15-git-rebase/before-github-pr.png)

그러면 이젠 터미널을 켜서 한번 진행해 봅시다.
- `$ git fetch [합치기를 원하는 브랜치]`

upstream이 존재하는 경우 이렇게 하면 된다.
```bash
$ git fetch upstream/main
```

- `$ git rebase [합치기를 원하는 브랜치]` 

![rebase](/assets/images/2020-11-15-git-rebase/git-rebase-1.png)
```bash
$ git rebase main
$ git rebase upstream/main
```
이렇게 치게 되면 위에 있는 내용과 같이 어떤 부분에서 `conflict`이 났는지 나오게 됩니다. 우리는 그것을 해결해야만 합니다!! 반드시!! 

한 번 현재 어떤 친구가 문제가 있는지에 대해서 확인하는 명령어를 사용합니다.
``` bash
$ git status
```
![status](/assets/images/2020-11-15-git-rebase/git-rebase-git-status-1.png)

그리고 문제가 되는 파일을 수정해야겠죠?!
``` bash
$ vi README.md
```

밑에 그림에서 보면 `HEAD`라는 부분과 `branch-1` 이라는 내용을 보면서 어떤 코드가 어떤 브랜치에 존재했던 친구인지 확인할 수 있음.
![conflict-1](/assets/images/2020-11-15-git-rebase/git-rebase-conflict-1.png)

저는 그냥 둘 다 수정했던 부분을 모두 사용하기 위해서 `conflict` 난 부분을 이렇게 합쳤습니다.
![conflict-2](/assets/images/2020-11-15-git-rebase/git-rebase-conflict-2.png)

실제로 코드를 수정한 이후에 해당 파일을 `git add`를 사용합니다.
![conflict-resolve](/assets/images/2020-11-15-git-rebase/git-rebase-conflict-resolve-1.png)

- `conflict`을 해결한 이후에 `git rebase --continue` or `git rebase --skip`을 통한 rebase 과정 진행합니다.
지속적으로 발생하는 `conflict`을 해결하고 나게 되면 끝까지 rebase 과정이 진행되고 끝마치게 된다.

- 그 이후 `git push origin head --force` or `git push origin head --force-with-lease`을 통해서 강제로 push를 진행합니다.
여기서 강제로 넣는 이유는 위에서 언급했던 것처럼 `rebase`는 새로운 commit을 만들게 된다. 그러기 때문에 기존에 origin에 존재하는 commit과 다르게 됩니다. (당연히 rebase을 진행하는 중에 code 수정을 하지 않은 경우 내용을 완전히 동일합니다.) 그러기 때문에 그냥 하게 되는 경우 `pull`을 하라고 하게 되는게 그렇게 하면 완전히 다시 돌아가기 때문에 강제로 `push --force`를 사용해야 됩니다.

```
--force             -- allow refs that are not ancestors to be updated
--force-with-lease  -- allow refs that are not ancestors to be updated if current ref matches expected value
```

참고로 `--force`에는 2가지 옵션이 있는데 `--force-with-lease`를 사용하게 되는 약간 안전할 수 있는게 만약 새로운 commit이 존재하는 경우에 강제로 push 하는 것을 진행하지 않는다. 

![push](/assets/images/2020-11-15-git-rebase/git-rebase-push-force-1.png)

## 결과
![after github pr](/assets/images/2020-11-15-git-rebase/after-github-pr.png)
위에 사진과 같이 rebase를 진행하게 되면 자연스럽게 해당 문제가 사라지게 된다.

## 추가적으로
당연히 folk를 통해서 작업을 진행하고 `upstream`으로 데이터 싱크를 맞추는 경우도 똑같이 진행하면 된다. 저 또한 folk를 통해서 발생하는 conflict 문제를 이렇게 해결했습니다. 동일 레포지토리에서 작업하더라도 이 같은 방식으로 rebase하면 문제를 해결할 수 있습니다.

# 마무리
어떤 방식이 best practice인지 알기 어렵다. 어떤 경우에는 rebase가 좋고 어떤 경우는 merge가 좋고. 하지만 명확한 것은 독립적인 브랜치를 사용하는 경우에는 rebase에 대한 위험이 크지 않다. 하지만 만약 그렇지 못한 경우에서 rebase를 사용하는 경우 큰 문제를 야기할 수 있다는 것을 알아 두어야 한다.

# ref
[git rebase](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0)

[HomoEfficio님의 블로그](https://homoefficio.github.io/2017/04/16/Git-%EA%B3%BC%EA%B1%B0%EC%9D%98-%ED%8A%B9%EC%A0%95-%EC%BB%A4%EB%B0%8B-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0/)
