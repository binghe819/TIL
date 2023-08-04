
Gradle이 실행되면 어떤 걸 읽고 어떤 순서로 실행되는지 정리.

<br>

Gradle이 실행되면 build.gradle 스크립트 파일을 위에서 아래로 실햄함.

Groovy로된 간단한 코드 보여주기 (println 등)

다음 글 참고 https://medium.com/@andrewMacmurray/a-beginners-guide-to-gradle-26212ddcafa8 

그리고 https://velog.io/@hyemin916/Gradle%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-2-Gradle-%EB%B9%8C%EB%93%9C-%EC%A7%9C%EB%B3%B4%EA%B8%B0 에서 Task안에 초기화 단계, 구성 단계, 실행 단계때 실행할 코드를 어디에 어떻게 작성하면 되는지 정리.

<br>

Project와 인스턴스에 대해서... 그리고 Project와 Task에 대한 관계... Project 1 : N Task 형태로 Project가 Task를 가지고있는 형태.

아래 글 참고해서 작성.
* https://medium.com/@andrewMacmurray/a-beginners-guide-to-gradle-26212ddcafa8
* https://kotlinworld.com/321
* https://blog.gradle.org/how-gradle-works-2

<br>

Gradle Build하면 발생하는 단계들 정리.

* Initialization Phase
* Configuration Phase
* Execution Phase

아래 글 참고해서 작성.
* https://blog.gradle.org/how-gradle-works-2
* https://proandroiddev.com/understanding-gradle-the-build-lifecycle-5118c1da613f

<br>




