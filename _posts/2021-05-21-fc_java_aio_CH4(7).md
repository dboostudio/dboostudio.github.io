---
title: <패스트캠퍼스> Java 웹개발마스터 - CH4. 스프링 입문(8) POST API
tags: LectureNote Fastcampus SpringBoot REST
---

## POST API

- 보통 쿼리 파라미터를 사용하지 않고, 데이터 바디에 정보를 실어서 전달한다.
- 데이터 형테는 XML이나 JSON으로 전달하고 보통 JSON을 사용한다.

짤막하게 JSON데이터 형식에 대해 알아보자.

### JSON

"key" : "value" 형식으로 나열하며, 키-값쌍들은 쉼표(,)로 구분한다.  
사용할 수 있는 변수는 다음과 같다.

1. string : value
2. number : value
3. boolean : true, false
4. object : {}
5. array : []

snake case : "phone_number"  
camel case : "phoneNumber"

~~~json
{
  "string" : "value",
  "number" : 10,
  "boolean" : false,
  "object" : {
    "var1" : "variable1",
    "var2" : 2,
  },
  "array" : [
    {
      "array1" : "1"
    },
    {
      "array2" : "2"
    }, ...
  ]
}
~~~

### POST Controller

기본적인 PostMapping은 다음과 같이한다.

~~~java
@PostMapping("/post")
public void post(@RequestBody Map<String, Object> requestData){
    requestData.forEach((key, value)->{
        System.out.println("key : " + key);
        System.out.println("value : " + value);
    });
}
~~~

GET방식과는 다르게 @RequestBody를 붙여준다.  
이번에도 DTO를 만들어서 데이터바디를 객체로 받아보자.

~~~java
public class PostRequestDto {
    private String account;
    private String email;
    private String address;
    private String password;

    //getter, setter, toString 생략
}
~~~

그러면 PostController에서

~~~java
@PostMapping("/post")
public void post(@RequestBody PostRequestDto postRequestDto){
    System.out.println(postRequestDto.toString());
}
~~~

와 같이 줄일 수 있다. GET에서와 다른 점은 DTO를 쓰더라도 @RequestBody를 생략하지 않는다.

그런데, DTO에서 받아야 하는 변수명이 snake-case로 오게 될 시, 자바에서의 camel-case와 어떻게 매핑
할것인지 문제가 있다.

~~~java
public class PostRequestDto {
    private String account;
    private String email;
    private String address;
    private String password;
    private String phoneNumber;

    //getter, setter, toString 생략
}
~~~

camel-case형식인 변수명 phoneNumber가 추가되었다.

기본적으로 JSON으로 넘어온 변수명들이 ObjectMapper를 통해서 매핑이 되는데, snake-case인 변수명을
따로 매핑정보를 줘야 한다.

- @JsonProperty 어노테이션을 이용한 변수명매핑

~~~java
public class PostRequestDto {
    private String account;
    private String email;
    private String address;
    private String password;

    @JsonProperty("phone_number")
    private String phoneNumber;

    //getter, setter, toString 생략
}
~~~

위 방법은 굳이 snake-case가 아니더라도, JSON 변수명을 매핑할때 사용할 수 있다.

이 방법 말고도 @JsonNaming 이라는 어노테이션으로도 해결할 수 있는데 이는 나중에 다시 알아보자.
