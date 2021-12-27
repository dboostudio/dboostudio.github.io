---
layout: post
author: Dboo
categories: 필기)인프런_스프링시큐리티_백기선
title: CH2. 아키텍쳐(4) Filter&FilterChainProxy, DelegatingFilterProxy
tags: LectureNote Inflearn Spring Spring-Security
---

## Filter와 FilterChainProxy

스프링 시큐리티가 제공하는 필터들
1. WebAsyncManagerIntergrationFilter
2. **SecurityContextPersistenceFilter**
3. HeaderWriterFilter
4. CsrfFilter
5. LogoutFilter
6. **UsernamePasswordAuthenticationFilter**
7. DefaultLoginPageGeneratingFilter
8. DefaultLogoutPageGeneratingFilter
9. BasicAuthenticationFilter
10. RequestCacheAwareFtiler
11. SecurityContextHolderAwareReqeustFilter
12. AnonymouseAuthenticationFilter
13. SessionManagementFilter
14. ExeptionTranslationFilter
15. FilterSecurityInterceptor  
...

우리는 저번에 인증요청이 왔을때 SecurityContextPersistenceFilter와
UsernamePasswordAuthenticationFilter가 동작하는 것을 보았는데, 이 두필터는 누구에 의해 호출되고
어떻게 호출되는지 알아보자.

Filter들은 기본적으로 FilterChainProxy에 의해서 호출되게 된다.

### FilterChainProxy

Proxy내부에서 일어나는 일은 다음과 같다.

1. getFilter함수에서 request와 매치되는 SecurityFilterChain에서 Filter목록을 가져온다.

~~~java
private List<Filter> getFilters(HttpServletRequest request){
  for(SecurityFilterChain chain : filterChains){
    if(chain.matches(request)){
      return chain.getFilters();
    }
  }
  return null;
}
~~~

2. 필터들을 순회하면서 순차적으로 수행한다.

> 이 FilterChain은 우리가 WebSecurityConfigurerAdapter의 구현체를 설정해준 것을 토대로 만들어진다.

그렇다면,WebSecurityConfigurerAdapter의 구현체를 여러개 만들면 여러개 쓸수 있다.

### Multiple FilterChain

기존에 있던 SecurityConfig 클래스를 복사해서 하나더 만들어주고 기존 SecurityConfig에서는 전체요청
을 허용, AnotherSecuirtyConfig에서는 모든 경로를 인증해야 하는것으로 변경하자.

~~~java
@Configuration
public class SecuirtyConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().permitAll();
        http.formLogin(); // 폼 로그인을 할 것이다.
        http.httpBasic(); //httpBasic도 적용할 것이다.
    }
}
~~~

~~~java
@Configuration
public class AnotherSecuirtyConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated();
        http.formLogin(); // 폼 로그인을 할 것이다.
        http.httpBasic(); //httpBasic도 적용할 것이다.
    }
}
~~~

그런데! 이렇게 작성하고 실행하면, 스프링시큐리티가 에러를 뱉는다. `@Order must be unique` 즉, 두 필
터체인에 순서를 정해줘야 한다는 것이다.

그러면 SecurityConfig, AnotherSecuirtyConfig에 각각
`@Order(Ordered.LOWEST_PRECEDENCE - 10)`,
`@Order(Ordered.LOWEST_PRECEDENCE - 15)`
을 부여해보자.

우선순위가 높은 필터체인이 먼저 걸려서 해당 필터로 인증을 수행하게 된다. AnotherSecuirtyConfig가 이
경우에는 우선순위가 높으므로 어떤 요청을 보내더라도 로그인 페이지로 보내주게 될 것이다.

## DelegatingFilterProxy

- DelegatingFilterProxy

  일반적인 서블릿 필터.
  서블릿 필터 처리를 스프링에 들어있는 빈으로 **위임**하고 싶을 때 사용하는 서블릿 필터.
  타겟 빈 이름을 설정한다.

  스프링 부트 없이 스프링 시큐리티 설정할 때는 AbstractSecurityWebApplicationInitializer를
  사용해서 등록.
  스프링 부트를 사용할 때는 자동으로 등록 된다. (SecurityFilterAutoConfiguration)

- FilterChainProxy

  보통 “springSecurityFilterChain” 이라는 이름의 빈으로 등록된다.

![](/assets/img/LectureNote/Inflearn/spring-sec/delegatingFilterProxy.png)
