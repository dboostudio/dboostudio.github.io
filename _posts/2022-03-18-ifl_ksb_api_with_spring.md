
---
layout: post
author: Dboo
categories: 특강필기
title: 그런 REST API로 괜찮은가?
tags: LectureNote REST API
---

# 스프링 기반 REST API 개발

스프링 기반의 REST API를 TDD개발방식을 활용하여 만들어보는 백기선님의 강의를 들으며 요약한다.

## REST API 및 프로젝트 소개

- REST 란? 이라는 질문을 굉장히 어디서 많이 듣기도 하고 보기도 한다.
  - 사실 해당 질문을 받았을 때, 굉장히 어물쩡하게 넘어간 적이 많다. 그냥 위키에 나와있는대로 아키텍쳐 스타일이라는 정도로만 이해하고 있었기 때문에
    해당 질문에 대한 개념적인거나 REST의 존재 의의나 목적에 대한 이해는 하지 못했기 때문이다.
  - 백기선님이 강의내에서 언급하고 정리해서 알려주시긴 하지만, 직접 해당 스피치의 링크르 따라 들어가 내용을 정리해 보았다.

## 이응준님의 DEVIEW 스피치 - 그런 REST API로 괜찮은가?

### 1. REST의 등장

일단 REST라는 개념이 어떻게 등장했는지에 대해서 먼저 설명해 주신다.

WEB의 원시적 단계를 거친 후 HTTP와 XML을 결합한 SOAP가 있었다.

그리고 앞으로 많이 언급하게될 Roy T. Fielding은 1994-1996년 HTTP/1.0 설계에 참가했고,

| 어떻게 WEB을 깨뜨리지 않으면서 HTTP를 발전시킬 수 있을까?

라는 고민에서 HTTP Object Model 이라는 것을 만들었다고 한다.

그리고 1998년, Microsoft Research에서 REST("Representational State Transfer")란 이름으로 발표를 하고,

2000년, 이것을 박사 논문으로 발표를 하게 된다.

### 2. SOAP와 REST

1998년, Microsoft에서는 XML-RPC라는 프로토콜을 만드는데 이게 SOAP이란 이름으로 바뀐다. 

그리고 Salesforce라는 회사에서 SOAP기반의 API를 공개한다.

그후 2004년, flickr에서도 API를 공개하는데 SOAP과 REST기반의 API를 둘다 내놓게 된다.

사람들은 REST API를 보면서 SOAP기반의 복잡한 API에 비해 새롭고 간단해보이는 REST를 선호하게 된다.

결과로, 2006년 아마존은 자사 85%의 API가 REST형식이라고 밝혔고 Salesforce는 2010년에 REST API를 추가하기에 이른다.

### 3. 그런데? REST가 아니라고?

2008년, CMIS라는 것이 나왔는데 EMC, IBM, Microsoft에서 함께 작업했으며 REST 바인딩을 지원했다.

그런데 Roy형은 "이거 REST 아님" (No REST in CMIS) 이라고 한다.

그 후에도 2016년, Microsoft에서 REST API Guidelines를 내놓는데...

또 Roy 형, "이것도 REST 아님, HTTP API임." ("s/REST API/HTTP API/")

또 블로그에 이런 말들을 적는다.
- "REST는 hyper-test driven 이어야 한다."
- "REST API를 위한 최고의 버저닝 전략은 버저닝을 안하는 것"

### 4. 그러면 뭐가 REST API 인데?

그러면 REST API가 도대체 뭔가 분석해 내려가 보자.

REST API : REST를 따르는 API
REST : 분산 하이퍼미디어 시스템(웹과 같은)을 위한 아키텍쳐 스타일

그러면 REST스타일에 어떤 제약조건들이 있는지 살펴보면,

- Client-Server
- Stateless
- Cache
- Uniform Interface
- Layered System
- Code-on-demand

와 같은 제약조건이 있다. 이중에서 Uniform Interface 말고는 보통 현재 수많은 개발자들이 만들고 있는 API가 조건을 부합한다.

