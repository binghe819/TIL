# 목차

<br>

<br>

# Apache HttpClient Connection Pool과 Timeout

<br>

# HttpClient는 요청을 어떻게 처리하는가?


<br>



<br>

# Connection Management
* 연결 지속성
  * 연결 지속성의 필요성.
    * 한 호스트에서 다른 호스트로 연결을 설정하는 프로세스는 매우 복잡하고, 두 엔드포인트 간에 여러 패킷 교환을 포함하므로 시간이 많이 소요된다.
    * 특히 HTTP와 같이 작은 HTTP 메시지를 교환하기위해 매번 TCP 연결은 매우 비효율적이다. 열린 연결을 재사용하여 여러 요청을 실행할 수 있다면 훨씬 더 높은 데이터 처리량을 만족할 수 있다.
  * HTTP/1.1의 Keep-Alive
    * HTTP/1.1부터 HTTP 연결이 기본적으로 여러 요청에 재사용될 수 있다고 명시한다.
    * 즉, 한번 TCP 연결을 맺고 여러번의 HTTP 요청과 응답을 주고 받는 것이다.
  * 그리고 이렇게 연결을 활성 상태로 유지하는 기능을 연결 지속성이라 부른다.
    * **HttpClient는 연결 지속성을 완벽하게 지원한다.**
  * 연결 지속성을 유지하는 방법
    * HTTP/1.1의 Keep-Alive
    * Connection Pool
* HTTP Connection Routing
  * HttpClient는 직접 또는 여러 중간 연결(홉)을 통해 타겟 호스트와 커넥션을 만들 수 있다.
    * 홉: https://en.wikipedia.org/wiki/Hop_(networking)
  * HttpClient는 다른 호스트와의 연결이 Plain인지 Tunneled인지 Layered인지 구분할 수 있습니다.
    * Plain (일반 경로)는 바로 타겟과 연결하거나 가장 앞선의 프록시와의 커넥션을 맺는다.
    * Tunnelled (터널링 경로)는 첫 번째 경로에 연결하고 프록시 체인을 통해 타겟과 커넥션을 맺는다. (프록시가 없는 경로는 터널링이 될 수 없습니다.)
    * Layered (계층화된 경로)는 기존 연결 위에 프로토콜을 계층화하여 커넥션을 맺는다. (프로토콜은 대상에 대한 터널 또는 프록시 없이 직접 연결을 통해서만 계층화될 수 있습니다.)
      * ex. TLS/SSL 연결
  * 대상 호스트에 대한 연결을 터널링하기 위해 여러 중간 프록시를 사용하는 것을 프록시 체인이라고 한다.
* Route Computation (경로 계산)
  * `RouteInfo` 인터페이스는 타겟 호스트까지의 정확한 경로 정보를 나타낸다.
    * 이때 경로 정보는 하나 혹은 하나 이상 홉의 대한 정보를 가지고있는다.
  * `HttpRoute`는 `RouteInfo`를 구현하는 불변 구현체이다.
  * `HttpTracker`는 `RouteInfo`를 구현한 가변 구현체로서, 타겟 호스트까지의 남은 홉 경로를 트래킹하기위해 사용된다.
    * 가변이기에 다음 홉에 도착하면 타겟 호스트까지의 경로 내용이 업데이트될 수 있다.
  * `HttpRouteDirector`는 타겟 호스트까지의 경로중 다음 단계를 계산하는데 도움을 주는 헬퍼 클래스이다.
  * `HttoRoutePlanner`는 실행 컨텍스트를 기반으로 최종 타겟 호스트까지의 전체 경로를 계산하는 전략을 나타내는 인터페이스이다.
    * `HttpClient`는 두 가지 `HttpRoutePlanner` 구현체를 제공한다.
      * `SystemDefaultRoutePlanner` -> `java.net.ProxySelector`를 기반으로 구현되어있으며, 기본적으로 시스템 속성이나 브라우저의 JVM의 프록시 설정를 사용한다.
      * `DefaultProxyRoutePlanner` -> 아무런 시스템의 설정을 참고하지않고, 기본 프록시를 통해 경로를 계산한다.

<br>

