---
layout: post
author: Dboo
categories: 인프런-스프링기반_REST_API_개발-백기선
title: CH 1. REST API 및 프로젝트 소개
tags: LectureNote REST API
---

## REST API 및 프로젝트 소개

- REST 란? 이라는 질문을 굉장히 어디서 많이 듣기도 하고 보기도 한다.
  - 사실 해당 질문을 받았을 때, 굉장히 어물쩡하게 넘어간 적이 많다. 그냥 위키에 나와있는대로 아키텍쳐 스타일이라는 정도로만 이해하고 있었기 때문에
    해당 질문에 대한 개념적인거나 REST의 존재 의의나 목적에 대한 이해는 하지 못했기 때문이다.
  - 백기선님이 강의내에서 언급하고 정리해서 알려주시긴 하지만, 직접 해당 스피치의 링크르 따라 들어가 내용을 정리해 보았다. 
  - [DEVIEW 스피치](https://youtu.be/RP_f5dMoHFc)
  - [정리글](https://dboostudio.github.io/%ED%8A%B9%EA%B0%95%ED%95%84%EA%B8%B0/2022/03/18/seminar_REST_deview.html)


### 1. Project 생성

- 프로젝트 생성시에, Self-descriptive Message를 달성하기 위한 Spring REST Docs 의존성과 HATEOAS를 달성하기 위한 Spring HATEOAS
의존성을 추가해 주었다.
- HTTP 통신 테스트를 위해 Postman 혹은 Chrome 플러그인인 Tanlend API를 사용한다.

### 2. Domain 생성하기


#### Event 도메인

Event라는 도메인, EventStatus라는 Enum을 생성했다.

~~~java
public enum EventStatus {

    DRAFT, PUBLISHED, BEGAN_ENROLLMENT;

}
~~~

~~~java
@Entity
@Getter @Setter @Builder @AllArgsConstructor @NoArgsConstructor
@EqualsAndHashCode(of = "id")
public class Event {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
    private LocalDateTime beginEnrollmentDateTime;
    private LocalDateTime closeEnrollmentDateTime;
    private LocalDateTime beginEventDateTime;
    private LocalDateTime endEventDateTime;
    private String location; // (optional) 이게 없으면 온라인 모임
    private int basePrice; // (optional)
    private int maxPrice; // (optional)
    private int limitOfEnrollment;

    private boolean offline;
    private boolean free;
    
    @Enumerated(EnumType.STRING)
    private EventStatus eventStatus = EventStatus.DRAFT;
}

~~~

- `@EqualsAndHashCode`에서 of를 사용하는 이유
  - @EqualsAndHashCode가 모든 필드를 기본적으로 참조하는데, 엔티티 연관관계가 묶여있는 필드가 들어가게 되면 스택오버플로우 발생한다.
- `@Data`를 사용하지 않는 이유
  - @EqualsAndHashCode가 기본적으로 적용되기 때문
- Lombok은 메타 어노테이션으로 동작하지 않아서 커스텀으로 줄일 수 없다.

#### 테스트 코드

~~~java

@Test
public void builder() throws Exception {
    Event event = Event.builder()
            .name("inflearn")
            .description("REST API")
            .build();

    assertThat(event).isNotNull();
}

@Test
public void javaBeanRule() throws Exception {
    Event event = new Event();
    event.setName("dboo");
    assertNotNull(event);
    assertThat(event.getName()).isEqualTo("dboo");
}

~~~

assertJ에서 제공하는 asssertThat을 이용했다. 테스트 내용은 객체생성이 잘 되는지, Setter,Getter가 잘 동작하는지 이다.


### 3. 이벤트 비즈니스 로직

- basePrice와 maxPrice에 대한 조건은 아래와 같다.

|-|-|-|
|basePrice|maxPrice|상황|
|-|-|-|
|0|0|무료 이벤트|
|0|100|선착순 등록|
|100|0|무제한 경매|
|100|200|제한가 선착순 등록|

