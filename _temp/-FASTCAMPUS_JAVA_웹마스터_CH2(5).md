---
title: <패스트캠퍼스> Java 웹개발마스터 - CH2. 객체지향 프로그래밍(5)
tags: LectureNote Fastcampus java syntax
---

## Inner Class, Lambda Expression, Stream

### Inner Class

- 클래스 내부에 구현한 클래스, 주로 외부 클래스 생성자에 내부 클래스를 생성

- 내부 클래스 유형

|:-:|:-:|:-:|:-:|
|종류|구현 위치|사용할 수 있는 외부 클래스 변수|생성 방법|
|-|-|-|-|
|인스턴스 내부 클래스|외부 클래스 멤버 변수와 동일|외부 인스턴스 변수<br>외부 전역 변수|외부 클래스를 먼저 만든 후 내부 클래스 생성|
|정적 내부 클래스|외부 클래스 멤버 변수와 동일|외부 전역 변수|외부 클래스와 무관하게 생성|
|지역 내부 클래스|메소드 내부에 구현|외부 인스턴스 변수<br>외부 전역 변수|메소드를 호출할 때 생성|
|익명 내부 클래스|메소드 내부에 구현<br>변수에 대입하여 직접 구현|외부 인스턴스 변수<br>외부 전역 변수|메소드를 호출할 때 생성되거나, 인터페이스 타입변수에 대입할 때 new예약어를 사용하여 생성|

- 인스턴스/정적 내부 클래스

~~~java

class OutClass{

  private int num = 10;
  private static sNum = 20;
  private InClass inClass;

  public OutClass() {
    inClass = new InClass();
  }

  class InClass{ //인스턴스 내부 클래스
    int iNum = 100;
    //static int sInNum = 200; static클래스 아니므로 사용불가

    void inTest() {
      System.out.println(num); //10
      System.out.println(sNum); //20
    }
  }

  public void usingInTest() {
    inClass.inTest();
  }

  static class InStaticClass {
    int inStaticNum = 100;
    static int sInStaticNum = 200;

    void inStaticTest() {
      System.out.println(num); //10
      System.out.println(sNum); //20
      System.out.println(inStaticNum); //100
      System.out.println(sInStaticNum); //200
    }

    static void sTest() { //Inner Static Class 안 Static method
      System.out.println(num); //10
      System.out.println(sNum); //20

      //아래 println은 전부 에러. 인스턴스 생성 후 멤버 변수 사용가능하기 때문에 클래스이름만으로 바로 호출은 안될것.

      // System.out.println(inStaticNum); //100
      // System.out.println(sInStaticNum); //200
    }
  }

}

public class Main {
  psvm{
    OutClass ouclass = new OutClass();
    outClass.usingInTest(); //inTest호출됨

    OutClass.InClass inClass = outClass.new InClass(); //InClass 인스턴스만들기 private아닐 때만 가능

    OutClass.InStaticClass.sTest(); //스태틱메소드 호출

    OutClass.InStaticClass sInClass = new OutClass.InStaticClass();
    sInClass.inStaticTest();
  }
}

~~~

- 지역/익명 내부 클래스

~~~java

class Outer {

  int outNum = 100;
  static int sNum = 200;

  Runnable getRunnable() {

    class MyRunnable implements Runnable {

      @Override
      public void run() {
        System.out.println(outNum);
        System.out.println(sNum);
      }
    }

    return new MyRunnable();
  }

}

~~~
