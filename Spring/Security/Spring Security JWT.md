# 목차







# Spring Security JWT

Spring Security를 사용하여 JWT를 도입하는 과정에서 모르는 내용들을 정리한 내용입니다.



## 1 JWT 개념

🤔 JSON Web Token이 무엇인가?



### 1-1 JWT란?

#### JWT란?

* JWT는 클라이언트와 서버 (서비스와 서비스 사이) 통신시 권한 인가를 위해 사용하는 토큰이다.
* JWT는 웹표준 [RFC 7519](https://tools.ietf.org/html/rfc7519)으로서 두 개체에서 JSON 객체를 사용하여 가볍고 자가수용적인방식으로 정보를 안전하게 전달해준다.



#### 자가 수용적 (self-contained)이다.

* JWT는 필요한 모든 정보를 자체적으로 지니고 있다.



#### 토큰 기반 인증

* JWT는 토근 기반 인증의 방법중 하나이다.

> 토큰 기반 인증에 대한 자세한 내용은 [여기](https://velopert.com/2350)를 참고 ( **꼭 참고하자**. )



### 1-2 JWT 구조와 생성

<img src="image/image-20201002224015534.png" width="600" />

```json
HEADER.PAYLOAD.SIGNATURE
```

JWT는 점을 기준으로 다음과 같이 구성된다.

* 헤더 (Header)
* 페이로드 (Payload)
* 서명 (Signature)

> JWT를 검증하는데 필요한 정보를 가진 JSON객체는 **Base64 URL-Safe 인코딩된 문자열**이다.



#### Header

**헤더는 JWT를 어떻게 검증(Verify)하는가**에 대한 내용을 담고 있다.

```json
{
  "alg" : "HS256",
  "typ" : "JWT"
}
```

* alg - 서명 시 사용하는 알고리즘.
  * 이 알고리즘은 토큰을 검증할 때 사용되는 signature 부분에서 사용된다.
* typ - 토큰의 타입을 지정.



#### Payload

**Payload는 JWT의 내용을 담고 있다.** 

Payload에 담는 정보의 **한 단위를 클레임 (Claim)이라 하며, name / value 형태**로 되어 있다.

Payload에 있는 속성들을 **클레임 셋 (Claim Set)**이라 부른다.

```json
{
  "uid" : "19234",
  "userName" : "binghe"
}
```

* Claim의 종류
  * 등록된 (registered) 클레임 - 이미 예약된 Claim
  * 공개 (public) 클레임 - 사용자 정의 Claim
    * 충돌 방지를 위해 URI 포맷을 이용해 저장한다.
  * 비공개 (private) 클레임 - 사용자 정의 Claim
    * 일반적인 정보를 저장한다. (이름의 충돌 위험이 있다.)



#### Signature

**점을 구분자**로 해서 **Header와 Payload를 합친 문자열을 서명한 값**이다.

즉, Header의 인코딩값과 Payload의 인코딩값을 합친후 주어진 비밀키로 해쉬를 사용하여 생성한다.

```
HSACSHA256 (
	base64UrlEncode(header) + "." +
	base64UrlEncode(payload),
	secret
)
```



## 1-3 JWT 구현

> jwt 토큰을 생성하고 검증하는데 사용되는 `jjwt` 라이브러리를 이용해 구현해보았다.

* jwt 토큰 생성
* jwt 토큰 파싱 및 검증



### Jwts

[Jwt Doc](https://github.com/jwtk/jjwt)



### 의존성 추가

```xml
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt</artifactId>
  <version>0.9.1</version>
</dependency>
<dependency>
  <groupId>jakarta.xml.bind</groupId>
  <artifactId>jakarta.xml.bind-api</artifactId>
  <version>2.3.3</version>
</dependency>
```

* JAVA 11버전에서 `jjwt`을 사용하려면 `DatatypeConverter`의 의존성이 필요하다. 



### 토큰 생성 및 검증 객체 구현

```java
public class Jjwt {

    /**
     * 토큰 생성
     * @param secretKey - 키
     * @param headers - 헤더 (Map)
     * @param payloads - 페이로드 (Map)
     * @return (String)
     */
    public static String createToken(String secretKey, Map<String, Object> headers, Map<String, Object> payloads) {
        return Jwts.builder()
                .setHeader(headers)
                .setClaims(payloads)
                .signWith(SignatureAlgorithm.HS256, secretKey.getBytes())
                .compact();
    }

    /**
     * 토른 파싱 및 검증
     * @param jwtTokenString - JWT 토큰 문자열
     * @param secretKey - 키
     */
    public static void getTokenFromJwtString(String jwtTokenString, String secretKey) {
        Claims claims = Jwts.parser()
                .setSigningKey(secretKey.getBytes())
                .parseClaimsJws(jwtTokenString)
                .getBody();

        Date expriration = claims.get("exp", Date.class);
        System.out.println(expriration);

        String data = claims.get("data", String.class);
        System.out.println(data);
    }

}

```



### 테스트 코드

```java
class JjwtTest {
    @Test
    void JWT_키_생성_및_검증() {
        // 비밀키
        String secretKey = "binghe";

        // 헤더
        Map<String, Object> headers = new HashMap<>();
        headers.put("typ", "JWT");
        headers.put("alg", "HS256");

        // 페이로드
        Map<String, Object> payloads = new HashMap<>();
        Long expiredTime = Long.valueOf(1000 * 600); // 만료기간
        Date now = new Date();
        now.setTime(now.getTime() + expiredTime); // 현재시간 + 만료기간
        payloads.put("exp", now);
        payloads.put("data", "Hello JwtWorld"); // 데이터

        String newToken = Jjwt.createToken(secretKey, headers, payloads);
        System.out.println(newToken); // 토큰 출력

        // 토큰 검증
        Claims tokenClaims = Jjwt.getTokenClaimsFromJwtString(newToken, secretKey);
        Date expiration = tokenClaims.get("exp", Date.class);
        System.out.println(expiration);

        String data = tokenClaims.get("data", String.class);
        assertEquals(data, "Hello JwtWorld");
    }
}
```





## 2 Spring Security JWT 도입



### 2-1 Spring Boot + Jwt Flow























### 2-2 JWT 커스터마이징 각자의 역할















# 참고

* https://meetup.toast.com/posts/239
* https://velopert.com/2389
* https://velopert.com/2350
* https://galid1.tistory.com/588













