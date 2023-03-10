# 목차

<br>

- [목차](#목차)
- [History of Java Logging Framework](#history-of-java-logging-framework)
- [1 System.out.println 사용](#1-systemoutprintln-사용)
  - [System.out.println 문제점](#systemoutprintln-문제점)
- [2 Log4j 탄생](#2-log4j-탄생)
- [3 JUL](#3-jul)
- [4 JCL](#4-jcl)
- [5 slf4j](#5-slf4j)
- [6 Logback과 log4j2](#6-logback과-log4j2)
- [끝으로](#끝으로)
- [참고](#참고)

<br>

# History of Java Logging Framework
'로그'는 운영체제나 소프트웨어가 실행중에 발생한 이벤트 및 메시지를 기록하는 데이터를 의미한다.

'로깅'은 이러한 '로그' 데이터 기록하는 행위를 의미하며, 애플리케이션에서 빼놓을 수 없는 것중 하나가 '로깅'이기도하다.

주변 자바 개발자를보면 대부분 로그 관련하여 굉장히 혼란스러워한다.

그 이유는 자바 진영의 굉장히 많은 로깅 프레임워크가 존재하기 때문인데, 실제로 자바는 Log4j, Logback, JUL, JCL 및 slf4j등등 굉장히 많은 로그 관련 프레임워크와 라이브러리가 존재한다.

이번 글은 이러한 자바 로깅 프레임워크의 역사에 대해서 살펴본다.

<br>

# 1 System.out.println 사용

💁‍♂️ **로깅 프레임워크가 존재하지않을 자바 초창기엔, `System.out.println`을 사용했다고한다.**

자바의 JDK 초기에는 마땅한 로깅 프레임워크가 존재하지않았기에, 대부분의 개발자들은 로그를 기록하는 가장 원시적으로 거친 방법으로 `System.out.println`과 `System.err.println`을 사용했다고한다.

<br>

## System.out.println 문제점

`System.out.println`은 굉장히 단순하지만 로깅용으로 사용하기엔 문제점이 많다.

<br>

1. **동작방식이 동기 (Synchronous)이다.**

`System.out.println`에서 `System.out`의 기본 구현체는 `PrintStream`으로 지정된다.

> PrintStream.java

```java
private void newLine() {
    try {
        synchronized (this) {
            ensureOpen();
            textOut.newLine();
            textOut.flushBuffer();
            charOut.flushBuffer();
            if (autoFlush)
                out.flush();
        }
    }
    catch (InterruptedIOException x) {
        Thread.currentThread().interrupt();
    }
    catch (IOException x) {
        trouble = true;
    }
}
```
실제로 위와 같이 소스코드에 `synchronized`사용하는 것을 볼 수 있다.

이때 **Lock을 걸게되는 객체는 `PrintStream`이며, 동시에 하나의 Thread만 로그를 남길 수 있는 구조다.**

이는 높은 동시성 처리가 필요한 환경에서 사용된다면 성능상 큰 낭패를 볼 수 있다.

**자칫 로깅으로 인해 많은 Thread들이 Blocked이 될 수도 있다.**

부가로직에 해당하는 로깅으로인해 핵심로직이 영향을 받는 구조이므로 높은 동시성 처리가 필요한 환경에선 무조건 사용하지 않아야한다.

<br>

2. **다양한 로그 설정을 할 수 없다.**

`System.out.println`은 로그 내용을 콘솔에 출력하는것이 아닌 파일에 저장하는등 여러 설정을 자체적으로 지원하지 않는다.

물론 아래 예시와 같이 `System.out`도 파일로 출력 내용을 저장할 수 있긴하다.

<details>
  <summary>System.out 로그내용 파일로 저장하는 코드</summary>
  
  ---
  
  ```java
  public class StdToFile {

    public static void main(String[] args) throws FileNotFoundException {
        // File로 라다이렉션
        File file = new File("{log 파일 저장할 디렉토리}}/test.log");
        PrintStream filePrintStream = new PrintStream(new FileOutputStream(file));

        // System의 printStream 수정
        System.setOut(filePrintStream);
        System.setErr(filePrintStream);

        System.out.println("======== Hello World! ========");
        System.out.println("Lorem Ipsum is simply dummy text of the printing and typesetting industry."
            + " Lorem Ipsum has been the industry's standard dummy text ever since the 1500s,"
            + " when an unknown printer took a galley of type and scrambled it to make a type specimen book."
            + " It has survived not only five centuries, but also the leap into electronic typesetting,"
            + " remaining essentially unchanged. "
            + "It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages,"
            + " and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.");
        System.err.println("======== Hello Error! ========");
        System.err.println("System.err.println");
    }
  }
  ```
  
  ---
</details>

위와 같이 콘솔에 출력되지않고 파일로 저장되는 이유는 코드에서 `FileOutputStream`의 `PrintStream`으로 `System`의 출력을 수정해줬기 때문이다.

즉, 디폴트로 콘솔에 출력되는 `PrintStream`을 가진 `System`을 이와 같은 구현체를 만들어서 주입해주었기 때문이다.

> 위 main 함수를 실행하면 `{log 파일 저장할 디렉토리}/test.log`에 `System`을 통해 출력되는 모든 내용이 저장된다.

하지만 위 방식은 파일에 대한 사이즈 및 로그 개수에따라 파일을 분리시키는등에 로깅 프레임워크가 제공하는 다양한 기능을 하나도 지원하지 않는다.

<br>

또한 환경에 따른 로그 설정을 할 수 없다.

즉, 베타와 운영등의 환경별 로그 설정을 따로 할 수 없다.

<br>

3. **로그 레벨이 세분화되어있지않다.**

로그 레벨이 따로 세분화되어있지않으므로, 로그 레벨을 통한 정보 분리하기가 어렵다.

예를 들어, debug 수준의 로그만을 특정 파일에 담기가 어렵다.

<br>

# 2 Log4j 탄생

`System.out.println`의 여러 문제점을 해결하고자 ceki Gulcü가 2001에 `Log4j`를 개발하여 오픈소스로 발표한다. 이때 Log4j는 많은 개발자들의 환호를 불러일으켰으며, Apache 재단의 탑레벨 프로젝트로 들어가게된다.

이후 Log4j는 자바 로깅 시스템의 마일스톤역할을 했으며, `Appender`, `Level`, `Logger`라는 개념(concept)`가 탄생하게되었다.

또한, 다양한 로그 레벨을 설정할 수 있게되었을 뿐만 아니라 다양한 `Appender`를 커스텀하여 사용할 수 있게되어 로그 레벨별 로그 저장방식도 쉽게 커스텀할 수 있게되었다. (ex. `SMTPAppender`, `SyslogAppender`, `JdbcAppender` [참고](https://stackoverflow.com/questions/31840/java-logging-vs-log4j/36482#36482))

지금도 비교적 많은 개발자가 Log4j를 선호하며, 여담으로 Apache에서 Log4j를 JDK에 가장 최우선 로깅 시스템으로 추천하였으나 sun에서 바로 거절했다고한다.

Log4j는 레거시 시스템이라면 사용하고있겠지만, Log4j 개발팀은 2015년을 끝으로 log4j를 더이상 업데이트하고있지않다.

그러므로 지금의 시스템이라면 굳이 사용할 이유가없다.

> 여기서 말하는 Log4j는 초기에 Log4j이다. 현재는 Log4j2가 나온 상태이다.

<br>

# 3 JUL

Log4j가 출시한 같은 시기 Sun에선 자신들만의 로깅 라이브러리인 JUL (Java Util Logging)을 만들었다.

JUL인 [JSR 47](https://jcp.org/en/jsr/detail?id=47)에 명시되어있으며, JDK 1.4부터 공식배포되었다.

Log4j와 다르게 `Appender`를 `Handler`라고 칭하고있으며, 오직 두 개의 `Handler`만 사용가능하다.

* `ConsoleHandler`
* `FileHandler`

JUL도 `System.out.println`의 문제점들은 대부분 해결하였지만, 일관성 없는 API 확장, 느린 성능, 설정 옵션 부족함, 문서화 부족등의 이유로 개발자들의 선택을 받진 못했다.

<br>

# 4 JCL

Log4j와 JUL가 출시하고 여러 개발자들이 사용되고있었다.

문제는 서로 호환되지않기때문에 만약 로깅 프레임워크를 변경한다면 소스 코드를 모두 변경해줘야하는 문제가 있었다.

이로인해, Apache 재단에선 Logging 시스템의 Facade Interface로 JCL (Jakarta Common Logging)을 만들게 되었다.

즉, JCL은 로깅의 구현체가 변경되어도 기존 소스코드엔 전혀 영향이 없도록하는 것이 목표였다. (JDBC와 비슷한 역할)

문제는 JCL도 다른 프레임워크에 대비해 설정이 굉장히 빈약했으며, JCL은 클래스로더를 커스터마이징하여 클래스패스 아래 로깅 구현체를 판단하여 로드하기때문에 메모리 누수가 발생하기 쉬운 구조였다고한다.

<br>

# 5 slf4j

위 문제점들을 바탕으로 Log4j를 만든 ceki Gulcü가 모든 로깅 프레임워크의 표준 인테페이스 스펙인 slf4j (Simple Logging Facade for Java)를 개발하게되었다.

즉, slf4j는 JUL, Log4j, Logback등의 다양한 로깅 프레임워크에 대한 추상화 (인터페이스) 역할을 수행한다.

slf4j는 세 가지 모듈을 제공한다.

* API
  * slf4j를 사용하기 위한 인터페이스를 제공한다.
* Binding
  * slf4j 인터페이스를 로깅 구현체와 연결하는 어댑터 역할을 수행하는 라이브러리다.
  * 사용하길 원하는 로깅 프레임워크에 대한 slf4j 바인딩을 추가해주면 된다. (동시에 반드시 한 개만 사용가능하다)
* Bridge
  * ceki Gulcü는 최대한 모든 로깅 시스템에 대한 호환성을 보장하기위해 다양한 bridge SDK도 개발해두었다.
    * ex. `log4j-over-slf4j`, `jcl-over-slf4j`
  * 다른 로깅 API로의 Logger 호출을 slf4j 인터페이스로 연결 (redirect)하여 slf4j API가 대신 처리할 수 있도록해주는 일종의 어댑터 역할을 수행하는 라이브러리다.
  * 다른 로깅 API -> Bridge(redirect) -> slf4j API

<br>

# 6 Logback과 log4j2

<br>

💁‍♂️ **Logback**

ceki Gulcü는 slf4j를 개발하던 비슷한 시기에 Log4j의 후속 프로젝트로 Logback을 개발하였다.

> 왜 Logback을 만들었는지는 [여기](https://logback.qos.ch/reasonsToSwitch.html)를 참고.

이는 slf4j의 구현체이며, 현재 자바 로깅 프레임워크의 가장 인기있는 구현체라고 볼 수 있다.

실제로 스프링은 slf4j + Logback 조합을 디폴트로 사용한다.

<br>

💁‍♂️ **log4j2**

log4j2는 가장 최근에 나온 프레임워크이며, Logback과 마찬가지로 slf4j를 구현한다.

이와 동시에 Logback이 지원하는 자동 리로드 기능과 고급 필터링 기능을 제공한다.

차이점은 Apache에 따르면 멀티 스레드 환경에서 비동기 로거인 Async Logger의 처리량이 대략 18배정도 높고 대기 시간이 더 짧다고한다.

관련된 더 자세한 내용은 [여기](https://logging.apache.org/log4j/2.x/)를 참고.

> 최근에 [log4j 보안 취약점 사태](https://namu.wiki/w/Log4j%20%EB%B3%B4%EC%95%88%20%EC%B7%A8%EC%95%BD%EC%A0%90%20%EC%82%AC%ED%83%9C)가 바로 log4j2에서 발견된 취약점이다.

<br>

# 끝으로
지금까지 자바의 로깅 프레임워크에 대한 히스토리를 간단히 살펴보았다.

어떤 프레임워크를 사용할지는 상황에따라 사용하면 될 듯하다.

다만 slf4j 인터페이스 사용하는 것을 추천한다. 그래야 추후에 소스코드 변경없이 쉽게 로깅 프레임워크 구현체를 변경할 수 있다.

> 최대한 여러 자료들을 참고하였지만, 틀린 내용이 있을 수 있다.

<br>

# 참고
* https://www.marcobehler.com/guides/java-logging
* https://levelup.gitconnected.com/development-history-of-java-logging-767e8955b553
* https://ko.wikipedia.org/wiki/Log4j
* https://en.wikipedia.org/wiki/Java_logging_framework

