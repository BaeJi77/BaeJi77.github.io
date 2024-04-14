---
title: "나의 작고 소중한 블로그 향상시키기 프로젝트 (1) - SEO"
description: ""

categories:
  - dev
  - blog
  - SEO
tags:
  - dev
  - blog
  - SEO
---

# 배경

저는 블로그를 작성만 했지 실질적으로 누구에게 많이 홍보되는 것에 대해서는 크게 생각하지는 않았습니다. 하지만 현재 블로그 글쓰는 활동인 [글또](https://geultto.github.io/docs/intro)에서 진행한 컨퍼런스(글또내에서는 반상회라고 불림)에서 SEO와 관련한 내용을 들었습니다.

생각은 해봤지만 실질적으로 조사하거나 적용해볼 생각을 해보지 않았지만 짧은 시간에 정말 다양하고 실질적으로 도움이 될만한 내용들이 많아서 이번에 한번 적용해보면서 직접 도움이 될만한 것들을 공유해보고 싶습니다.

해당 내용은 [재그재그의 SEO 글](https://wormwlrm.github.io/2023/05/07/SEO-for-Technical-Blog.html)를 바탕으로 글을 많이 요약해서 작성할 예정입니다. 핵심적으로 현재 제가 사용하고 있는 `jekyll`에서 할수 있는 것과 제가 적용했던 것에 대해서 이야기해보려고 합니다.

저는 SEO에 대한 간단한 설명만 할뿐이며 더욱 구체적인 내용이 궁금하다면 `재그재그님의 블로그` 혹은 `구글에서 제공해주는 공식 가이드`를 참고해보세요.

# SEO란

![seo 정의 위키]()

SEO란 `Search Engine Optimization`라는 뜻이며 간단하게 말하면 검색 엔진에 어떻게 더 잘 선택받을 수 있는지에 대한 것입니다. 다른 검색엔진도 있겠지만 가장 많이 사용되는 검색 엔진이 구글에서 검색이 잘되는 방법에 대한 이야기를 하는것입니다.

해당 내용은 구글에서도 공식적으로 안내를 하고 있습니다. [검색엔진 최적화(SEO) 기본 가이드 by Google](https://developers.google.com/search/docs/fundamentals/seo-starter-guide?hl=ko)

# SEO를 위한 방법

On-Page, Techincal, Off-page 이렇게 3가지 접근 방법이 있다고 합니다. 

## On-Page SEO: Page 내부적으로. 컨텐츠에 대한 방법

- 양질의 콘텐츠 제작

간단하게 좋은 글을 쓰면 좋은 SEO 전략이 될 수 있습니다. 그 기준은 여러가지 내용들이 있겠지만 구글 공식에 존재하는 내용을 가져와보겠습니다. 

![구글 seo 컨텐츠 관련]()

- 태그 이용

html에는 여러 태그들을 이용해서 해당 사이트 혹은 해당 링크에 대한 내용을 설명해줄 수 있습니다. 그 중에 밑에 있는 태그들을 이용하면 SEO에 도움이 된다고 합니다.

- title tag: html `<title>`
- meta tag: html `<meta>` 
- og(Open Graph) tag: html의 meta 태그에 property중 하나 `<meta property="og:title", ... \>`
- semantic tag: 해당 태그는 특정 태그를 말한다기보다 구조적인 태그 구성을 통해서 검색 엔진이 잘 이해할 수 있도록 해야된다는 내용입니다.

## Technical SEO: 색인 과정에서 SEO가 잘될 수 있도록 하는 방법

- sitemap: 웹 사이트 구조를 설명해주는 메타데이터 입니다. 크롤링할 때 웹 사이트를 잘 이해할 수 있도록 합니다.
- robots.txt: 원래는 크롤링에 대한 제한 및 허용을 위해서 사용되는 파일인데 해당 파일로 크롤러가 더 잘 웹 사이트를 이해할 수 있도록 한다고 합니다.
- Lighthouse 점수: `Lighthouse`는 구글에서 제공해주는 웹 사이트 성능 측정 도구입니다. 높은 점수 일수록 더욱 SEO가 잘된다고 합니다.
- 구글 서치 콘솔 등록: 구글 검색과 관련해서 어떤 경로를 통해서 접근했는지에 대한 인사이트를 제공해줍니다. 내가 등록한 사이트에 대한 특정 url에 대한 색인 결과도 볼 수 있습니다.
- 네이버 서치 어드바이저 등록: 많은 개발자분들이 구글 검색을 통해서 결과를 얻지만 네이버를 통해서도 검색할수 있습니다. 구글 서치 콘설처럼 네이버에도 비슷한 서비스가 있습니다.

## Off-Page SEO: 글과 색인 과정 그 이후에 SEO를 올리는 방법

간단하게 말한다면 사이트에 대한 평판과 관련된 내용입니다. 

- 다양한 플랫폼에서의 백링크 확보: 백링크란 다른 사이트에서 제 사이트로 연결되는 링크를 말합니다. 한마디로 여러 사이트에서 나의 사이트에 대한 정보가 많다면 크롤러가 높은 신뢰를 해준다는 것입니다.
- 전략적인 포스트 공유: 재그지그님 블로그에도 있지만 저의 개발 블로그도 평일에는 트래픽이 나오다가 주말이되면 트래픽이 확 줄어드는 패턴이 있습니다. 그런 패턴을 이용해서 최대한 사용자들이 많이 볼 수 있는 시간에 글을 포스터하는 것도 하나의 전략일 수 있다는 것입니다.

# SEO에 대한 요약

SEO는 검색 엔진이 나의 글을 더 위에 보여줄 수 있도록 하는 최적화 과정입니다. 내가 좋은 글을 쓰는것도 매우 중요하지만 그것을 다양한 방법으로 그리고 간단한 설정들로도 충분히 좋은 SEO를 만들 수 있다는 것을 알게되었습니다.

그러면 지금부터는 내가 할 수 있는 간단한 SEO 설정들에 대해서 알아보겠습니다.

# 간단하게 SEO를 향상 시킬 수 있는 방법들

- 실습 환경

현재 저는 `github blog`를 이용하고 있으며 `jekyll` 라는 플랫폼을 이용해서 블로그를 운영하고 있습니다. 제가 소개하는 것들은 `jekyll`에는 다양한 플러그인을 활용하는 방법이지만 다른 블로그 플랫폼뿐만 아니라 naver, tistory와 같이도 제공받을 수 있는 것들이 있다고 알고 있습니다.

## [jekyll-sitemap](https://github.com/jekyll/jekyll-sitemap)

밑에 있는 내용은 실제 해당 github readme에 존재하는 내용을 가져왔습니다. `Gemfile`을 업데이트하고 `_config.yml`하는 방법입니다.

1. Add gem 'jekyll-sitemap' to your site's Gemfile and run bundle
2. Add the following to your site's _config.yml:

```md
url: "https://example.com" # the base hostname & protocol for your site
plugins:
  - jekyll-sitemap
```

## [jekyll-seo-tag](https://github.com/jekyll/jekyll-seo-tag)

`jekyll-seo-tag`는 SEO를 높일 수 있는 방법들을 자동으로 만들어주고 처리해줍니다. 위에서 언급했던 og tag아 관련된 내용들고 여러 기법들을 자동으로 해주는것 설명하고 있습니다.

밑에 있는 내용은 해당 플러그인이 어떤 것들을 해주는지에 대한 내용을 readme에서 가져와봤습니다.

```
Jekyll SEO Tag adds the following meta tags to your site:
- Page title, with site title or description appended
- Page description
- Canonical URL
- Next and previous URLs on paginated pages
- JSON-LD Site and post metadata for richer indexing
- Open Graph title, description, site title, and URL (for Facebook, LinkedIn, etc.)
- Twitter Summary Card metadata
```

## Google Search Console

1. [google search console](https://search.google.com/search-console/about)에 들어가게 되면 도메인을 등록하는 방법이 있습니다.
2. github blog를 사용하고 있는 경우 해당 [링크](https://docs.github.com/ko/enterprise-cloud@latest/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)를 확인하고 알려주는대로 진행하면 됩니다. dns가 업데이트되는데 시간이 걸릴 수도 있습니다.
3. 제대로 search console에 등록이 완료되었다면 sitemap을 추가하시면 됩니다. ![sitemap 등록]()
4. 설정에 들어가서 `robots.txt`를 등록합니다. ![robots.txt 등록]()

추가적으로 원한다면 [google analytics](https://analytics.google.com/analytics)에 등록도 할 수 있습니다. 등록하게 되면 조금 더 구체적인 유저 접근 및 유저 접근 시간 또한 구체적으로 알수도 있습니다.

## Naver search advisor




