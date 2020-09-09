# 목차









# Bean

스프링에서 사용하는 Bean이 무엇이며, 어떻게 등록하고 사용되는지 정리한 자료이다.



## 1 Bean이란?

🤔  **`spring bean`이란?**

- **간단하게 말하면 자바 객체이다.**
  - **스프링 컨테이너**에 의해서 **자바 객체**가 만들어 지게 되면 이 객체를 스프링은 **스프링 빈**이라고 부른다.
  - **스프링 빈과 자바 일반 객체와의 차이점은 없다. 다만 스프링 컨테이너에서 만들어질 뿐이다.**
- **Reference**
  - **스프링 IoC컨테이너에 의해서 관리되고 애플리케이션의 핵심을 이루는 객체들을 스프링에서는 빈즈(beans)라고 부른다.**
  - 빈은 **스프링 IoC컨테이너에 의해**서 **인스턴스화되어 조립되거나 관리되는 객체**를 말한다.
  - **이같은 점을 제외하고 빈은 수많은 객체들중의 하나일 뿐이다.**
  - 빈과 빈 사이의 의존성은 컨테이너가 사용하는 메타데이터 환경설정에 반영된다.



Bean의 주요 속성

* class(필수) : 정규화된 자바 클래스 이름
* id : bean의 고유 식별자
* scope : 객체의 범위
* constructor-arg : 생성 시 생성자에 전달할 인수
* property : 생성 시 bean setter에 전달할 인수
* init method와 destroy method



## 2 Bean 설정 방법

🙋‍♂️ **스프링에서 Bean을 설정하는 방법은 크게 3가지 이다.**

* xml을 이용한 빈 설정
* Component Scan을 이용한 빈 설정
* 자바 설정파일을 이용한 빈 설정



### 2-1 XML을 이용한 빈 설정

🙋‍♂️ XML (`application.xml`)에 등록하고 싶은 빈을 설정하고 `ClassPathXmlApplicationContext`를 사용하여 빈을 불러오는 방식

`application.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookService" class="com.springtest.demo.BookService"
          scope="singleton" autowire="default">
        <property name="bookRepository" ref="bookRepository"/>
    </bean>

    <bean id="bookRepository" class="com.springtest.demo.BookRepository"
          scope="singleton" autowire="default"/>

</beans>
```

`POJO`

```java
public class BookService {

    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}

public class BookRepository{
}

public class Client {
  public static void main(String[] args){
    // application.xml설정파일 등록(읽기)
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    // 모든 빈들의 이름 가져오기
    String[] beanDefinitionNames = context.getBeanDefinitionNames();
    System.out.println(Arrays.toString(beanDefinitionNames));
    BookService bookService = (BookService)context.getBean("bookService");
    System.out.println(bookService.bookRepository != null); // 의존성이 주입된 것을 볼 수 있다.
  }
}
// 결과
[BookService, BookRepository]
true
```

* `ApplicationContext`에게 `application.xml`을 등록하면, IoC컨테이너가 해당 설정 파일에 명시된 대로 의존성을 주입하여 객체들을 생성하여 주입해준다.
  * **이후에 클라이언트가 `getBean`을 사용해서 의존성이 모두 주입된 객체를 바로 받아서 사용할 수 있다.**



### 2-2 Component Scan을 이용한 빈 설정

🙋‍♂️ 스프링은 리플렉션과 애노테이션을 사용해서 Component scan을 지원한다.

`application.xml`

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.springtest.demo"/>

</beans>
```



```java
@Service
public class BookService {

  	@Autowired
    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}

@Repository
public class BookRepository{
}

public class Client {
  public static void main(String[] args){
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    String[] beanDefinitionNames = context.getBeanDefinitionNames();
    System.out.println(Arrays.toString(beanDefinitionNames));
    BookService bookService = (BookService)context.getBean("bookService");
    System.out.println(bookService.bookRepository != null); // 의존성이 주입된 것을 볼 수 있다.
  }
}

