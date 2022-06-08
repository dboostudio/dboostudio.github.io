---
layout: post
author: Dboo
categories: 인프런-더자바_테스트-백기선
title: CH 2. Mockito
tags: LectureNote Test Java Junit5 Mockito
---

### 1. Mockito 소개

- Mock: 진짜 객체와 비슷하게 동작하지만 프로그래머가 직접 그 객체의 행동을 관리하는 객체
- 단위 테스트?
	+ 마틴 파울러의 유닛 테스트의 정의
	+ https://martinfowler.com/bliki/UnitTest.html
	+ 모든 의존성을 끊고 테스트해야만 유닛 테스트인가?
	+ 단위의 개념을 행동의 단위라고 생각하는것이 적절하다. 는 의견도 있다.
	+ Bango payment : Payment Mocking 서비스 (테스트 환경)

### 2. Mockito 시작하기

- spring-boot-starter 의존성 내에 포함되어 있다.
- 없다면, mockito-core 와 mockito-junit-jupiter를 추가해서 쓴다.

**Mock을 활용하기**
	1. Mock을 만드는 법
	2. Mock이 어떻게 동작할지 관리하는 법
	3. Mock의 행동을 검증하는 법

### 3. Mock 객체 만들기

- StudyService 인스턴스에 대한 테스트를 진행한다고 가정
- StudyService 에서 사용하는 의존성은 StudyRepository와 MemberService가 있다고 생각해보자.
- 테스트에서 다른 서비스나 리포지토리의 의존성을 끊기 위해 Mock객체를 만들어서 사용한다.
	1. Mockito.mock 으로 Mock객체 생성 (mock()을 static import해서 더 짧게 사용가능)
	2. @Mock 어노테이션으로 Mock객체 생성 (@ExtendWith(MockitoExtesion.class) 필요)
		~~~java
		@ExtendWith(MockitoExtension.class)
		class StudyServiceTest{
    		@Mock
    		MemberService memberService;
    		@Mock
    		StudyRepository studyRepository;
		
    		@Test
    		public void createStudyService() throws Exception {
        		// Mockito.mock
		//        MemberService memberService = Mockito.mock(MemberService.class);
		//        StudyRepository studyRepository = mock(StudyRepository.class);
		
        		StudyService studyService = new StudyService(memberService, studyRepository);
	
        		assertNotNull(studyService);
    		}
		}
		~~~
	3. 테스트 메소드의 파라미터로 전달
		~~~java
		@Test
	    public void createStudyService(@Mock MemberService memberService,
	                                    @Mock StudyRepository studyRepository) throws Exception {
	    }
		~~~

### 4. Mock 객체 Stubbing (행동 조작)

**기본 행동**

- void 메소드
    - 아무 일도 발생하지 않음 (예외 X)
- return값이 있는 메소드
    - null을 리턴
    - Optional 타입인 경우 Optional.empty 반환
- Mock 객체 내에
    - primitive 타입의 필드가 있으면 기본 primitive 값을 따름
    - 컬렉션은 모두 비어있는 컬렉션으로 만들어짐

**Stubbing**

- Mock 객체를 다음과 같이 조작할 수 있다.
	+ 특정한 매개변수를 받고 : 특정한 값을 리턴 or 예외 리턴
	+ 메소드가 동일한 매개변수로 여러번 호출 : 각기 다르게 행동

1. when().thenReturn()
	~~~java
	@Test
	public void createStudyService() throws Exception {
    	Member member = new Member();
    	member.setId(1L);
    	String email = "dboo.studio@gmail.com";
    	member.setEmail(email);
	
    	when(memberService.findById(1L)).thenReturn(Optional.of(member));
	
    	assertThat(memberService.findById(1L).get().getEmail()).isEqualTo(email);
    	assertThat(memberService.findById(2L).get().getEmail()).isEqualTo(email);
	
	}
	~~~
	+ 1L로 아이디를 찾을때 생성한 member를 리턴하도록 정의했다.
	+ 2L로 아이디를 찾으면 member를 리턴하지 않는다.
	
	| any()를 이용하여 어떤 파라미터가 들어와도 객체가 동일하게 행동하도록 정의하기
		* mockito.ArgumentMathcer.any 를 이용하면 어떤 아이디로 찾더라도 동일한 member객체를 반환하도록 정의할 수 있다.
		* 이 외에도 anyInt, artThat 등으로 특정한 파라미터값에 대한 조건을 걸어줄 수 있다.
	~~~java
	when(memberService.findById(any())).thenReturn(Optional.of(member));
	~~~

2. when().thenThrow(), doThrow().when()
	+ when().thenThrow() : 특정 조건에서 예외를 발생시킨다.
	+ doThrow().when() : void메소드의 특정조건에서 예외를 발생시킨다.

3. 동일하게 호출되어도 다르게 행동하도록 조작
	- then문을 여러번 호출하도록 stubbing해주면 된다.
	~~~java
	when(memberService.findById(any()))
                	.thenReturn(Optional.of(member))
                	.thenThrow(new RuntimeException())
                	.thenReturn(Optional.empty());
	~~~


