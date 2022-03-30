---
layout: post
author: Dboo
categories: 인프런-더자바_테스트-백기선
title: CH 1. JUnit5
tags: LectureNote Test Java Junit5
---

### 1. JUnit5 소개

- JUnit5 구성 : JUnit Platform 위에 Jupitor, Vintage가 올라가는 형태
    - Platform : 테스트를 실행해주는 런처 제공. TestEngine API 제공
    - Jupitor : TestEngine API 구현체
    - Vintage : JUnit 4,3을 지원하는 TestEngine API 구현체

- SpringBoot 2.2 이상부터 JUnit5가 기본으로 설정되어 있다. (JUnit-Jupitor-Engine)

### 2. JUnit5 시작하기

- Test Class나 Method를 public으로 선언하지 않아도 된다. (Java의 Reflection..?)

- 어노테이션 소개
    + @BeforeAll : 모든 테스트 전 1번만 수행된다. static void이어야 함.
    + @AfterAll : 모든 테스트 후 1번만 수행된다. static void이어야 함.
    + @BeforeEach : 모든 테스트 함수 실행 전 1번씩 수행된다.
    + @AfterEach : 모든 테스트 함수 실행 후 1번씩 수행된다.
    + @Disabled : 전체 테스트 실행 시 무시된다.

### 3. JUnit5 테스트 이름 표시하기

- 테스트 이름 표기 전략 : 기본은 함수 이름이 나온다.

- @DisplayNameGeneration()
    + 클래스, 함수에 사용가능
    + 테스트 함수명 표기 전략을 설정해줄 수 있다.
    + ex) @DisplayNameGeneration(DisplayNameGeneration.ReplaceUnderscores.class)
        * `_`를 빈 공백 문자로 변경

- @DisplayName(`원하는 함수 디스플레이명`)을 권장

### 4. Assertion

- Assertions
    + assertThat
    + assertNotNull
    + assertTrue
    + assertEquals ...
    + assertTimeout (Blocking)

- Suppliers
    + 아래와 같이 assert검증실패 시 출력할 메세지를 넣어줄 수 있다.
    + ex) assertEquals(~~~, ~~~, () -> "두 인자가 같아야한다.");
    + assertThat()을 쓰는경우, 체인메소드로 마지막에 `.withFailMessage()` 를 활용한다.

- assertAll()
    + assertAll() 내부에 assert문들을 묶으면 assert함수 여러개를 테스트 통과하지 못할 때 한번에 알 수 있다.  
        ~~~java
        assertAll(
            () -> assertNotNull(~~~~),
            () -> assertEquals(~~~), 
            ...
        );
        ~~~

### 5. 조건에 따라 테스트 실행하기 (Assume)

- 테스트 코드를 어떤 조건(OS, java version, 환경변수 등)에 따라 실행하거나 실행하지 말아야 한다면

- assumeTrue()
    ~~~java
    assumeTure("LOCAL".equalsIgnoreCase(System.getenv("TEST_ENV")));
    ~~~

- assumingThat()
    ~~~java
    assumingThat(~~~condition~~~, () -> {});
    ~~~

- @EnabledOnOs({OS.MAC, OS.LINUX}), @DisabledOnOs()
- @EnabledOnJre(JRE.JAVA_8), @DisabledOnJre()

### 6. 태깅, 필터링

- 태스트 태깅
    + 테스트 그룹화
    + @Tag("description")
    + 인텔리제이] Test실행 Configuration에서 Test kind 를 변경하여 같은 태그의 테스트 그룹만 실행 가능
    + 커맨드라인에서 그룹 테스트 실행하려면, 빌드 설정을 해주어야 한다.
    
### 7. 커스텀 어노테이션

~~~java

@Target(ElementType.METHOD) //어노테이션을 어디에 적용할 것인지
@Retention(RetentionPolicy.RUNTIME)

// Composed Annotation 내용을 아래와 같이 달아준다.
@Test
@Tag("fast")

public @interface FastTest {
}

