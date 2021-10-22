---
title: <인프런> Vue.js 시작하기 CH2. Vue.js 소개
tags: LectureNote Inflearn Vue.js
---

ref : [장기효님 블로그](https://joshua1988.github.io/web-development/vuejs/vuejs-tutorial-for-beginner/), Inflearn Vue.js시작하기 강의

## Vue.js 살펴보기

### Vue는 무엇인가?

MVVM패턴의 뷰모델(ViewModel)레이어에 해당하는 화면단 라이브러리

![](/assets/img/LectureNote/Inflearn/startvue/what-is-vue.png)

DOM Listener 가 DOM단에서 일어나는 이벤트를 받아서 javascript단에 전달하고, 거꾸로 javascript
단에서 일어나는 데이터 변화 등을 Data Bindings를 통해 화면으로 반영하게 된다.

하기 항목은 강의자이신 장기효님 블로그에서 발췌한 설명문 입니다.

- 데이터 바인딩과 화면 단위를 컴포넌트 형태로 제공하며, 관련 API 를 지원하는데에 궁극적인 목적이 있음
- Angular에서 지원하는 양방향 데이터 바인딩 을 동일하게 제공
- 하지만 컴포넌트 간 통신의 기본 골격은 React의 단방향 데이터 흐름(부모 -> 자식)을 사용
- 다른 프런트엔드 프레임워크(Angular, React)와 비교했을 때 상대적으로 가볍고 빠름.
- 문법이 단순하고 간결하여 초기 학습 비용이 낮고 누구나 쉽게 접근 가능

### VSCode 코드 실습 중 팁

- 빈 HTML파일에 !+tab 을 입력하면 기본 HTML 코드를 자동 완성 해준다.
- div 생성할때, `div#app` 이라고만 쳐도 `<div id="app"></div>`로 만들어준다.
- 기본적으로 주로 쓰는 태그들은 자동완성을 전부 지원한다. script태그도 마찬가지.

### 기존 프론트 작업방법

~~~html
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div id="app">
        <!-- hell world! -->
    </div>
    <script>
        // $('#app');과 동일한 하기 코드
        var div = document.querySelector('#app');
        // console.log(div);
        div.innerHTML = 'hello World!!!';
        //자바스크립트의 기능 : 해당 태그나 돔의 정보를 조작하는것.

        var str = 'hello world!';
        div.innerHTML = str;

        str = 'hello world?'; // 바뀐 데이터 내용은 화면에 반영되지 않는다.

        // 기존방법
        div.innerHTML = str;

        // 여기까지가 기존 프론트 개발 방식

    </script>
</body>

</html>
~~~

### Reactivty 구현

~~~html
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div id="app">

    </div>

    <script>
        var div = document.querySelector('#app');
        var viewModel = {};

        // Object.defineProperty(대상 객체, 객체의 속성, {
        //     //정의할 내용
        // });
        //객체의 동작을 재정의하는 api이다.
        //mdn object defineproperty로 검색하면 문서로 볼 수 있음.

        Object.defineProperty(viewModel, 'str', {
            // 속성에 접근했을 때 동작을 정의
            get: function () {
                console.log('접근');
            },
            // 속성에 값을 할당했을 때 동작을 정의
            set: function (newValue) {
                console.log('할당' + newValue);
                div.innerHTML = newValue;
            }
        });
        // str값을 바꿀때마다, 화면에 실시간으로 반영된다.
        // 이것을 reactivity라고 하며, view의 핵심이다.
        // 어떤 값의 변경을 라이브러리에서 감지하여 화면까지 적용시켜주는 것이다.
    </script>
</body>

</html>
~~~

위와 같이 viewModel 의 str이라는 속성에 값을 접근하거나 부여할때마다 어떤 동작을 정의를 해놓고 해당 동
작을 실행시키면,

![](/assets/img/LectureNote/Inflearn/startvue/reactivity.png)

값을 변경할 때마다 화면에 새로운 값이 표출되게 된다.

## Reactivity 코드 라이브러리화

vue에서는 다음과 같은 구조로 짜여져 있다고 이해하면 좋다.

~~~html
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div id="app">

    </div>

    <script>
        var div = document.querySelector('#app');
        var viewModel = {};

        // Object.defineProperty(대상 객체, 객체의 속성, {
        //     //정의할 내용
        // });
        //객체의 동작을 재정의하는 api이다.
        //mdn object defineproperty로 검색하면 문서로 볼 수 있음.

        (function () { //즉시 실행 함수 문법, 변수 유효범위를 관리하기 위함이다.
            function init() {
                Object.defineProperty(viewModel, 'str', {
                    // 속성에 접근했을 때 동작을 정의
                    get: function () {
                        console.log('접근');
                    },
                    // 속성에 값을 할당했을 때 동작을 정의
                    set: function (newValue) {
                        console.log('할당' + newValue);
                        render(newValue);
                    }
                });
                // str값을 바꿀때마다, 화면에 실시간으로 반영된다.
                // 이것을 reactivity라고 하며, view의 핵심이다.
                // 어떤 값의 변경을 라이브러리에서 감지하여 화면까지 적용시켜주는 것이다.
            }

            function render(value) {
                div.innerHTML = value;
            }

            init();
        })();

    </script>
</body>

</html>
~~~

기존에 있던 Object.defineProperty 함수를 init함수로 정의하고 render함수를 추가해 innerHTML에
값을 적용하는 부분을 따로 떼낸다. 마지막으로 즉시실행함수로 감싸 변수의 scope를 컨트롤한다.

위와 같은 방식으로 vue.js라이브러리가 이루어져있다고 생각하면 편하다.

## Hello Vue.js와 Vue개발자 도구

일단 getting-started 코드를 보자.

~~~html
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Getting Started</title>
  </head>
  <body>
    <div id="app">
      {{ message }}
    </div>

    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
      new Vue({
        el: '#app',
        data: {
          message: 'Hello Vue.js'
        }
      })
    </script>
  </body>
</html>
~~~

해당 HTML을 라이브서버로 올리면 Vue를 사용했기 때문에 Vue 개발자 도구 탭을 검사창에서 확인할 수 있다.

![](/assets/img/LectureNote/Inflearn/startvue/hello-vue-js.png)

Component창에 Root를 눌러보면, data속성안에 message가 Hello Vue.js 로 나타나있다. 해당 값을
더블클릭하여 변경하면 화면에도 적용된다.
