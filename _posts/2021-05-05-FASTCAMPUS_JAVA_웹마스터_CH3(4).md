---
title: <패스트캠퍼스> Java 웹개발마스터 - CH3. 스프링 입문(3) 데코레이터 패턴
tags: LectureNote Fastcampus Design_Pattern
---

## Decorator Pattern

데코레이터 패턴은 기존 뼈대(클래스)는 유지하되, 이후 필요한 형태로 꾸밀 때에 사용한다. 확장이 필요한 경우
상속의 대안으로도 활용된다. SOLID원칙 중에서 개방폐쇄원칙 (OCP), 의존역전원칙 (DIP)를 따른다.

~~~java
public interface ICar {

  int getPrice();
  void showPrice();

}
~~~

~~~java
public class Audi implements ICar {

  private int price;
  public Audi(int price){
    this.price = price;
  }

  @Override
  public int getPrice(){
    return price;
  }

  @Override
  public void showPrice(){
    System.out.println("this audi costs : " + this.price);
  }
}
~~~

~~~java
//psvm
ICar audi = new Audi(1000);
audi.showPrice(); // 1000;
~~~

아우디의 차종은 여러개이고, 가격도 제각각이므로 이걸 구현하기 위해 데코레이터 패턴을 적용해보자.

~~~java
public class AudiDecorator implements ICar{

  protected ICar audi;
  protected String modelName;
  protected int modelPrice;

  public AudiDecorator(ICar audi, String modelName, int modelPrice){
    this.audi = audi;
    this.modelName = modelName;
    this.modelPrice = modelPrice;
  }

  @Override
  public int getPrice(){
    return audi.getPrice() + modelPrice;
  }

  @Override
  public void showPrice(){
    System.out.println("audi " +modelName + "costs : " + getPrice());
  }
}
~~~

아우디 데코레이터를 다 만들었으니, 이제 아우디 모델들을 만들어 보자.

A3
~~~java
public class A3 extends AudiDecorator{
  public A3(ICar audi, String modelName){
    super(audi, modelName, 1000);
  }
}
~~~

A4
~~~java
public class A4 extends AudiDecorator{
  public A3(Audi audi, String modelName){
    super(audi, modelName, 2000);
  }
}
~~~

A5
~~~java
public class A5 extends AudiDecorator{
  public A3(Audi audi, String modelName){
    super(audi, modelName, 3000);
  }
}
~~~

아우디 모델을 전부 만들었다. 각각 가격을 A3는 +1000, A4는 +2000, A5는 +3000 해주었다. 이제 메인
함수에 적용시켜 보자.

~~~java
//psvm
ICar audi = new Audi(1000);
audi.showPrice(); // 1000;

//A3
ICar audi3 = new A3(audi, "A3");
audi3.showPrice(); // 2000
//A4
ICar audi4 = new A4(audi, "A4");
audi4.showPrice(); // 3000
//A5
ICar audi5 = new A5(audi, "A5");
audi5.showPrice(); // 4000
~~~

데코레이터 패턴을 사용하여 각 아우디 모델에 대한 가격을 바꾸도록 하였다 끗.