// 결과
[bookRepository, bookService, org.springframework.context.annotation.internalConfigurationAnnotationProcessor, org.springframework.context.annotation.internalAutowiredAnnotationProcessor, org.springframework.context.annotation.internalCommonAnnotationProcessor, org.springframework.context.event.internalEventListenerProcessor, org.springframework.context.event.internalEventListenerFactory]
true
```

* XML파일을 통해서 컴포넌트 스캔을 활성화하고 `@Autowired`, `@Service`, `@Repository`를 사용해서 빈을 설정하는 예시이다.
* 컴포넌트 스캔
  * 설정 방법
    * XML설정에서 `context:component-scan`
    * 자바 설정에서 `@ComponentScan`
  * **스프링은 특정 패키지 이하의 모든 클래스 중에 `@Component` 애노테이션을 사용한 클래스를 빈으로 자동으로 등록해준다.**



### 2-3 자바 설정파일을 이용한 빈 Scan

```java
// Java Config 객체
@Configuration
public class ApplicationConfig {
  	// 빈 등록
    @Bean
    public BookRepository bookRepository() {
        return new BookRepository();
    }

    // 빈 등록
    @Bean
    public BookService bookService() {
        BookService bookService = new BookService();
        bookService.setBookRepository(bookRepository()); // 의존성 주입
        return bookService;
    }
}

public class BookService {
  
    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}

public class BookRepository{
}

public class Client {
  public static void main(String[] args){
    ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class); // Java Config 읽기.
    String[] beanDefinitionNames = context.getBeanDefinitionNames();
    System.out.println(Arrays.toString(beanDefinitionNames));
    BookService bookService = (BookService)context.getBean("bookService");
    System.out.println(bookService.bookRepository != null); // 의존성이 주입된 것을 볼 수 있다.
  }
}
// true
```

* `AnnotationConfigApplicationContext`에  `ApplicationConfig` 자바 설정 파일을 등록하여 빈을 등록하는 예제

* 빈을 등록할 때 의존성을 주입하지 않고, 해당 클래스안에서 `@Autowired`나 생성자 혹은 Setter로 의존성을 주입해줘도 된다.

  * ```java
    public class BookService {
      @Autowired
      BookRepository bookRepository;
    }
    ```

    



### 3-4 ComponentScan을 사용한 빈 Scan

```java
@Configuration
@ComponentScan(basePackageClasses = DemoApplication.class)
public class ApplicationConfig {
  
}
```

* 일일이 `@Bean`을 설정하지 않아도, `baseClass`를 기반으로 애노테이션들을 탐색해서 빈들을 스캔한다.





## 3 Bean의 라이프사이클

![img](./image/22107F505693B43E29.png)





[Spring Bean Life Cycle](https://javaslave.tistory.com/48)

[Spring bean LifeCycle](http://wonwoo.ml/index.php/post/1820)



















## 4 Bean의 스코프

🙋‍♂️ 모든 빈에는 스코프가 존재한다.

<img src="./image/spring-bean-scope.png" />

```java
@Component
public class Single {
}

@Component @Scope("prototype")
public class Proto{
}

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext context;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("==== proto ====");

        System.out.println(context.getBean(Proto.class));
        System.out.println(context.getBean(Proto.class));
        System.out.println(context.getBean(Proto.class));

        System.out.println("==== Single ====");

        System.out.println(context.getBean(Single.class));
        System.out.println(context.getBean(Single.class));
        System.out.println(context.getBean(Single.class));

    }
}
// 결과
==== proto ====
com.springtest.demo.Proto@648d0e6d
com.springtest.demo.Proto@79e66b2f
com.springtest.demo.Proto@17273273
==== Single ====
com.springtest.demo.Single@5f69e2b
com.springtest.demo.Single@5f69e2b
com.springtest.demo.Single@5f69e2b
```







## 참고

* [스프링 레퍼런스](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-scopes)
* [백기선님 핵심 강의]()
* [Spring Bean의 개념과 Bean Scope 종류 - 블로그](https://gmlwjd9405.github.io/2018/11/10/spring-beans.html)