~~~

- 위와 같이 커스텀 어노테이션을 만들어 자주 사용하거나 분류할 어노테이션을 커스텀하게 만들어 사용할 수 있다.
- 위 경우, @FastTest어노테이션 하나로 @Test, @Tag("fast") 두 개의 어노테이션 효과를 줄 수 있다.

### 8. 테스트 반복하기

#### RepeatedTest

~~~java
@DisplayName("반복테스트")
@RepeatedTest(value = 10, name = "{displayName}, {currentRepetition}/{totalRepetitions}")
public void repeat(RepetitionInfo info) throws Exception {
    System.out.println(info.getCurrentRepetition() + "times");
}
~~~

#### ParameterizedTest

- 반복테스트시, 매번 다른 값을 가지고 테스트를 하고 싶다.
    + @ParameterizedTest
    + @ValueSource

~~~java
@DisplayName("반복테스트_파라미터")
@ParameterizedTest(name = "{index} {displayName} message={0} ")
@ValueSource(strings = {"날씨가", "슬슬", "더워지고", "있네요"})
public void param_repeat(String message) throws Exception {
    System.out.println(message);
}
~~~


- @ValueSource()
    + ints, strings, .... 여러가지 인자를 사용가능
- 여러가지 파라미터를 사용 가능, 암묵적 타입변환 해줌(SimpleArgumentConverter구현으로 강제할 수 있음)
- @NullSource, @EmptySource, @NullAndEmptySource ... 등
- @CvsSource : 여러타입의 인자를 넘겨줄 수 있다.

~~~java
@ParameterizedTest
@CsvSource({"10, '자바'" , "20, '스프링'"})
public void csvSource(Integer integer, String string) throws Exception {
    System.out.println("integer = " + integer + ", string = " + string);
}
~~~

- ArgumentAccessor

~~~java
@ParameterizedTest
@CsvSource({"10, '자바'" , "20, '스프링'"})
public void csvSource_argumentAccessor(ArgumentsAccessor argumentsAccessor) throws Exception {
    System.out.println(argumentsAccessor.get(0) + "," + argumentsAccessor.get(1));
}
~~~

- 혹은 ArgumentAggregator를 구현해서 사용할 수 있다. (static inner class || public class 이어야 함)

~~~java
@ParameterizedTest
@CsvSource({"10, '자바'" , "20, '스프링'"})
public void csvSource_args_aggregate(@AggregateWith(StudyAggregator.class) Study study) throws Exception {
    System.out.println(study);
}

static class StudyAggregator implements ArgumentsAggregator {
    @Override
    public Object aggregateArguments(ArgumentsAccessor accessor, ParameterContext context)
        throws ArgumentsAggregationException 
    {
        return new Study(accessor.getString(1), accessor.getInteger(0));
    }
}
~~~

### 9. 테스트 인스턴스

- JUnit은 테스트 메소드마다 테스트 인스턴스를 새로 만드는 것을 기본 전략으로 한다.

- @TestInstance(TestInstance.Lifecycle.PER_CLASS)
    + 만일 테스트끼리 공유할 수 있는 변수나 메소드를 쓰고 싶다면 테스트 인스턴스의 라이프 사이클을 조정해 줄 수 있다.
    + @BeforeAll, @AfterAll을 static으로 선언할 필요가 없다.

### 10. 테스트 순서

- JUnit5에서는 기본적으로 테스트가 작성되어있는 순서대로 실행이 되기는 한다.
- 하지만 혹시 테스트 코드끼리 의존성이 있거나 순서가 필요한 경우에 강제로 순서를 줄 수 있다.
    + Use case를 테스트하는 경우나 통합테스트에 유용할 수 있다.

- @TestMethodOrder(`MethodOrder구현체`)
    + 구현체는 세가지를 제공한다.
        * Alphanumeric
        * OrderAnnotation > @Order() 어노테이션으로 설정한다. (jupitor의 것을 써야함)
        * Random

### 11. junit-platform.properties