# HTTP Connection Managers
* HTTP와 Thread-Safe
  * HTTP 커넥션은 굉장히 복잡하고 프로토콜 차원에서 Thread-Safe를 지원해주지않는다. 그러므로 HTTP을 사용하는 애플리케이션에서 Thread-Safe하게 관리를 해주어야한다.
  * HTTP 커넥션은 한 번에 하나의 스레드에서만 연결되며 사용될 수 있다. (하나의 커넥션을 두 개의 스레드에서 연결할 수 없다.)
* `HttpClientConnectionManager` (커넥션 연결 관리자)
  * 개념
    * `HttpClientConnectionManager` 인터페이스는 HTTP 커넥션 관리를 위한 엔티티이다. (HTTP 연결에 대한 모든 관리를 담당한다.)
  * 목적
    * 새로운 HTTP 커넥션을 생성하는 팩토리 역할을 한다.
    * 지속 연결된 커넥션의 생명주기를 관리하며, 한 번에 하나의 스레드만 커넥션에 액세스할 수 있도록 커넥션을 동기화한다. (동시에 두 개의 스레드가 하나의 커넥션에 액세스할 수 없다.)
* `ManagedHttpClientConnection`
  * 개념
    * 실제 커넥션에 대한 프록시 역할을 하는 인터페이스.
    * 실제 TCP와 소켓 커넥션 정보를 가지고있는 프록시 역할. 이 프록시 객체를 통해 실제 커넥션에 접근한다.
    * 커넥션 연결 관리자가 커넥션 상태를 관리하고 I/O (소켓으로부터 자원 I/O) 작업을 제어하는데 사용되는 인스턴스이다.
    * 커넥션 연결 관리자에 의해 관리되고있는 커넥션이 해제되거나, 명시적으로 커넥션이 닫히면 실제 커넥션은 프록시로부터 분리되고 커넥션 관리자에게 다시 반환된다(돌아간다).

<br>

커넥션 연결 관리자로부터 커넥션을 획득하는 예시.

```java
HttpClientContext context = HttpClientContext.create();
HttpClientConnectionManager connMrg = new BasicHttpClientConnectionManager();
HttpRoute route = new HttpRoute(new HttpHost("localhost", 80));
// Request new connection. This can be a long process
ConnectionRequest connRequest = connMrg.requestConnection(route, null);
// Wait for connection up to 10 sec
HttpClientConnection conn = connRequest.get(10, TimeUnit.SECONDS);
try {
    // If not open
    if (!conn.isOpen()) {
        // establish connection based on its route info
        connMrg.connect(conn, route, 1000, context);
        // and mark it as route complete
        connMrg.routeComplete(conn, route, context);
    }
    // Do useful things with the connection.
} finally {
    connMrg.releaseConnection(conn, null, 1, TimeUnit.MINUTES);
}
```
만약 커넥션을 명시적으로 사전에 종료하고싶다면 `ConnectionRequest#cancel()`을 호출하면 된다.

호출하면 `ConnectionRequest#get()` 메서드에서 차단된 스레드가 차단 해제된다.

<br>

* Simple connection manager
  * `BasicHttpClientConnectionManager`는 한 번에 하나의 커넥션만 유지 관리하는 간단한 커넥션 연결 관리자이다.
  * 이 클래스는 스레드로부터 안전하지만, 하나의 실행 스레드에서만 사용해야한다.
  * 이 클래스는 동일한 경로를 사용하는 후속 요청에 대해서 커넥션을 재사용하려고 노력한다. 그러나 만약 커넥션이 맺어진 경로와 새로운 요청에 대한 경로가 일지하지않으면 기존 커넥션을 닫고 새로운 경로에 대한 커넥션을 다시 맺는다.
  * 만약 커넥션이 이미 할당되어있는 경우 `java.lang.IllegalStateException`이 발생한다.
  * 이 클래스는 EJB 컨테이너 내부에서 사용되야한다.
* Pooling connection manager
  * `PoolingHttpClientConnectionManager`는 커넥션 풀을 관리하여 여러 실행 스레드의 커넥션 요청을 처리할 수 있다.
  * 커넥션 풀은 경로별로 풀링된다. (key가 경로라고 보면 된다!)
  * 





