# TIL (Today I Learned) - feat. WIL (What I Learned)

[![Blog](https://img.shields.io/badge/Blog-binghe.github.io-green.svg)](https://binghedev.tistory.com/) [![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fbinghe819%2FTIL&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

❗️ 기술과 지식을 나만의 언어로 정제하지 않고 정리하는 저장소입니다. 비교적 정제된 내용은 [블로그](https://mark-kim.blog/)를 참고바랍니다.

<br>

## Spring

* Spring Core
  * IoC / DI - 예정
  * AOP - 예정
  * [빈 생명주기 및 콜백 메서드](./Spring/Core/bean%20lifecycle/bean%20lifecycle.md)
* Servlet
  * [Head First Servlet & First](./Spring/Servlet/Head%20First%20Servlets%20%26%20JSP/README.md)
  * [Servlet Filter](./Spring/Servlet/ServletFilter.md)
  * [Servlet Listener](./Spring/Servlet/ServletListener.md)
* Spring MVC
  * [Spring MVC 기초 개념](./Spring/MVC/Spring%20MVC%20%EA%B8%B0%EC%B4%88%20%EA%B0%9C%EB%85%90.md)
  * [Spring MVC Flow](./Spring/MVC/Spring%20MVC%20flow.md)
  * [서블릿에 SpringMVC 연동](./Spring/MVC/%EC%84%9C%EB%B8%94%EB%A6%BF%EC%97%90%20%EC%8A%A4%ED%94%84%EB%A7%81%20%EC%97%B0%EB%8F%99.md)
  * [DispatcherServlet](./Spring/MVC/DispatcherServlet.md)
  * [Spring MVC 설정](./Spring/MVC/Spring%20MVC%20%EC%84%A4%EC%A0%95.md)
  * [Spring MVC 활용](./Spring/MVC/Spring%20MVC%20%ED%99%9C%EC%9A%A9.md)
* Spring Boot
  * [Spring Boot 필터 설정](./Spring/SpringBoot/SpringBootFilter.md)
* Spring Cache
  * [Spring Cache Core (with 학습테스트)](./Spring/Cache/spring%20boot%20cache%20core/spring%20boot%20cache%20core.md)
* Spring Data Redis
  * [Spring RedisTemplate Serializer 종류와 사용시 주의할 점](./Spring/Redis/redis%20serializer/serializer.md)
  * [Spring으로 Redis를 사용하기 전 보면 좋은 글 (Redis Client, RedisTemplate, RedisRepository)](./Spring/Redis/spring으로%20redis를%20사용하기%20전에%20보면%20좋은%20큰%20그림/spring으로%20redis를%20사용하기%20전에%20보면%20좋은%20큰%20그림.md)
  * [내장 Redis 서버 - Embedded Redis](./Spring/Redis/spring%20boot%20embedded%20redis%20server/spring%20boot%20embedded%20redis%20server.md)
* Spring Batch
  * [배치 애플리케이션이란?](./Spring/Batch/배치애플리케이션이란/batch-application.md)
  * [Hello World로 보는 Spring Batch 시작하기](./Spring/Batch/spring-batch-helloworld-and-enabledbatchprocessing/spring-batch-helloworld-and-enabledbatchprocessing.md)
  * [디버깅을 통해 알아보는 Spring Batch 동작 원리](./Spring/Batch/spring-batch-helloworld-workflow/spring-batch-helloworld-workflow.md)
* DB
  * [Spring Boot + JPA기반의 Replication 구현 (Multi DataSource)](./Spring/DB/Replication/Replication%20With%20JPA.md)
* Test
  * [Spring DB 테스트 격리 (DatabaseCleaner를 곁들인)](./Spring/Test/DB%20테스트%20격리/DB%20테스트%20격리.md)
* Event
  * [Spring Boot기반 Event 기초](./Spring/Event/Spring%20Event%20Basic/SpringEventBasic.md)
* Spring Security
  * [JWT (with JJWT)](./Spring/Security/JWT.md)
* [토비의 스프링](https://github.com/binghe819/TIL/tree/master/Spring/%ED%86%A0%EB%B9%84%EC%9D%98%20%EC%8A%A4%ED%94%84%EB%A7%81)
* Validation
  * [Bean Validation와 Hibernate Validator](./Spring/Validation/Bean%20Validation와%20Hibernate%20Validator.md)
* Logging
  * [Spring Boot 기반 Logback 설정 및 전략](./Spring/Logging/logback/logback.md)
* 예외처리
  * [Spring Exception 처리 개념 및 전략 정리](./Spring/%EA%B8%B0%ED%83%80/%EC%8A%A4%ED%94%84%EB%A7%81%20%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%AC%20%EA%B0%9C%EB%85%90%20%EB%B0%8F%20%EC%A0%84%EB%9E%B5.md)
* Rest Client
  * RestTemplate
    * [RestTemplate - Quick Start](./Spring/RestClient/RestTemplate/quick-start/quick-start.md)
  * WebClient
  * 서킷 브레이커
* MSA
  * 서킷 브레이커
    * [서킷 브레이커 개념과 동작 원리](./Spring/MSA/circuitbreaker-개념/circuitbreaker-개념.md)
    * [resilience4j를 이용한 서킷 브레이커]()
    * [Spring기반의 resilience4j 서킷 브레이커]()
* AWS
  * [Spring Boot 환경 AWS SNS와 SQS를 이용한 이벤트 처리](./Spring/AWS/SNS_SQS/spring_boot_sns_sqs.md)
* 커스텀
  * [프록시를 이용한 Http Method 포함 인터셉터 설정기](./Spring/MVC/프록시를%20이용한%20Http%20Method%20포함%20인터셉터%20설정기.md)
  * [다이내믹 프록시를 이용한 QueryCounter 적용기](./Spring/기타/QueryCounter/다이내믹%20프록시를%20이용한%20QueryCounter%20적용기.md)
* 기타
  * [DataSource](./Spring/%EA%B8%B0%ED%83%80/DataSource.md)
  * [KeyHolder](./Spring/%EA%B8%B0%ED%83%80/KeyHolder.md)
  * [@Component vs @Bean](./Spring/기타/@Component%20vs%20@Bean.md)

<br>

## JPA
* [JPA 개념 정리 및 학습 테스트](https://github.com/binghe819/jpa-learning-sandbox)
* [cascade.REMOVE vs orphanremoval=true](./JPA/기타/cascade_remove_vs_orphanremoval/cascade_remove_vs_orphanremoval.md)
* N + 1
  * [N + 1 문제와 해결 방법](./JPA/N+1/N+1%20문제와%20해결%20방법.md)

<br>

## 빌드
* Gradle 이해하기
  * [Gradle 이해하기 시리즈의 첫번째 - Gradle이란 무엇인가?](./Build/Gradle%20기본/Gradle_이해하기_1_Gradle이란_무엇인가.md)
  * [Gradle 이해하기 시리즈의 두번째 - Gradle 시작하기](./Build/Gradle%20기본/Gradle_이해하기_2_Gradle_시작하기.md)
  * [Gradle 이해하기 시리즈의 세번째 - Gradle Daemon](./Build/Gradle%20기본/Gradle_이해하기_3_Gradle_Daemon.md)
  * [Gradle 이해하기 시리즈의 네번째 - Build Phases (LifeCycle)](./Build/Gradle%20기본/Gradle_이해하기_4_Build_Phases.md)
  * [Gradle 이해하기 시리즈의 다섯번째 - Build Script](./Build/Gradle%20기본/Gradle_이해하기_5_Build_Script.md)
* Gradle 의존성
  * [오픈소스 버전 별 의미](./Build/Gradle_Publish/publish_version.md)
  * [Gradle 모듈 Publish](./Build/Gradle_Publish/Gradle_Publish_Plugin.md)
* Gradle 멀티 모듈
  * [멀티 모듈 적용하기 with Gradle, Spring](./Build/멀티%20모듈/멀티%20모듈%20적용%20with%20Gradle.md)

<br>

## 운영체제
* [운영체제와 정보기술의 원리](./OS/운영체제와%20정보기술의%20원리/README.md)
* 리눅스
  * [리눅스 자주사용되는 명령어 정리](./OS/리눅스/리눅스%20명령어%20모음/linux_command_zip.md)
* [Blocking, Non-Blocking and Synchronous, Asynchronous](./OS/blocking_nonblocking_%26_synchronous_asynchronous/blocking_nonblocking_%26_synchronous_asynchronous.md)

<br>

## Infra & DevOps
* AWS
  * [IAM 개념과 AWS 계정 안전하게 사용하는 모범 사례](./Infra%26DevOps/AWS/IAM/IAM.md)
  * VPC
    * [VPC 이해하기 시리즈의 첫번째 - 일반적인 네트워크 통신 방식](./Infra%26DevOps/AWS/VPC/VPC_1.md)
    * [VPC 이해하기 시리즈의 두번째 - 실습을 통한 VPC 개념 이해하기](./Infra&DevOps/AWS/VPC/VPC_2.md)
  * ELB
    * [ELB 이해하기 시리즈의 첫번째 - ELB 개념과 특징](./Infra&DevOps/AWS/ELB_ASG/ELB_1.md)
  * SQS
    * [SQS 개념과 동작 원리 이해하기](./Infra%26DevOps/AWS/SQS/SQS.md)
    * [Spring Boot 환경 AWS SNS와 SQS를 이용한 이벤트 처리](./Spring/AWS/SNS_SQS/spring_boot_sns_sqs.md)
* CI/CD
  * [CI/CD란?](./Infra&DevOps/CI:CD/CI:CD란/CI:CD란.md)
  * Jenkins
    * [Jenkins 개념 및 설치 - 미완성](./Infra&DevOps/CI:CD/Jenkins/Jenkins%20개념%20및%20설치.md)
    * [Jenkins와 Github 연동하기](./Infra&DevOps/Jenkins/../CI:CD/Jenkins/freestyle/Jenkins와%20Github%20연동하기.md)
    * [freestyle을 통한 CI/CD](./Infra&DevOps/Jenkins/../CI:CD/Jenkins/freestyle/freestyle을%20통한%20CI:CD.md)
    * [pipeline을 통한 CI/CD](./Infra&DevOps/Jenkins/../CI:CD/Jenkins/pipeline/pipeline을%20통한%20이용한%20CI:CD.md)
* NginX
  * [NginX 설치](./Infra&DevOps/NGINX/NGINX%20설치.md)
  * [NginX 설정](./Infra&DevOps/NGINX/NGINX%20설정.md)
  * [NginX를 이용한 로드 밸런싱 (헬스 체크 모듈을 곁들인)](./Infra&DevOps/NGINX/NGINX%20로드%20밸런싱%20(무료%20헬스체크를%20곁들인).md)
* Pinpoint
  * [Pinpoint 소개 및 적용](./Infra%26DevOps/pinpoint/pinpoint_introduce.md)
* 성능 테스트
  * [웹 성능 진단](./Infra&DevOps/웹%20성능%20진단/웹%20성능%20진단.md)
  * [nGrinder 완벽가이드 - 개념, 설치, 부하테스트 Hello World](./Infra&DevOps/nGrinder/nGrinder%20%EC%99%84%EB%B2%BD%EA%B0%80%EC%9D%B4%EB%93%9C.md)
* 카프카
  * [카프카 Hello World (Quick Start)](./Infra%26DevOps/Kafka/hello-world/kafka-hello-world.md)

<br>

## 네트워크
* [모두의 네트워크](./Network/모두의%20네트워크/README.md)
* [컴퓨터 네트워크 - kocw](./Network/%EC%BB%B4%ED%93%A8%ED%84%B0%20%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/README.md)
* [REST / REST API / RESTful](./Network/REST%20API/REST%20API.md)
* [SSH](./Network/SSH/ssh.md)
* [암호화 기초](./Network/Security/%EC%95%94%ED%98%B8%ED%99%94%20%EA%B8%B0%EC%B4%88.md)
* [CORS](./Network/CORS/CORS.md)
* [OAuth 2.0](./Network/OAuth%202.0/OAuth2.0.md)
* RPC
  * [JAVA로 gRPC 실습해보기 (feat. 4가지 통신 기법)](./Network/grpc/grpc_java/grpc_java.md)
* HTTP
  * [HTTP 진화과정 이해하기 첫번째 - HTTP 0.9부터 HTTP 1.1까지](./Network/HTTP_0_9_to_1_1/HTTP_0_9_to_1_1.md)
  * [HTTP 진화과정 이해하기 두번째 - HTTP 2.0](./Network/HTTP2_0/HTTP2_0.md)
  * [HTTP 완벽 가이드 - 진행중](./Network/HTTP%20The%20Definitive%20Guide/README.md)
  * [HTTPS](./Network/HTTPS/HTTPS.md)
* [Data Serialization Format](https://en.wikipedia.org/wiki/Comparison_of_data-serialization_formats)
  * 

<br>

## 데이터베이스

* [데이터베이스 첫걸음](./DB/RDB/%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%B2%A0%EC%9D%B4%EC%8A%A4%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%B2%A0%EC%9D%B4%EC%8A%A4%20%EC%B2%AB%EA%B1%B8%EC%9D%8C.md)
* 데이터 모델링
  * [데이터 모델링 기초 및 DB 설계](./DB/RDB/%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%AA%A8%EB%8D%B8%EB%A7%81/%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%AA%A8%EB%8D%B8%EB%A7%81%20%EA%B8%B0%EC%B4%88%20%EB%B0%8F%20DB%20%EC%84%A4%EA%B3%84.md)
  * [정규형](./DB/RDB/%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%AA%A8%EB%8D%B8%EB%A7%81/%EC%A0%95%EA%B7%9C%ED%98%95.md)
  * [NoSQL - 빅데이터 세상으로 떠나는 간결한 안내서](./DB/NoSQL_book/README.md)
  * [파티셔닝과 샤딩](./DB/RDB/partitioning_vs_sharding/partitioning_vs_sharding.md)
* 관계형 데이터베이스
  * [트랜잭션](./DB/RDB/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98/Transaction.md)
  * 인덱스 - 예정
* [h2](./DB/RDB/h2/h2.md)
* SQL
  * [SQL 정리](./DB/RDB/SQL/SQL%EC%A0%95%EB%A6%AC.md)
  * [JOIN](./DB/RDB/JOIN/join.md)
  * [IN, NOT IN, EXISTS, NOT EXISTS](./DB/RDB/IN_EXISTS_NOT_IN_NOT_EXISTS/IN_EXISTS_NOT_IN_NOT_EXISTS.md)
* MySQL
  * [MySQL 5.7 설치 및 설정](./DB/MySQL/MySQL%20설치%20및%20설정/MySQL%205.7%20설치%20및%20설정.md)
  * [InnoDB Lock 종류 및 이해하기](./DB/MySQL/InnoDB%20Lock/InnoDB%20Lock.md)
  * [MySQL DeadLock 확인](./DB/MySQL/DeadLock%20확인/MySQL%20InnoDB%20DeadLock%20확인.md)
  * [Insert Intention Waiting DeadLock 원인 및 해결 (Insert시 DeadLock 발생)](./DB/MySQL/insert%20intention%20waiting%20DeadLock/insert%20intention%20waiting%20deadlock.md)
  * [Real MySQL](./DB/MySQL/RealMySQL/README.md)
  * [샘플 데이터](./DB/MySQL/샘플%20데이터/샘플%20데이터.md)
  * [Replication 개념과 MySQL을 이용한 실습](./DB/MySQL/Replication/MySQL%20Replication.md)
* Redis
  * [Redis 설치](./DB/Redis/Redis설치/Redis설치.md)
  * [Redis 설정](./DB/Redis/Redis설정/Redis설정.md)
  * [Redis는 무엇이고, 어떻게 사용하는 것이 좋은가](./DB/Redis/Redis는%20무엇이고,%20어떻게%20사용하는%20것이%20좋은가/Redis는%20무엇이고,%20어떻게%20사용하는%20것이%20좋은가.md)
  * [Redis 자료구조와 명령어 & 활용사례](./DB/Redis/Redis%20자료구조와%20명령어/Redis%20자료구조와%20명령어%20&%20활용사례.md)
  * [Redis Memory Analyzer (RMA)](./DB/Redis/rma/rma.md)
* ElasticSearch
  * [ElasticSearch 설치 및 실행](./DB/ES/installation/es_installation.md)
* MongoDB
  * [MongoDB 설치](./DB/MongoDB/installation/mongo_installation.md)
* 기타
  * [B-Tree 이해하기](./DB/기타/B-Tree/B-Tree.md)
  * [CAP, PACELC 이론과 DB 선택의 기준](./DB/CAP_PACELC/CAP_PACELC_theorem.md)
  * [Inverted Index (역색인)](./DB/기타/Inverted_Index/Inverted_Index.md)

<br>

## JAVA
* JAVA Core
  * [JVM 구조](./JAVA/JVM/jvm_structure.md)
  * [ClassLoader](./JAVA/JVM/classloader.md)
  * [reflection](./JAVA/JVM/reflection.md)
  * [가비지 컬렉션 이해하기 - 가비지 컬렉션이 무엇이고 어떻게 동작하는가?](./JAVA/JVM/garbage_collection_how_to_works.md)
  * [JAR 개념 및 구조](./JAVA/jar/jar.md)
* [JCF (Java Collections Framework)](./JAVA/JCF/README.md)
* [Thread & Thread Pool](./JAVA/Thread%26ThreadPool/README.md)
  * [Thread 기본 개념 (Thread란 무엇인가)](./JAVA/Thread%26ThreadPool/Thread%20%EA%B0%9C%EB%85%90.md)
  * [Java Thread 기본](./JAVA/Thread%26ThreadPool/Java%20Thread%20%EA%B8%B0%EB%B3%B8.md)
  * [Executor, ExecutorService (feat. Thread Pool)](./JAVA/Thread%26ThreadPool/Thread%20Pool.md)
* [비동기 프로그래밍](./JAVA/Asynchronous%20Programming/READMD.md)
  * [Blocking, Non-Blocking and Synchronous, Asynchronous](./OS/blocking_nonblocking_%26_synchronous_asynchronous/blocking_nonblocking_%26_synchronous_asynchronous.md)
  * [자바 비동기 처리와 Future](./JAVA/Asynchronous%20Programming/Java%20Asynchronization-Future.md)
  * [Future 이해하기 (1) - Future은 어떻게 비동기 결과를 얻는가?](./JAVA/Asynchronous%20Programming/Java%20Asynchronization-Understand-Future-1.md)
  * [Future 이해하기 (2) - FutureTask의 동작 원리]()
  * [안정적 비동기 프로그래밍 - CompletableFuture](./JAVA/Asynchronous%20Programming/Java%20Asynchronization-CompletableFuture.md)
* Network
  * I/O
    * [사례를 통해 이해하는 네트워크 논블로킹 I/O와 Java NIO](./JAVA/network/nio/io-nio-example-server/io-nio-example-server.md)
    * [사례를 통해 이해하는 Event Loop](./JAVA/network/nio/event_loop/event_loop.md)
  * Netty
    * [Netty가 빠른 이유와 동작 흐름](./JAVA/netty/example_netty_workflow/index.md)
  * HTTP
    * URLConnection
      * [URLConnection & HttpURLConnection](./JAVA/network/UrlConnection&HttpUrlConnection/UrlConnection&HttpUrlConnection.md)
    * HttpClient (HttpComponents)
      * [Apache HttpClient 4 사용 방법 (Quick-Start)](./JAVA/network/HttpClient/apache_httpclient_quick_start.md)
      * [Apache HttpClient 4 코어 - 핵심 구조 (ExecChain)](./JAVA/network/HttpClient/apache_httpclient_core.md)
* Reactive Programming
  * [Reactive Streams](./JAVA/Reactive%20Programming/Reactive%20Streams/Reactive%20Streams.md)
* 로깅
  * [자바 로깅 프레임워크 히스토리](./JAVA/Logging/history_of_java_logging/history_of_java_logging.md)
  * [slf4j 이해하기](./JAVA/Logging/slf4j/slf4j.md)
  * [Logback 이해하기](./JAVA/Logging/Logback/Logback.md)
* Serialization / Deserialization (직렬화 / 역직렬화)
  * [자바 직렬화 이해하기 - 사용시 주의할 점과 사용을 추천하지 않는 이유](./JAVA/serialization_deserialization/java_serialization/java_serialization.md)
  * [Jackson 이해하기 - 동작 방식과 사용 방법 SOON](./JAVA/jackson/jackson.md)
    * [TypeRefrence - SOON]()
  * [Gzip with Jackson - JSON 용량 줄이기 SOON]()
* [자바의 정석](./JAVA/자바의%20정석/README.md)
* [Effective Java 3/E](./JAVA/Effective%20Java/README.md)
* [모던 자바 인 액션](./JAVA/Modern%20Java%20In%20Action/README.md)
* Test
  * [JUnit 5](./Test/JUnit/JUnit5.md)
  * [AssertJ](./Test/AssertJ.md)
  * [Mockito](./Test/Mockito/Mockito.md)
* 기타
  * [Stream](./JAVA/%EB%AA%A8%EB%8D%98%20%EC%9E%90%EB%B0%94/Stream.md)
  * [Optional](./JAVA/%EB%AA%A8%EB%8D%98%20%EC%9E%90%EB%B0%94/Optional.md)
  * [ThreadLocal](./JAVA/%EA%B8%B0%ED%83%80/ThreadLocal.md)
  * [불변 객체](./JAVA/%EA%B8%B0%ED%83%80/%EB%B6%88%EB%B3%80%20%EA%B0%9D%EC%B2%B4.md)
  * [Enum](./JAVA/%EA%B8%B0%ED%83%80/%EC%97%B4%EA%B1%B0%ED%98%95(enum).md)
* Java Convention
  * [SUN Java Convention](./JAVA/%EA%B8%B0%ED%83%80/SUN%20Java%20Convention.md)
  * [Google Java Convention](./JAVA/%EA%B8%B0%ED%83%80/google%20java%20style%20guide.md)

<br>

## Kotlin
* [Kotlin In Action](./Kotlin/Kotlin%20In%20Action/README.md)

<br>

## Python
* [Python 설치 및 venv 가상 환경 - mac](./Python/install_&_venv_for_mac/install_&_venv_for_mac.md)
* [folium](./Python/folium/README.md)

<br>

## OOP 및 아키텍처

* 책
  * [객체지향의 사실과 오해 정리](./OOP%26%EC%84%A4%EA%B3%84/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5%EC%9D%98%20%EC%82%AC%EC%8B%A4%EA%B3%BC%20%EC%98%A4%ED%95%B4/README.md)
  * [오브젝트 - 코드로 이해하는 객체지향 설계](./OOP&설계/%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8)
  * [만들면서 배우는 클린 아키텍처](./OOP&설계/Get%20your%20hands%20dirty%20on%20clean%20architecture/README.md)
  * [도메인 주도 개발 시작하기](./OOP%26%EC%84%A4%EA%B3%84/DDD/%EB%8F%84%EB%A9%94%EC%9D%B8%20%EC%A3%BC%EB%8F%84%20%EA%B0%9C%EB%B0%9C%20%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/README.md)
* DDD
  * [DDD 사실과 오해](./OOP&설계/DDD/DDD의_사실과_오해/ddd_fact_and_misunderstand.md)
  * [DDD 지향 마이크로 서비스 디자인](./OOP&설계/DDD/DDD_지향_마이크로_서비스_디자인/ddd_oriented_micro_service_design.md)
* SOLID
  * [SRP](./OOP&설계/SOLID/SRP.md)
  * [OCP](./OOP&설계/SOLID/OCP.md)
  * [LSP](./OOP&설계/SOLID/LSP.md)
  * [ISP](./OOP&설계/SOLID/ISP.md)
  * [DIP](./OOP&설계/SOLID/DIP.md)
* 디자인 패턴
  * [전략 패턴](./OOP&설계/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4/Strategy%20Pattern.md)
  * [템플릿 메서드 패턴](./OOP&설계/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4/Template%20Method%20Pattern.md)
  * [상태 패턴](./OOP&설계/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4/State%20Pattern.md)
  * [프론트 컨트롤 패턴](./OOP&설계/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4/Front%20Controller%20Pattern.md)
  * [데코레이터 패턴](./OOP&설계/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4/Decorator%20Pattern.md)
  * [프록시 패턴와 데코레이터 패턴](./OOP&설계/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4/../디자인패턴/Proxy%20Pattern와%20Decorator%20pattern.md)
  * [다이내믹 프록시](./OOP&설계/디자인패턴/Dynamic%20Proxy.md)
* 기타
  * [DTO, VO](./OOP&설계/기타/DTO%2C%20VO.md)
  * [원시값 포장과 VO](./OOP&설계/기타/원시값%20포장과%20VO.md)

<br>

## JavaScript

* [Vanilla JS](./JS/Vanilla%20JS/README.md)
* [WEB APIs](./JS/web/README.md)
* [Node.js와 NPM](./JS/node.js와%20npm/node.js와%20npm.md)

<br>

## React

* [React 이해하기](./React/README.md)

<br>

## 지도
* [공간정보 기본 이해하기 feat. GIS](./ETC/공간정보/공간정보%20기본%20이해하기%20-%20GIS/공간정보_기본_이해하기.md)
* [좌표계 기본 이해하기](./ETC/공간정보/좌표계%20기본%20이해하기/좌표계_기본_이해하기.md)
* [Geofence 기본 개념과 벡터 기반의 PIP(Point In Polygon) 알고리즘](./ETC/공간정보/Point%20in%20polygon%20-%20GeoFence/Point%20in%20polygon%20-%20GeoFence.md)
* [GeoHash 기본 개념과 Geo-hash를 이용한 Point In Polygon](./ETC/공간정보/Point%20in%20polygon%20-%20GeoHash/Point%20in%20polygon%20-%20GeoHash.md)
* [GeoHash 학습 테스트 - Python](https://nbviewer.org/github/binghe819/TIL/blob/master/ETC/%EA%B3%B5%EA%B0%84%EC%A0%95%EB%B3%B4/Point%20in%20polygon%20-%20GeoHash/Geo-Hash.ipynb)
* [H3 기본 개념](./ETC/공간정보/H3/H3_basic.md)
* [H3 학습 테스트 - Python](https://nbviewer.org/github/binghe819/TIL/blob/master/ETC/%EA%B3%B5%EA%B0%84%EC%A0%95%EB%B3%B4/H3/H3.ipynb)
* [H3를 이용한 Point In Polygon](./ETC/공간정보/H3/Point%20in%20polygon%20-%20H3.md)

<br>

## ETC

* [Clean Code](./ETC/CleanCode/README.md)
* 테스트
  * [단위 테스트](./Test/TestGuide/%EB%8B%A8%EC%9C%84%ED%85%8C%EC%8A%A4%ED%8A%B8.md)
  * [책 - 단위 테스트](./Test/Unit%20Testing/README.md)
  * [테스트 더블](./Test/TestDouble/TestDouble.md)  
* [컴퓨터 개발 세팅](./ETC/컴퓨터세팅/setting.md)
* [Git 명령어 모음](./ETC/git/git-command.md)
