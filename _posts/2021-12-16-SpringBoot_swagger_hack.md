---
layout: post
author: Dboo
categories: 삽질은_아안돼
title: SpringBoot 2.6.x 버전에서 Springfox 3.0.0(Swagger, AOP) 강제적용하기
tags: SpringBoot Swagger
---

일시 2021/12/17 인 현재 시점에서 스프링부트 2.6.x 버전에는 스프링폭스 3.0.0을 적용이 안된다.
의존성에 둘을 추가하면, 다음과 같은 에러를 뱉는다.

~~~
org.springframework.context.ApplicationContextException: Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException: Cannot invoke "org.springframework.web.servlet.mvc.condition.PatternsRequestCondition.getPatterns()" because "this.condition" is null
    at org.springframework.context.support.DefaultLifecycleProcessor.doStart(DefaultLifecycleProcessor.java:181)
    at org.springframework.context.support.DefaultLifecycleProcessor.access$200(DefaultLifecycleProcessor.java:54)
    at org.springframework.context.support.DefaultLifecycleProcessor$LifecycleGroup.start(DefaultLifecycleProcessor.java:356)
    at java.base/java.lang.Iterable.forEach(Iterable.java:75)]
    ...(중략)
~~~

처음에는 스웨거 설정을 잘못 한 줄 알고 한참을 다시 얼마 고칠것도 없는 스웨거 설정을 변경해보면서 왜안되지?
의 늪에 빠져있다가, 이미 스웨거가 잘 설정되어 있는 프로젝트랑 비교를 하니 스웨거 설정의 문제가 아니고
다른건 스프링부트 버전밖에 없다는 생각이 들었다. 스프링부트 버전을 2.5.0 으로 낮추니 스웨거가 동작하였다.

그래서 혹시 2.6.X 버전 스프링부트에 적용할 수 있는지 찾아보았는데 스프링폭스 깃허프 저장소에 이슈로 등록된
글이 있었다.

https://github.com/springfox/springfox/issues/3462 (게시글)  
https://github.com/springfox/springfox/issues/3462#issuecomment-978707909 (해결책 답글)

해당 게시글을 읽다보니 누군가 `강제로` 스프링 부트가 스웨거를 인식하는 방법이 있다고해서 따라해보았는데
그 결과가 성공적이어서 공유한다.

1. spring.mvc.pathmatch.matching-strategy 옵션을 ant-path-matcher 로 변경한다.
  (application.properties에서 변경하면 된다.)
2. WebMvcRequestsHandlerProvider 클래스 파일을 찾아서 통째로 복사해서 내 소스 안에 붙여넣는다.

-> 해당 파일을 복사할 때 주의할 점은 패키지명이 그대로 살아있어야 한다는 점이다. 복사해서 넣으면 보통
IDE들이 해당경로를 패키지명으로 넣어줄 텐데 그 패키지명을 지우고 원래의 패키지명을 유지하도록 해주면 된다.

끝~
