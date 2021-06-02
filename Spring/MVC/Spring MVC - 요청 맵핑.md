[백기선님의 MVC 강의](https://www.inflearn.com/course/%EC%9B%B9-mvc/dashboard)와 [Spring Reference](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#spring-web)를 바탕으로 정리한 자료입니다.

<br>

# 목차

<br>

- [목차](#목차)
- [요청 맵핑하기](#요청-맵핑하기)
  - [1 HTTP Method](#1-http-method)
    - [@RequestMapping](#requestmapping)
    - [@GetMapping](#getmapping)
  - [2 URI 패턴 맵핑](#2-uri-패턴-맵핑)
    - [URI, URL, URN](#uri-url-urn)
    - [요청 식별자로 맵핑하기](#요청-식별자로-맵핑하기)
    - [정규식](#정규식)
    - [패턴이 중복된다면?](#패턴이-중복된다면)
    - [URI 확장자 맵핑](#uri-확장자-맵핑)
  - [3 미디어 타입](#3-미디어-타입)
    - [특정한 타입의 데이터를 담고 있는 요청만 처리하는 핸들러 - consumes](#특정한-타입의-데이터를-담고-있는-요청만-처리하는-핸들러---consumes)
    - [특정한 타입의 응답을 만드는 핸들러 - produces](#특정한-타입의-응답을-만드는-핸들러---produces)
  - [4 헤더와 매개변수](#4-헤더와-매개변수)
  - [5 커스텀 애노테이션](#5-커스텀-애노테이션)

<br>

# 요청 맵핑하기

<br>

## 1 HTTP Method

스프링에서 HTTP 요청을 핸들러(`Controller`)에 맵핑하는 방법을 다룬다.

<br>

### @RequestMapping

Annotation for mapping web requests onto methods in **request-handling classes with flexible method signatures.** - [RequestMapping](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html)

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
  String name() default "";  
  String[] value() default {}; 
  String[] path() default {};
  RequestMethod[] method() default {};
  String[] params() default {};
  String[] headers() default {};
  String[] comsumes() default {};
  String[] produces default {};
}
```
* name - 맵핑에 이름 부여
* [value, path](#2-uri-패턴-맵핑)
* [params](#4-헤더와-매개변수)
* [headers](#4-헤더와-매개변수)
* [comsumes](#3-미디어-타입)
* [produces](#3-미디어-타입)

<br>

:point_right: 예시

```java
@RequestMapping(value = "/hello", method = {RequestMethod.GET, RequestMethod.PUT})
public String hello() {
  ...
}
```
* `/hello` 요청에 대해서 GET과 PUT 요청을 처리.

<br>

### @GetMapping

`@GetMapping` is a shorthand for `@RequestMapping(method = RequestMethod.GET)` - [stackoverflow](https://stackoverflow.com/questions/50351590/difference-between-path-and-value-attributes-in-requestmapping-annotation)

<p align="center"><img src="image/image-20201007134035637.png" width="400" /></p>

> HTTP 메서드의 맵핑 애노테이션이 모두 이와 같이 설정되어 있다.

<br>

## 2 URI 패턴 맵핑

<br>

### URI, URL, URN

<p align="center"><img src="image/FbaKm.png" width="300" /><br>출처 : https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-a</p>

<br>

### 요청 식별자로 맵핑하기

* `?` : 한글자 
  * `/author/???` => `/author/123`
* `*` : 여러 글자
  * `/author/*` => `/author/binghe`
* `**` : 여러 Path
  * `/author/**` => `/author/binghe/book`

<br>

### 정규식

정규식으로 맵핑할 수도 있다.

* `/{name:정규식}` : 정규식에 해당하는 문자열을 name으로 받겠다.
  * `/{name:[a-z]+}` : 소문자 알파벳은 name으로 받는다.

```java
@GetMapping("/{name:[a-z]+}")
@ResponseBody
public String hello(@PathVariable String name) {
  return "hello " + name;
}
```

<br>

### 패턴이 중복된다면?

가장 구체적으로 맵핑되는 핸들러를 선택한다.

```java
@Controller
@RequestMapping("/hello")
public class SampleController {

    @GetMapping("/binghe")
    @ResponseBody
    public String hellobinghe() {
        return "hello binghe";
    }

    @GetMapping("/**")
    @ResponseBody
    public String hello() {
        return "hello";
    }
}
```

* `localhost:8080/hello/binghe` 요청을 해보면 `hellobinghe` 핸들러가 실행된다.

<br>

### URI 확장자 맵핑

```java
@RequestMapping("/binghe") // localhost:8080/binghe.json도 지원
public String hello() {}
```
* 확장자 맵핑이란?
  * 스프링 MVC (부트말고)에서는 `/binghe` 를 맵핑하면 `/binghe.json` 이라는 요청을 받아들인다.
* **이 기능은 권장하지 않는다.** (스프링 부트에서는 기본으로 이 기능을 사용하지 않도록 설정 해준다.)
  * 보안 이슈 (RFD Attack)
  * URI 변수, Path 매개변수, URI 인코딩을 사용할 때 불명확함.

> **요즘은 URI에 `binghe.*` 처럼 명시적으로 하기보다는 HTTP header에 `Accept` 속성을 이용한다고 한다.**

<br>

## 3 미디어 타입

요청을 맵핑할 때 컨텐츠 타입, `Accept` 헤더 속성을 사용하는 방법

<br>

### 특정한 타입의 데이터를 담고 있는 요청만 처리하는 핸들러 - consumes

* `@ReqeustMapping(consumes=MediaType.APPLICATION_JSON_UTF8_VALUE)`
* **Content-Type 헤더로 필터링**
* 매치 되지 않는 경우에 415 Unsupported Media Type 응답

```java
// @RequestMapping(value = "/hello", consumes = MediaType.APPLICATION_JSON_VALUE)
@RequestMapping(value = "/hello", consumes = "application/json")
@ResponseBody
public String hellobinghe() {
  return "hello";
}
```

<p align="center"><img src="image/image-20201007142416399.png" width="500" /></p>

* 응답이 제대로 오는 것을 볼 수 있다. 만약 `Content-Type`을 헤더에 명시하지 않으면 415 코드가 날라온다.

<br>

### 특정한 타입의 응답을 만드는 핸들러 - produces

* `@RequestMapping(produces="application/json")` - json타입을 원하는 요청만 처리
* Accept 헤더로 필터링
  * **요청 헤더에 Accept를 정의하지 않으면 아무거나 다 받겠다는 의미로 받아들인다.**
* 매치 되지 않는 경우에 406 Not Acceptable 응답

```java
@RequestMapping(
  value = "/hello",
  consumes = MediaType.APPLICATION_JSON_VALUE,
  produces = "application/json"
)
@ResponseBody
public String hellobinghe() {
  return "hello";
}
```

* 요청 Content-Type이 json이며, Accept가 json인 요청을 처리한다.
  * Accept를 정의하지 않았어도 요청을 처리한다.

<br>

> 메서드에 선언한 `@RequestMapping`은 클래스에 선언한 `@RequestMapping`을 오버라이딩한다.
>
> ```java
> @Controller
> @RequestMapping(consumes = MediaType.APPLICATION_XML_VALUE)
> public class SampleController {
> 
>     @RequestMapping(
>             value = "/hello",
>             consumes = MediaType.APPLICATION_JSON_VALUE,
>     )
>     @ResponseBody
>     public String hellobinghe() {
>         return "hello";
>     }
> }
> ```
>
> * `hellobinghe` 핸들러는 json 타입만을 받게 된다. xml은 받지 않는다.

<br>

## 4 헤더와 매개변수

* 특정한 헤더가 있는 요청을 처리하고 싶은 경우
  * `@RequestMapping(headers="key")`
    * ex)  `@RequestMapping(headers=HttpHeaders.ACCEPT)`
* 특정한 헤더가 없는 요청을 처리하고 싶은 경우
  * `@RequestMapping(header="!key")`
    * ex)  `@RequestMapping(headers="!" + HttpHeaders.ACCEPT)`
* 특정한 헤더 키/값이 있는 요청을 처리하고 싶은 경우
  * `@RequestMapping(headers="key=value")`
    * ex)  `@RequestMapping(headers=HttpHeaders.ACCEPT + "=" + "application/json")`
* 특정한 요청 매개변수(URL변수) 키를 가지고 있는 요청을 처리하고 싶은 경우
  * `@RequestMapping(params="a")`
    * ex) `@RequestMapping(params = "name")`
* 특정한 요청 매개변수(URL변수)가 없는 요청을 처리하고 싶은 경우
  * `@RequestMapping(params="!a")`
* 특정한 요청 매개변수(URL변수) 키/값을 가지고 있는 요청을 처리하고 싶은 경우
  * `@RequestMapping(params = "a=b")`

<br>

## 5 커스텀 애노테이션

**메타(Meta) 애노테이션**
* 애노테이션에 사용할 수 있는 애노테이션
* 스프링이 제공하는 대부분의 애노테이션은 메타 애노테이션으로 사용할 수 있다.

<br>

**조합(Composed) 애노테이션**
* 한개 혹은 여러 메타 애노테이션을 조합해서 만든 애노테이션
* 코드를 간결하게 줄일 수 있다.
* 보다 구체적인 의미를 부여할 수 있다.

<br>

**@Retention**
* 해당 애노테이션 정보를 언제까지 유지할 것인가를 설정할 수 있다.
  * `Source`: 소스 코드까지만 유지. 즉, 컴파일 이후엔 해당 애노테이션 정보는 사라진다.
  * `Class`: 컴파일한 `.class`파일에도 유지된다. 즉, 런타임에 클래스를 메모리로 읽어오면 해당 정보는 사라진다.
  * `Runtime`: 클래스를 메모리에 읽어왔을 때까지 유지된다. 코드에서 이 정보를 바탕으로 특정 로직을 실행할 수 있다.

<br>

**@Target**
* 해당 애노테이션을 어디에 사용할 수 있는지 결정한다. (ex. 클래스, 메서드, 매개변수...)

<br>

**@Documented**
* 해당 애노테이션을 사용한 코드의 문서에 그 애노테이션에 대한 정보를 표기할 지 결정한다.

