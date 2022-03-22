---
layout: post
author: Dboo
categories: 인프런-스프링_시큐리티-백기선
title: CH1. 폼 인증(3)
tags: LectureNote Inflearn Spring Spring-Security
---

지금까지 시큐리티 설정을 계속 웹브라우저에서 테스트하였는데, 테스트 코드를 통해 테스트 하는 방법을 알아보고
활용해보자.

## 스프링 시큐리티 테스트 1부

일단 spring-security-test 라는 의존성을 추가해준다.

~~~
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
    <version>${spring-security.version}</version>
</dependency>
~~~

test시에만 사용할 것이기 때문에 scope은 test로 주고, version은 spring-security의 버전과 동일
한 버전을 사용하도로 명시해준다.

그 후 컨트롤러에서 cmd + n 을 입력하여 (intellij) test를 만들어 보자.

> 이상하게도, mockMvc.perform(get())을 작성하면 get()에 대한 static method가 자동으로 import
되지 않는 현상이 일어났다.

> 그래서, 해당하는 MockMvcRequestBuilder의 메소드들을 수기로 import시켜주었다.  
`import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;`

강의에선 JUnit4를 사용하여 테스트코드를 작성했지만, 필자는 JUnit5가 탑재된 스프링 부트 버전을 사용했으
므로, JUnit5의 사용법을 적용하여 테스트코드를 작성하면서 수업을 따라갔음을 미리 알린다.

### Test With MockMvc

처음으로 간단하게 index페이지를 호출하여 정상응답이 내려오는지 테스트코드를 통해 확인해보자.

~~~java
@SpringBootTest
@AutoConfigureMockMvc
class AccountControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void index_anonymous() throws Exception {
        mockMvc.perform(get("/").with(anonymous()))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
~~~

그 다음, user로 인증을 받은 계정을 통해 요청을 보내는 코드를 작성해보자.

~~~java
@Test
public void index_user() throws Exception {
    mockMvc.perform(get("/")
                //가짜 유저가 로그인 한 상태를 가정
                .with(user("dboo").roles("USER")))
            .andDo(print())
            .andExpect(status().isOk());
}
~~~

그 다음, admin페이지에 user권한을 가진 사용자가 접근하는 테스트 코드이다.

~~~java
@Test
public void admin_user() throws Exception {
    mockMvc.perform(get("/admin")
            //가짜 유저가 로그인 한 상태를 가정
            .with(user("dboo").roles("USER")))
            .andDo(print())
            .andExpect(status().isForbidden()); //403
}
~~~

다음, admin페이지에 admin권한으로 접근하는 테스트 코드이다.

~~~java
@Test
public void admin_admin() throws Exception {
    mockMvc.perform(get("/admin")
            //가짜 유저가 로그인 한 상태를 가정
            .with(user("dboo").roles("ADMIN")))
            .andDo(print())
            .andExpect(status().isOk());
}
~~~

### Test With MockMvc With Annotaion

여기에서 더 나아가서 `.with`로 일일이 user를 주지 않고, @WithUser 어노테이션을 이용해 코드를 조금
줄일 수 있다.

~~~java
@Test
@WithAnonymousUser
public void index_anonymous() throws Exception {
    mockMvc.perform(get("/"))
            .andDo(print())
            .andExpect(status().isOk());
}

@Test
@WithMockUser(username = "dboo", roles = "USER")
public void index_user() throws Exception {
    mockMvc.perform(get("/"))
            .andDo(print())
            .andExpect(status().isOk());
}

@Test
@WithMockUser(username = "dboo", roles = "USER")
public void admin_user() throws Exception {
    mockMvc.perform(get("/admin"))
            .andDo(print())
            .andExpect(status().isForbidden()); //403 forbidden
}

@Test
@WithMockUser(username = "dboo", roles = "ADMIN")
public void admin_admin() throws Exception {
    mockMvc.perform(get("/admin"))
            .andDo(print())
            .andExpect(status().isOk());
}
~~~

### Test With MockMvc With Custom Annotaion

그런데 여기서 더 코드양을 줄일 수 있는 방법은 customAnnotation을 만들어 붙여주는 것이다.

test안에 account패키지에다가 `WithUser`, 'WithAdmin'라는 어노테이션을 만들어보자.

- @WithUser

~~~java
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(username = "dboo", roles = "USER")
public @interface WithUser {
}
~~~

- @WithAdmin

~~~java
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(username = "dboo", roles = "ADMIN")
public @interface WithAdmin {
}
~~~

그 후, 테스트코드에서
@WithMockUser(username = "dboo", roles = "USER") -> @WithUser  
@WithMockUser(username = "dboo", roles = "ADMIN") -> @WithAdmin  
으로 각각 바꿔준다.

> 어노테이션을 @Test까지 합쳐서 @TestWithUser 이런식으로 사용도 가능할 것으로 보인다!
