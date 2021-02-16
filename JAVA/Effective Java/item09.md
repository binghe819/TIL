> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 9. try-finally 보다는 try-with-resources를 사용하라

- [아이템 9. try-finally 보다는 try-with-resources를 사용하라](#아이템-9-try-finally-보다는-try-with-resources를-사용하라)
  - [close](#close)
  - [try - finally의 문제](#try---finally의-문제)
    - [예외가 무시되버려 디버깅이 어려워진다.](#예외가-무시되버려-디버깅이-어려워진다)
    - [코드가 지저분](#코드가-지저분)
  - [try-with-resources](#try-with-resources)
    - [AutoCloseable](#autocloseable)
    - [try-with-resources 예시](#try-with-resources-예시)
  - [핵심 정리](#핵심-정리)

<br>

## close

자바 라이브러리에는 `close` 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.

* `InputStream`
* `OutputStream`
* `java.sql.Connection` 등등

자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 한다.

상당수가 안전망으로 `finalizer`를 활용하고는 있지만 `finalizer` 는 믿을만하지 못하다. ([아이템 8](./item08.md))

<br>

## try - finally의 문제

자원이 제대로 닫힘을 보장하는 수단으로 `try - finalliy`가 많이 사용되었다.

<br>

### 예외가 무시되버려 디버깅이 어려워진다.

```java
// 하나의 자원을 회수
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine(); // 여기서도 예외 발생 (첫 번째 예외)
    } finally {
        br.close();           // 여기서도 예외 발생 (두 번째 예외)
    }
}
```

* 만약, try 블록 (`br.readLine()`)과 finally 블록 (`br.close()`)가 기기의 물리적 이유로 동시에 예외가 발생한다면 **두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다.**
  * 이럴 경우 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어, **디버깅이 몸시 힘들어진다.**
  * 실제 근본 문제는 `br.readLine()`인데 에러 메시지에는 전혀 나오지 않게 된다.. 노답

<br>

### 코드가 지저분

```java
// 두 개의 자원을 회수
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

* 자원이 둘 이상이면 너무 지저분하다.

<br>

## try-with-resources

이런 문제들은 자바 7이 투척한 `try-with-resources`덕에 모두 해결되었다.

<br>

### AutoCloseable

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

**이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.**

즉, 닫아야하는 자원을 뜻하는 클래스를 작성한다면 `AutoCloseable`을 반드시 구현해라.

<br>

### try-with-resources 예시

```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

* 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다.
  * **`readLine`과 `close` 호출 양쪽에서 예외가 발생하면, `close` 예외는 숨겨지고 `readLine` 예외가 기록된다.**
  * 또한 숨겨진 예외도 모두 스택 추적 내역에 '숨겨졌다(suppressed)'라는 꼬리표를 달고 출력된다.

* 또한, 보통의 try - finally처럼 `catch` 절을 쓸 수 있다.

<br>

## 핵심 정리

꼭 회수해야 하는 자원을 다룰 때는 `try-finally` 말고, `try-with-resources`를 사용하자.

예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. `try-finally`로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, `try-with-resources`로는 정확하고 쉽게 자원을 회수할 수 있다