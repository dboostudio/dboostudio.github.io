---
title: <인프런> Vue.js 시작하기 CH3. Vue 인스턴스 소개
tags: LectureNote Inflearn Vue.js
---

ref : [장기효님 블로그](https://joshua1988.github.io/web-development/vuejs/vuejs-tutorial-for-beginner/), Inflearn Vue.js시작하기 강의

## Vue Instance

Vue Instance 는 뷰로 개발할 때 필수로 생성해야하는 코드이다.

### Create Vue Instance

인스턴스는 아래와 같이 생성하고, 해당 인스턴스를 콘솔에 출력하면 어떤 속성과 API가 있는지 확인 할 수 있다.

~~~javascript
var vm = new Vue();
console.log(vm);
~~~

라이브 서버 콘솔에서 다음과 같이 확인 가능하다.

![](/assets/img/LectureNote/Inflearn/startvue/vue-instance.png)

뷰에서 제공하는 API와 속성들이 나오는 것을 볼 수 있다.

그럼 이 뷰 인스턴스를 어떻게 사용하느냐 하면, vue인스턴스 생성자 함수에서 vue인스턴스 기능을 사용하고 싶
은 태그를 연결해준다.

~~~html
<div id="app"></div>

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    var vm = new Vue({
        el: '#app',
        // #app을 찾아서 vue인스턴스를 붙여준다.
        // vue 인스턴스를 붙이게 되면 id가 app인 태그에서 vue인스턴스가 가진 기능들을 사용할 수 있게 된다.
        data: {
            message: 'hi'
        }
    });
    console.log(vm);
</script>
~~~

이렇게 연결하고 나서 뷰 개발자 탭으로 가보자.

![](/assets/img/LectureNote/Inflearn/startvue/vue-instance-apply.png)

Root라는 녀석이 생겼고, 그녀석의 data속성안에 message에 'hi'가 들어있는 것을 확인 가능하다.

## 인스턴스와 생성자 함수

javscript에서 객체를 만들어 사용할때 다음과 같이 한다.

![](/assets/img/LectureNote/Inflearn/startvue/object-constructor.png)

vue에서는 생성자를 활용할 때 다음과 같이한다.

![](/assets/img/LectureNote/Inflearn/startvue/vue-constructor.png)

이렇게 vue생성자를 활용하여 필요한 공통기능을 정의해놓으면 vue인스턴스를 사용할때 재사용 가능하도록 할 수
있고 vue인스턴스를 쓰는 가장 큰 이유라고 할 수 있다.

## Vue 인스턴스 옵션

인스턴스에서 사용할 수 있는 속성과 API에 대해 간략히 알아보자.

~~~javascript
new Vue({
  el: ,
  template: ,
  data: ,
  methods: ,
  created: ,
  watch: ,
});
~~~

- el : 인스턴스가 그려지는 화면의 시작점(특정 HTML태그)
- template : 화면에 표시할 요쇼 (HTML, CSS 등)
- data : 뷰의 반응성(Reactivity)가 반영된 데이터 속성
- methods : 화면의 동작과 이벤트 로직을 제어하는 메소드
- created : 뷰의 라이프 사이클과 관련된 속성
- watch : data에서 정의한 속성이 변화했을 때, 추가동작을 수행할 수 있게 정의하는 속성
