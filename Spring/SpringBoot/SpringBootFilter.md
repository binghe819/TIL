# 목차

- [Spring Boot 필터 설정](#spring-boot-필터-설정)
  * [1 FilterRegistrationBean](#1-filterregistrationbean)
  * [2 @WebFilter 애너테이션 필터 등록](#2-webfilter-애너테이션-필터-등록)
- [참고](#참고)





# Spring Boot 필터 설정

일반적인 서블릿 애플리케이션에서는 `web.xml` 을 통해서 필터를 설정한다. [서블릿 필터 설정 방법](https://github.com/binghe819/TIL/blob/master/Spring/Servlet/ServletFilter.md)

```xml
<filter>
  <filter-name>필터 이름</filter-name>
  <filter-class>필터 클래스</filter-class>
</filter>

<filter-mapping>
  <filter-name>필터 이름</filter-name>
  <url-pattern>필터를 적용할 URL</url-pattern>
</filter-mapping>
```

스프링 부트에는 더이상 `web.xml` 을 사용하지 않는다. 그러므로 필터를 설정하는 방법도 자바 설정을 통해 설정해줘야한다.

스프링 부트에서 필터를 설정하는 방법은 두 가지이다.

* `FilterRegistrationBean` 을 이용하는 방법
* 스프링 부트 내장 컨테이너(톰캣)이 지원하는 `@WebServlet`, `@WebFilter`, `@WebListener` 선언해서 자동 등록 후 `@ServletComponentScan` 로 이용하는 방법



## 1 FilterRegistrationBean

`org.springframework.boot.web.servlet` 의 `FilterRegistrationBean` 을 사용하여 필터 클래스를 설정파일에 빈으로 등록하는 방식.

```java
public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("Test Filter Init()");
    }

    @Override
    public void destroy() {
        System.out.println("Test Filter Destory()");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("request - Test Filter doFilter()");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("response - Test Filter doFilter()");
    }
}
```

```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean getFilterRegistrationBean(){
        FilterRegistrationBean registrationBean = new FilterRegistrationBean(new TestFilter());
//        registrationBean.setUrlPatterns(Arrays.asList("/article/*"));
        registrationBean.addUrlPatterns("/test");
        return registrationBean;
    }
}
```



## 2 @WebFilter 애너테이션 필터 등록

스프링부트에서 지원하는 `@WebFilter` 를 사용하여 자동 등록후 `@ServletComponentScan` 으로 사용하는 방식

```java
@WebFilter(urlPatterns = "/test")
public class TestFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("Test Filter Init()");
    }

    @Override
    public void destroy() {
        System.out.println("Test Filter Destory()");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("request - Test Filter doFilter()");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("response - Test Filter doFilter()");
    }
}
```

```java
@SpringBootApplication
@ServletComponentScan // 서블릿컴포넌트스캔 추가!
public class Application {

    public static void main(String[] args) {
        Application.run(Application.class, args);
    }

}
```





# 참고

* https://linked2ev.github.io/gitlog/2019/09/15/springboot-mvc-13-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-MVC-Filter-%EC%84%A4%EC%A0%95/
* https://taetaetae.github.io/2020/04/06/spring-boot-filter/









