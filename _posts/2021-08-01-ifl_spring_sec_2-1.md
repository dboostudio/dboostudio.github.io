---
layout: post
author: Dboo
categories: INFLEARN_SPRING_SEC
title: <인프런> 스프링 시큐리티 CH2. 아키텍쳐(1) SecurityContextHolder
tags: LectureNote Inflearn Spring Spring-Security
---

## SecurityContextHolder와 Authentication

이번시간부터는 Spring Security Architecure에 관한 내용이다.

일단 이번에 살펴볼 항목들은 SecurityContextHolder 와 Authentication 부분이다.

### SecurityContextHolder와 Authentication

1. SecurityContextHolder
  - SecurityContext를 제공하며, 기본적으로 ThreadLocal을 사용한다.

  >ThreadLocal : 한 쓰레드 안에서 공유하는 저장소라고 생각하면 편함.

2. SecurityContext
  - Authentication을 제공.

3. Authentication
  - `Principle`, `GrantedAuthority`, Details, credentials, authenticated 제공.

4. Principal
  - “누구"에 해당하는 정보.
  - UserDetailsService에서 리턴한 그 객체.
  - 객체는 UserDetails 타입.

5. GrantedAuthority
  - “ROLE_USER”, “ROLE_ADMIN”등 Principal이 가지고 있는 “권한”을 나타낸다.
  - 인증 이후, 인가 및 권한 확인할 때 이 정보를 참조한다.

5. UserDetails
  - 애플리케이션이 가지고 있는 유저 정보와 스프링 시큐리티가 사용하는 Authentication 객체 사이의 어댑터.

6. UserDetailsService
  - 유저 정보를 UserDetails 타입으로 가져오는 DAO (Data Access Object) 인터페이스.
    구현은 마음대로! (우리는 스프링 데이터 JPA를 사용했습니다.)

### Authentication

지난시간에 짰던 코드의 /dashboard 요청을 받는 컨트롤러에서 SampleService라는 녀석의 dashboard라
는 메소드를 탄다고 가정해보자.

~~~java
@GetMapping("/dashboard")
public String dashboard(Model model, Principal principal){
    model.addAttribute("message", "dashboard");
    sampleService.dashboard();
    return "dashboard";
}
~~~

~~~java
@Service
public class SampleService {
    public void dashboard() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        Object principal = authentication.getPrincipal(); //사실상 UserDetails타입일 것.
        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        Object credentials = authentication.getCredentials(); //인증을 한 다음에는 크리덴셜을 가지고 있지 않을것.
        boolean authenticated = authentication.isAuthenticated();

    }
}
~~~

그러면 dashboard메소드 안에서, 요청한 사용자의 Authentication정보를 위와같이 살펴볼 수 있는데
Authentication객체를 들어가보면 다음과 같다.

~~~java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();

    Object getCredentials();

    Object getDetails();

    Object getPrincipal();

    boolean isAuthenticated();

    void setAuthenticated(boolean var1) throws IllegalArgumentException;
}
~~~

폼인증의 경우 Authentication은 UsernamePasswordAuthenticationToken에서 구현하게 되고, 해당
객체가 SecurityContext안에 담기게 된다.

### Authentication -> Principal

Principal 또한 살펴보자.

![](/assets/img/LectureNote/Inflearn/spring-sec/principal.png)

Principal의 경우는 userdetails패키지 안에 User라는 객체로 구현되어진다. 해당 구현 객체를 쫓아들어가
보면

~~~java
private String password;
private final String username;
private final Set<GrantedAuthority> authorities;
private final boolean accountNonExpired;
private final boolean accountNonLocked;
private final boolean credentialsNonExpired;
private final boolean enabled;
~~~

위와같은 정보들을 담을수 있도록 하고 있다.

### Authentication -> GrantedAuthority

GrantedAuthority를 SimpleGrantedAuthority라는 개체가 구현을 하고 있고, 해당 객체를 까보면
role이라는 변수 하나가 선언되어있다.

~~~java
private final String role;
~~~

그런데 role = "ROLE_USER"로 들어가 있는데 이는 User객체의 빌더가 roles를 받을때 추가해주는 것이다.

해당코드는 User객체를 다시 보면 알수 있는데 User.UserBuilder의 roles메소드에 정의되어 있다.

~~~java
authorities.add(new SimpleGrantedAuthority("ROLE_" + role));
~~~

즉 User객체를 생성할때 ROLE_를 붙여서 SimpleGrantedAuthority객체를 생성한다.
