# 목차

<br>

- [목차](#목차)
- [개요](#개요)
- [1 JVM 개념](#1-jvm-개념)
- [2 JVM 구성요소](#2-jvm-구성요소)
  - [2-1 클래스 로더](#2-1-클래스-로더)
  - [2-2 메모리](#2-2-메모리)
  - [2-3 실행 엔진](#2-3-실행-엔진)
- [3 JIT 컴파일러](#3-jit-컴파일러)

<br>

# 개요
자바를 사용하면서 여러 프레임워크와 라이브러리들의 도움을 많이 받았다.

그리고 이러한 프레임워크와 라이브러리들의 동작 방식에 대해서 알아보다보니 자연스레 JVM의 대한 궁금증으로 이어졌다.

이번 글은 JVM의 개념과 구조에 대해서 정리하였다.

# 1 JVM 개념

<br>

🤔 JVM
* JVM (Java Virtual Machine)은 자바 바이트코드를 실행할 수 있게 해주는 주체다.
* 바이트코드로 컴파일 되는 대표적인 언어가 바로 자바다. 그 외에도 코틀린, 스칼라등이 있다.

<br>

💁‍♂️ JVM의 특징
* WORA (Write Once, Run Anywhere)
  * JVM은 플랫폼에 독립적이며, 모든 자바 가상 머신은 자바 가상 머신 규격에 정의된 대로 자바 바이트 코드를 실행한다. (스펙)
  * 모든 자바 프로그램은 CPU나 운영 체제의 종류와 무관하게 동일하게 동작하는 것을 보장한다.
  * 윈도우, 맥, 리눅스.. 등등 운영체제에 종속적이지 않다.
* GC
  * 클래스 인스턴스는 사용자 코드에 의해 명시적으로 생성되고 GC에 의해 자동적으로 소멸된다.

<br>

# 2 JVM 구성요소

<p align="center"><img src="./image/jvm_architecture.png"><br>출처: https://dzone.com/articles/jvm-architecture-explained</p>

<br>

## 2-1 클래스 로더
> **클래스 로더: 컴파일 된 `.class`에서 바이트 코드를 읽고 JVM(메모리)에 저장하는 역할.**
* 로딩: 클래스를 읽어오는 과정
* 링크: 레퍼런스를 연결하는 과정
* 초기화: static 값들 초기화 및 변수에 할당

<br>

## 2-2 메모리
* 메서드 영역: 메모리 영역에는 클래스 수준의 정보 (클래스 이름, 부모 클래스 이름, 메서드, 변수)를 저장한다.
  * 공유 자원. (스레드와 상관없이 공유)
* 힙 영역: 클래스 인스턴스(객체)를 저장한다.
  * 공유 자원. (스레드와 상관없이 공유)
* 스택 영역: 스레드마다 런타임 스택을 만들고, 그 안에 메서드 호출을 스택 프레임(메서드 콜)이라 부르는 블럭으로 쌓는다. 스레드 종료하면 런타임 스택도 사라진다.
* PC (Program Counter) 레지스터: 스레드마다 스레드 내 현재 실행할 instruction의 위치를 가리키는 포인터가 생성됨.
* 네이티브 메서드 스택: 메서드에 `Native` 키워드가 붙어있고, 자바가 아닌 C나 C++로 돌아가는 콜스택을 의미한다.
  * 네이티브 메서드 인터페이스(JNI)를 호출하여 메서드를 실행한다.
  * 네이티브 메서드 인터페이스의 구현체는 네이티브 메서드 라이브러리에 위치해있다.

<br>

## 2-3 실행 엔진
* 인터프리터: 동적으로 바이트 코드를 한 줄씩 컴파일하여 실행.
* JIT 컴파일러: 인터프리터 효율을 높이기 위해, 인터프리터가 반복하는 코드를 발견하면 JIT 컴파일러로 반복되는 코드를 모두 네이티브 코드로 바꿔둔다. 그 다음부터 인터프리터는 네이티브 코드로 컴파일된 코드를 바로 사용한다.
* GC: 더 이상 참조되지 않는 객체를 모아서 정리한다.
* JNI: 자바 애플리케이션에서 C, C++, 어셈블리로 작성된 함수를 사용할 수 있는 방법을 제공한다. (네이티브 메서드 인터페이스)
* 네티이브 메서드 라이브러리: C, C++로 작성된 라이브러리 (JNI의 구현체)

> 가장 중요한 부분은 GC이다. 나머지는 어떻게 동작하는지만 이해하면 된다.

<br>

# 3 JIT 컴파일러

💁‍♂️ **전통적인 컴퓨터 프로그램을 만드는 방법**
* 인터프리터 방식 - 실행 중 프로그래밍 언어를 읽어가면서 해당 기능에 대응하는 기계어 코드를 실행 (동적)
  * 코드를 한줄 씩 번역하여 실행한다.
* 컴파일 방식 - 실행하기 전에 프로그램 코드를 기계어로 번역하여 실행 (정적)
  * 코드를 한번에 번역하고 실행한다.
* **JIT 컴파일러는 이 두 가지 방식을 혼합한 방식이다.**

<br>

🤔 **JIT 컴파일러의 동작방식**
* 바이트코드를 네이티브 코드로 컴파일해주는 역할
* **인터프리터 효율을 높이기 위해, 인터프리터가 반복되는 코드를 발견하면 JIT 컴파일러로 반복되는 코드를 모두 네이티브 코드로 바꿔둔다.**
* **그 다음부터 인터프리터는 네이티브 코드로 컴파일된 코드를 바로 사용한다.**

<br>

💁‍♂️ **JIT 컴파일러의 장점**
* **실행 시점에서 인터프리터 방식으로 기계어 코드를 생성하면서 그 코드를 캐싱하여, 같은 함수가 여러 번 불릴 때 매번 기계어 코드를 생성하는 것을 방지한다. 그러므로 인터프리터보다 실행속도가 빠르다.**
