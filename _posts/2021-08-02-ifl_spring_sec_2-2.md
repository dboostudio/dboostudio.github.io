---
layout: post
author: Dboo
categories: 인프런_스프링시큐리티_백기선
title: CH2. 아키텍쳐(2) AuthenticationManager
tags: LectureNote Inflearn Spring Spring-Security
---

## AuthenticationManager와 Authentication

지난시간에는 Authentication가 무엇이고, 어떤 정보들을 담고 있는지에 대해 배웠고, 이번 시간에는 이
Authentication을 실제로 만들고 인증을 처리하는 AuthenticationManager에 대해 배워보자.

~~~java
Authentication authenticate(Authentication authentication) throws AuthenticationException;
~~~

AuthenticationManager 안에는 authenticate라는 메소드 하나만 존재한다.

인자로 받은 Authentication이 유효한 인증인지 확인하고 Authentication 객체를 리턴한다.
인증을 확인하는 과정에서 비활성 계정, 잘못된 비번, 잠긴 계정 등의 에러를 던질 수 있다.

보통, ProviderManager라는 클래스가 구현을 담당한다.

유효한 인증인지 확인
사용자가 입력한 password가 UserDetailsService를 통해 읽어온 UserDetails 객체에 들어있는 password와 일치하는지 확인
해당 사용자 계정이 잠겨 있진 않은지, 비활성 계정은 아닌지 등 확인

Authentication 객체를 리턴
Authentication
Principal: UserDetailsService에서 리턴한 그 객체 (User)
Credentials: 인증후에는 비어있음, 인증전 비밀번호를 담기위한 변수
GrantedAuthorities

### ProviderManager

ProviderManager에서 구현된 authenticate의 주요내용만 살펴보면,

~~~java
private List<AuthenticationProvider> providers;

...
Iterator var9 = this.getProviders().iterator();

        while(var9.hasNext()) {
            AuthenticationProvider provider = (AuthenticationProvider)var9.next();
            if (provider.supports(toTest)) {
                if (logger.isTraceEnabled()) {
                    Log var10000 = logger;
                    String var10002 = provider.getClass().getSimpleName();
                    ++currentPosition;
                    var10000.trace(LogMessage.format("Authenticating request with %s (%d/%d)", var10002, currentPosition, size));
                }

                try {
                    result = provider.authenticate(authentication);
                    if (result != null) {
                        this.copyDetails(authentication, result);
                        break;
                    }
                } catch (InternalAuthenticationServiceException | AccountStatusException var14) {
                    this.prepareException(var14, authentication);
                    throw var14;
                } catch (AuthenticationException var15) {
                    lastException = var15;
                }
            }
        }
~~~

ProviderManager가 직접 인증을 처리하는 것이 아니라 AuthenticationProvider `들`에게 위임하는
구조로 되어있다.

그런데 현재 이 ProviderManager는 AuthenticationProvider를 딱 하나만 가지고 있는데 그 타입이
AnonymousAuthenticationProvider로 익명사용자를 인증하는 것이다.
현재 Authentication객체는 실제 UsernamePasswordAuthenticationToken 타입이고,
AnonymousAuthenticationProvider는 해당 타입의 인증을 처리할 수 없다.
이 때는 Parent에게 인증이 다시 위임되게 된다.

Parent의 ProviderManager에는 AuthenticationManager로써 DaoAuthenticationProvider를
가지고 있다. 이 Provider는 UsernamePasswordAuthenticationToken을 처리할 수 있다.

DaoAuthenticationProvider를 잠깐 살펴보면,

~~~java
protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
    this.prepareTimingAttackProtection();

    try {
        UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
        if (loadedUser == null) {
            throw new InternalAuthenticationServiceException("UserDetailsService returned null, which is an interface contract violation");
        } else {
            return loadedUser;
        }
    } catch (UsernameNotFoundException var4) {
        this.mitigateAgainstTimingAttack(authentication);
        throw var4;
    } catch (InternalAuthenticationServiceException var5) {
        throw var5;
    } catch (Exception var6) {
        throw new InternalAuthenticationServiceException(var6.getMessage(), var6);
    }
}
~~~

이 retrieveUser안에 `this.getUserDetailsService` 가 우리가 만든 (UserDetailsService를
구현한) AccountService로 연결이 된다. 그 후, 계정에 대한 추가적인 확인및 cache를 하고 Authentication
을 최종적으로 리턴하게 되고 이 Authentication객체는 우리가 만든 User객체이다.

그 다음으로 이 User객체가 SecurityContextHolder로 Principal로써 들어가게 되는 것이다.


## ThreadLocal

- Java.lang 패키지에서 제공하는 쓰레드 범위 변수. 즉, 쓰레드 수준의 데이터 저장소.
- 같은 쓰레드 내에서만 공유.
- 따라서 같은 쓰레드라면 해당 데이터를 메소드 매개변수로 넘겨줄 필요 없음.
- SecurityContextHolder의 기본 전략.
- Transaction처리에 기본적으로 사용됨.

~~~java
public class AccountContext {

    private static final ThreadLocal<Account> ACCOUNT_THREAD_LOCAL
            = new ThreadLocal<>();

    public static void setAccount(Account account) {
        ACCOUNT_THREAD_LOCAL.set(account);
    }

    public static Account getAccount() {
        return ACCOUNT_THREAD_LOCAL.get();
    }

}
~~~
