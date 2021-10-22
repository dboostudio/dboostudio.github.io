---
layout: post
author: Dboo
categories: Inflearn_Spring_Security
title: <인프런> 스프링 시큐리티 CH1. 폼 인증(4)
tags: LectureNote Inflearn Spring Spring-Security
---

## 스프링 시큐리티 테스트 2부

이번시간에는 폼 로그인 기능이 제대로 작동하는지 테스트해보는 시간을 가질 것이다.

현재는 스프링시큐리티가 제공해주는 기본 formLogin을 사용하고 있지만, 추후에 직접 제작한 폼 로그인을 사용
하려면 이 테스트가 필요하게 될 것이기 때문이다.

~~~java
@Test
public void login() throws Exception {
    mockMvc.perform(formLogin().user("dboo").password("123"))
            .andExpect(authenticated());
}
~~~

그런데, 지금 user정보가 없기때문에 해당 테스트는 실패한 것으로 나올 것이다.

새 Account 객체와, accountService를 이용하여 가짜 유저를 만들어준 후 로그인을 시도해보자.

> 추가팁 : 테스트코드에서 DB에 있는 내용을 변경하는 코드가 있는 경우 @Transactional 어노테이션을 붙
여주는 것이 좋다. (테스트에서는 Transactional이 기본적으로 RollBack으로 동작)

~~~java
@Test
@Transactional
public void login_success() throws Exception {
    String username = "dboo";
    String password = "123";
    this.createUser(username, password);

    mockMvc.perform(formLogin().user(username).password(password))
            .andExpect(authenticated());
}

private Account createUser(String username, String password) {
    Account account = Account.builder()
            .username(username)
            .password(password)
            .role("USER")
            .build();
    accountService.createNew(account);
    return account;
}
~~~

그리고, 엉뚱한 유저로 로그인하는 테스트를 작성해보자.

~~~java
@Test
@Transactional
public void login_fail() throws Exception {
    String username = "dboo";
    String password = "123";
    this.createUser(username, password);

    mockMvc.perform(formLogin().user(username).password(password+"0"))
            .andExpect(unauthenticated());
}
~~~
