---
layout: post
author: Dboo
categories: Inflearn_Spring_Security
title: <인프런> 스프링 시큐리티 CH2. 아키텍쳐(3) SecurityContextPersistenceFilter
tags: LectureNote Inflearn Spring Spring-Security
---

## Authentication와 SecurityContextHolder

AuthenticationManager가 인증을 마친 뒤 리턴 받은 Authentication 객체의 행방은?

> Authentication은 SecurityContextHolder에 담겨져 Application전반에 사용할 수 있게 된다.

그러면 이 Authentication객체를 SecurityContextHolder에 담아주는 녀석은 누구일까?

- UsernamePasswordAuthenticationFilter

  폼 인증을 처리하는 시큐리티 필터
  인증된 Authentication 객체를 SecurityContextHolder에 넣어주는 필터
  SecurityContextHolder.getContext().setAuthentication(authentication)

- SecurityContextPersistenceFilter

  SecurityContext를 HTTP session에 캐시(기본 전략)하여 여러 요청에서 Authentication을 공유할 수 있 공유하는 필터.
  SecurityContextRepository를 교체하여 세션을 HTTP session이 아닌 다른 곳에 저장하는 것도 가능하다.

### Authentication 객체

인증을 마친 Authentication객체에서 정보를 꺼내오는것은 다음과 같다.

~~~java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
UserDetails principal = (UserDetails) authentication.getPrincipal();
~~~

Principal의 경우 authentication 에서 getPrincipal을 하면 Object타입으로 리턴되는데 Principal
안의 정보를 꺼내오려면 이를 UserDetails나 User로 타입캐스팅해서 사용하면 된다.

### SecurityContextPersistenceFilter 로그인 이전

인증이 필요한 어떤 페이지에 접근을 요청하면, 먼저 SecurityContextPersistenceFilter가 기존에 캐시
되어있는 세션이 있는지, 있다면 해당인증을 불러오려고 한다.

### UsernamePasswordAuthenticationFilter

SecurityContextPersistenceFilter가 캐시된 세션을 찾지 못하고, 폼 로그인 화면으로 리다이렉트된 후
인증을 시도하면 폼 로그인을 처리하는 UsernamePasswordAuthenticationFilter가 동작하게 된다.

UsernamePasswordAuthenticationFilter는 AuthenticationManager를 통해서 인증을 시도한다.

~~~java
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
    if (this.postOnly && !request.getMethod().equals("POST")) {
        throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
    } else {
        String username = this.obtainUsername(request);
        username = username != null ? username : "";
        username = username.trim();
        String password = this.obtainPassword(request);
        password = password != null ? password : "";
        UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
        this.setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }
}
~~~

그러면, 지난시간에 배웠던 것처럼 AuthenticationManager의 구현체인 ProviderManager가 인증을 처리
하게 된다. 인증이 될때 SecurityContextPersistenceFilter를 통해서 세션을 캐시한다.

인증이 처리가 된 후, 원래 접근을 시도했던 요청화면으로 리다이렉트를 시키는데 그러면 다시 새로운 요청이 발생
하는 것이므로 SecurityContextPersistenceFilter가 작동하게 된다.

### SecurityContextPersistenceFilter 로그인 이후

인증이 필요한 어떤 페이지에 접근을 요청하면, 먼저 SecurityContextPersistenceFilter가 기존에 캐시
되어있는 세션이 있는지, 있다면 해당 세션을 통해 context를 불러오게 된다.

그런데 이번에는 이미 캐시된 세션이 있으므로 HttpSessionSecurityContextRepository에서 context를
불러와서 넣어주게 된다.
