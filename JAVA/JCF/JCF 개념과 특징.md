# 목차

<br>

- [목차](#목차)
- [JCF 도입 배경](#jcf-도입-배경)
- [JCF 개념](#jcf-개념)
- [JCF 특징](#jcf-특징)
- [참고](#참고)

<br>

# JCF 도입 배경

<br>

🤔 본격적으로 JCF의 개념과 특징에 대해서 알아보기 이전에 **JCF가 왜 도입되었는지 알아본다.**

<br>

JDK 1.2 이전에는 자바에서 자료구조로 `Arrays`(배열), `Vectors`, `Hashtables`를 제공했다.

위 자료구조들은 **구조체 방식에 가깝게 구현되어 있으며, 공통 인터페이스가 없었기 때문에 데이터 작업을 수행하는 것이 쉽지 않았다.**

이를테면, **정렬과 같은 처리를 위해선 자료구조로부터 get하여 외부에서 처리해줘야했으며, 각 자료구조마다 사용하는 메서드, 문법, 생성자가 모두 달랐기에 혼동하기 쉬웠다.**

개발자들은 **모든 데이터 구조에서 일관되게 작동하고 작업을 효율적으로 수행하는 공통 인터페이스가 필요했다고 한다.**

이에 JDK 1.2에 **Java는 다양한 데이터 구조에서 수행할 수 있는 다양한 작업으로 구성된 클래스, 인터페이스 그룹인 Collections Framework를 제시했다.** 

<br>

<br>

💁‍♂️ **필자가 생각하는 JCF의 도입 배경은 다음과 같다.**

> 필자가 [공식 문서](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)를 보고 재해석한 것이며, JCF가 추구하는 컨셉과 다를 수 있다.

<br>

1. 조금 더 자율적으로
   * [JDK 1.0 Vector](https://github.com/barismeral/Java-JDK-1.0-src/blob/master/src/java/util/Vector.java)를 보면 데이터에 대한 검색, 정렬, 특정 처리를 제공하지 않는다. 그저 데이터에 대한 CRUD만을 지원한다. 즉, 데이터에 대한 처리를 위해선 `get`를 통해 가져와서 외부에서 처리해줘야한다. (절차지향적)
   * JCF가 소개된 이후엔 메시지를 통해 데이터에 대한 처리를 할 수 있게 되었다. (`sort`, `indexOf`...등등)
   * 필자가 생각하기에 이는 자료구조에 대한 객체를 조금 더 자율적으로 만들어 객체지향의 이점을 잘 활용했다고 생각한다.
2. 조합을 통한 모듈화
   * JCF는 여러 인터페이스를 제공한다. (`Iterable`, `Comprator`, `Collection` 등등)
   * 이는 개발자가 인터페이스의 대한 구현체(ex. 정렬 알고리즘)를 몰라도 손쉽게 조합하여 사용할 수 있게 해주었다.
3. 사용자 편의성을 위해 미리 구현한 클래스
   * JCF는 `List`, `Set`, `Map`을 기반으로 다양한 자료구조에 대한 구현체를 제공한다.

**결론적으론 기존의 구조체 객체처럼 사용되던 것을 객체지향의 특성을 많이 살린것이 아닐까 싶다.**

> [JCF의 장점](#jcf-특징)을 보면 더 와닿는 부분이 많을 듯 하다.

<br>

# JCF 개념

<br>

🤔 Collection과 Framework의 의미

JCF는 Java Collections Framework의 약자이다. 여기서 의문은 `Collection과 Framework란 무엇인가?`이다.

* Collection
  * Collection은 객체들의 그룹을 정의하는 것을 의미한다. (a group of objects)
  * Collection은 일반적으로 둘 이상의 객체로 구성된 단일 단위이다.
* Framework
  * Framework의 목적은 코드의 재사용성과 편의성이다.
  * 필자의 경우 JCF는 라이브러리에 가깝지않나 싶었지만, 기본적인 틀을 제공해준다는 의미에서 프레임워크라고 부르는 듯 하다. (JCF는 제어가 역전된 것인가? 사실 잘 모르겠다..)
* Collections Framework
  * Collections Framework는 컬렉션을 표현하고 처리하기 위한 통합 아키텍처로, 구현체의 세부 사항과 독립적으로 처리될 수 있다.
    * 컴파일타임땐 추상적인 것, 런타임땐 구체적인 것에 의존하므로, 구현체 세부 사항에 따라 다르게 동작한다.

<br>

🤔 JCF란?
* JCF (Java Collections Framework)는 컬렉션을 표현하고 처리하기위한 통합 아키텍처이다.
  * 컬렉션에 저장된 데이터를 처리하기 위한 인터페이스와 그 구현 클래스 및 알고리즘등이 포함되어 있다.
  * `Collection` 인터페이스는 `List`, `Set` 및 `Queue`와 같은 다른 인터페이스에 의해 확장된다.
* 하나 이상의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 클래스의 집합을 의미한다.

<br>

# JCF 특징
JCF의 장점은 다음과 같다. 

JCF의 장점을 통해 JCF가 무엇을 추구하려고 하는지 알 수 있다.

* **Reduces programming effort** by providing data structures and algorithms so you don't have to write them yourself.
* **Increases performance** by providing high-performance implementations of data structures and algorithms. Because the various implementations of each interface are interchangeable, programs can be tuned by switching implementations.
* **Provides interoperability between unrelated APIs** by establishing a common language to pass collections back and forth.
* **Reduces the effort required to learn APIs** by requiring you to learn multiple ad hoc collection APIs.
* **Reduces the effort required to design and implement APIs** by not requiring you to produce ad hoc collections APIs.
* **Fosters software reuse** by providing a standard interface for collections and algorithms with which to manipulate them.

<br>

# 참고
* [Oracle Java Document - Collections Framework](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)