그러면 Uniform Interface조건을 만족하려면 어떻게 해야하는지 항목을 살펴보면,

- Identical of resource
- Manipulation of resource
- `Self-descriptive Message`
- `Hypermedia as the engine of application state` (HAETOS)

마지막 두개의 조건을 대다수가 지키지 못한다고 한다. 그러면 이 두 제약조건은 어떤의미를 가지고 있으며 이 두개를 만족하려면 어떻게 해야하는지,

그리고 이 두 항목이 채워지면 어떤 의미를 갖는지 알아보도록 하자.

### 4. 어떤 부분이 REST하지 않다는 걸까?

#### 1. Self-Descriptive Message

이것을 직역해보면 스스로 설명하는 메세지이다. 즉, 메세지를 받은 클라이언트가 어떤 부차적인 정보(예를 들면 API 정의서)를 보지 않고도

메세지를 해석할 수 있어야 한다는 뜻이다.

다음과 같은 응답이 있다고 해보자.

~~~
HTTP/1.1 200 OK
[{
  "op": "remove",
  "path": "/a/b/c/"
}]
~~~

위와 같은 응답이 있을 때, 아무런 제약 조건 없이 위 응답을 해석하기는 어렵다. 왜냐하면 우리가 본문의 내용이 JSON인지 아니면 또 어떤 다른 형식인지 알 수 없고,
또한 JSON안의 내용 op라던가 path에 대해 어떤 정보도 메세지 내에서 주어지지 않기 때문이다.

#### 2. HAETOS

HAETOS라는 것은, 어플리케이션의 상태가 Hyperlink를 통해서만 전이되어야 한다는 것이다.

예를 들면, HTML의 경우 a태그의 링크를 통해서 HATEOAS를 만족한다.

다음 API 응답을 보자.

~~~
HTTP/1.1 200 OK
Content-Type : application/json
Link : </articles/1>; rel="previous",
       </articles/3>; rel="next";
{
    "title" : "The second article",
    "contents" : "foo bar ..."
}
~~~

위 응답을 보면 Link라는 헤더에 주어진 하이퍼링크 정보로 HATEOAS를 만족하고 있다.
하지만 `Self-descriptive`는 아직도 만족하지 못하고 있는데, 내용이 json인지는 Content-Type으로 알 수 있지만, title과 contents가 어떤 의미인지
해석할 수 없기 때문이다.

#### 3. 이 부분을 지킨다고 뭐가 달라지나? 따르기 어렵다면 따라야 하나?

REST에서 이 부분이 중요한 이유는, 독립적 진화를 위해서이다. 위 조건들을 만족하면 클라이언트 입장에서는 서버가 어떻게 바뀐다고 한들, 자체적으로 변경할 필요가 
없이 독립적으로 진화가 가능하다. 
그런데, 우리 개발자 입장에서는 위 두개의 조건을 포함한 REST를 만족하는 API를 만드려면 시간이 오래걸리고 논의도 많이 필요한 것이 사실이다. 

그러면 꼭 REST여야 할까?

우리 Roy형 또 이렇게 말한 바 있다. "시스템 전체를 통제할 수 있다고 생각하거나, 진화에 관심이 없다면, REST에 대해 따지느라 시간을 낭비하지 마라."

### 5. 그럼 이제 어떻게 할까?

1. REST API를 구현하고 REST API라고 부른다.
2. REST API를 구현하고 HTTP API라고 부른다.
3. REST API가 아니지만 REST API라고 부른다.

보통 우리는 3번 상태에 있을 것이라고 생각된다. 1로 가려면 어떤 노력이 필요하고 어떻게 해야하는지 짚어보자.

#### 1. Self-Descriptive 만족하기

사실 HTML만을 사용한다면 우리는 이미 Self-Descriptive Message를 충족한다. 하지만 JSON을 활용한 HTTP 통신을 하게 되면서 Self-Descriptive하지 
않게 되었다.

- 방법 1: Media Type

미디어 타입을 정의하고 IANA에 등록함으로써 해당 JSON통신을 해석할 수 있도록 한다.

