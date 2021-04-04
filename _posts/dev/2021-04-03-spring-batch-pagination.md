---
title: "spring batch: Reader에서 pagination 마음대로 사용해보기"
description: "How to set custom pagination about spring batch"

categories:
  - dev
  - spring
tags:
  - dev
  - spring batch
---

> 코드와 관련해서는 [github](https://github.com/BaeJi77/blog-code/tree/main/2021-04/spring-batch-pagination)를 참고하세요.

# Batch
**주기적**으로 **특정 실행(Job)**을 해야되는 경우가 있다. 예를 들면 하루에 모든 데이터를 다른 곳에 저장을 한다던지 특정 데이터들을 정제해서 다른 곳으로 보낸다는 작업을 해야되는 경우가 있다.

이런 작업을 `Batch`을 라고 부른다. 위키를 보니 이렇게 설명되어 있다.

> 일괄 처리(batch processing)란 최종 사용자의 개입 없이 또는 (자원이 허가한다면) 실행을 스케줄링할 수 있는 작업(job)의 실행을 의미한다.[1] 컴퓨터 프로그램 흐름에 따라 순차적으로 자료를 처리하는 방식이다.

# Spring Batch
Spring에서도 Batch 작업을 할 수 있는 기능을 제공해준다. Spring batch라는 친구가 바로 그 친구이다. 

Spring batch와 관련해서는 내가 잘 설명할 자신이 없다. 간단하게 검색해보면 간단한 사용법은 활용과 관련해서 바로 찾아서 볼 수 있다. 

# 시작
내가 Batch를 만들어서 활용하는데 Reader 역할을 하려고 했던 source들은 api를 요청해서 가져와야했었다. 그런데 Spring batch 대부분의 sql에 대해서 약간 특화되어 있다는 생각이 들었다. 그러다보니 api를 통해서 가져오는 것에 대해서 잘 구현되어있지 않았다. 아! json 데이터를 읽을 수 있는 방법이 있어서 그것을 활용할 수는 있다. 

하지만 나는 Pagination되어진 http api를 통해서 데이터를 모으는 작업을 하고 있었다.

## 여기서 질문
> Spring batch를 왜 사용했나요? 

꼭 Spring batch를 써야되는 것은 아니었지만 그래도 
1. 어느정도 자동화해주는 부분이 있었고 batch 시스템을 제공해준다는 것이 있었기에 선택하게 되었다. 
2. 기존에 서버에 붙어 있던 컴포넌트였고 자바로 되어 있었기 때문에 migration도 편하다는 장점이 있었다. 

> pagination은 그냥 api 요청한 데이터 가져와서 모아서 처리하면 되지 않나요?

Spring batch에서는 `chuck`라는 단위가 있다. 그 단위는 데이터를 처리하는 트랙잭션 단위라고 생각하며 되며 만약 실패하게 되면 그 단위로 데이터를 또 가져와서 처리할 수 있기 때문에 api 요청에 대한 페이지 크기만큼 하면 장점이 있다고 생각했다.

# [AbstractPagingItemReader](https://docs.spring.io/spring-batch/docs/current/api/org/springframework/batch/item/database/AbstractPagingItemReader.html)

Paging을 간편하게 만들어주는 추상클래스. Jpa를 이용하거나 다른 db를 접속해서 chuck 단위로 처리할 때 이친구를 사용하게 된다. 

해당 클래스의 핵심은 데이터를 페이지 단위로 읽어서 지속적으로 리턴을 로직이 짜여져 있는 추상 클래스이다.

## 추상 클래스의 핵심 코드
``` java
protected T doRead() throws Exception {

    synchronized (lock) {

        if (results == null || current >= pageSize) {

            if (logger.isDebugEnabled()) {
                logger.debug("Reading page " + getPage());
            }

            doReadPage();
            page++;
            if (current >= pageSize) {
                current = 0;
            }

        }

        int next = current++;
        if (next < results.size()) {
            return results.get(next);
        }
        else {
            return null;
        }

    }

}
```
Page 데이터가 끝에 갈때까지 지속적으로 읽는다. 실제 페이지에서 데이터를 읽어보는 `doReadPage()`를 구현하면 된다.

## 추상 클래스 구현
``` java
public class PaginationReader extends AbstractPagingItemReader<People> {

    private static final ObjectMapper MAPPER = new ObjectMapper();

    public PaginationReader(int pageSize) {
        setPageSize(pageSize);
    }

    int index = 0;

    @Override
    protected void doReadPage() {
        if (results == null) {
            results = new ArrayList<>();
        } else {
            results.clear();
        }

        if (index == 2) {
            return;
        }

        try {
            ClassPathResource classPathResource = new ClassPathResource(String.format("test_%d.json", index++));
            System.out.println(classPathResource.getFile().exists());

            BufferedInputStream inputStream = new BufferedInputStream(classPathResource.getInputStream());

            List<People> peopleList = MAPPER.readValue(inputStream, new TypeReference<List<People>>() {});
            results.addAll(peopleList);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void doJumpToPage(int itemIndex) {

    }
}
```
- `results` 라는 List객체를 만들어줘야 한다. 해당 데이터에 지속적으로 데이터가 들어가고 그것을 계속 읽기 때문이다. 그리고 clear를 해주지 않으면 해당 이전 데이터를 읽지 못하게 된다. 그뿐만 아니라 밑에와 같은 무한루프에 콘솔 메시지를 볼 수 있기 때문에 꼭 해주기를 바란다.
- 그 외에 코드들은 파일에 있는 데이터들을 지속적으로 읽도록 하였고 `results`라는 list 하게 만들었다.

## Chuck 동작
기본적으로 어떻게 동작하는지 알기 위해서 writer을 아주 간단하게 만들었다. (+ step set-up)
``` java
@Bean
    public Step paginationStep() {
        return stepBuilderFactory.get("paginationStep")
                .<People, People>chunk(CHUCK_SIZE)
                .reader(new PaginationReader(PAGE_SIZE))
                .writer(printWrite())
                .build();
    }

    @Bean
    public ItemWriter<People> printWrite() {
        return list -> {
            log.info("writer={}", list);
        };
    }
```

> 나는 총 3개의 데이터 2개의 파일에 저장했다. 총 데이터 6개가 되는 것이다.

### page = 3, chuck = 2
``` bash
===시작===
Read page. page number=0
writer=[People(number=1, name=hoon, job=developer), People(number=2, name=baeji, job=developer)]
Read page. page number=1
writer=[People(number=3, name=얄리얄리, job=developer), People(number=4, name=hoon, job=developer)]
writer=[People(number=5, name=baeji, job=developer), People(number=6, name=얄리얄리, job=developer)]
===끝===
```
총 2번의 읽었고 실제로 writer을 통해서 데이터 3번 보여지는 것을 알 수 있다.

### page = 3, chcuk = 3
``` bash
===시작===
Read page. page number=0
writer=[People(number=1, name=hoon, job=developer), People(number=2, name=baeji, job=developer), People(number=3, name=얄리얄리, job=developer)]
Read page. page number=1
writer=[People(number=4, name=hoon, job=developer), People(number=5, name=baeji, job=developer), People(number=6, name=얄리얄리, job=developer)]
===끝===
```

# 느낀점
생각보다 DB를 제외한 작업에 대해서 친절한 편은 아니었다고 생각한다. DB에 대해서는 편한 부분이 있다고 생각하지만 다른 자원을 reader로 활용해서 한다면 다른 것도 한번 더 생각해보긴 해야될 것 같다.

# ref
[우아한 형제 개발 블로그](https://woowabros.github.io/experience/2020/02/05/springbatch-querydsl.html)
