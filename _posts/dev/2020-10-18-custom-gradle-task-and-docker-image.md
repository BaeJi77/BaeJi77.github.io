---
title:  "Java Gradle custom task로 docker image 만들기 & 저장소에 올리기 (feat. harbor)"

categories:
  - spring
  - dev
tags:
  - dev
  - spring
  - java
  - gradle
  - docker
  - harbor
  - container
---

# Gradle task
task는 gradle project의 작업 단위.
기본적으로 gradle에서는 제공해주는 task들이 있음. 
![사진1](/assets/images/custom-gradle-task-and-docker-image-1.png)
이런 친구들을 `./gradlew [taskName]` 을 활용해서 실행할 수 있다.
대표적으로 `./bradlew bootJar` 를 실행하는 경우 `.jar` 파일이 만들어지고 그것을 활용하는 경우가 많다. (이 jar 파일을 container image로 만들어서 사용할 수 있다.)

# 기본적인 명령어
## task 선언
``` groovy
task TASK_NAME() {
    SOME_CODE
}
```

실행은 밑에 터미널 창에서 gradle과 함께 task 이름을 실행시키면 된다.
``` bash
$ ./gradlew TASK_NAME
```

## 코드 상에 우선순위
``` groovy
task TASK_NAME() {
    doFirst {
      // doLast{} 보다 먼저 실행될 친구
    }

    doList {
      // doFirst{} 보다 니증에 실행될 친구
    }
}

--- 밑에도 가능함

task TASK_NAME {
    doFirst {
      // doLast{} 보다 먼저 실행될 친구
    }
}

TASK_NAME.doLast {
  // 요렇게도 가능합니다.
}
```

## 외부 파라미터를 활용한 실행
-P를 이용하는 경우 project에 대한 파라미터 변수를 넣을 수 있다.
-D를 사용하는 경우 System에 해당하는 변수를 넣을 수 있다.

``` groovy
task TASK_NAME(){
    doLast{
        if (project.hasProperty("args")) {
            println "Project property ${project.getProperty("args")}"
        }
        println "System property ${System.getProperty("args")}"
    }
}
```

실행 및 결과
``` bash
$ gradlew TASK_NAME -Pargs=hello -Dargs=world

--- result ---
Project property hello
System property world
```

## 해당 테스크에 의존성 추가
``` groovy
task TASK_NAME(){
    dependsOn OTHER_TASK_NAME

}
```
의존성이 있는 경우에는 `doFrist`와 `doLast`에 대한 코드가 실행되기 전에 실행됩니다. 강제로 순서를 정하는 방법은 잘 모르겠군요.


## 변수 선언
`group`과 `description`를 통해서 그룹핑과 해당 task에 대한 description을 할 수 있고  `gradle task`를 보는데 편함을 제공해줄 수 있다.
``` groovy
task TASK_NAME() {
    group = '태스크의 그룹'
    description = '태스크 설명'
}

// 전역 동적변수 선언
ext {
    buildVersion = project.hasProperty("buildVersion") ? project.getProperty("buildVersion") : ${proejct.version}
    containerName = "baeji77/blog"
}

task TASK_NAME(){
    ext.name = "baeji"
    ext.job = "developer"
}

task OTHER_TASK_NAME() { // 요렇게 다른 task에서 변수로 사용 가능
    println tasks.TASK_NAME.name // print baeji
    println tasks.TASK_NAME.job  // print developer
}
```

동적변수이다 보니 해당 데이터를 외부 파라미터로 넘어온 값으로 설정할 수도 있다.
위에 보이는 `buildVersion` 이라는 것을 활용해서 만약 input이 있는 경우는 그것에 대한 것을 사용하고 그렇지 않은 경우 project에 존재하는 version값을 사용하도록 했다.

# Docker image 만들기
이 방법은 `gradle bootBuildImage` task를 이용한 방법입니다. 해당 글과 관련된 저의 글을 참고하세요. [참고](https://baeji77.github.io/java/spring/dev/Spring-boot-gradle-build/)

일단 bootBuildImage는 이미 존재하고 있는 task이며 존재하는 task도 커스텀 할 수 있다.
``` groovy
bootBuildImage {
    imageName = "spring-boot-dockerized:${project.version}"
}

task buildDockerImage {
    dependsOn 'bootBuildImage'

    println "projectVersion = ${project.version}"
}
```
![사진2](/assets/images/custom-gradle-task-and-docker-image-2.png)

# harbor에 이미지 올리기
저희 팀에서는 harbor을 이용한 docker registry를 가지고 있기 때문에 harbor에 대한 방법을 공유드리겠습니다. (다른 방법도 비슷할 것이라고 생각합니다.)

**기본적으로 제가 레퍼런스한 harbor 관련 글에 존재하는 로그인이 끝난 이후라고 가정하고 진행하겠습니다.**

## gradle.build
``` groovy
ext {
    buildVersion = project.hasProperty("buildVersion") ? project.getProperty("buildVersion") : "${project.version}"
    containerProjectName = 'pring-boot-dockerized'
    dockerImageName = "${containerProjectName}:${buildVersion}"
    harborRepository = "HARBOR_URL/${dockerImageName}"
}

task buildHarborImage {
    dependsOn 'bootBuildImage'

    println("buildVersion = ${buildVersion}")
    println("dockerImageName = ${dockerImageName}")
    println("harborRepository = ${harborRepository}")

    doLast {
        exec {
            commandLine "docker", "tag", "${dockerImageName}", "${harborRepository}"
        }
        exec {
            commandLine "docker", "push", "${harborRepository}"
        }
        println("harbor push done!")
    }

}
```

## 실행 커맨드
``` bash
$ ./gradlew buildHarborImage -PbuildVersion=0.0.1
```


# ref
[중년개발자님의 블로그](https://blog.naver.com/PostView.nhn?blogId=sharplee7&logNo=221413629068)

[권남님 블로그](https://kwonnam.pe.kr/wiki/gradle/task)

[Gradle parameter와 관련된 글](https://www.baeldung.com/gradle-command-line-arguments)

[harbor reference](https://goharbor.io/docs/1.10/working-with-projects/working-with-images/pulling-pushing-images/)

[gradle properties](https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_system_properties)