### 5. Mock 객체 Verify (확인하기)

1. Mock객체의 Method호출 확인
	+ StudyService에서 createStudy메소드를 호출할 때, MemberService의 notify(Study study), notify(Member member)라는 메소드를 차례로 호출한다고 가정
	+ Test에서, verify를 활용해 notify가 호출되는지 확인
	
	~~~java
	@Test
	public void createNewStudy() throws Exception {
    	StudyService studyService = new StudyService(memberService, studyRepository);
    	assertThat(studyService).isNotNull();
	
    	Member member = new Member();
    	String email = "dboo.studio@gmail.com";
    	member.setId(1L);
    	member.setEmail(email);
	
    	when(memberService.findById(1L).thenReturn(Optional.of(member));
	
    	assertThat(memberService.findById(1L).get().getEmail()).isEqualTo(email);
	
    	Study study = new Study(10, "java");
    	when(studyRepository.save(study)).thenReturn(study);
    	studyService.createNewStudy(1L, study);
	
    	verify(memberService, times(1)).notify(study);
    	verify(memberService, times(1)).notify(any());
	
	}
	~~~
	+ notify라는 메소드가 1번 memberService에서 호출됨을 검증하는것이고, 어떤 파라미터와 함께 호출되었는지도 확인이 가능하다.
	+ verify가 확인하는 호출에서도 argumentMatcher기능을 사용하여 any(), anyInt(), ... 등을 사용할 수 있다.
	
2. Method 호출 순서 확인
	+ Method 호출 순서를 확인할 필요가 있을때에는 InOrder객체의 기능을 사용한다.
	+ 호출되는 순서 차례로 verify하면 된다.
	~~~java
	InOrder inOrder = inOrder(memberService);
    inOrder.verify(memberService).notify(study);
    inOrder.verify(memberService).notify(member);
	~~~

3. timeout 확인
	~~~java
	verify(memberService, tiemout(10000)).findById(any());
	~~~

4. 더이상 객체의 활동이 없는 것을 확인
	+ verifyNoMoreInteraction() 으로 확인한다.
	+ 주의할 것은 테스트가 깨지지 않기 위해서는 마지막 action을 verifyNoMoreInteraction() 전에 확인해야 한다.
	
### 5. BDD 스타일 Mockito API
- BDD : Behavior Driven Development 의 약자로, 어플리케이션이 어떻게 행동해야되는지에 대한 공통된 이해를 구성하는 방법.
- 행동에 대한 스펙
	+ Title
	+ Narrative : As a, I want, So that
	+ Acceptance Criteria : Given, When, Then
- BddMockito 클래스를 통해 제공됨
	+ 기존 mock의 when -> given, verify -> then
	+ times() 는 then() 뒤에 오는 shoud()안에 넣어서 적용
	+ verifyNoMoreInteractions()는 shouldHaveNoMoreInteraction()으로 적용
- BDD 스타일 적용 전후 비교
	+ 기존 mockito 스타일 TDD
		~~~java
		@Test
    	public void createNewStudy_before_BDD() throws Exception {
        	// given
        	StudyService studyService = new StudyService(memberService, studyRepository);
        	assertThat(studyService).isNotNull();
	
        	Member member = new Member();
        	String email = "dboo.studio@gmail.com";
        	member.setId(1L);
        	member.setEmail(email);
	
        	Study study = new Study(10, "java");
        	when(memberService.findById(any())).thenReturn(Optional.of(member));
        	when(studyRepository.save(study)).thenReturn(study);
	
        	// when
        	studyService.createNewStudy(1L, study);
	
        	// then
        	assertThat(member).isEqualTo(study.getOwner());
        	verify(memberService, times(1)).notify(study);
        	verifyNoMoreInteractions(memberService);
    	}
		~~~
	+ mockito BDD 스타일 적용
		~~~java
	    @Test
    	public void createNewStudy_after_BDD() throws Exception {
        // given
        StudyService studyService = new StudyService(memberService, studyRepository);
        assertThat(studyService).isNotNull();

        Member member = new Member();
        String email = "dboo.studio@gmail.com";
        member.setId(1L);
        member.setEmail(email);

        Study study = new Study(10, "java");

        given(memberService.findById(any())).willReturn(Optional.of(member));
        given(studyRepository.save(study)).willReturn(study);

        // when
        studyService.createNewStudy(1L, study);

        // then
        assertThat(member).isEqualTo(study.getOwner());
        then(memberService).should(times(1)).notify(study);
        then(memberService).shouldHaveNoMoreInteractions();
    }
		~~~

- BDD 스타일은 when().thenReturn()문이 given 영역에서 이뤄지는것이 불편한 이들을 위한 것이라고 생각된다.
- 두 스타일 모두 쓰는데에는 큰 지장이 없고, 취향껏 선택해서 쓰면 됨