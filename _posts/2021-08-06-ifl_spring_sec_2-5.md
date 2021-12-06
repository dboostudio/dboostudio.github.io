---
layout: post
author: Dboo
categories: 인프런_스프링시큐리티_백기선
title: CH2. 아키텍쳐(5) AccessDecisionManager
tags: LectureNote Inflearn Spring Spring-Security
---

지금까지는 **인증**에 대한 스프링시큐리티의 동작원리를 살펴보았다면, 앞으로는
**인가** 즉 권한관리(Access Control)을 스프링시큐리티가 어떻게 하고있을까? 에 대해 알아보자.

![](/assets/img/LectureNote/Inflearn/spring-sec/access-decision-voting.png)

## AccessDecisionManager

- Access Control 결정을 내리는 인터페이스로, 구현체 3가지를 기본으로 제공한다.

  **AffirmativeBased**: 여러 Voter중에 한명이라도 허용하면 허용. 기본 전략.  
  ConsensusBased: 다수결  
  UnanimousBased: 만장일치  

- AccessDecisionVoter

  해당 Authentication이 특정한 Object에 접근할 때 필요한 ConfigAttributes를 만족하는지 확인한다.  
  **WebExpressionVoter**: 웹 시큐리티에서 사용하는 기본 구현체, ROLE_Xxxx가 매치하는지 확인.  
  RoleHierarchyVoter: 계층형 ROLE 지원. ADMIN > MANAGER > USER

### AffirmativeBased

~~~java
Iterator var5 = this.getDecisionVoters().iterator();

while(var5.hasNext()) {
    AccessDecisionVoter voter = (AccessDecisionVoter)var5.next();
    int result = voter.vote(authentication, object, configAttributes);
    switch(result) {
    case -1:
        ++deny;
        break;
    case 1:
        return;
    }
}

if (deny > 0) {
    throw new AccessDeniedException(this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
} else {
    this.checkAllowIfAllAbstainDecisions();
}
~~~

AccessDecisionManager의 구현체인 (정확히는 AbstractAccessDecisionManager)AffirmativeBased
의 주요코드이다. Voter들을 불러온뒤에 해당권한이 있는지 체크하면서 하나라도 허용이 나오면(case 1) 그냥
리턴을 하고, 전부 허용이 없다면 AccessDeniedException을 던진다.

## AccessDecisionManager 또는 Voter를 커스터마이징 하는 방법

USER권한으로만 접근할수있는 user라는 페이지가 있다고 하자.

- Controller
~~~java
@GetMapping("/user")
public String user(Model model, Principal principal){
    model.addAttribute("message", "user");
    return "user";
}
~~~

- WebSecurityConfigurerAdapter 구현체
~~~java
.mvcMatchers("user").hasRole("USER")
~~~

그다음, user페이지에 ADMIN권한으로 접근하면 Forbidden에러를 뱉는다. 이는 ADMIN계정이 ROLE_ADMIN
만 가지고 있기 때문인데, ADMIN Authority가 부여될 때 USER Authority를 부여해보자.

- WebSecurityConfigurerAdapter 구현체

여기에 직접 우리가 사용할 AccessDecisionManager를 만든다.
~~~java
public AccessDecisionManager accessDecisionManager() {
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER"); // ROLE_USER의 권한보다 ROLE_ADMIN이 상위권한이다.

    DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler(); //Voter에 넣을 핸들러
    handler.setRoleHierarchy(roleHierarchy); // 핸들러에 RoleHierarchy를 설정

    WebExpressionVoter voter = new WebExpressionVoter(); // AccessDecisionManager에 넘겨줄 Voter
    voter.setExpressionHandler(handler); //Voter에 handler설정
    List<AccessDecisionVoter<? extends  Object>> voters = Arrays.asList(); //AccessDecisionManager에 넘겨줄 VoterList
    return new AffirmativeBased(voters);
}
~~~

그리고 HttpSecurity를 파라미터로받는 configure메소드에 accessDecisionManager를 우리가 커스텀
한 녀석으로 명시해준다.

~~~java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .mvcMatchers("/", "/info", "/account/**").permitAll()
            .mvcMatchers("admin").hasRole("ADMIN")
            .mvcMatchers("user").hasRole("USER")
            .anyRequest().authenticated()
    .accessDecisionManager(accessDecisionManager());
    http.formLogin();
    http.httpBasic();
}
~~~

이렇게 하고 나면, USER권한의 인가가 명시된 요청에 대하여 ADMIN권한으로도 접근할 수 있게된다.

### 위 코드를 조금 줄이는법

configure 메소드 안에서 AccessDecisionManager를 명시해주는 방법을 알아보았는데 명시하기 위해서
우리가 커스텀한 AccessDecisionManager를 직접 설정하는 과정에서 코드가 조금 길어지게 된다.

이를 조금 줄이려면 configure메소드 안에서 AccessDecisionManager를 명시하는게 아니라,
expressionHandler를 명시해주는 방법을 사용하면 코드가 살짝 줄어들게 된다.

뭐.. 근데 그게 그거같다.

~~~java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .mvcMatchers("/", "/info", "/account/**").permitAll()
            .mvcMatchers("admin").hasRole("ADMIN")
            .mvcMatchers("user").hasRole("USER")
            .anyRequest().authenticated()
    .expressionHandler(expressionHandler());
    http.formLogin();
    http.httpBasic();
}
~~~

~~~java
public SecurityExpressionHandler expressionHandler() {
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER"); // ROLE_USER의 권한보다 ROLE_ADMIN이 상위권한이다.

    DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler(); // 핸들러
    handler.setRoleHierarchy(roleHierarchy); // 핸들러에 RoleHierarchy를 설정

    return handler;
}
~~~
