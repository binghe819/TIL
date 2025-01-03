<p align="center"><img src="http://image.yes24.com/momo/TopCate239/MidCate002/23813585.jpg" width="200"><br> </p>

[빅데이터 세상으로 떠나는 간결한 안내서. NoSQL](http://www.yes24.com/Product/Goods/8510944)을 읽고 정리한 글입니다.

<br>

# 목차
* [1장. 왜 NoSQL인가?](./ch01_%EC%99%9C_nosql%EC%9D%B8%EA%B0%80.md)
* [2장. 집합적 데이터 모델](./ch02_%EC%A7%91%ED%95%A9%EC%A0%81_%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%AA%A8%EB%8D%B8.md)
* [3장. 데이터 모델 상세](./ch03_%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%AA%A8%EB%8D%B8_%EC%83%81%EC%84%B8.md)
* [4장. 분산 모델](./ch04_%EB%B6%84%EC%82%B0_%EB%AA%A8%EB%8D%B8.md)
* [5장. 일관성](./ch05_일관성.md)

<br>

# NoSQL 설계시 주의할 점
* NoSQL은 클러스터 (분산)에 맞게 설계되어야한다.
  * 특정 비즈니스에 사용되는 필요한 데이터는 하나의 노드에 존재하는 것이 좋다.
    * 여러 RDB는 샤드를 적용할 경우, 쿼리나 참조 정합성, 트랜잭션, 일관성 제어등이 굉장히 복잡해진다. 이 경우 분산에 적합하지않다.
    * NoSQL은 분산에 최적화되게 설계되어야하며, 필요한 데이터를 집합이라는 단위로 저장해야한다. 그리고 가능한 하나의 노드안 집합에서 필요한 데이터를 가져오도록 설계하는 것이 좋다.
* 트랜잭션 관련
  * RDB는 여러 테이블의 데이터 일관성을 위해 ACID 트랜잭션을 이용한다. NoSQL은 대부분 이를 지원하지 않는다.
  * 대신 NoSQL은 집합 단위로 데이터의 일관성을 지원한다. 그러므로 데이터 일관성이 필요하다면 집합 단위를 잘 설계하는 것이 좋다.
  * NoSQL은 보통 원자적 연산으로 집합을 업데이트하고 데이터 저장소와도 집합 단위로 통신한다.
* 집합 모델에서 중요한 것은 애플리케이션에서 그 데이터에 어떻게 접근할 지에 대해 생각해야한다는 것이다. (조회 요구사항)
  * NoSQL의 집합 경계나 설계는 정규식과 다르게 정답이 없다. 그러므로 애플리케이션에서 데이터에 접근하는 요구사항을 잘 분석해서 설계하는 것이 중요하다.
* 집합의 특징
  * 집합은 검색할 수 있도록 key로 색인되어있다.
  * 집합은 클러스터에서 실행되는 데 가장 중요한 개념으로, 한 집합으로 구성된 데이터는 모두 한 노드에 저장됨을 데이터베이스가 보장한다.
  * 집합은 업데이트에 대한 원자적 단위로 동작해, 제한적이지만 유용한 정도의 트랜잭션 제어를 제공한다.
* NoSQL은 무스키마이다.
  * 무스키마의 장점 - 스키마에 구속되지 않으며 필요한 것을 쉽게 저장할 수 있다. (균일하지 않은 데이터를 처리하는 것이 쉽다.)
  * 무스키마의 단점 - 데이터에 접근하는 프로그램이 항상 암묵적 스키마에 의존해야한다. 스키마가 데이터베이스에서 애플리케이션 코드로 이동한다.
* NoSQL은 정규화와 다르게 중복적인 데이터 저장을 허용한다. 이 점을 꼭 기억하자.


