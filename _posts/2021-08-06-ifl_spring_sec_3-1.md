---
layout: post
author: Dboo
categories: INFLEARN_SPRING_SEC
tags: LectureNote Inflearn Spring Spring-Security
---

# <인프런> 스프링 시큐리티 CH3. 웹 어플리케이션 시큐리티(1) WebAsyncManagerIntegrationFilter
## 스프링 시큐리티 ignoring()

WebSecurity의 ignoring()을 사용해서 시큐리티 필터 적용을 제외할 요청을 설정할 수 있다.

~~~java
@Override
public void configure(WebSecurity web) throws Exception {
    web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
}
~~~

스프링 부트가 제공하는 PathRequest를 사용해서 정적 자원 요청을 스프링 시큐리티 필터를 적용하지 않도록 설정.

### 다른 방법에 대한 조언

http.authorizeRequests()
.requestMatchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()

- 이런 설정으로도 같은 결과를 볼 수는 있지만 스프링 시큐리티 필터가 적용된다는 차이가 있다.
  동적 리소스는 http.authorizeRequests()에서 처리하는 것을 권장합니다.
  정적 리소스는 WebSecurity.ignore()를 권장하며 예외적인 정적 자원 (인증이 필요한 정적자원이 있는
  경우)는 http.authorizeRequests()를 사용할 수 있습니다.

요청url매칭방법은 antMatcher, mvcMatcher, regexpMatch등 여러가지 방법이 있다. 해당 강의에서는
개인적으로 파보라고 던져주었다. 나중에 개인적으로 정리해서 올려야겠다.



## WebAsyncManagerIntegrationFilter

간단히 말하면, Spring MVC Async 핸들러를 지원하는 기능이다. Security Context 가 threadLocal
에 존재하기 때문에 외부 thread에서는 Security Context를 꺼내쓸 수 없다.

그런데 Spring MVC 에서 Async한 기능에서 이 Security Context를 꺼내 쓸 수 있도록 해주는 필터이다.

- 스프링 MVC의 Async 기능(핸들러에서 Callable을 리턴할 수 있는 기능)을 사용할 때에도 SecurityContext를 공유하도록 도와주는 필터.  
  PreProcess: SecurityContext를 설정한다.  
  Callable: 비록 다른 쓰레드지만 그 안에서는 동일한 SecurityContext를 참조할 수 있다.  
  PostProcess: SecurityContext를 정리(clean up)한다.

### 코드로 이해하기

다음과 같은 컨트롤러가 있다고 해보자.

~~~java
@GetMapping("/async-handler")
@ResponseBody
public Callable<String> asyncHandler(){
    return new Callable<String>() {
        @Override
        public String call() throws Exception {
            return "Async Handler";
        }
    };
}
~~~

Callable을 리턴하게 되면, Callable안에 있는 코드를 수행하기 전에 응답을 내보낸다.

이 안에서, 쓰레드 이름, 호출된 메소드, 그리고 SecurityContextHolder에서 꺼내온 principal정보를
로그를 찍는 클래스를 따로 빼서 만들자.

~~~java
public class SecurityLogger {

    public static void log(String message) {
        System.out.println(message);

        Thread thread = Thread.currentThread();
        System.out.println("thread : " + thread.getName());

        Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        System.out.println("principal : " + principal);
    }
}
~~~

~~~java

~~~
