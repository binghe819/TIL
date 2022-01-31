# 목차

<br>

- [목차](#목차)
- [들어가며](#들어가며)
- [설치 및 실행](#설치-및-실행)
- [Gradle Wrapper](#gradle-wrapper)
  - [Gradle Wrapper란?](#gradle-wrapper란)
  - [어떻게 동작하는가](#어떻게-동작하는가)
- [Hello World를 통해 보는 Gradle 빌드 구조](#hello-world를-통해-보는-gradle-빌드-구조)
  - [Gradle 빌드 구조](#gradle-빌드-구조)
- [마치며](#마치며)

<br>

# 들어가며
이번 글은 `Gradle 시작하기`의 주제로 간단히 어떻게 Gradle을 설치 및 시작하는지에 대해서 다룬다.

이 글을 읽기 전 [Gradle이란 무엇인가?](./Gradle이란%20무엇인가?.md)를 먼저 보는 것을 추천한다.

> 이 글은 [Gradle docs - Gettign Started](https://docs.gradle.org/current/userguide/getting_started.html)을 정리하며 살을 조금 붙인 글입니다.

<br>

# 설치 및 실행
아마 대부분의 개발자들은(특히, Java나 Spring) **IDE에서 프로젝트를 생성하기때문에 따로 Gradle을 설치하지 않는다.**

<br>

🤔 어떻게 Gradle을 설치하지 않고도 Gradle을 이용하여 빌드할 수 있을까?

* Gradle은 꼭 설치하지 않아도 사용할 수 있는 기능을 제공하기 때문이다.
* Gradle의 설치는 두 가지로 나눈다.
    1. Gradle설치 -> 설치 (컴퓨터에 전역적으로 사용할 Gradle을 설치하는 방법)
    2. Gradle Wrapper -> 무설치 (프로젝트 상위 디렉토리에 [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html#gradle_wrapper)를 사용하는 방법)

<br>

직접 Gradle을 설치해서 사용하는 경우는 거의 없기에 이번 글에선 Gradle Wrapper를 통한 Gradle의 설치와 사용에 대해서 다룬다..

> 만약, Gradle의 설치 방법을 알고싶다면 [여기](https://docs.gradle.org/current/userguide/installation.html#installation)를 참고.

<br>

# Gradle Wrapper
빌드 루트에 식별할 수 있는 [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html#gradle_wrapper)가 존재한다면 `gradlew`또는 `gradlew.bat`을 실행함으로써 굳이 Gradle을 설치하지 않고도 프로젝트를 빌드할 수 있다.

> 단, Gradle은 JVM위에서 동작하기 때문에 컴퓨터에 JDK 8이상이 설치되어있어야한다.

<br>

실제 Gradle 공식 Docs에서는 `Gradle Wrapper`방식을 더 추천한다.

그 이유는 다음과 같다. (Gradle Wrapper 방식의 장점)

1. 새로운 환경에서 매번 Gradle을 수동으로 설치할 필요없이 쉽게 프로젝트를 빌드할 수 있다. (종속적이지 않다)
2. 주어진 Gradle 버전에 따라 프로젝트를 표준화하여 더 안정적이고 강력한 빌드를 할 수 있다. (프로젝트를 빌드하는 모든 컴퓨터 환경을 일치하게 할 수 있다)
3. Wrapper의 내용을 수정함으로써 간단히 다른 사용자 및 실행 환경 (IDE, Jenkins등등)에 새 Gradle 버전을 프로비저닝할 수 있다.

<br>

## Gradle Wrapper란?

<br>

🤔 그렇다면 Wrapper란 무엇을 의미할까?
* **Wrapper는 선언된 버전의 Gradle을 호출하여 필요한 경우 미리 다운로드하는 스크립트다.**
  * 이는 **빌드시 사용되는 버전에 맞는 gradle 등을 다운받는 코드를 의미한다.**
  * 더 정확히는 `프로젝트/gradle/wrapper`에 위치한 두 파일을 가리킨다.
  * `gradle-wrapper.jar`
  * `gradle-wrapper.properties`

<br>

## 어떻게 동작하는가

<br>

🤔 Gradle Wrapper는 어떻게 동작하는가?

<p align="center"><img src="./image/wrapper-workflow.png" width="400"><br>wrapper workflow</p>

* `gradlew`을 통해 빌드하면 `gradle-wrapper.properties`에 정의된 Gradle 정보에 빌드가 진행된다.
  * 이때, **정의된 Gradle 버전이 로컬에 없으면 외부 서버로부터 다운받아 로컬에 저장시키고 이를 사용하여 빌드를 진행한다.**

<br>

# Hello World를 통해 보는 Gradle 빌드 구조
> Gradle Wrapper 방식의 구조라고 보면 된다.

<br>

## Gradle 빌드 구조

🤔 **Gradle 빌드 구조**

Hello World를 통해 간단히 Java Gradle 프로젝트를 생성하여 구조를 알아본다.

```bash
$ gradlw init

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4] 2

Select implementation language:
  1: C++
  2: Groovy
  3: Java
  4: Kotlin
  5: Scala
  6: Swift
Enter selection (default: Java) [1..6] 3

Split functionality across multiple subprojects?:
  1: no - only one application project
  2: yes - application and library projects
Enter selection (default: no - only one application project) [1..2] 1

Select build script DSL:
  1: Groovy
  2: Kotlin
Enter selection (default: Groovy) [1..2] 1

Generate build using new APIs and behavior (some features may change in the next minor release)? (default: no) [yes, no] no
Select test framework:
  1: JUnit 4
  2: TestNG
  3: Spock
  4: JUnit Jupiter
Enter selection (default: JUnit Jupiter) [1..4] 4

Project name (default: gradle-getting-started): gradle-getting-started
Source package (default: gradle.getting.started):

> Task :init
Get more help with your project: https://docs.gradle.org/7.3.3/samples/sample_building_java_applications.html

$ tree

.
├── app
│   ├── build.gradle
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── gradle
│       │   │       └── getting
│       │   │           └── started
│       │   │               └── App.java
│       │   └── resources
│       └── test
│           ├── java
│           │   └── gradle
│           │       └── getting
│           │           └── started
│           │               └── AppTest.java
│           └── resources
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── settings.gradle

```
* `app`
  * `build.gradle`: 빌드할 때의 Task를 정의하는 파일. (빌드 스크립트)
  * `src`: 자바 소스코드
* `gradle.wrapper`
  * `gradle-wrapper.jar`: Wrapper 실행 파일. 실행 스크립트가 실행되면 wrapper에 맞는 환경을 로컬 캐시에 다운로드 받은 뒤에 실제 명령에 해당하는 task를 실행한다.
    * code required for downloading the correct Gradle version when you run the build
    * The Wrapper JAR file containing code for downloading the Gradle distribution.
  * `gradle-wrapper.properties`: Wrapper 설정 파일. (gradle 버전등등)
    * file to configure the wrapper’s properties such as the Gradle version
    * A properties file responsible for configuring the Wrapper runtime behavior e.g. the Gradle version compatible with this version.
* `gradlew`: 유닉스용 wrapper 실행 스크립트. 컴파일, 빌드등 Gradle의 Task를 실행할 때 사용한다. (ex. `gradlew {task}`)
* `gradlew.bat`: 윈도우용 wrapper 실행 스크립트.
* `setting.gradle`: 빌드 관련 설정

> 더 자세한 내용은 [Building Java Applications Sample](https://docs.gradle.org/current/samples/sample_building_java_applications.html)를 참고.

<br>

💁‍♂️ **빌드 해보기**

**구조에 대해서 간단히 알아보았으니 이제 빌드를 진행해본다.**

```bash
# 빌드
$ ./gradlew build

BUILD SUCCESSFUL in 21s

# 빌드 및 자바 실행
$ ./gradlew run

> Task :app:run
Hello World!

BUILD SUCCESSFUL in 1s
2 actionable tasks: 2 executed
```

<br>

**`gradle tasks`를 통해 실행 가능한 명령을 볼 수 있다.**

```bash
$ ./gradlew tasks

> Task :tasks

------------------------------------------------------------
Tasks runnable from root project 'gradle-getting-started'
------------------------------------------------------------

Application tasks
-----------------
run - Runs this project as a JVM application

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
classes - Assembles main classes.
clean - Deletes the build directory.
jar - Assembles a jar archive containing the main classes.
testClasses - Assembles test classes.

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Distribution tasks
------------------
assembleDist - Assembles the main distributions
distTar - Bundles the project as a distribution.
distZip - Bundles the project as a distribution.
installDist - Installs the project as a distribution as-is.

Documentation tasks
-------------------
javadoc - Generates Javadoc API documentation for the main source code.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'gradle-getting-started'.
dependencies - Displays all dependencies declared in root project 'gradle-getting-started'.
dependencyInsight - Displays the insight into a specific dependency in root project 'gradle-getting-started'.
help - Displays a help message.
javaToolchains - Displays the detected java toolchains.
outgoingVariants - Displays the outgoing variants of root project 'gradle-getting-started'.
projects - Displays the sub-projects of root project 'gradle-getting-started'.
properties - Displays the properties of root project 'gradle-getting-started'.
tasks - Displays the tasks runnable from root project 'gradle-getting-started' (some of the displayed tasks may belong to subprojects).

Verification tasks
------------------
check - Runs all checks.
test - Runs the test suite.

To see all tasks and more detail, run gradlew tasks --all

To see more detail about a task, run gradlew help --task <task>

BUILD SUCCESSFUL in 978ms
```

<br>

# 마치며
이번 글은 정말 간단히 어떻게 Gradle을 설치 및 시작하는지에 대해서 다뤘다.

간단히 다뤘지만, 글을 작성하면서 이전엔 몰랐던 내용들을 많이 알 수 있었다.

특히 이전엔 자바 프로젝트에서 소스코드부분(`src`)만 수정하고 다른 부분은 쳐다도 보지않았다. (반성..)

직접 CLI환경에서 Gradle 프로젝트를 생성하고 빌드하는 과정을 통해 각각의 파일들이 무슨 역할하는지를 알게된 것 같다.

다음 글은 [Gradle을 알아가는 과정](https://docs.gradle.org/current/userguide/getting_started.html#authoring_gradle_builds)에 따라 Gradle 스크립트 작성에 대해서 다룬다.
