# 목차

- [SQL 정리](#sql-정리)
  * [조회](#조회)
    + [SELECT](#select)
    + [DISTINCT](#distinct)
    + [WHERE](#where)
      - [LIKE](#like)
      - [IN](#in)
      - [Between](#between)
    + [ORDER BY](#order-by)
    + [GROUP BY](#group-by)
      - [그룹 함수](#그룹-함수)
      - [HAVING](#having)
    + [JOIN](#join)
      - [LEFT OUTER JOIN](#left-outer-join)
      - [INNER JOIN](#inner-join)
      - [FULL OUTER JOIN](#full-outer-join)
  * [삽입](#삽입)
  * [갱신](#갱신)
  * [삭제](#삭제)
  * [테이블](#테이블)
    + [CREATE TABLE](#create-table)
    + [ALTER TABLE](#alter-table)
    + [DROP TABLE](#drop-table)
- [참조](#참조)



# SQL 정리

🤔  SQL

* SQL (Structrued Query Language, 구조화 질의어)는 RDBMS의 데이터를 관리하기 위해 설계된 특수 목적의 프로그래밍 언어이다.



## 조회

> 조회 명령어

### SELECT

```sql
// User테이블에서 UserID, Name 열 데이터 조회
SELECT UserID, Name FROM USERS;
```

* 조회



### DISTINCT

```sql
// 이름의 중복을 제거하고 모든 이름 조회
SELECT DISTINCT NAME FROM USERS;

// 이름과 이메일에서 동시에 중복을 제거할 수 있는 행을 제거하고 조회
SELECT DISTINCT NAME, EMAIL FROM USERS;
```

* 중복 제거

  <img src="./image/065_2.jpg" width="400" />

  <center> 출처 : 모두의 SQL</center>

  * **SELECT 해온 전체 레코드 단위(모든 열)에 대해 중복값을 제거하고 출력**

* 사전 의미 : 뚜렷한, 분명한, 별개의





### WHERE

```sql
// 이름이 홍길동인 유저 조회
SELECT * FROM USERS WHERE Name = '홍길동'

// 나이가 20살인 유저의 이름과 이메일 조회
SELECT Name, Email FROM USERS WHERE Age = 20

// 나이가 20살이 아닌 유저의 이름과 이메일 조회
SELECT Name, Email FROM USERS WHERE Age <> 20

// USERID가 2이며 유저의 이름이 홍길서 이거나 나이가 23인 유저 조회
SELECT * FROM USERS WHERE UserID = 2 AND (NAME = '홍길서' OR Age = 23)

// USERID가 2이거나 유저의 이름이 홍길동이면서 나이가 20인 유저 조회
SELECT * FROM USER WHERE USERID = 2 OR ( NAME = '홍길동' AND AGE = 20)
```

* 조건문

| 명령어  | 설명                                             |
| ------- | ------------------------------------------------ |
| `=`     | Equal                                            |
| `>`     | Greater than                                     |
| `<`     | Less than                                        |
| `>=`    | Greater than or equal                            |
| `<=`    | Less than or Equal                               |
| `<>`    | Not Equal (`!=`)                                 |
| BETWEEN | Between a certain range                          |
| LIKE    | Search for a pattern                             |
| IN      | To specify multiple possible values for a column |



#### LIKE

```sql
// '홍'으로 시작하는 유저 조회
SELECT *  FROM USER WHERE NAME LIKE '홍%'

// '홍'이 들어가는 유저 조회
SELECT *  FROM USER WHERE NAME LIKE '%홍%'

// '동'으로 끝나는 유저 조회
SELECT *  FROM USER WHERE NAME LIKE '%동'
```

* 시작, 포함, 끝나는 단어 조회



#### IN

```sql
// 나이가 20, 23인 유저 조회
SELECT * FROM USER WHERE AGE IN(20,23)
```

* ~이거나



#### Between

```sql
// 나이가 20 ~ 23인 유저 조회
SELECT * FROM USER WHERE AGE BETWEEN 20 AND 23
```

* ~부터 ~까지



### ORDER BY

```sql
// 나이를 기준으로 오름차순
SELECT * FROM USER ORDER BY AGE

// 나이를 기준으로 내림차순
SELECT * FROM USER ORDER BY AGE DESC;

// 이름을 기준으로 A~Z순으로 정렬
SELECT * FROM USER ORDER BY NAME

// 이름을 기준으로 Z~A순으로 정렬
SELECT * FROM USER ORDER BY NAME DESC

// 이름을 기준으로 A~Z순으로 정렬하고, 이름이 같은 경우 나이를 기준으로 내림차순
SELECT * FROM USER ORDER BY NAME ASC, AGE DESC
```





### GROUP BY

```sql
SELECT 컬럼 FROM 테이블 GROUP BY 그룹화할 컬럼

// 이름을 기준으로 그룹화하여 조회
SELECT NAME FROM USERS GROUP BY NAME

// ORDER BY와 함께 할 경우
SELECT 컬럼 FROM 테이블 [WHERE 조건] GROUP BY 그룹화할 컬럼 [HAVING 조건] ORDER BY 컬럼1 [, 컬럼2, ..]
```

* **지정된 특정 컬럼의 같은 값을 가진 행을 그룹화 한다.**
  
  * 컬럼을 그룹짓고(중복을 제거하고) 이에 대해 건수나 값의 합을 계산할 때 사용됨.
  * `GROUP BY`로 지정한 열을 '집약 키'나 '그룹화 키'라고 부른다.
  
* `SELECT` 절에 그룹 함수와 컬럼명이 같이 기술된 경우 해당 컬럼은 반드시 `GROUP BY` 절에 그룹화 돼야 한다.

* `GROUP BY`의 원리

  ![img](./image/129.jpg)

  <center>출처 : 모두의 sql</center>

  * 내부적으로 임시 테이블을 만든다.
  * **임시 테이블의 컬럼은 `GROUP BY`에 사용된 컬럼과 `SELECT`에서 사용된 집합함수가 된다.**



#### 그룹 함수

> `GROUP BY`는 보통 그룹함수와 많이 사용된다.

| 함수명   | 기능                     | 예시           |
| -------- | ------------------------ | -------------- |
| COUNT    | 행의 수를 계산           | COUNT(컬럼)    |
| MAX      | 값들 중에 최대 값을 반환 | MAX(컬럼)      |
| MIN      | 값들 중에 최소 값을 반환 | MIN(컬럼)      |
| AVG      | 평균 값을 계산           | AVG(컬럼)      |
| SUM      | 총 합계를 계산           | SUM(컬럼)      |
| VARIANCE | 분산을 계산              | VARIANCE(컬럼) |
| STDDEV   | 표준편차를 계산          | STDDEV(컬럼)   |

* 반드시 하나의 값만을 반환한다.
* `GROUP BY` 설정 없이 일반 컬럼과 기술될 수 없다.

```sql
// 각 이름의 개수 조회
SELECT COUNT(NAME), NAME FROM USERS GROUP BY NAME
```



#### HAVING

```sql
// 이름을 기준으로 그룹화하고 나이가 25인 유저 조회
SELECT NAME, AGE FROM USER GROUP BY NAME HAVING AGE = 25
```

* 특정 컬럼을 그룹화한 결과에 조건



> `WHERE`과 `HAVING`의 차이
>
> * **`WHERE`은 그룹화 하기 전, `HAVING`은 그룹화 후에 조건.**
>
>   * 조건 처리 후에 컬럼 그룹화
>
>   * ```sql
>     SELECT 컬럼 FROM 테이블 WHERE 조건식 GROUP BY 그룹화할 컬럼
>     ```
>
>   * 컬럼 그룹화 후에 조건 처리
>
>   * ```sql
>     SELECT 컬럼 FROM 테이블 GROUP BY 그룹화할 컬럼 HAVING 조건식.
>     ```





### JOIN

```sql
// 내부 결합
SELECT 선택하고 싶은 열 리스트 FROM 첫 번째 테이블명 INNER JOIN 두 번째 테이블명 ON 결합 조건

// 외부 결합
SELECT 선택하고 싶은 열 리스트 FROM 첫 번째 테이블 명 LEFT OUTER JOIN 두 번째 테이블명 ON 결합 조건
```

* 결합을 사용하는 이유
  * 매 테이블은 한 가지의 테마만을 가짐으로써 테이블을 분리시킨다.
  * 분리시킨 테이블은 기본키와 외래키를 사용하여 결합함으로써 분리시킨 테이블을 결합시켜 조회할 수 있다.

* 내부 결합

  * `ON`이 지정한 결합 조건에 일치하는 행만을 2개의 테이블로부터 가져올 수 있다.

* 외부 결합

  * 한 쪽 테이블을 기준으로 전체 행을 표시하고 다른 테이블은 값이 있으면 표시하고 없을 경우 NULL로 표시

    

:point_right: 예시

![img](./image/D093EADF-6256-4987-8E0E-671E69A98BF0.png)

#### LEFT OUTER JOIN

<img src="./image/image-20200914163512018.png" width="450" />

<center> 출처 : https://sql-joins.leopard.in.ua/ </center>

```sql
SELECT * FROM article LEFT JOIN USER ON article.user = USER.uid
```

![image-20200914162025023](./image/image-20200914162025023.png)

* `USER`에 만약 `article` 에서 `user` 와 동일한 키가 없다면 그냥 `NULL` 을 출력한다.
  * 왼쪽 테이블 (`article`)에는 값이 있는데 오른쪽 테이블 (`USER`)에는 대응되는 값이 없다.

![image-20200914163119572](./image/image-20200914163119572.png)



#### INNER JOIN

<img src="./image/image-20200914163742547.png" width="450" />

<center> 출처 : https://sql-joins.leopard.in.ua/ </center>

![image-20200914164248021](./image/image-20200914164248021.png)

* `INNER JOIN` 은 두 테이블에 동일하게 존재하는 것만 출력된다. (교집합)



#### FULL OUTER JOIN

<img src="./image/image-20200914165249361.png" width="450" />

<center> 출처 : https://sql-joins.leopard.in.ua/ </center>

![image-20200914165130774](./image/image-20200914165130774.png)

* `FULL JOIN` 은 왼쪽과 오른쪽에 있는 행 모두를 합성해서 하나의 표를 만든다. (합집합)
  * `FULL JOIN` =  `LEFT JOIN` UNION `RIGHT JOIN`

> 전혀 사용되지는 않지만.. JOIN의 원리를 이해하는데 도움이 되는 것 같다.



## 삽입

```sql
INSERT INTO 테이블명(열1[, 열2, 열3, ...]) VALUES (값1[, 값2, 값3 ...])

// 데이터 삽입
INSERT INTO USERS( USERID , EMAIL , NAME , AGE ) VALUES (3, 'test1@naver.com', '홍길서', 25);
INSERT INTO USERS VALUES (4, 'test2@naver.com', '홍길동', 24)
```





## 갱신

```sql
UPDATE 테이블명 SET 열명 = 값 WHERE 조건

// UserID가 2인 유저의 이름을 김길동으로 갱신 (열 갱신)
UPDATE USERS SET Name = '김길동' WHERE UserID = 2

// UserID가 2인 유저의 이름을 김길동으로 나이를 24로 갱신 (복수 열 갱신)
UPDATE USERS SET Name = '김길동', AGE = 24 WHERE UserID = 2
```





## 삭제

```sql
DELETE FROM 테이블명 WHERE 조건

// UserID가 2인 유저 삭제
DELETE FROM USERS WHERE UserID = 2
```

* 행 단위로 삭제한다.







## 테이블



### CREATE TABLE

```sql
CREATE TABLE 테이블명(
  컬럼명 타입(크기) NOT NULL auto_increment, // NULL값이 들어갈 수 없음
  컬럼명 타입 NULL DEFAULT(값), // NULL값 허용, 초기값 설정
	PRIMARY KEY (컬럼명) // PK 설정
  FOREIGN KEY (컬럼명) REFERENCES 테이블(컬럼명) // 외래키 설정
)

// 예시
CREATE TABLE User (
UserID INTEGER,
Email VARCHAR(255),
Name VARCHAR(255),
PRIMARY KEY (UserID)
);
```

* `PRIMARY KEY` : 기본키 정의
* `UNIQUE KEY` : 고유키 정의
* `NOT NULL` : `NULL` 값의 입력을 금지
* `CHECK` : 입력할 수 있는 값의 범위를 제한. `boolean`로 평가할 수 있는 논리식을 설정
* `FOREIGN KEY` : 외래키



### ALTER TABLE

```sql
// 칼럼 추가
ALTER TABLE 테이블명 ADD 컬럼명 컬럼 속성

// 컬럼 수정
ALTER TABLE 테이블명 ALTER 컬럼명 컬럼 속성

// 컬럼 삭제
ALTER TABLE 테이블명 DROP COLUMN 칼럼명
```





### DROP TABLE

```sql
DROP TABLE 테이블명
```









# 참조

* https://coding-factory.tistory.com/80?category=772898

* https://keep-cool.tistory.com/37

* [모두의 sql](https://thebook.io/006977/)

* [생활코딩 - SQL JOIN](https://opentutorials.org/course/3884)