~~~
GET /todos HTTP/1.1
Host: exampel.org

HTTP/1.1 200 OK
Content-Type : application/vnd.todos+json

[
  {"id" : 1, "title" : "회사 가기"},
  {"id" : 2, "title" : "집에 가기"}
]
~~~

Content-Type을 공신력있는 IANA에 등록하고, 해당 JSON을 어떻게 해석해야 하는지에 대한 API정의서를 등록하면 된다. 
하지만 이 방법은 매번 IANA에다가 새로운 JSON형태에 대한 API정의서를 등록해야 하기 때문에 번거롭고 시간도 오래 걸릴 것이다.

- 방법 2: Profile

Link헤더에 profile relation으로 명세를 작성한 페이지의(API정의) 링크를 제공하는 방법.

~~~
GET /todos HTTP/1.1
Host: exampel.org

HTTP/1.1 200 OK
Content-Type : application/json
Link: <https://example.org/docs/todos>; rel="profile"

[
  {"id" : 1, "title" : "회사 가기"},
  {"id" : 2, "title" : "집에 가기"}
]
~~~

#### 2. HATEOAS 만족하기

- 방법 1: data에 하이퍼링크 포함하기

HTTP Body에 해당하는 부분에 link를 삽입하여 명시한다.

~~~
GET /todos HTTP/1.1
Host: exampel.org

HTTP/1.1 200 OK
Content-Type : application/json
Link: <https://example.org/docs/todos>; rel="profile"

[
  {
    "link": "https://example.org/todos/1",
    "id" : 1, "title" : "회사 가기"
  },
  {
    "link": "https://example.org/todos/2",
    "id" : 2, "title" : "집에 가기"
  }
]
~~~

그런데 위 같은 방법은 기존 API서버를 너무 많이 고쳐야 한다.

- 방법 2: HTTP 헤더로

~~~
GET /todos HTTP/1.1
Host: exampel.org

HTTP/1.1 200 OK
Content-Type : application/json
Location: /todos/1
Link: <https://example.org/docs/todos>; rel="profile"
      </todos/2>; rel="next"

[
  {"id" : 1, "title" : "회사 가기"},
  {"id" : 2, "title" : "집에 가기"}
]
~~~

Location으로 현재 페이지의 하이퍼링크, Link로 전이할 수 있는 페이지의 하이퍼링크 정보를 표기한다.

### 7. ???

REST를 만족하려고 할 때, 다음과 같은 질문사항이 있을 수 있다.

| Q. HyperLink는 반드시 uri여야 하는가?

|-|-|
|종류|예|
|-|-|
|uri|https://~~~/users/dboo|
|uri reference(absolute)|/users/dboo|
|uri reference(relative)|dboo|
|uri template|users/{username}|

어떤 표현방식이던 상관없다.

| Q. Media Type 등록은 필수인가?

Roy : "A REST API should be entered with no prior knowledge beyond the inital URI and set of standardized media types
that are apporpriate for the intended audience" -> No

예를 들면, 사내에서만 사용되는 API이고 해당 Media Type을 이해하고 있다면 굳이 등록할 필요는 없다.

### 8. 정리

- 오늘날 대부분의 "REST API"는 사실 REST를 따르지 않고 있다.
- REST의 제약조건 중에서도 특히 Self-Descriptive Message와 HATEOAS를 잘 만족하지 못한다.
  - Self-Descriptive는 custom media type이나 profile link relation 등으로 만족시킨다.
  - HATEOAS는 HTTP 헤더나 본문에 링크를 담아 만족시킬 수 있다.
- REST는 긴 시간을 걸쳐 진화하는 웹 어플리케이션을 위한 것이다.
- REST를 따를 것인지는 API를 설계하는 이들이 스스로 판단하여 결정해야 한다.
- REST를 따르지 않겠다면, "REST를 만족하지 않는 REST API"를 뭐라고 부를지 결정해야 할 것이다.
  - HTTP API라고 부를 수도 있고
  - 그냥 이대로 REST API라고 부를 수도 있다. (Roy가 싫어함ㅋㅋㅋ)


