---
title: <패스트캠퍼스> Java 웹개발마스터 - CH3. 스프링 입문(2) 싱글톤 패턴
tags: LectureNote Fastcampus Design_Pattern
---

## Proxy Pattern

Proxy는 대리인 이라는 뜻으로, 뭔가를 대신해서 처리하는 것  
Proxy Class를 통해서 대신 전달하는 형태로 설계되며, 실제Client는 Proxy로부터 결과를 받는다.  

Cache의 기능으로도 활용이 가능하다. SOLID 원칙 중에서 개방폐쇄원칙과(OCP)과 의존 역전 원칙(DIP)을 따른다.

### Proxy Pattern 구현예제

~~~java
public interface IBrowser{
  Html show();
}
~~~

~~~java
public class Html{

  private String url;

  public Html(String url){
    this.url = url;
  }
}
~~~

~~~java
public class Browser implements IBrowser {

  private String url;

  public Browser(String url){
    this.url = url;
  }

  @override
  public Html show(){
    System.out.println("browser loading html from : " + url);
    return new Html(url);
  }
}
~~~

~~~java
//psvm
Browser browser = new Browser("www.somthing.com");
browser.show();
browser.show();
browser.show();
browser.show();
~~~

여기까지는 `browser.show();`를 할때마다 매번 불러오게 되는데, 프록시 패턴을 사용하여 캐시기능을 만들어
봅시다.

~~~java
public class BroswerProxy implements IBroswer {

  private String url;
  private Html html;

  public BroswerProxy(String url){
    this.url = url;
  }

  @override
  public Html show {
    if(html==null){
      this.html = new Html(url);
      System.out.println("BroswerProxy loading html from : " + url);
    } else {
      System.out.println("BroswerProxy use cache from : " + url);
    }

    return html;
  }

}
~~~

이제 캐싱기능이 추가된 BroswerProxy를 통해서 show()메소드를 호출해보자.

~~~java
//psvm
IBroswer broswer = new BroswerProxy("www.somthing.com");
browser.show(); //처음에만 loading으로 호출되고,
browser.show(); //이 아래로는 cache로부터 로드된다.
browser.show();
browser.show();
~~~

### AOP 패턴

~~~java
public class AopBroswer implements IBroswer {
  private String url;
  private Html html;
  private Runnable before;
  private Runnable after;

  public AopBroswer(String url, Runnable before, Runnable after){
    this.url = url;
    this.before = before;
    this.after = after;    
  }

  @override
  public Html show {
    before.run();

    if(html==null){
      this.html = new Html(url);
      System.out.println("AopBroswer loading html from : " + url);
    } else {
      System.out.println("AopBroswer use cache from : " + url);
    }

    try{
      Thread.sleep(1500);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }    
    after.run();

    return html;
  }
}
~~~

~~~java
//psvm

AtomicLong start = new AtomicLong();
AtomicLong end = new AtomicLong();

IBroswer aopBroswer = new AopBroswer("www.something.com",
  ()->{
    System.out.println("before");
    start.set(System.cvurrentTimeMillis());
  },
  () ->{
    long now = System.cvurrentTimeMillis();
    end.set(now - start.get());
  }
);
aopBroswer.show();
System.out.println("loading time : " + end.get());
aopBroswer.show();
System.out.println("loading time : " + end.get());
~~~
처음 show()메소드를 호출 하면 1500ms보다 살짝 더 걸리는 지연시간이 나오는데, 두번째 로딩할 때는 로딩
시간이 0ms으로 나온다. 이게 캐싱의 힘인가..?
