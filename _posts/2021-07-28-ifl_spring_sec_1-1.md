---
layout: post
author: Dboo
categories: 인프런_스프링시큐리티_백기선
title: CH1. 폼 인증(1)
tags: LectureNote Inflearn Spring Spring-Security
---

## 스프링 웹 프로젝트 만들기

일단 스프링 이니셜라이져를 통해서 스프링 부트 어플리케이션을 하나 만들어준다.  
의존성은 web, thymeleaf 두개만 추가해주면 된다.

resource아래에 template폴더에(없으면 생성) index, info, admin, dashboard 페이지를 만들어준다.

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Hello</title>
</head>
<body>
    <h1 th:text="${message}">Hello</h1>
</body>
</html>
~~~

title과 h1의 내용만 변경해서 만들어 준 후에, 컨트롤러를 만들어준다.

~~~java
@Controller
public class SampleController {

    @GetMapping("/")
    public String index(Model model){
        model.addAttribute("message", "Hello Spring Sec");
        return "index";
    }
    @GetMapping("/info")
    public String info(Model model){
        model.addAttribute("message", "info");
        return "info";
    }
    @GetMapping("/dashboard")
    public String dashboard(Model model, Principal principal){
        model.addAttribute("message", "dashboard");
        return "dashboard";
    }
    @GetMapping("/admin")
    public String admin(Model model, Principal principal){
        model.addAttribute("message", "admin");
        return "admin";
    }
}
~~~

dashboard 와 admin은 로그인한 사용자를 대상으로 제공할 것이기 때문에 Principal객체를 파라미터로 받
도록 한다.

여기서 추가로 index페이지에서 로그인한 사용자와 로그인하지 않은 사용자를 구분하고 싶다면 다음과 같이 변경
한다.

~~~java
@GetMapping("/")
public String index(Model model, Principal principal){
    if(principal == null){
        model.addAttribute("message", "Hello Spring Sec");
    } else{
        model.addAttribute("message", "Hello" + principal.getName());
    }
    return "index";
}
~~~

principal == null 인 경우에는 로그인하지 않은 사용자이기 때문에 기본 메세지를 보여주고, 로그인 한 경
우에는 사용자 이름을 가져와서 보여주도록 한다.

그런데 아직 우리는 Spring Security를 적용하지 않았기 때문에 로그인이 동작하지는 않을 것이다. 이제
Spring Security를 적용해보자.

## Spring Security 연동

일단 Spring Security의 의존성을 추가해보자.

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
~~~

적용한후 서버를 다시 시작해보면, 루트에 접근하려고만 해도 로그인 페이지가 뜨는것을 볼 수 있다.

우리가 유저정보에 대한 설정을 하지 않았기때문에 스프링시큐리티는 기본 사용자 정보를 제공해주는데 username
은 user, password는 서버 콘솔창에 띄워주게 된다.

해당 정보로 로그인을 하게되면 우리가 만든 index페이지로 접근하면서, username이 user이기 때문에
Hello user 라는 메세지를 보게 된다.

Info, Dashboard, Admin 에 접근이 가능한데 admin의 경우 나중에 권한을 설정해주어서 user의 권한으
로는 접근하지 못하도록 막을 예정이다.

## 스프링 시큐리티 설정하기

이제, 본격적으로 Spring Security의 인증과 권한에 대한 디테일을 설정해보는 시간이다.

SecurityConfig라는 클래스를 만들어주고 거기에 @Configuration, @EnableWebSecurity 어노테이션을
붙인 후 WebSecurityConfigurerAdapter를 상속하면 된다. 어노테이션이나 상속받는 클래스에 대한 설명
은 나중에 정리해주신다고 한다.

~~~java
@Configuration
@EnableWebSecurity
public class SecuirtyConfig extends WebSecurityConfigurerAdapter {
}
~~~

다음 WebSecurityConfigurerAdapter의 configure라는 메소드를 재정의 하는데, 파라미터로 HttpSecurity
를 받는 녀석을 재정의 해준다.

~~~java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http.authorizeRequests()
          .mvcMatchers("/", "/info").permitAll() // 루트(/)와 /info 경로의 접근은 모두 허용
          .mvcMatchers("admin").hasRole("ADMIN") // admin은 ADMIN 권한이 있을때만 허용
          .anyRequest().authenticated() // 그외의 경로로 접근하는 경우 인증(로그인)을 해야 허용
          .and()// and()를 통해 메소드 체이닝을 하는것은 필수는 아니다.
      .formLogin() // 폼 로그인을 할 것이다.
          .and()// http.httBasic() 이런식으로 끊어서 사용이 가능하다.
      .httpBasic(); //httpBasic도 적용할 것이다.
}
~~~

## 인메모리 유저 추가

이제는 직접 유저를 추가 및 설정을 해보자. 여러가지 방법중 가장 간단한 방법인 인메모리 유저를 추가하는 방법
을 알아보자.

기존 스프링 시큐리티가 유저정보를 만들어줄때를 잘 보면, 콘솔에 UserDeatailServiceAutoConfiguration
이라는 클래스로부터 패스워드가 만들어짐을 볼 수 있다.

UserDeatailServiceAutoConfiguration를 열어서 보면 inMemoryUserDetailsManager라는 녀석이
있고 SecurityProperties타입의 객체로부터 유저정보를 받아오는것을 알 수 있다.  
SecurityProperties를 타고 들어가보면, 유저 이름, 패스워드, 권한등을 멤버변수로 가지고 있다.

> 인메모리 유저정보를 사용해 로그인을 하는것은 간단한 설정으로 로그인테스트를 위해서 하는것뿐이지 실 서비스에
적용하는것은 매우 위험하다.

그럼 이 인메모리 유저를 어디다가 추가하느냐 하면 application.properties에 추가해주면 된다.

~~~
spring.security.user.name=admin
spring.security.user.password=123
spring.security.user.roles=ADMIN
~~~

위방법은 계정을 하나밖에 설정하지못한다. 다른 방법으로도 추가할 수 있는데 우리가 만들어뒀던 SecurityConfig
로 돌아가서 이번엔 파라미터로 AuthenticationManagerBuilder타입의 객체를 받는 configure메소드를
재정의해준다.

~~~java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
            //{} prefix는 어떤 인코더를 사용할지 알려주는 것이다. noop : 사용하지 않음
            .withUser("dboo").password("{noop}123").roles("USER").and()
            .withUser("admin").password("{noop}!@#").roles("USER").and()
            // 혹은 다음과 같이 유저를 추가해줄수도 있다.
            .withUser(
                    //이렇게 default 패스워드 인코더를 적용해줄수도 있지만 테스트용이다.
                    User.withDefaultPasswordEncoder()
                    .username("user").password("1234").roles("USER")
            );
}
~~~
