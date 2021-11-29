# 목차

<br>

<br>

# 들어가며
필자는 토큰 저장소와 캐시 저장소로 Redis 저장소를 이용한다.

WAS는 Spring Boot를 이용했으며, Redis Client로는 Lettuce를 사용했다.

🤔 **왜 Lettuce를 사용했을까?? Redis 도입할 당시의 나는 크게 두 가지 이유때문이었다.**

1. **성능** -> 캐싱 기능을 모두 구현하고, 간단히 브라우저를 통해 비교해보니 Lettuce의 Latency가 더 좋은 성능을 보였다. (눈으로 확인..)
   * 좋은 성능 측정 방식은 아니다. 그저 직접 브라우저를 통해 요청을 몇십번 날리며 Latency를 비교하였다.
   * 또한, [이동욱님의 글 - Jedis 보다 Lettuce 를 쓰자](https://jojoldu.tistory.com/418)을 보면 Lettuce가 빠르다고 단정지었다.
2. **설정** -> Jedis는 Connection Pool을 이용하기 때문에 설정해줘야할 것이 많으며, 적절한 Pool Size를 찾는 것 또한 비용으로 다가왔다. 반면에, Lettuce는 비동기 방식이기에 설정해줘야할 사항이 비교적 적다고 생각했다.

시간이 흘러 **실제 면접때 `왜 Lettuce를 사용했느냐?`란 질문을 받고 위 2가지 이유를 말하였다.**

**하지만 면접이 끝난 후 복기를 하다 뜨끔했다. 과연 브라우저를 통한 Latency 비교가 적절한 선택 기준이었을까?.. 아니라고 본다.**

실제 Latency의 차이도 50-100ms 내외였다. 항상 Lettuce가 빠른 것도 아니였다. 이는 두 Redis Client의 성능의 차이라고 보기 어렵다. 

**이번 글은 이전에 무지성으로 Lettuce를 사용한 자신을 반성하며, 필자가 적용한 캐싱을 이용하여 Lettuce와 Jedis의 성능을 측정해보고자 한다.**

<br>

# 0 환경 구축

## 0-1 인프라 사양
테스트에 사용된 의존성 환경은 다음과 같다.

* Spring Boot 2.5.2
* Spring Boot Data Redis 2.5.2
* Jedis 3.6.1
* Lettuce 6.1.3
* Redis 4.0.9

<br>

인프라 구조는 다음과 같다.

<p align="center"><img src="./image/project_infra.png" > </p>

프로젝트의 인프라는 위와 같다. NginX는 HTTPS를 적용시켰으며 적절히 튜닝해둔 상태이다.

Redis 또한 RDB/AOF 기능을 OFF해두었으며, MaxClient는 50,000으로 설정해둔 상태입니다.

모든 애플리케이션은 EC2에 실행되고 있으며, `t2.medium`을 사용했다.

> 기존 프로젝트에서의 사양을 그대로 사용하고자, DB Replication은 그대로 두었다. 
> 
> WAS의 부하도 테스트하기 위해 로드 밸런싱은 우선 제거하고 하나의 WAS로만 테스트 하고자한다.

<br>

## 0-2 프로젝트 환경
필자가 Redis를 적용시킨 부분은 캐싱이며, 캐싱을 적용시킨 부분은 SNS에서의 홈피드 조회 요청이다.

Redis 캐싱 설정은 다음과 같다.

> RedisCacheConfiguration.java
```java
@EnableCaching
@Configuration
@Profile("!test")
public class RedisCachingConfiguration {

    private final RedisConnectionFactory redisConnectionFactory;
    private final ObjectMapper objectMapper;

    public RedisCachingConfiguration(
        RedisConnectionFactory redisConnectionFactory,
        ObjectMapper objectMapper
    ) {
        this.redisConnectionFactory = redisConnectionFactory;
        this.objectMapper = objectMapper;
    }

    public CacheManager redisCacheManager() {
        CollectionType collectionType = objectMapper.getTypeFactory()
            .constructCollectionType(ArrayList.class, PostResponseDto.class);

        RedisCacheConfiguration redisCachingConfiguration = RedisCacheConfiguration
            .defaultCacheConfig()
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()
                )
            )
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new Jackson2JsonRedisSerializer<>(
                        collectionType
                    )
                )
            )
            .entryTtl(Duration.ofMinutes(30));

        return RedisCacheManager
            .RedisCacheManagerBuilder
            .fromConnectionFactory(redisConnectionFactory)
            .cacheDefaults(redisCachingConfiguration)
            .build();
    }
}
```

<br>

홈피드 조회 서비스단에 캐싱을 적용시킨 코드는 다음과 같다.

> PostFeedService.java
```java
@Cacheable(
  key = "#homeFeedRequestDto.page",
  value = "homeFeed",
  condition = "#homeFeedRequestDto.guest == true",
  unless = "#result == null || #result.empty"
)
public List<PostResponseDto> homeFeed(HomeFeedRequestDto homeFeedRequestDto) {
  Pageable pageable = getPagination(homeFeedRequestDto);
  if (homeFeedRequestDto.isGuest()) {
    return PostDtoAssembler.assembleFrom(null,  postRepository.findAllPosts(pageable));
  }
  User requestUser = findUserByName(homeFeedRequestDto.getRequestUserName());
  List<Post> result = postRepository.findAllAssociatedPostsByUser(requestUser, pageable);
  return PostDtoAssembler.assembleFrom(requestUser, result);
}
```

<br>

최종적으로 캐싱은 다음과 같이 되도록 구현하였다.

<p align="center"><img src="./image/project_cache_type.png" > </p>

* `key : value` = `홈 피드 페이지 : 응답 JSON` 형식으로 저장되며, Redis에서는 `string:string` 형식이다.
  * Application단에서 결과 DTO를 JSON으로 직렬화하고 역직렬화하는 형식.

> 자세한 구현 방법에 대해서는 [여기](https://github.com/binghe819/TIL/blob/master/Spring/Cache/spring%20boot%20cache%20with%20redis/spring%20boot%20cache%20with%20redis%20-%20%EC%A0%81%EC%9A%A9%EA%B8%B0.md)를 참고.

<br>

## 0-3 테스트 데이터 양 및 Ngrinder 설정

테스트에 사용될 데이터 양은 다음과 같다.

<p align="center"><img src="./image/db_data_info.png" ><br>테스트 DB에 저장된 데이터 개수 </p>

**주요 대상인 게시물은 대략 100만건이다.**

<br>

부하 테스트는 [Ngrinder](https://naver.github.io/ngrinder/)를 사용했다.

* 테스트 시간: 10분
* VUser: 600
* 조회 범위: 1-1000 랜덤 페이지

<br>

# 1 Jedis
Jedis는 ...

<br>

먼저 Jedis의 의존성을 추가해준다.

> build.gradle
```gradle
...
implementation 'org.springframework.boot:spring-boot-starter-cache:2.5.2'
implementation group: 'redis.clients', name: 'jedis'
implementation group: 'org.apache.commons', name: 'commons-pool2', version: '2.6.2'
implementation ('org.springframework.boot:spring-boot-starter-data-redis') {
    exclude group: 'io.lettuce', module: 'lettuce-core'
}
implementation ('it.ozimov:embedded-redis:0.7.3') {
    exclude group: 'org.slf4j', module: 'slf4j-simple'
}
...
```

## 1-1 Default Setting (Pool Size - 8)
먼저 Jedis의 기본 설정을 통해 테스트를 진행해보았다.

> RedisConfiguration.java
```java
@EnableRedisRepositories
@Configuration
public class RedisConfiguration {

    private final String host;
    private final String password;
    private final int port;

    public RedisConfiguration(
        @Value("${security.redis.host}") String host,
        @Value("${security.redis.password}") String password,
        @Value("${security.redis.port}") int port
    ) {
        this.host = host;
        this.password = password;
        this.port = port;
    }

    @Bean
    @ConditionalOnMissingBean(RedisConnectionFactory.class)
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration configuration =
            new RedisStandaloneConfiguration(host, port);
        configuration.setPassword(password);
        
        return new JedisConnectionFactory(configuration);
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<byte[], byte[]> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }
}
```

<br>

**성능 테스트 결과**



<br>

## 1-2 Custom Setting (Pool Size - 128)
Jedis의 기본 설정을 통해 다음과 같은 결과를 얻었다.

* TPS가 생각보다 잘 나왔다.
* Redis Connection과 EC2 서버의 CPU가 여유로웠다.

**그래서 그 다음으로 시도한 테스트는 Jedis의 Connection Pool Size를 커스텀하여 테스트하는 것이다.**

<br>

커스텀하는 코드는 다음과 같다. 

> RedisConfiguration.java

```java
@EnableRedisRepositories
@Configuration
public class RedisConfiguration {

    ...

    @Bean
    @ConditionalOnMissingBean(RedisConnectionFactory.class)
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration configuration =
            new RedisStandaloneConfiguration(host, port);
        configuration.setPassword(password);

        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(configuration);

        GenericObjectPoolConfig<Jedis> poolConfig = jedisConnectionFactory
            .getPoolConfig();
        poolConfig.setMaxTotal(128);
        poolConfig.setMaxIdle(128);
        poolConfig.setMinIdle(36);
        poolConfig.setTestOnBorrow(true);
        poolConfig.setTestOnReturn(true);
        poolConfig.setTestWhileIdle(true);
        poolConfig.setMinEvictableIdleTimeMillis(Duration.ofSeconds(60).toMillis());
        poolConfig.setTimeBetweenEvictionRunsMillis(Duration.ofSeconds(30).toMillis());
        poolConfig.setNumTestsPerEvictionRun(3);
        poolConfig.setBlockWhenExhausted(true);

        return jedisConnectionFactory;
    }

    ...
}

```

<br>

**성능 테스트 결과**

<br>

## 1-3 Jedis 결론



<br>

# 2 Lettuce
Lettuce는 ...

Lettuce로 코드를 변경하여 다시 테스트를 진행해보았다.

> build.gradle

```gradle
...
implementation 'org.springframework.boot:spring-boot-starter-cache:2.5.2'
implementation 'org.springframework.boot:spring-boot-starter-data-redis:2.5.2'
implementation ('it.ozimov:embedded-redis:0.7.3') {
    exclude group: 'org.slf4j', module: 'slf4j-simple'
}
...
```
* spring boot data redis는 기본적으로 Lettuce를 사용한다.

<br>

> RedisConfiguration.java
```java
@EnableRedisRepositories
@Configuration
public class RedisConfiguration {

    ...

    @Bean
    @ConditionalOnMissingBean(RedisConnectionFactory.class)
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration configuration =
            new RedisStandaloneConfiguration(host, port);
        configuration.setPassword(password);
        return new LettuceConnectionFactory(configuration);
    }
    
    ...
}
```

<br>

**성능 테스트 결과**

<br>

# 3 성능 비교
* Redis 커넥션 수
* WAS 자원 사용량
* Redis 자원 사용량 (CPU 사용량, )

<br>

# 마치며
* 측정 대상: TPS, Latency
* 리소스 모니터링: CPU, 메모리, Connection개수

<br>



