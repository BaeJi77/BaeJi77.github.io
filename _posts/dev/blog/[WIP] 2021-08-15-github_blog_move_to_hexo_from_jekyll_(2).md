---
title: "github hexo blog framework를 이용해서 꾸며보기 + icarus theme 적용해보기 (2)"
description: "How to move jekyll theme blog to hexo blog theme about github blog"

categories:
  - dev
  - blog
tags:
  - dev
  - blog
---

이전 글을 이어서 작성하며 완벽하게 migration하는 과정에 대해서 이야기해보겠습니다.

# Migration Post

migration이 이루어진 이후에 모든 링크가 깨지면 안된다. 이전에 똑같은 링크로 들어왔을 때 똑같은 결과를 보여주어야 한다.

하지만 기본 link를 만드는 방식이 조금 다르다. 

``` 
$ hexo
permalink: :year/:month/:day/:title/
=> http://localhost:4000/2021/09/12/hello-world/

$ jekyll
=> http://localhost:4000/hello-world/
```

위에 보이는 예와 같이 hexo에 설정대로라면 글을 작성한 날짜로 링크가 만들어진다. 그러기 때문에 우리가 기대했던대로 똑같은 링크가 만들어지지 않는다.

그래서 나는 해당 설정을 이렇게 바꿨다.

``` 
$ hexo
permalink: :title/
=> http://localhost:4000/hello-world/
```



# ref

[hexo.io](https://hexo.io/)

[Getting Started with Icarus](https://ppoffice.github.io/hexo-theme-icarus/uncategorized/getting-started-with-icarus)

[icarus github](https://github.com/ppoffice/hexo-theme-icarus)

