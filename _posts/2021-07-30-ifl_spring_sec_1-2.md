---
layout: post
author: Dboo
categories: 인프런_스프링시큐리티_백기선
title: CH1. 폼 인증(2)
tags: LectureNote Inflearn Spring Spring-Security
---

## 스프링 시큐리티에 JPA 연동하기

이전 시간에는 직접 만든 유저정보로 로그인을 했지만 실서비스에서는 DB에 암호화된 유저정보를 가지고 있다가,
해당 유저정보에 맞는 로그인 요청에 대해서 인가를 해줘야 한다.

따라서 DB에 있는 유저정보를 통해 로그인이 가능하도록(DAO를 통해) 해보자.

일단 JPA를 연동하기 위해 JPA의존성과 인메모리 DB로 사용할 H2데이터베이스의 의존성을 추가하자.

~~~
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
~~~

유저정보를 담을 엔티티를 만들어준다.

~~~java
@Data
@Entity
public class Account {

    @Id
    @GeneratedValue
    private Integer id;

    @Column(unique = true)
    private String username;

    private String password;
    private String role;
}
~~~

다음으로 Account엔티티에대한 리포지토리를 만든다.

~~~java
public interface AccountRepository extends JpaRepository<Account, Integer> {}
~~~

다음으로는 UserDetailsService를 구현하는 AccountService를 만들어준다. UserDetailsService가
하는 일은, username을 가지고 UserDetails의 객체를 리턴하는 것이다.

그러면 AccountService를 만들어보자.

~~~java
@Service
public class AccountService implements UserDetailsService {

    @Autowired
    AccountRepository accountRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account account = accountRepository.findByUsername(username);
        if(account == null){ // 조회된 계정이 없다면
            // 유저정보를 찾지못함 예외를 던진다.
            throw new UsernameNotFoundException(username);
        }
        // 조회된 계정이 있다면
        // UserDetails 타입으로 리턴을 해줘야한다.
        // 이를 편리하게 하도록 스프링 시큐리티가 User라는 객체타입을 제공한다.
        return User.builder()
                .username(account.getUsername())
                .password(account.getPassword())
                .roles(account.getRole())
                .build();
    }
}
~~~

코드를 보면 알겠지만 스프링 시큐리티가 제공하는 User객체를 사용하면, 우리가 직접 Account객체를
UserDetails 객체로 변환할 필요 없이 간단하게 리턴할 수 있도록 제공하고 있다.

그럼 이제 마지막으로 우리가 직접 Account를 등록할 수 있도록 Controller를 추가해보자.

실서비스에서는 PathVariable을 통해서 게정정보를 받으면 안되지만... 간소화를 위해 PathVariable을
통해서 계정정보를 입력하도록 하자.

~~~java
@RestController
public class AccountController {

    @Autowired
    AccountRepository accountRepository;

    @GetMapping("/account/{role}/{username}/{password}")
    public Account createAccount(@ModelAttribute Account account){
        return accountRepository.save(account);
    }
}
~~~

그런데 우리가 계정을 만들기위해서 매핑해준 경로가 /account/~~~ 이기 때문에 해당 경로를 인증이 필요없도
록 만들어줘야한다.

