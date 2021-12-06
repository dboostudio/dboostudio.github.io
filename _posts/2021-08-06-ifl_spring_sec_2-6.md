---
layout: post
author: Dboo
categories: 인프런_스프링시큐리티_백기선
title: CH2. 아키텍쳐(6) FilterSecurityInterceptor, Summary
tags: LectureNote Inflearn Spring Spring-Security
---

이번 시간에는 AccessDecisionManager를 누가 호출해주는지 알아보도록 하자.

## FilterSecurityInterceptor

AccessDecisionManager를 사용하여 Access Control 또는 예외 처리 하는 필터.
대부분의 경우 FilterChainProxy에 제일 마지막 필터로 들어있다.

![](/assets/img/LectureNote/Inflearn/spring-sec/filtersecurityinterceptor.png)

FilterSecurityInterceptor 의 부모인 AbstractSecurityInterceptor를 들어가보면,
attempAuthorize라는 메소드가 있는데 이부분이 AccessDecisionManager를 호출하는 곳이다.

~~~java
private void attemptAuthorization(Object object, Collection<ConfigAttribute> attributes, Authentication authenticated) {
    try {
        this.accessDecisionManager.decide(authenticated, object, attributes);
    } catch (AccessDeniedException var5) {
        if (this.logger.isTraceEnabled()) {
            this.logger.trace(LogMessage.format("Failed to authorize %s with attributes %s using %s", object, attributes, this.accessDecisionManager));
        } else if (this.logger.isDebugEnabled()) {
            this.logger.debug(LogMessage.format("Failed to authorize %s with attributes %s", object, attributes));
        }

        this.publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated, var5));
        throw var5;
    }
}
~~~

## ExceptionTranslationFilter

필터 체인에서 발생하는 AccessDeniedException과 AuthenticationException을 처리하는 필터

- AuthenticationException 발생 시

  AuthenticationEntryPoint 실행  
  AbstractSecurityInterceptor 하위 클래스(예, FilterSecurityInterceptor)에서 발생하는
  예외만 처리.  
  그렇다면 UsernamePasswordAuthenticationFilter에서 발생한 인증 에러는?  

- AccessDeniedException 발생 시

  익명 사용자라면 AuthenticationEntryPoint 실행  
  익명 사용자가 아니면 AccessDeniedHandler에게 위임  

## Security Summary

![](/assets/img/LectureNote/Inflearn/spring-sec/security-summary.png)
