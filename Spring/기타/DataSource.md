# DataSource

스프링에서 DB 커넥션을 할때 사용하는 DataSource에 대해서 간단히 다뤄보고자 한다.



## 1 DataSource

[DataSource API](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/javax/sql/DataSource.html)

### 1-1 DataSource란?

🤔 **`DataSource`란?**

* 자바에서 제공하는 **DB 커넥션 인터페이스**이다.

* 기존 JDBC 프로그램 구현으로 DB와의 `Connection`을 맺은 객체를 얻어야 했다. `DataSource`는 이러한 커넥션을 연결하는데 유용한 기술과 인터페이스를 통한 느슨한 연결을 지원한다.
* JNDI Server를 통해서 이용된다.
* `DataSource`객체를 통해서 필요한 `Connection`을 획득, 반납 등의 작업을 한다.



💁‍♂️ **`DataSource`는 이미 잘 구현된 클래스가 많기 때문에 ,직접 구현할 일은 거의 없다.**

* 대부분의 DataSource구현체는 DB의 종류나 아이디, 비밀번호를 지정할수 있게 되어있다.



### 1-2 DataSource의 장점

💁‍♂️ **DataSource의 장점**

* Connection Pool을 지원한다.
  * `Connection`객체를 프로그램이 실행될 때마다 생성하는 것이 아닌, 웹 애플리케이션이 서비스되기 전에 미리 생성하여 준비한 다음, 필요할 때 준비된 `Connection`을 가져다 사용함으로써 JDBC의 단점들을 개선한 기술.
  * **`DataSource`를 커넥션 풀의 `Connection`을 관리하기 위한 객체라고도 한다.**

* 트랜잭션 처리



### 1-3 JNDI

🤔  JNDI란?

* Java Naming and Directory Interface
* 디렉토리 서비스에서 제공하는 데이터 및 객체를 발견하고 참고하기 위한 자바 API



🤔  JNDI가 하는일 == `Naming & Directory` 

* Naming & Directory 서비스는 실제 어떤 자원을 가지고 서비스 한다는 의미가 아니라, 어떤 서버나 애플리케이션에서 **분산환경에 서비스하고자 하는 자원을 이 Naming & Directory 서버에 이름값과 실제 자원을 연결하여 등록하면, 해당 자원을 이용하고자 하는 다른 애플리케이션에서 Naming & Directory 서버에 접근하여 이름값만을 가지고 자원을 연결하여 이용할 수 있게 하는 개념입니다.**
* **네이밍 서비스의 대표적인 예로 DNS서버가 이런 기능을 한다.**
* **Naming & Directory 서버 또한 분산환경에서 자원을 연결해주는 기능을 합니다.**





## 2 DataSource 예제



### 2-1 XML

`Server.xml`

```xml
<GlobalNamingResources>
    .... 생략 ....
    <Resource driverClassName="oracle.jdbc.driver.OracleDriver"
              url="jdbc:oracle:thin:@127.0.0.1:1521:xe"
              username="scott"
              password="tiger"
              name="jdbc/myoracle"
              type="javax.sql.DataSource"
              maxActive="4"
              maxIdle="2"
              maxWait="5000" />    
</GlobalNamingResources>
```

```java
//1. JNDI 서버 객체 새성
InitialContext ic= new InitialContext();
 
//2. lookup()
DataSource ds = (DataSource) ic.lookup("java:comp/env/jdbc/myoracle");
 
//3. getConnection()
Connection conn = ds.getConnection();
```





### 2-2 Java Config

```java
@Bean
public DataSource dataSource() {
  SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
  
  dataSource.setDriverClass(org.h2);
  dataSource.setUrl("jdbc:..");
  dataSource.setUsername("id");
  dataSource.setPassword("password");
  
  return dataSource;
}
```







## 참고

* https://opentutorials.org/module/3569/21223

* https://gmlwjd9405.github.io/2018/05/15/setting-for-db-programming.html