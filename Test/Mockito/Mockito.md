# 목차

<br>

- [목차](#목차)
- [Mockito](#mockito)
  - [Mock과 Mockito 개념](#mock과-mockito-개념)
  - [Mockito 사용법](#mockito-사용법)
    - [의존성 추가](#의존성-추가)
    - [Mock 객체가 필요한 이유](#mock-객체가-필요한-이유)
    - [Mock 객체 만들기](#mock-객체-만들기)
    - [Stubbing](#stubbing)
      - [모든 Mock 객체의 행동 (조작하지 않는 경우)](#모든-mock-객체의-행동-조작하지-않는-경우)
      - [Mock 객체 조작](#mock-객체-조작)
  - [Mock 객체 확인](#mock-객체-확인)
  - [BDD 스타일 API](#bdd-스타일-api)
    - [BDD란?](#bdd란)
    - [행동에 대한 스펙](#행동에-대한-스펙)
    - [Mockito에서의 BDD](#mockito에서의-bdd)
- [참고](#참고)

<br>

# Mockito

<br>

## Mock과 Mockito 개념
> 사전적 의미: 거짓된, 가짜의
* Mock
  * 진짜 객체와 비슷하게 동작하지만 프로그래머가 직접 그 객체의 행동을 관리하는 객체.
  * 실제 객체를 만들어 사용하기에 시간, 비용 등의 Cost가 높거나 혹은 객체 서로간의 의존성이 강해 구현하기 힘들 경우 가짜 객체를 만들어 사용하는 방법.
  * Java Mock Framework: Mockito, EasyMock, JMock
* Mock객체
  * **모의 객체**(Mock Object)란 주로 객체 지향 프로그래밍으로 개발한 프로그램을 테스트 할 경우 **테스트를 수행할 모듈과 연결되는 외부의 다른 서비스나 모듈들을 실제 사용하는 모듈을 사용하지 않고 실제의 모듈을 "흉내"내는 "가짜" 모듈**을 작성하여 테스트의 효용성을 높이는데 사용하는 객체이다.
  * **사용자 인터페이스(UI)나 데이터베이스 테스트 등과 같이 자동화된 테스트를 수행하기 어려운 때 널리 사용된다.**
  * 실제 객체를 만들기에는 비용과 시간이 많이 들거나 의존성이 크게 걸쳐져 있어서 테스트 시 제대로 구현하기 어려울 경우 가짜 객체를 만들어서 사용하는 기술이다.
* Mockito
  * Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공한다.

> 이미 존재하는 객체를 Mocking하는 것보다는 외부 서비스를 Mocking하여 테스트하는 것이 좋아보인다.
> 
> ex. HTTP 요청

<br>

## Mockito 사용법
> 세 가지만 알면 Mock을 활용한 테스트를 쉽게 작성할 수 있다고 한다.
> * Mock을 만드는 방법
> * Mock이 어떻게 동작해야 하는지 관리하는 방법
> * Mock의 행동을 검증하는 방법

<br>

### 의존성 추가
* 스프링부트의 경우
  * `starter-test`를 통해 의존성이 자동으로 주입된다.
* 스프링부트가 아닌경우
  * `mockito-junit-jupiter`와 `mockito-core`를 의존성으로 추가해주면 된다.

<br>

### Mock 객체가 필요한 이유
```java
@Test
void createStudyService() {
    // StudyService에 의존성을 주입해주기 위해 MemberService의 구현체를 만들어준다.
    MemberService memberService = new MemberService() {
        @Override
        public Member findById(Long memberId) {
            return null;
        }
    };

    // StudyService에 의존성을 주입해주기 위해 StudyRepository의 구현체를 만들어준다.
    StudyRepository studyRepository = new StudyRepository() {
        @Override
        public Study save(Study study) {
            return null;
        }
    };

    // StudyService를 만들기 위해선 MemberService와 StudyRepository가 필요하다.
    StudyService studyService = new StudyService(memberService, studyRepository);

    assertNotNull(studyService);
}
```
* **위와 같이 구현체는 없지만, 어떤 객체를 만들때 구현체가 필요한 경우가 Mock 객체를 만들기 가장 좋은 순간이다.**
  * 위 코드에선 StudyService를 만들기 위해선 MemberService와 StudyRepository가 필요하다.
  * 그래서 두 구현체를 직접 만들어 주입해주었다.
* **하지만 Mock을 사용할 경우 직접 구현해주지 않아도 된다.**

<br>

### Mock 객체 만들기
1. `Mockito.mock()` 메서드로 만드는 방법

```java
@Test
void createStudyService() {
    // Mock 객체를 만들어준다.
    MemberService memberService = Mockito.mock(MemberService.class);
    StudyRepository studyRepository = Mockito.mock(StudyRepository.class);

    StudyService studyService = new StudyService(memberService, studyRepository);

    assertNotNull(studyService);
}
```

<br>

2. `@Mock`애노테이션으로 만드는 방법
   * JUnit 5 extension으로 MockitoExtension을 사용해야 한다.
   * 필드와 메서드 매개변수에 사용할 수 있다.

```java
// 필드
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Mock
    private MemberService memberService;

    @Mock
    private StudyRepository studyRepository;

    @Test
    void createStudyService() {
        StudyService studyService = new StudyService(memberService, studyRepository);

        assertNotNull(studyService);
    }
}
```

```java
// 매개변수
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Test
    void createStudyService(@Mock MemberService memberService, @Mock StudyRepository studyRepository) {
        StudyService studyService = new StudyService(memberService, studyRepository);

        assertNotNull(studyService);
    }
}
```

<br>

### Stubbing
> Stubbing이란 Mock 객체 조작하는 것을 의미한다.

<br>

#### 모든 Mock 객체의 행동 (조작하지 않는 경우)
* `return`값이 있는 메서드는 `Null`을 리턴한다.
 * `Optional`의 경우는 `Optional.empty`을 리턴한다.
* 상태로 `Primitive` 타입을 가지고 있다면 기본값을 가진다. (int는 0, boolean이면 false...)
* 컬렉션은 비어있는 컬렉션
* `void` 메서드는 예외를 던지지 않고 아무런 일도 발생하지 않는다.

<br>

#### Mock 객체 조작
* 특정한 매개변수를 받은 경우 **특정한 값을 리턴하거나 예외를 던지도록 만들 수 있다.**
  * [Mockito Stubbing 공식 문서](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#stubbing)
  * [Argument Matchers (여러가지 매개변수 조작 방법)](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#argument_matchers)
```java
Member member = new Member(1L, "binghe819@naver.com");

// Stubbing -> 이 메서드 안에서 memberService.findById(1L)을 하면 위의 member가 리턴된다.
when(memberService.findById(1L)).thenReturn(member);
assertAll(
    () -> assertEquals(memberService.findById(1L).getId(), 1L),
    () -> assertEquals(memberService.findById(1L).getEmail(), "binghe819@naver.com");
);
```

> 더 많은 예외는 공식 문서를 참고하자!

<br>

* void메서드 특정 매개변수를 받거나 호출된 경우 예외를 발생시킬 수 있다.
  * [Stubbing void methods with exceptions](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#5)

```java
// memberService.validate는 void 메서드
doThrow(new IllegalArgumentException()).when(memberService).validate(1L);
assertThrows(IllegalArgumentException.class, () -> memberService.validate(1L));
assertDoesNotThrow(() -> memberService.validate(2L));
```

<br>

* 메서드가 동일한 매개변수로 여러변 호출될 때 각기 다르게 행동하도록 조작할 수 있다.
  * [Stubbing consecutive calls](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#10)

```java
Member member = new Member(1L, "binghe819@naver.com");

when(memberService.findById(any()))
    .thenReturn(member)
    .thenThrow(new RuntimeException())
    .thenReturn(null);

assertEquals(memberService.findById(1L).getEmail(), "binghe819@naver.com");
assertThrows(RuntimeException.class, () -> memberService.findById(1L));
assertNull(memberService.findById(1L));
```

<br>

## Mock 객체 확인
> Mock 객체가 어떻게 사용됐는지를 확인할 수 있다. (`verify`)

* 특정 메서드가 특정 매개변수로 몇번 호출 되었는지, 최소 한번은 호출 됐는지, 전혀 호출되지 않았는지
  * [몇번 호출되었는지 검증](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#exact_verification)

```java
 //using mock
 mockedList.add("once");

 mockedList.add("twice");
 mockedList.add("twice");

 mockedList.add("three times");
 mockedList.add("three times");
 mockedList.add("three times");

 //following two verifications work exactly the same - times(1) is used by default
 verify(mockedList).add("once");
 verify(mockedList, times(1)).add("once");

 //exact number of invocations verification
 verify(mockedList, times(2)).add("twice");
 verify(mockedList, times(3)).add("three times");

 //verification using never(). never() is an alias to times(0)
 verify(mockedList, never()).add("never happened");
```

<br>

* 어떤 순서로 호출했는지
  * [호출 순서 검증](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#in_order_verification)

```java
 // A. Single mock whose methods must be invoked in a particular order
 List singleMock = mock(List.class);

 //using a single mock
 singleMock.add("was added first");
 singleMock.add("was added second");

 //create an inOrder verifier for a single mock
 InOrder inOrder = inOrder(singleMock);

 //following will make sure that add is first called with "was added first", then with "was added second"
 inOrder.verify(singleMock).add("was added first");
 inOrder.verify(singleMock).add("was added second");

 // 만약 아래와 같이 호출하면 에러가 발생한다.
 inOrder.verify(singleMock).add("was added second");
 inOrder.verify(singleMock).add("was added first");

 // B. Multiple mocks that must be used in a particular order
 List firstMock = mock(List.class);
 List secondMock = mock(List.class);

 //using mocks
 firstMock.add("was called first");
 secondMock.add("was called second");

 //create inOrder object passing any mocks that need to be verified in order
 InOrder inOrder = inOrder(firstMock, secondMock);

 //following will make sure that firstMock was called before secondMock
 inOrder.verify(firstMock).add("was called first");
 inOrder.verify(secondMock).add("was called second");
```

<br>

* 특정 시간 이내에 호출됐는지
  * [timeout 검증](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#verification_timeout)

```java
//passes when someMethod() is called no later than within 100 ms
//exits immediately when verification is satisfied (e.g. may not wait full 100 ms)
verify(mock, timeout(100)).someMethod();
//above is an alias to:
verify(mock, timeout(100).times(1)).someMethod();

//passes as soon as someMethod() has been called 2 times under 100 ms
verify(mock, timeout(100).times(2)).someMethod();

//equivalent: this also passes as soon as someMethod() has been called 2 times under 100 ms
verify(mock, timeout(100).atLeast(2)).someMethod();
```

<br>

* 특정 시점 이후에 아무 일도 벌어지지 않았는지
  * [no more interactions](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#finding_redundant_invocations)

```java
//using mocks
mockedList.add("one");
mockedList.add("two");

verify(mockedList).add("one");

//following verification will fail
verifyNoMoreInteractions(mockedList);
```

<br>

## BDD 스타일 API

<br>

### BDD란?
> BDD (Behaviour-Driven Development)
* BDD
  * **애플리케이션이 어떻게 "행동"해야 하는지에 대한 공통된 이해를 구성하는 방법**으로, TDD에서 창안됐다.

<br>

### 행동에 대한 스펙
* Title
* Narrative (묘사, 서술)
  * As a (~로서) / I want (~을 원한다) / so that (~그래서 결과는 어떻다)
* Acceptance criteria (인수 테스트)
  * Given / When / Then

<br>

### Mockito에서의 BDD
> Mockito는 BDDMockito라는 클래스를 통해 BDD 스타일의 API를 제공한다. 최상위

```java
// Non - BDD Style
when(memberService.findById(1L)).thenReturn(member);
when(studyRepository.save(study)).thenReturn(study);

// BDD Style
given(memberService.findById(1L)).willReturn(member);
given(studyRepository.save(study)).willReturn(study);
```

```java
// Non - BDD Style
verify(memberService, times(1)).notify(study);
verifyNoMoreInteractions(memberService);

// BDD Style
then(memberService).should(times(1)).notify(study);
then(memberService).shouldHaveNoMoreInteractions();
```

> API
> * https://javadoc.io/static/org.mockito/mockito-core/3.2.0/org/mockito/BDDMockito.html
> * https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#BDD_behavior_verification


<br>

# 참고
* https://ko.wikipedia.org/wiki/%EB%AA%A8%EC%9D%98_%EA%B0%9D%EC%B2%B4
* [백기선님 더 자바 테스트 강의](https://www.inflearn.com/course/the-java-application-test/dashboard)
