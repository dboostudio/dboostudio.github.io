---
layout: post
author: Dboo
categories: 인프런-스프링기반_REST_API_개발-백기선
title: CH 2. 이벤트 생성 API 개발
tags: LectureNote REST API
---

## 이벤트 생성 API 개발

### 1. 이벤트 API 테스트 클래스 생성, 201 응답받기

- 먼저 EventController의 Test클래스를 만들어준다.

~~~java
@ExtendWith(SpringExtension.class)
@WebMvcTest
public class EventControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Autowired
    ObjectMapper objectMapper;

    @Test
    public void createEvent() throws Exception {

        Event event = Event.builder()
                .id(100L)
                .name("dboo")
                .description("dboo desc")
                .beginEnrollmentDateTime(LocalDateTime.of(2022, 03, 22, 9, 40))
                .closeEnrollmentDateTime(LocalDateTime.of(2022, 03, 24, 9, 40))
                .beginEventDateTime(LocalDateTime.of(2022, 4, 1, 9, 40))
                .endEventDateTime(LocalDateTime.of(2022, 4, 3, 9, 30))
                .basePrice(100)
                .maxPrice(200)
                .free(true)
                .offline(true)
                .eventStatus(EventStatus.PUBLISHED)
                .build();

        mockMvc.perform(post("/api/events/")
                        .contentType(MediaType.APPLICATION_JSON)
                        .accept(MediaTypes.HAL_JSON_VALUE)
                        .content(objectMapper.writeValueAsString(event)))
                .andDo(print())
                .andExpect(status().isCreated())
                .andExpect(jsonPath("id").exists())
                .andExpect(header().exists(HttpHeaders.LOCATION))
                .andExpect(header().string(HttpHeaders.CONTENT_TYPE, MediaTypes.HAL_JSON_VALUE));
    }

}
~~~

- 강좌에서는 JUnit4를 사용하고 있지만, JUnit5의 변경점을 반영해서 작성했다.

- @WebMvcTest -> 웹 관련 빈들만 등록해준다. (슬라이싱 테스트)

- MockMvc
    + 스프링 MVC 테스트 핵심 클래스
    + MVC 요청 과정을 확인 가능, 컨트롤러 테스트로 주로 사용
    + DispatcherServlet가 개입되어 있어 단위테스트라고 보기 어렵다

- Location헤더가 존재하는지, Content-Type이 알맞은지는 맨 아래서 검증하고 있다.

~~~java
@Controller
@RequestMapping(value = "/api/events", produces = MediaTypes.HAL_JSON_VALUE)
public class EventController {

    @PostMapping
    public ResponseEntity createEvent(@RequestBody Event event){

        // location URL 만들기
        URI uri = linkTo(methodOn(EventController.class).createEvent(null)).slash("{id}").toUri();
        event.setId(Long.valueOf(10));
        return ResponseEntity.created(uri).body(event);
    }

}
~~~

- 강좌에서는 produces 옵션에 MediaTypes.HAL_JSON_UTF89_VALUE 를 사용했었는데 표준이 아니라서 없어졌다고 한다.

### 2. 이벤트 Repository

- 이벤트 객체를 DB에 저장하도록 설정해본다.
- 기존에 Entity 및 Id 설정은 해줬고, 추가로 Enum이라고 알려주는 부분만 추가한다.

~~~java
@Enumerated(EnumType.STRING)
private EventStatus eventStatus;    
~~~

#### Repository 인터페이스 생성

~~~java
@Repository
public interface EventRepository extends JpaRepository<Event, Long> {   
}
~~~

다음으로 Controller에서 event객체를 저장해준다.

~~~java
Event newEvent = eventRepository.save(event);
URI uri = linkTo(methodOn(EventController.class).createEvent(null)).slash(newEvent.getId()).toUri();
return ResponseEntity.created(uri).body(newEvent);
~~~

- 이 상태에서 테스트를 돌리면 제대로 동작하지 않을 것.
- @WebMvcTest는 웹에 관련된 의존성만을 가져오기 때문에 Repository를 가져오지 않는다.
    - EventRepository를 Mock빈으로 만들어서 사용한다.  
        -> mock객체는 return이 null이 나와서 NPE발생. stubbing을 해줘야 동작할 것  
        ~~~java
        @MockBean
        EventRepository eventRepository;
        ~~~
        ~~~java
        when(eventRepository.save(event)).thenReturn(event);
        ~~~

### 3. 입력값 제한하기

- id, free, offline 필드값은 미리 정해지거나 계산되어야 한다.
- DTO를 사용하는 방법으로 해결해보자.
    + @Ignore~~~ 어노테이션으로 해결도 가능하지만, 너무 어노테이션이 많아질 수 있기 때문에(그 외 Validation, json관련) DTO와 Entity를 분리해서 사용하자.  

~~~java
@Builder @NoArgsConstructor @AllArgsConstructor
@Data
public class EventDto {

    private String name;
    private String description;
    private LocalDateTime beginEnrollmentDateTime;
    private LocalDateTime closeEnrollmentDateTime;
    private LocalDateTime beginEventDateTime;
    private LocalDateTime endEventDateTime;
    private String location; // (optional) 이게 없으면 온라인 모임
    private int basePrice; // (optional)
    private int maxPrice; // (optional)
    private int limitOfEnrollment;

}
~~~

- EventDto를 분리해서 받을 필드를 명시하면서 입력값을 제한했다.
- 기존 컨트롤러에서 Entity객체로 받던 것을 Dto로 바꾸는데, repository에는 Entity를 사용해야하기 때문에 ModelMapper 를 이용해서 객체를 변환해준다.
- Dto 클래스에 converToEvent 라던지 하는 함수를 정의해서 Event객체를 반환하는것도 좋은 방법이다.

- 그런데 테스트에서 오류가 난다. stubbing해준 event객체와 EventDto를 통해서 만든 event객체가 달라서 그렇다고 한다.
- Repository Mocking을 빼고 실제 Repository에 저장을 하면서 테스트를 진행한다.

- 더 이상 Slicing 테스트가 아니기 때문에, @WebMvcTest -> @SpringBootTest 로 변경하고 MockMvc 객체를 사용하기 위해 @AutoConfigureMockMvc를 붙여준다.
- 추가로, 입력값이 잘 제한되었는지 확인하는 코드를 추가한다.

~~~java
        .andExpect(jsonPath("id").value(Matchers.not(100)))
        .andExpect(jsonPath("free").value(Matchers.not(true)))
        .andExpect(jsonPath("eventStatus").value(EventStatus.DRAFT.name()))
~~~

