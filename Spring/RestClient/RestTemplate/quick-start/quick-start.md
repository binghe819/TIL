# 목차

<br>

- [목차](#목차)
- [RestTemplate - Quick Start](#resttemplate---quick-start)
- [테스트용 서버](#테스트용-서버)
- [자주 사용되는 메서드](#자주-사용되는-메서드)
- [GET 메서드](#get-메서드)
  - [getForEntity()](#getforentity)
  - [getForObject()](#getforobject)
  - [exchange()](#exchange)
- [HEAD 메서드](#head-메서드)
  - [headForHeaders()](#headforheaders)
- [POST 메서드](#post-메서드)
  - [getForEntity()](#getforentity-1)
  - [getForObject()](#getforobject-1)
  - [change()](#change)
- [PUT](#put)
  - [put()](#put-1)
  - [exchange()](#exchange-1)
- [DELETE](#delete)
  - [DELETE()](#delete-1)
- [마치며](#마치며)

<br>

# RestTemplate - Quick Start
RestTemplate은 스프링 3.0부터 지원하는 Spring Rest Client중 하나이다.

RestTemplate은 스프링을 통해 다른 서버에 HTTP 요청을 하기쉽도록 추상화시켜놓았으며, 사용자로하여금 쉽게 HTTP 요청을 할 수 있도록 도와준다.

이번 글은 RestTemplate을 이용하여 HTTP 요청을 해보는 Quick-Start이다.

간단한 회원 CRUD 서버를 띄워놓고, RestTemplate의 기능들을 테스트한다.

> 현재는 스프링 5.0의 WebClient가 나오면서 Deprecated 되었다.
> 
> 그럼에도 아직까지도 RestTemplate을 이용하는 프로젝트는 많기에.. 이번 글을 정리하게되었다.
> 
> 만약 새로운 프로젝트를 시작하는 것이라면 WebClient 사용하는 것을 추천한다. (이유는 이번 글의 목적과 다르기에 추후에 자세히 다룬다.)

<br>

# 테스트용 서버
이번 글은 직접 RestTemplate을 이용하여 특정 서버에 HTTP 요청을 보내본다.

그러기위해선 간단한 응답을 해줄 서버가 하나 필요하기에, 필자가 간단히 회원 CRUD만을 지원하는 서버를 만들었다.

<br>

💁‍♂️ **API 명세**

생성된 회원 서버의 API 명세는 아래와 같다.

* 회원 조회 - `GET /users/{id}`
* 회원 리스트 조회 - `GET /users`
* 회원 등록 - `POST /users`
* 회원 수정 - `PUT /users/{id}`
* 회원 삭제 - `DELETE /users/{id}`

<br>

자세한 코드는 [여기](https://github.com/binghe819/learning-sandbox/tree/master/spring-rest-template)를 참고하면 된다.

이번 글의 테스트는 모두 위 서버를 로컬 (`localhost:8080`)에 실행해둔 상태에서 진행된다.

<br>

# 자주 사용되는 메서드
본격적으로 RestTemplate가 지원하는 각각의 메서드를 살펴보기전에, 자주 사용되는 메서드를 간단히 정리해본다.

* `getForEntity()` - GET 요청을 실행하고 HTTP 응답의 상태 값과 바디 값(지정한 객체로 변환하여)을 `ResponseEntity`에 넣어서 반환한다.
* `getForObject()` - `getForEntity()`와 동일하지만, HTTP의 응답 바디 값(지정한 객체로 변환된)만을 바로 반환한다.
* `exchange()` - GET, POST, PUT과 같은 HTTP 메서드를 실행하며, HTTP 응답의 상태 값과 바디 값(지정한 객체로 변환하여)을 `ResponseEntity`에 넣어서 반환한다.
* `execute()` - `exchange()`과 유사하지만, `RequestCallback`과 `ResultSetExtractor`를 파라미터로 설정할 수 있다.
* `headForHeaders()` - HEAD 요청을 실행하며, 지정된 URL에 대한 모든 HTTP 헤더를 반환한다.
* `optionsForAllow()` - OPTIONS 요청을 실행하며, Allows 헤더를 사용하여 지정된 URL에서 허용되는 HTTP 메서드를 반환한다.
* `delete()` - 주어진 URL에 DELETE 요청을 실행한다.
* `put()` - 주어진 URL에 PUT 요청을 실행한다. (반환 값은 없다.)
* `postForEntity()` - POST 요청을 통해 새로운 리소스를 생성하며, HTTP 응답의 상태 값과 새로 생성된 바디 값(지정한 객체로 변환하여)을 `ResponseEntity`에 넣어 반환한다.
* `postForObject()` - `postForEntity()`와 동일하지만, HTTP의 응답 바디 값(지정한 객체로 변환된)만을 바로 반환한다.
* `postForLocation()` - POST 요청을 통해 새로운 리소스를 생성하며, 생성된 리소스의 주소를 반환한다.

> 더 많은 메서드는 [RestTemplate docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)를 참고.

<br>

# GET 메서드
GET 메서드를 테스트하기위해 먼저 회원(`User`)정보 3개를 미리 생성하고 진행한다.

<br>

## getForEntity()
```java
@Test
void Get_getForEntity() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    Long id = 1L;
    String url = LOCAL_URL + "/" + id;

    // when
    ResponseEntity<UserDto> response = restTemplate.getForEntity(url, UserDto.class);
    
    // then
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    assertThat(response.getBody().getId()).isEqualTo(id);
}
```
응답 바디 값을 `String`으로 받아서 직접 `ObjectMapper`로 변환해줘도된다.

하지만 해당 부분은 매번 반복해야하는 부분이기에, RestTemplate은 주어진 객체 타입으로 자동 변환해준다.

<br>

## getForObject()
```java
@Test
void Get_getForObject() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    Long id = 1L;
    String url = LOCAL_URL + "/" + id;

    // when
    UserDto response = restTemplate.getForObject(url, UserDto.class);
    
    // then
    assertThat(response.getId()).isEqualTo(id);
}
```
`getForObject()`는 `getForEntity()`와 유사하지만, `ResponseEntity`가 아닌, 응답 값만을 반환한다.

<br>

## exchange()
```java
@Test
void Get_exchange() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    Long id = 1L;
    String url = LOCAL_URL + "/" + id;

    HttpHeaders headers = new HttpHeaders();
    headers.add("Accept", MediaType.APPLICATION_JSON_VALUE);

    HttpEntity<UserDto> request =
            new HttpEntity<>(headers);

    // when
    ResponseEntity<UserDto> response = restTemplate.exchange(url, HttpMethod.GET, request, UserDto.class);

    // then
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    assertThat(response.getBody().getId()).isEqualTo(id);
}
```

<br>

# HEAD 메서드

<br>

## headForHeaders()
```java
@Test
void Head_headForHeaders() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    String url = LOCAL_URL;

    // when
    HttpHeaders response = restTemplate.headForHeaders(url);

    // then
    assertThat(response.getContentType().includes(MediaType.APPLICATION_JSON)).isTrue();
}
```

<br>

# POST 메서드

<br>

## getForEntity()
```java
@Test
void Post_postForEntity() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    String url = LOCAL_URL;
    String name = "test";
    int age = 27;

    HttpHeaders headers = new HttpHeaders();
    headers.add("Accept", MediaType.APPLICATION_JSON_VALUE);

    HttpEntity<UserDto> request =
            new HttpEntity<>(new UserDto(null, name, age), headers);

    // when
    ResponseEntity<Long> response = restTemplate.postForEntity(url, request, Long.class);

    // then
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    assertThat(response.getBody()).isOfAnyClassIn(Long.class);
    System.out.println(response.getBody()); // 새로 만들어진 User의 Entity ID
}
```

<br>

## getForObject()
```java
@Test
void Post_postForObject() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    String url = LOCAL_URL;
    String name = "test";
    int age = 27;

    HttpHeaders headers = new HttpHeaders();
    headers.add("Accept", MediaType.APPLICATION_JSON_VALUE);

    HttpEntity<UserDto> request =
            new HttpEntity<>(new UserDto(null, name, age), headers);

    // when
    Long response = restTemplate.postForObject(url, request, Long.class);

    // then
    assertThat(response).isNotNull();
    System.out.println(response); // 새로 만들어진 User의 Entity ID
}
```

<br>

## change()
```java
@Test
void Post_exchange() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    String url = LOCAL_URL;
    String name = "test";
    int age = 27;

    HttpHeaders headers = new HttpHeaders();
    headers.add("Accept", MediaType.APPLICATION_JSON_VALUE);

    HttpEntity<UserDto> request =
            new HttpEntity<>(new UserDto(null, name, age), headers);

    // when
    ResponseEntity<Long> response = restTemplate.exchange(url, HttpMethod.POST, request, Long.class);

    // then
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    System.out.println(response); // 새로 만들어진 User의 Entity ID
}
```

<br>

# PUT

<br>

## put()
```java
@Test
void Put_put() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    HttpHeaders headers = new HttpHeaders();
    headers.add("Accept", MediaType.APPLICATION_JSON_VALUE);

    String url = LOCAL_URL + "/" + 1L;
    String updatedName = "updated_user_name";
    int updatedAge = 27;
    HttpEntity<UserDto> request =
            new HttpEntity<>(new UserDto(null, updatedName, updatedAge), headers);

    // when
    restTemplate.put(url, request);
}
```

<br>

## exchange()
```java
@Test
void Put_Put() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    HttpHeaders headers = new HttpHeaders();
    headers.add("Accept", MediaType.APPLICATION_JSON_VALUE);

    String updatedName = "updated_user_name";
    int updatedAge = 27;
    HttpEntity<UserDto> request =
            new HttpEntity<>(new UserDto(null, updatedName, updatedAge), headers);

    // when
    ResponseEntity<UserDto> response = restTemplate.exchange(LOCAL_URL + "/" + 1L, HttpMethod.PUT, request, UserDto.class);

    // then
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    assertThat(response.getBody().getName()).isEqualTo(updatedName);
    assertThat(response.getBody().getAge()).isEqualTo(updatedAge);
}
```

<br>

# DELETE

<br>

## DELETE()
```java
@Test
void DELETE() {
    // given
    RestTemplate restTemplate = new RestTemplate();

    String url = LOCAL_URL + "/" + 1L;

    restTemplate.delete(url);
}
```

<br>

# 마치며
이번 글은 간단히 RestTemplate을 어떻게 사용하는지에 대한 Quick-Start에 대해서 알아보았다.

다음 글에선 RestTemplate의 동작 원리와 설정하는 방법에 대해서 다룬다.