- JUnit 설정 파일로, src/test/resources/ 경로에 junit-platform.properties파일을 만들어 적용한다.

~~~properties
#테스트 인스턴스 라이프사이클 설정
junit.jupiter.testinstance.lifecycle.default = per_class

#확장팩 자동 감지 기능
junit.jupiter.extensions.autodetection.enabled = true

#@Disabled 무시하고 실행하기
junit.jupiter.conditions.deactivate = org.junit.*DisabledCondition

#테스트 이름 표기 전략 설정
junit.jupiter.displayname.generator.default = org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
~~~

### 12. 확장 모델

- JUnit4 : @RunWith(), TestRule, MethodRule
- JUnit5 : @ExtendWith()

- Extension 등록 방법
    + 선언적 등록 : @ExtendWith
    + 프로그래밍 등록 : @RegisterExtension
    + 자동 등록 : 자바 ServiceLoader 이용

#### 1) 선언적 등록 @ExtendWith()

- 간단한 Extension을 만들어보자.

~~~java


public class FindSlowTestExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    private static final long THRESHOLD = 1000;

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        ExtensionContext.Store store = getStore(context);
        Method requiredTestMethod = context.getRequiredTestMethod();
        String methodName = requiredTestMethod.getName();
        SlowTest annotation = requiredTestMethod.getAnnotation(SlowTest.class);

        long start_time =  store.remove("START_TIME", long.class);
        long duration = System.currentTimeMillis() - start_time;


        if(duration > THRESHOLD && annotation == null){
            System.out.printf("Please consider mark method [%s] with @SlowTest \n", methodName);
        }
    }

    @Override
    public void beforeTestExecution(ExtensionContext context) throws Exception {
        ExtensionContext.Store store = getStore(context);
        store.put("START_TIME", System.currentTimeMillis());
    }

    private ExtensionContext.Store getStore(ExtensionContext context){
        String className = context.getRequiredTestClass().getName();
        String methodName = context.getRequiredTestMethod().getName();
        return context.getStore(ExtensionContext.Namespace.create(className, methodName));
    }

}

~~~

- @ExtendWith(FindSlowTestExtension.class)라고 선언하면 Extension이 적용된다.

#### 2) 프로그래밍으로 등록 @RegisterExtension

- 만일 Extension내에 어떤 변수를 선언하고, 테스트마다 다른 값을 넣어서 사용하고 싶어 생성자를 작성해뒀다면 위와같이 @ExtendWith()으로는 사용이 불가하다.
- 생성자를 이용해 선언하고, @RegisterExtension 어노테이션을 붙여준다.

~~~java
public class RegisterExtensionTest {

    @RegisterExtension
    static FindSlowTestExtension findSlowTestExtension = new FindSlowTestExtension(2000L);

    //Test Codes...
}

~~~

#### 3) ServiceLoader를 통한 자동등록

- junit-platform.properties
    + junit.jupiter.extensions.autodetection.enabled = true 로 하고, 설정을 추가로 해주면 된다고 하는데 수업에서는 소개하지 않았다.

### 13. Junit4 마이그레이션

- junit-vintage-engine을 의존성으로 추가하면, JUnit 5의 junit-platform으로 JUnit 3과 4로 작성된 테스트를 실행할 수 있다.
    + @Rule은 기본적으로 지원하지 않지만, junit-jupiter-migrationsupport 모듈(의존성 추가 필요)이 제공하는 @EnableRuleMigrationSupport를 사용하면 다음 타입의 Rule을 지원한다.
        * ExternalResource
        * Verifier
        * ExpectedException

|:-:|:-:|
|JUnit4|JUnit5|
|-|-|
|@Category(Class)|@Tag(String)|
|@RunWith, @Rule, @ClassRule|@ExtendWith, @RegisterExtension|
|@Ignore|@Disalbed|
|@Before, @After, @BeforeClass, @AfterClass|@BeforeEach, @AfterEach, @BeforeAll, @AfterAll|
