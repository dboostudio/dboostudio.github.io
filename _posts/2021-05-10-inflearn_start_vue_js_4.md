---
title: <인프런> Vue.js 시작하기 CH4. 컴포넌트
tags: LectureNote Inflearn Vue.js
---

ref : [장기효님 블로그](https://joshua1988.github.io/web-development/vuejs/vuejs-tutorial-for-beginner/), Inflearn Vue.js시작하기 강의

## Vue Component

컴포넌트는 화면의 영역을 구분하여 개발할 수 있는 뷰의 기능이다. 컴포넌트 기반으로 화면을 개발하게 되면 코
드의 재사용성이 올라가고 빠르게 화면을 제작할 수 있다.

![](/assets/img/LectureNote/Inflearn/startvue/vue-component.png)

### Vue 전역 컴포넌트

~~~html
<div id="app">
    <!-- 컴포넌트 태그를 만들어 주면 컴포넌트에 등록한 내용이 바로 반영이 된다. -->
    <app-header></app-header>
    <app-content></app-content>
</div>

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

<script>
    // Vue 전역 컴포넌트
    //Vue.component('컴포넌트 이름', 컴포넌트 내용);
    Vue.component('app-header', {
        template: '<h1>Header</h1>'
    }); //컴포넌트를 등록한뒤, 사용하려면 컴포넌트 이름에 해당하는 태그를 만들어줘야한다.

    Vue.component('app-content', {
        template: '<div>content</div>'
    });

    new Vue({
        el: '#app',
    });
</script>
~~~

### Vue 지역 컴포넌트

~~~html
<div id="app">
    <!-- 지역 컴포넌트 -->
    <app-footer></app-footer>
</div>

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

<script>
    new Vue({
        el: '#app',
        components: { //객체리터럴{}
            //'키' : '값',
            //'컴포넌트 이름' : 컴포넌트 내용,
            'app-footer': {
                template: '<footer>footer</footer>'
            }
        }
    });
</script>
~~~

전역 컴포넌트인 app-header, app-content 지역 컴포넌트인 app-footer를 적용하면 vue 개발자도구에
서 다음과 같이 root아래 컴포넌트들이 생긴것을 볼 수 있다.

![](/assets/img/LectureNote/Inflearn/startvue/vue-component-apply.png)

## Vue 전역 컴포넌트와 지역 컴포넌트의 차이

- 지역 컴포넌트 등록시, component 가 아닌 components이다. s를 빼먹지 않도록 주의할 것.  
(라우터에는 component로 s가 안붙음. component를 여러개 설정할 수 있는지의 차이이다.)
- 전역 컴포넌트의 경우, 전역적으로 사용해야 하는 플러그인이나 라이브러리를 등록할 때 주로 사용한다.

## Component와 Instance의 관계

~~~html
<div id="app2">
    <!-- 전역 컴포넌트이므로, 인스턴스를 따로 생성하지 않아도 적용된다. -->
    <app-header></app-header>
    <!-- 지역 인스턴스 이므로 app2에는 app-footer가 등록되어 있지 않다. -->
    <app-footer></app-footer>
</div>
~~~

위의 태그를 추가하면, 전역컴포넌트로 생성한 인스턴스로 인하여 app-header태그는 적용이 된다.  
하지만, app-footer태그는 지역컴포넌트로 app태그에만 적용할 수 있기 때문에 app-footer는 적용되지 않
는다.
