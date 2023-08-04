Gradle Build가 실행되면 Build Script인 build.gradle이 실행됨.

이번 글은 buidl.gradle을 해석해보는 시간을 가짐.

자바 gradle 만들면 있는 기본적인 아래 내용을 바탕으로 해석.

```build.gradle
plugins {
    id 'java'
}

group 'com.bignhe'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.1'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.1'
}

test {
    useJUnitPlatform()
}
```

plugins이 뭔지도 설명.

<br>

Task 관련 정리.

아래 글 참고
* https://kwonnam.pe.kr/wiki/gradle/task
* https://yearnlune.github.io/java/gradle-task-01/#%EA%B3%B5%ED%86%B5
* https://velog.io/@hyemin916/Gradle%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-2-Gradle-%EB%B9%8C%EB%93%9C-%EC%A7%9C%EB%B3%B4%EA%B8%B0

<br>


아래 글 참고

* https://blog.gradle.org/how-gradle-works-3

