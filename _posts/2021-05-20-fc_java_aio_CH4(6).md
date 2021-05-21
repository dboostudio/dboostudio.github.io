---
title: <패스트캠퍼스> Java 웹개발마스터 - CH4. 스프링 입문(6) IoC/DI
tags: LectureNote Fastcampus Spring
---

## Spring Framwork

- "테스트의 용이성", "느슨한 결합"에 중점을 두고 개발

![](/assets/img/LectureNote/Fastcampus/Java_All_In_One/spring-pojo.png)

- 가운데 POJO를 중심에 두고, 3가지 특징을 중심으로 개발하도록 설계된 프레임워크이다.

## IoC / DI

### IoC (Inversion of Control)

- 제어의 역전이라는 뜻이다. 스프링에서는 일반적인 Java 객체를 new로 생성하여 개발자가 관리하는 것이 아
니라, Spring Container에게 맡긴다. 객체 제어의 권한을 프레임워크가 갖고 있음을 뜻한다.

- 프레임워크의 Spring Container안에서 싱글톤 객체로 관리된다.

### DI (Dependency Injection)

- 의존성주입이란 뜻으로, Spring Container로부터 사용할 객체를 주입받아 사용한다.
- 코드 테스트에 용이하다.
- DI를 통하여, Mock을통해 안정적 테스트가 가능하다.
- 코드를 확장하거나 변경 할 때 영향을 최소화 한다.
- 순환참조를 막을 수 있다.

### DI 코드 예제

- IoC/DI 적용 전

~~~java
public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";

    // Base 64 encoding
    Base64Encoder encoder = new Base64Encoder();
    String result = encoder.encode(url);
    System.out.println(result);
}
~~~

~~~java
public class Base64Encoder {
    public String encode(String message){
        return Base64.getEncoder().encodeToString(message.getBytes());
    }
}
~~~

위는 그저 Encoder라는 객체를 통해 url을 인코딩했다.  
그런데 여기서 여러개의 Encoder가 늘어난다면 어떻게 될까? URLEncoder를 추가하면 아래와 같다.

~~~java
public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";

    // Base 64 encoding
    Encoder encoder = new Encoder();
    String result = encoder.encode(url);
    System.out.println(result);

    // URL encoding
    UrlEncoder urlEncoder = new UrlEncoder();
    String urlResult = urlEncoder.encode(url);
    System.out.println(urlResult);
}
~~~

~~~java
public class UrlEncoder {
    public String encode(String message){
        try {
            return URLEncoder.encode(message, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            return null;
        }
    }
}
~~~

이제 여기에서 추상화를 들어간다.

~~~java
public interface IEncoder {
    String encode(String message);
}
~~~

encode메소드를 정의하는 인터페이스를 하나 만들고, 기존 Base64Encoder, UrlEncoder가 이를 상속받
도록 한다. 그러면 메인함수에서 다음과 같이 쓸수있다.

~~~java
public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";
    // 추상화 후
    IEncoder encoder = new Base64Encoder();
    String iResult = encoder.encode(url);
    System.out.println(iResult);

}
~~~

이제 DI가 들어가는 부분이 나온다. 매번 IEncoder를 new해서 써야하는 불편함을 제거하기 위해서, 다음과
같이 한다.

IEncoder를 멤버변수로 받고, encode를 정의한 Encoder클래스를 하나 정의한다.

~~~java
public class Encoder {

    private IEncoder iEncoder;

    public Encoder(){
        this.iEncoder = new Base64Encoder();
    }

    public String encode(String message){
        return iEncoder.encode(message);
    }
}
~~~

이렇게 하면 main이 다음과 같이 바뀐다.

~~~java
public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";

    Encoder encoder = new Encoder();
    String result = encoder.encode(url);
    System.out.println(result);
}
~~~

매우 간단해진것을 알 수 있다.  
그런데 지금은 Base64Encoder로 인코딩하는 encoder이기때문에 여러 인코더를 사용하려며 어떻게 하느냐?
이것이 DI의 핵심이다.

DI는 외부에서 내가 사용하는 객체를 주입받는 것이다.  
방법은 간단하다. 첫째로 Encoder객체의 생성자에서 어떤 인코더를 사용할것인지를 주입받는다.

~~~java
public class Encoder {

    private IEncoder iEncoder;

    public Encoder(IEncoder iEncoder){
        this.iEncoder = iEncoder;
    }

    public String encode(String message){
        return iEncoder.encode(message);
    }
}
~~~

그 후, main에서 사용할 때에는 인코더를 new해서 넘겨주면 된다.

~~~java
public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";

    // Encoder encoder = new Encoder(new Base64Encoder()); 사용하고 싶은 인코더를
    Encoder encoder = new Encoder(new UrlEncoder()); // 주입시켜 주면 된다.
    String result = encoder.encode(url);
    System.out.println(result);
}
~~~

그렇다면 IoC라 함은, 위와같은 의존 주입을 Spring Container가 해준다고 이해할 수 있다.
