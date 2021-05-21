---
title: <패스트캠퍼스> Java 웹개발마스터 - CH4. 스프링 입문(7) IoC
tags: LectureNote Fastcampus Spring
---

### IoC 코드 예제

일단 Spring Application 프로젝트를 하나 만든다. Spring Initializer로 간단한게 만들 수 있다.

이전 포스트에서 만들었던 클래스 및 인터페이스를 모조리 복사해 온 후, 패키지 명을 맞춰준다.

DI예제에서는 주입하는 객체를 개발자가 직접 new키워드를 통해 만들어서 객체를 직접 주입하고 있다.

Spring Framework에서는 IoC특성상 주입되는 객체를 Spring Container가 관리해주도록 할 수 있다.

어떻게 하느냐? 클래스위에 `@Component`라는 어노테이션을 붙이면 스프링이 해당 클래스의 인스턴스를 `빈`
객체로써 등록을 해 주게 된다. `빈`객체는 싱글톤형태로 관리되게 된다.

그러면, 우리는 등록된 `빈`객체에 어떻게 접근하고 사용할 수 있는가?  
ApplicationContextAware를 스프링으로부터 상속받아서 정의하고 사용가능하다.

~~~java
@Component
public class ApplicationContextProvider implements ApplicationContextAware {

    private static ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        context = applicationContext;
    }

    public static ApplicationContext getContext(){
        return context;
    }
}
~~~

그리고, 기존 Encoder에 IEncoder를 바꾸는 set메소드를 하나 만들어주자.

~~~java
public class Encoder {

    private IEncoder iEncoder;

    public Encoder(IEncoder iEncoder){
        this.iEncoder = iEncoder;
    }

    public void setIEncoder(IEncoder iEncoder){
        this.iEncoder = iEncoder;
    }

    public String encode(String message){
        return iEncoder.encode(message);
    }
}

~~~

이러면 다음과 같이 메인에서 사용 가능하다.

~~~java
@SpringBootApplication
public class IocApplication {

    public static void main(String[] args) {
        SpringApplication.run(IocApplication.class, args);

        ApplicationContext context = ApplicationContextProvider.getContext();

        Base64Encoder base64Encoder = context.getBean(Base64Encoder.class);
        UrlEncoder urlEncoder = context.getBean(UrlEncoder.class);

        Encoder encoder = new Encoder(base64Encoder);

        String url = "www.naver.com/boooks/it?page=10&size=20&name=spring-boot";
        String result = encoder.encode(url);
        System.out.println(result);

        encoder.setIEncoder(urlEncoder);
        result = encoder.encode(url);
        System.out.println(result);
    }
}
~~~

아직까지는 Encoder도 직접 관리하고 있는데, 이또한 스프링에게 제어하도록 할 것이다. 일단 Encoder에
`@Component`어노테이션을 붙여보자.

~~~java
@Component
public class Encoder {

    private IEncoder iEncoder;

    public Encoder(@Qualifier("base64Encoder") IEncoder iEncoder){
        this.iEncoder = iEncoder;
    }

    public void setIEncoder(IEncoder iEncoder){
        this.iEncoder = iEncoder;
    }

    public String encode(String message){
        return iEncoder.encode(message);
    }
}
~~~

그러면, 생성자쪽에서 에러가 나는것을 알 수 있는데 이는 Spring이 등록된 Bean중에 어떤 녀석을 붙여야 할
지 알수 없기 때문에 생기는 에러이다. `@Qualifier`어노테이션을 통해 명시해주도록 하자.

`빈`객체의 이름은 기본적으로 클래스명 맨앞글자만 소문자로 바꿔주면 되는데, 혹시 따로 빈객체의 이름을 명명
하고 싶으면 `@Component("명명할이름")`으로 어노테이션을 붙여주면 된다.

이제 Encoder도 빈객체로 등록했기 때문에 메인을 정리해보자.

~~~java
@SpringBootApplication
public class IocApplication {

    public static void main(String[] args) {
        SpringApplication.run(IocApplication.class, args);

        ApplicationContext context = ApplicationContextProvider.getContext();

//        Base64Encoder base64Encoder = context.getBean(Base64Encoder.class);
//        UrlEncoder urlEncoder = context.getBean(UrlEncoder.class);

        Encoder encoder = context.getBean(Encoder.class);

        String url = "www.naver.com/boooks/it?page=10&size=20&name=spring-boot";
        String result = encoder.encode(url);
        System.out.println(result);

        encoder.setIEncoder(context.getBean(UrlEncoder.class));
        result = encoder.encode(url);
        System.out.println(result);
    }
}
~~~

한클래스내에서 여러개의 빈을 등록하고 싶을때에는 다음과 같이 한다.

~~~java
@Configuration
class AppConfig{

    //Base64Encoder, UrlEncoder가 빈으로 등록되어있기때문에 아래 주입받는 곳에 자동으로 매칭됨.

    @Bean("base64Encode")
    public Encoder encoder(Base64Encoder base64Encoder){
        return new Encoder(base64Encoder);
    }

    @Bean("urlEncode")
    public Encoder encoder(UrlEncoder urlEncoder){
        return new Encoder(urlEncoder);
    }
}
~~~

사용할때는 다음과 같이 한다.

~~~java
@SpringBootApplication
public class IocApplication {

    public static void main(String[] args) {
        SpringApplication.run(IocApplication.class, args);

        ApplicationContext context = ApplicationContextProvider.getContext();

        Encoder encoder = context.getBean("base64Encode", Encoder.class);
        String url = "www.naver.com/boooks/it?page=10&size=20&name=spring-boot";
        String result = encoder.encode(url);
        System.out.println(result);
    }
}
~~~

이제 개발자가 관리하는 객체는 없고 전부 Spring이 객체를 관리하고 있다. IoC특징이 적용된 것이다. 등록된
빈의 생명주기를 스프링이 관리를 해준다.
