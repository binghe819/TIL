# 목차

<br>

- [목차](#목차)
- [Apache HttpClient (HttpComponents) 기초](#apache-httpclient-httpcomponents-기초)
- [1 HTTP 실행 요청](#1-http-실행-요청)
  - [1-1 HTTP 요청](#1-1-http-요청)
  - [1-2 HTTP 응답](#1-2-http-응답)
  - [1-3 HTTP 헤더](#1-3-http-헤더)
  - [1-4 HTTP 엔티티](#1-4-http-엔티티)
  - [1-5 응답 핸들러](#1-5-응답-핸들러)
- [2 HttpClient 인터페이스](#2-httpclient-인터페이스)
- [3 HTTP Execution Context](#3-http-execution-context)
- [4 인터셉터](#4-인터셉터)
- [5 예외 처리](#5-예외-처리)
- [6 요청 중단](#6-요청-중단)

<br>

# Apache HttpClient (HttpComponents) 기초
이 글은 이전 글인 Quick-Start 글을 읽고 온 사람을 대상으로 작성된 글입니다.

<br>

# 1 HTTP 실행 요청

> 이번 챕터는 HttpClient를 이용하여 HTTP를 실행. 즉, HTTP 요청하고 응답을 처리하는데 필요한 기본적인 지식을 정리한 챕터이다.

HttpClient의 가장 기본적인 역할은 HTTP 메서드를 실행하는 것이다. 

HTTP 메서드 실행이란 HttpClient에서 내부적으로 처리되는 하나 이상의 HTTP 요청과 응답 (교환이라고도 말한다)을 의미한다.

HttpClient를 사용하는 개발자는 실행해야하는 요청 객체를 HttpClient에게 제공해야하며, HttpClient는 요청을 대상 서버에 전송하여 해당 응답 객체를 반환하거나 실패한 경우 예외를 던져줘야한다.

HttpClient를 이용한 실행 과정을 간단히 살펴보면 아래와 같다.

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    <...>
} finally {
    response.close();
}
```
중요한 점은 HttpClient 라이브러리에서 HTTP 요청에 대한 인터페이스의 이름이 `HttpClient`라는 것이다.

<br>

## 1-1 HTTP 요청
앞서 말했듯이, HttpClient를 사용하는 개발자는 실행해야하는 요청 객체를 만들어 HttpClient에게 제공해야한다.

HttpClient에서 제공하는 요청 객체는 HTTP/1.1에서 지원하는 모든 HTTP 메서드를 지원한다.

그리고 명시적인 구현체로 `HttpGet`, `HttpHead`, `HttpPost`, `HttpPut`, `HttpDelete`, `HttpTrace`, and `HttpOptions` 객체들을 미리 정의해두고 제공해준다.ㅇ

각 객체들의 상위 클래스는 RequestBody를 가지느냐에따라 다르다. 하지만 모든 요청 구현체의 최상위 인터페이스는 `HttpRequest`로 동일하다.

요청 객체를 생성하는 방법은 아래와 같다.

```java
HttpGet httpget = new HttpGet(
     "http://www.google.com/search?hl=en&q=httpclient&btnG=Google+Search&aq=f&oq=");
```

이외에도, 아래와 같이 URIBuilder를 이용하여 명시적으로 생성해줄 수도 있다.

```java
URI uri = new URIBuilder()
        .setScheme("http")
        .setHost("www.google.com")
        .setPath("/search")
        .setParameter("q", "httpclient")
        .setParameter("btnG", "Google Search")
        .setParameter("aq", "f")
        .setParameter("oq", "")
        .build();
HttpGet httpget = new HttpGet(uri);
System.out.println(httpget.getURI());
```

<br>

## 1-2 HTTP 응답
요청 객체를 만들어 HttpClient에게 넘겨 실행시키면 HTTP 응답을 반환하게된다.

임의적으로 응답 객체를 만들면 아래와 같다.

```java
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, HttpStatus.SC_OK, "OK");

System.out.println(response.getProtocolVersion());              // HTTP/1.1
System.out.println(response.getStatusLine().getStatusCode());   // 200
System.out.println(response.getStatusLine().getReasonPhrase()); // OK
System.out.println(response.getStatusLine().toString());        // HTTP/1.1 200 OK
```

<br>

## 1-3 HTTP 헤더
HTTP 메시지 (요청과 응답 포함)는 ContentLength, ContentType과 같이 메시지의 속성을 설명하는 여러 헤더를 포함하고있다.

HttpClient도 당연히 헤더를 쉽게 사용할 수 있는 편의 메서드를 지원한다.

```java
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, 
    HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie", "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie", "c2=b; path=\"/\", c3=c; domain=\"localhost\"");
Header h1 = response.getFirstHeader("Set-Cookie");
System.out.println(h1);                                  // Set-Cookie: c1=a; path=/; domain=localhost
Header h2 = response.getLastHeader("Set-Cookie");
System.out.println(h2);                                  // Set-Cookie: c2=b; path="/", c3=c; domain="localhost"
Header[] hs = response.getHeaders("Set-Cookie");
System.out.println(hs.length);                           // 2
```

<br>

## 1-4 HTTP 엔티티
HTTP 메시지의 가장 아래 부분을 Body라고 부르며, 이 영역은 다양한 컨텐츠를 전달할 때 사용한다.

이러한 **Body에 담기는 컨텐츠를 HttpClient에서는 HTTP Entity (이하 엔티티)라고 부른다.**

엔티티를 포함하는 요청을 Entity Enclosing 요청이라고 부르며, `POST`와 `PUT` 요청시 사용된다.

응답은 일반적으로 엔티티를 포함한다. 물론 예외적으로 `204 No Content`과 같은 응답에는 포함되지 않는다.

HttpClient는 컨텐츠의 출처에 따라 크게 3가지의 엔티티로 구분한다.

* `streamed` : 컨텐츠가 스트림 (소켓)에서 수신되는 경우. 보통 HTTP 응답으로부터 수신되는 엔티티는 `streamed`이다. 그리고 이렇게 수신되는 엔티티는 반복적으로 읽을 수 없다. (한번 읽고 끝)
  * 응답의 엔티티를 의미한다.
* `self-contained` : 콘텐츠가 메모리에 있거나 연결 또는 기타 엔티티와 독립적인 수단을 통해 수신되는 경우. 일반적으로 반복해서 읽을 수 있으며, HTTP 요청의 엔티티가 이에 해당된다.
  * 요청의 엔티티를 의미한다.
* `wrapping` : 컨텐츠를 다른 엔티티에서 가져오는 경우.

이렇게 3가지로 엔티티를 구분하는 이유는 HTTP 응답에서 컨텐츠를 스트리밍할 때 연결 관리에 중요한 구분점이 되기때문이다.

<br>

**HTTP 엔티티 사용하기**

엔티티는 이진과 문자열을 모두 나타낼 수 있으므로 문자 인코딩을 지원한다.

보통 엔티티는 동봉된 컨텐츠가 있는 요청을 실행할 때나 요청이 성공하고 응답 본문을 사용하여 결과를 클라이언트에 다시 보낼 때 생성된다.

엔티티에서 컨텐츠를 읽으려면 `java.io.InputStream`을 반환하는 `HttpEntity#getContent()` 메서드를 통해 입력 스트림을 읽을 수 있다.

엔티티에 컨텐츠를 쓰려면 `HttpEntity#writeTo(OutputStream)` 메서드에 OutputStream을 주입해주면 된다.

<br>

**리소스 해제를 보장해줘야한다**

시스템 리소스의 적절한 해제를 보장하려면 **애플리케이션과 Socket 간의 연결된 콘텐츠 스트림이나 소켓 자체를 닫아야한다.**

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        InputStream instream = entity.getContent();
        try {
            // do something useful
        } finally {
            instream.close();
        }
    }
} finally {
    response.close();
}
```
컨텐츠 스트림을 닫는 것과 응답을 닫는 것은 차이가 있다.

* 컨텐츠 스트림을 닫는 것: TCP 커넥션은 유지하되 소켓으로부터 컨텐츠를 소비 (읽는)것을 멈춘다는 의미.
* 응답을 닫는 것: TCP 커넥션을 즉시 종료한다는 의미이다.

`HttpEntity#writeTo(OutputStream)`과 `HttpEntity#getContent()` (InputStream) 모두 finally 절에는 컨텐츠 스트림을 닫아줘야한다.

그리고 `EntityUtils#consume(HttpEntity)`를 통해 엔티티 컨텐츠가 완전히 사용되었고, 기본 스트림이 닫혔는지 확인할 수 있다.

만약 응답으로부터 읽어야하는 컨텐츠가 작다면, 전체 응답 컨텐츠를 소비하고 컨텐츠 및 커넥션을 닫는 것보다, 바로 커넥션을 닫는 것이 효율적일 수 있다.

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        InputStream instream = entity.getContent();
        int byteOne = instream.read();
        int byteTwo = instream.read();
        // Do not need the rest
    }
} finally {
    response.close();
}
```
위와 같이 커넥션이 재사용되지 않지만, 커넥션이 보유한 모든 리소스는 올바르게 닫힌다.

> 닫힌다 -> 할당 해제.

<br>

**엔티티 컨텐츠 소비** (Consuming Entity Content)

**엔티티 컨텐츠를 사용하는 데 권장하는 방법은 `HttpEntity#getContent()` 또는 `HttpEntity#writeTo(OuputStream)` 메서드를 사용하는 것이다.**

물론 HttpClient는 이러한 메서드를 활용하여 엔티티 컨텐츠를 더 쉽게 읽을 수 있도록하는 헬프 정적 메서드를 제공해준다. (`EntityUtils`)

`EntityUtils`를 이용하면 직접 `java.io.InputStream`을 읽지않고, 쉽게 전체 컨텐츠 본문을 문자열 혹은 바이트 배열로 가져올 수 있다.

다만, 응답 엔티티가 신뢰할 수 있는 HTTP 서버로부터 오고, 길이가 제한된 것으로 알려진 경우가 아니라면 `EntityUtils`를 사용하는 것은 좋지 않다고한다.

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        long len = entity.getContentLength();
        if (len != -1 && len < 2048) {
            System.out.println(EntityUtils.toString(entity));
        } else {
            // Stream content out
        }
    }
} finally {
    response.close();
}
```

<br>

어떤 상황에서는 엔티티 컨텐츠를 두 번이상 읽어야할 수도 있다. (InputStream은 한번 읽으면 다시 읽는 것은 불가능하다.)

이 경우 엔티티 컨텐츠를 한번 읽을 때 메모리나 디스크에 버퍼링하면 된다. HttpClient는 `BufferedHttpEntity` 클래스를 통해 이러한 기능을 제공한다.

```java
CloseableHttpResponse response = <...>
HttpEntity entity = response.getEntity();
if (entity != null) {
    entity = new BufferedHttpEntity(entity);
}
```

<br>

**엔티티 컨텐츠 제작** (Producing Entity Content)

HttpClient는 문자열, 바이트 배열, 입력 스트림 및 파일과 같은 엔티티 컨텐츠를 쉽게 전송할 수 있는 클래스를 제공한다.

물론 OutputStream을 사용하여 데이터를 전송할 수 있다.

```java
File file = new File("somefile.txt");
FileEntity entity = new FileEntity(file, 
    ContentType.create("text/plain", "UTF-8"));        

HttpPost httppost = new HttpPost("http://localhost/action.do");
httppost.setEntity(entity);
```

StreamEntity들은 모두 기본 컨텐츠 스트림에서 한 번만 읽을 수 있으므로 반복할 수 없다.

그래서 일반적으로 StreamEntity를 사용하는 것보단 사용자 지정 HttpEntity 클래스를 구현하는 것이 좋다고한다. (ex. FileEntity)

<br>

## 1-5 응답 핸들러
HttpClient를 사용하면서 HTTP 응답을 처리하는 가장 간단하고 편리한 방법은 `ResponseHandler` 인터페이스를 사용하는 것이다.

`ResponseHandler` 인터페이스는 HTTP 요청의 성공과 실패 상관없이 커넥션을 자동으로 닫아준다.

즉, `ResponseHandler#handleResponse(HttpResponse)` 메서드는 커넥션 관리(ex. close())에 대한 코드없이 쉽게 사용할 수 있다.

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/json");

ResponseHandler<MyJsonObject> rh = new ResponseHandler<MyJsonObject>() {

    @Override
    public JsonObject handleResponse(
            final HttpResponse response) throws IOException {
        StatusLine statusLine = response.getStatusLine();
        HttpEntity entity = response.getEntity();
        if (statusLine.getStatusCode() >= 300) {
            throw new HttpResponseException(
                    statusLine.getStatusCode(),
                    statusLine.getReasonPhrase());
        }
        if (entity == null) {
            throw new ClientProtocolException("Response contains no content");
        }
        Gson gson = new GsonBuilder().create();
        ContentType contentType = ContentType.getOrDefault(entity);
        Charset charset = contentType.getCharset();
        Reader reader = new InputStreamReader(entity.getContent(), charset);
        return gson.fromJson(reader, MyJsonObject.class);
    }
};
MyJsonObject myjson = client.execute(httpget, rh);
```

<br>

# 2 HttpClient 인터페이스
HttpClient 인터페이스는 HTTP 요청 실행을 위한 가장 필수적인 역할을 수행한다.

인터페이스이므로 HTTP 요청 응답 프로세스에 대한 제한이나 특정 세부 사항을 강제하지 않는다. 

대신 커넥션 관리, 상태 관리, 인증 및 리다이렉션 처리의 구현은 개발자의 구현에 맡긴다. (유연하다는 의미)

이러한 특징으로 인해 응답 컨텐츠를 캐싱한다는 등 여러가지 기능을 쉽게 추가할 수 있다.

> 공식문서에서는 이를 Decorate한다고한다. 즉, 데코레이터 패턴과 같이 새로운 기능을 쉽게 추가 설정할 수 있다는 의미.

<br>

HttpClient는 퍼사드 패턴에서의 퍼사드 (건물의 정면) 역할을 한다.

리다이렉션, 인증 처리, 커넥션 관리, 커넥션 설정등 HTTP 이용할 때 필요한 기능들의 퍼사드 역할을 한다.

즉, 개발자는 HttpClient을 통해 HTTP 기능들을 모두 이용할 수 있다.

이를 통해 개발자는 구현체를 바꿔껴주기만하면, 쉽게 HTTP 요청과 응답에 대한 설정을 커스텀할 수 있다. 

> KeepAlive 전략 설정하는 예시.
```java
ConnectionKeepAliveStrategy keepAliveStrat = new DefaultConnectionKeepAliveStrategy() {

    @Override
    public long getKeepAliveDuration(
            HttpResponse response,
            HttpContext context) {
        long keepAlive = super.getKeepAliveDuration(response, context);
        if (keepAlive == -1) {
            // Keep connections alive 5 seconds if a keep-alive value
            // has not be explicitly set by the server
            keepAlive = 5000;
        }
        return keepAlive;
    }
};
CloseableHttpClient httpclient = HttpClients.custom()
        .setKeepAliveStrategy(keepAliveStrat)
        .build();
```

또한, 개발자는 부가 로직 (리다이렉션, 인증 처리, 커넥션 관리등)을 신경쓰지않고, 핵심 로직 (HTTP 요청과 응답)에만 집중할 수 있도록 해준다.

<br>

**스레드 안전**

HttpClient의 구현체는 스레드로부터 안전해야한다. 또한, 매 요청마다 인스턴스를 생성하는 것보단, 한번 만들어서 재사용하는 것이 좋다.

<br>

**자원 할당 해제**

`CloseableHttpClient`의 인스턴스가 더 이상 필요하지 않는다면, 아래와 같이 `close()` 메서드를 통해 해당 인스턴스와 연결된 커넥션 관리자에게 자원 해제 요청을해주어야한다.

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
try {
    <...>
} finally {
    httpclient.close();
}
```

<br>

# 3 HTTP Execution Context

HTTP는 stateless하며, 요청-응답 지향의 프로토콜이다. 그러나 실제 응용 프로그램은 종종 stateful하게 상태를 유지하여 몇몇 요청-응답을 주고받아야할 때가 존재한다.

> 예를 들어, 쿠키/세션 기반의 로그인.

이러한 상태를 유지할 수 있도록 HttpClient는 HTTP 요청이 특정 Execution Context (실행 컨텍스트)내에서 실행되도록 허용한다.

연속적인 요청 간에 동일한 컨텍스트가 재사용되는 경우엔 논리적으로 관련된 여러 요청이 논리적 세션에 참여할 수 있다.

HTTPContext는 HTTP 요청 프로세스의 실행 상태를 나타낸다. 속성을 저장하는 것은  `java.util.Map<String, Object>`와 유사하다. 즉, 단순한 딕셔너리 형태의 컬렉션이다.

HTTPContext의 주요 목적은 논리적으로 관련된 다양한 구성 요소 간의 정보 공유를 용이하게 하는 것이다.  (HttpContext는 데이터를 저장하고 데이터를 편리하게 주고 받기 위해서 사용하는 객체다.)

HttpClient를 이용하면 요청 실행 전에 컨텍스트 속성을 채우거나 실행이 완료된 후 컨텍스트를 검사할 수 있다.

HttpContext는 여러 오브젝트 객체를 포함할 수 있으며 이는 여러 스레드 간에 공유하는 것이 안전하지 않을 수 있다.

이로인해 실행의 각 스레드는 자체 컨텍스트르 유지하는 것이 좋다.

<br>

HTTP 요청 실행 과정에서 HttpClient는 실행 컨텍스트에 다음 속성을 추가한다.

* `HttpConnection` - 타겟 서버에 대한 실제 커넥션을 나타내는 HttpConnection 객체.
* `HttpHost` - 커넥션을 맺을 타겟 서버을 나타내는 객체.
* `HttpRoute` - 타겟 서버에 커넥션하기 위한 전체 연결 경로를 나타내는 객체.
* `HttpRequest` - HTTP 요청을 나타내는 객체.
* `HttpResponse` - HTTP 응답을 나타내는 객체.
* `java.lang.boolean` - 실제 요청이 커넥션 맺은 타겟으로 완전히 전송되었는지 여부를 나타내는 플래그.
* `RequestConfig` - 실제 요청에 대한 설정을 나타내는 객체.
* `java.util.List<URI>` - 요청 실행 과정에서 수신된 모든 리다이렉션 위치의 컬렉션을 나타내는 객체.

아래와 같이 `HttpClientContext` 어댑터 클래스를 사용하여 컨텍스트 상태와의 상호 작용을 단순화할 수도 있다.

```java
HttpContext context = <...>
HttpClientContext clientContext = HttpClientContext.adapt(context);
HttpHost target = clientContext.getTargetHost();
HttpRequest request = clientContext.getRequest();
HttpResponse response = clientContext.getResponse();
RequestConfig config = clientContext.getRequestConfig();
```

<br>

논리적으로 관련된 세션을 나타내는 여러 요청 시퀀스는 요청 간의 대화 컨텍스트 및 상태 정보의 자동 전파를 보장하기 위해 동일한 HttpContext 인스턴스로 실행되어야 한다.

아래 예시는 초기 요청에 의해 설정된 요청 구성은 HttpContext에 유지되고 동일한 컨텍스트를 공유하는 연속 요청을 실행한다.

```java
CloseableHttpClient httpclient = HttpClients.custom()
        .addInterceptorLast(new HttpRequestInterceptor() {

            public void process(
                    final HttpRequest request,
                    final HttpContext context) throws HttpException, IOException {
                AtomicInteger count = (AtomicInteger) context.getAttribute("count");
                request.addHeader("Count", Integer.toString(count.getAndIncrement()));
            }

        })
        .build();

AtomicInteger count = new AtomicInteger(1);
HttpClientContext localContext = HttpClientContext.create();
localContext.setAttribute("count", count);

HttpGet httpget = new HttpGet("http://localhost/");
for (int i = 0; i < 10; i++) {
    CloseableHttpResponse response = httpclient.execute(httpget, localContext);
    try {
        HttpEntity entity = response.getEntity();
    } finally {
        response.close();
    }
}
```

> HttpClient 사용할 때 사용되는 모든 설정도 여기에 설정해둔다. (소켓 timeout등등)

<br>

# 4 인터셉터
HttpClient는 HTTP 요청 프로세스에 인터셉터 기능을 제공한다. (데코레이터 패턴을 활용한다.)

이를 활용하면 HTTP 요청이나 응답에서 특정 동작을 쉽게 추가할 수 있다. (HTTP 헤더뿐만 아니라 컨텐츠 엔티티도 조작할 수 있다.)

또한, 인터셉터는 HTTP Execution Context를 통해 처리 상태와 같은 정보를 공유할 수 있다.

만약 인터셉터를 사용한다면 Thread-Safe하게 사용해야한다. 서블릿과 유사하게 동기화되지 않는 인스턴스 변수를 사용하면 안된다.

아래는 연속적인 요청 간에 처리 상태를 유지하는 예시이다.

```java
CloseableHttpClient httpclient = HttpClients.custom()
        .addInterceptorLast(new HttpRequestInterceptor() {

            public void process(
                    final HttpRequest request,
                    final HttpContext context) throws HttpException, IOException {
                AtomicInteger count = (AtomicInteger) context.getAttribute("count");
                request.addHeader("Count", Integer.toString(count.getAndIncrement()));
            }

        })
        .build();

AtomicInteger count = new AtomicInteger(1);
HttpClientContext localContext = HttpClientContext.create();
localContext.setAttribute("count", count);

HttpGet httpget = new HttpGet("http://localhost/");
for (int i = 0; i < 10; i++) {
    CloseableHttpResponse response = httpclient.execute(httpget, localContext);
    try {
        HttpEntity entity = response.getEntity();
    } finally {
        response.close();
    }
}
```

<br>

# 5 예외 처리
HttpClient는 HTTP 프로토콜을 처리하면서 두 가지의 예외를 던질 수 있다.

* `java.io.IOException`
  * socket timeout, socket rest, HTTP 프로토콜 위반등의 경우 발생할 수 있는 예외.
  * 보통의 IOException은 치명적이지 않아 복구 가능하다.
* HTTP 프로토콜 위반 (`ClientProtocolException`)
  * 보통 IOException은 치명적이지 않아 복구 가능하지만, HTTP 프로토콜 위반의 경우는 자동 복구가 안되기에 치명적일 수 있다.
  * 이로 인해 HttpClient는 이 경우 바로 IOException을 던지지않고, IOException의 하위 클래스인 ClientProtocolException을 던진다.
  * ClientProtocolException을 이용하면 일반 IOException과 HTTP 프로토콜 위반을 모두 처리할 수 있다.

<br>

**자동 예외 복구**

HttpClient는 기본적으로 IOException이 발생하면 자동으로 복구를 시도한다.

기본 자동 복구 메커니즘은 안전하다고 알려진 몇 가지 예외만을 대상으로 발생한다.

* HttpClient는 개발자가 작성한 논리적 오류 또는 HTTP 프로토콜 오류는 자동 복구를 시도하지않는다.
* HttpClient는 멱등성으로 간주되는 메서드(GET, HEAD)를 자동으로 다시 시도한다.
* HttpClient는 HTTP 요청을 대상 서버로 전송하는 과정에서 발생하는 예외의 경우 자동 복구를 시도한다. (커넥션 관련 오류)
  * ex. 요청이 서버로 완전히 전송되지 않은 경우.

<br>

**요청 재전송 핸들러**

아래와 같이 사용자가 직접 재전송에 대한 핸들러를 정의할 수 있다. 이때 `HttpRequestRetryHandler` 인터페이스를 구현해줘야한다.

```java
HttpRequestRetryHandler myRetryHandler = new HttpRequestRetryHandler() {

    public boolean retryRequest(
            IOException exception,
            int executionCount,
            HttpContext context) {
        if (executionCount >= 5) {
            // Do not retry if over max retry count
            return false;
        }
        if (exception instanceof InterruptedIOException) {
            // Timeout
            return false;
        }
        if (exception instanceof UnknownHostException) {
            // Unknown host
            return false;
        }
        if (exception instanceof ConnectTimeoutException) {
            // Connection refused
            return false;
        }
        if (exception instanceof SSLException) {
            // SSL handshake exception
            return false;
        }
        HttpClientContext clientContext = HttpClientContext.adapt(context);
        HttpRequest request = clientContext.getRequest();
        boolean idempotent = !(request instanceof HttpEntityEnclosingRequest);
        if (idempotent) {
            // Retry if the request is considered idempotent
            return true;
        }
        return false;
    }

};
CloseableHttpClient httpclient = HttpClients.custom()
        .setRetryHandler(myRetryHandler)
        .build();
```
`StandardHttpRequestRetryHandler`는 멱등성을 지원하는 메서드인 GET, HEAD, PUT, DELETE, OPTIONS, TRACE등에 자동 재시도 메커니즘이 구현되어있다.

<br>

# 6 요청 중단
서버에 부하로인해 HTTP 요청이 예상 시간내에 완료되지 않을 가능성이있다.

이러한 상황에서 요청을 조기에 종료하여 커넥션이 맺어진 블록킹된 스레드를 빠르게 해제하는 것이 좋다.

HttpClient는 이러한 상황을 대비하여 `HttpUriRequest#abort()`를 지원하며, 이 메서드를 호출하면 HTTP 프로세스의 모든 단계가 중단된다.

이 메서드는 스레드로부터 안전하며 모든 스레드에서 호출할 수 있다.

HTTP 요청이 중단되면 실행 스레드 (현재 커넥션맺고 I/O작업에서 차단된 스레드)는 InterruptedIOException을 던져 스레드를 해제한다.