~~~java
@Configuration
@EnableWebSecurity
public class SecuirtyConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/", "/info", "/account/**").permitAll() // 루트(/)와 /info 경로의 접근은 모두 허용
                .mvcMatchers("admin").hasRole("ADMIN") // admin은 ADMIN 권한이 있을때만 허용
                .anyRequest().authenticated(); // 그외의 경로로 접근하는 경우 인증(로그인)을 해야 허용
        http.formLogin(); // 폼 로그인을 할 것이다.
        http.httpBasic(); //httpBasic도 적용할 것이다.
    }
}
~~~

/account/** 로 /account를 통한 모든경로에 대해 permitAll을 해준다. 그런데 이 상태로 되느냐?
무엇을 빠뜨렸는지 생각해보자.

위 경로를 통해 계정을 만들고 로그인을 시도하면 패스워드인코더가 없다는 에러 메세지가 나온다. 그러면 계정정
보를 받아서 계정을 만들때에 인코딩을 해주도록 변경해보자.

- Controller

~~~java
@RestController
public class AccountController {

    @Autowired
    AccountService accountService;

    @GetMapping("/account/{role}/{username}/{password}")
    public Account createAccount(@ModelAttribute Account account){
        return accountService.createNew(account);
    }
}
~~~

기존에 accountRepository.save()만 수행했던 부분을 accountService쪽으로 이관한 후,

- Service

~~~java
public Account createNew(Account account) {
    account.encodePassword();
    return this.accountRepository.save(account);
}
~~~

Account에 encodePassword를 통해 인코딩을 수행하고, 서비스단에서 Repository에 저장하도록 변경,

- Entity

~~~java
@Data
@Entity
public class Account {

    @Id
    @GeneratedValue
    private Integer id;

    @Column(unique = true)
    private String username;

    private String password;
    private String role;

    public void encodePassword() {
        this.password = "{noop}"+this.password;
    }
}
~~~

Entity내에 선언된 메소드를 통해 문자열 앞에 인코딩 prefix를 붙이도록 변경한다.

이렇게 데이터베이스를 통해서 계정정보를 저장하고 인증을 하는 방법을 알아보았다.

두가지 추가적인 설명을 덧붙이면,
1. 패스워드의 경우 스프링 시큐리티에 이미 PasswordEncoder가 따로 있어서 그 인코더가 인코딩을 담당하도록 하는 것이 좋다.
2. UserDetailsService구현체인 AccountService의 경우 원래 명시적으로 SecurityConfig에 authManger
에게 AccountService가 구현체임을 알려줘야 하지만, 빈으로 등록되어있는 경우 그럴필요가 없다.

## PasswordEncoder

기존에 패스워드를 prefix인 `{noop}`을 붙여서 했는데, 그 과정을 하드코딩으로 하지 않고 스프링이 제공하
는 PasswordEncoder에게 위임하도록 해보자.

PasswordEncoder를 Bean으로 등록하자.

~~~java
@Bean
public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
}
~~~

여기서 리턴하고 있는 NoOpPasswordEncoder타입은 deprecated된 것이므로 권장하지 않지만 수업을 위해
그냥 사용하기로 한다.

PasswordEncoder를 Autowired로 끌어와서 기존 하드코딩들을 교체해준다.

- Service

~~~java
public Account createNew(Account account) {
    account.encodePassword(passwordEncoder);
    return this.accountRepository.save(account);
}
~~~

- Account Entity

~~~java
public void encodePassword(PasswordEncoder passwordEncoder) {
    this.password = passwordEncoder.encode(this.password);
}
~~~

NoOpPasswordEncoder는 시큐리티5 이전에 많이 사용했지만, deprecated되었기 때문에 버전업을 하면 인
증이 되지 않는 현상이 생길 수 있으므로 사용하지 않는게 좋다. 그리고 다양한 알고리즘을 통해 패스워드 인코딩
을 지원하려다 보니 {}포맷을 선택하여 지원하게 되어있다.

## 시큐리티 5에서 권장하는 인코딩 방식

최종적으로 시큐리티 5에서 권장하는 패스워드 인코딩 방식은 다음과 같다.

~~~java
@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
~~~

PasswordEncoderFactories를 이용하여 패스워드 인코더를 리턴하는 것인데, createDelegatingPasswordEncoder
메소드를 들어가보면 다음 코드들을 볼 수 있다.

~~~java
public static PasswordEncoder createDelegatingPasswordEncoder() {
    String encodingId = "bcrypt";
    Map<String, PasswordEncoder> encoders = new HashMap();
    encoders.put(encodingId, new BCryptPasswordEncoder());
    encoders.put("ldap", new LdapShaPasswordEncoder());
    encoders.put("MD4", new Md4PasswordEncoder());
    encoders.put("MD5", new MessageDigestPasswordEncoder("MD5"));
    encoders.put("noop", NoOpPasswordEncoder.getInstance());
    encoders.put("SHA-256", new MessageDigestPasswordEncoder("SHA-256"));
    ...
~~~

스프링 시큐리티의 PasswordEncoderFactories는 `bcrypt`인코딩방식을 기본으로 채택하고 있으며 그 외
에 다양한 인코딩 방식을 지원한다.
