# 목차

<br>

<br>

# 개요
이번 글에선 [Gradle 공식 문서]()를 참고하여 Gradle 빌드 스크립트 작성법에 대해서 다룬다.

깊은 내용보다는 Gradle을 처음 사용하는 개발자를 대상으로 쉬운 예시를 통해 작성법에 대해 이야기한다.

<br>

# Projects, Plugins, Tasks
> 본격적인 스크립트 작성법을 다루기 전에 스크립트의 기본이 되는 Projects, Plugins, Tasks에 대해서 설명한다.

* Projects
  * 모든 Gradle 빌드는 하나 혹은 그 이상의 Projects로 만들어진다.
  * Projects는 Gradle로 수행하는 작업을 의미한다. (ex. 라이브러리 JAR, 웹 애플리케이션등등)
* Tasks
  * 

<br>

# Hello World

**gradle명령어**

Gradle의 빌드 스크립트를 실행시키는 명령어는 `gradle`이다.

`gradle`은 현재 위치에 존재하는 `build.gradle`파일에게 명령을 보내는 명령어라고 이해하면 된다.

<br>

**간단한 예시**

```gradle
<!-- build.gradle -->
tasks.register('hello') {
    doLast {
        println 'Hello world!'
    }
}
```
```bash
gradle -q hello
```
`build.gradle`을 위와 같이 작성하고 `gradle -q hello` 명령하면 `Hello world!`가 나오는 것을 볼 수 있다.

<br>






