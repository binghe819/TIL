# 목차

<br>

- [목차](#목차)
- [MongoDB 설치](#mongodb-설치)
- [MongoDB 도커 컨테이너 사용](#mongodb-도커-컨테이너-사용)

<br>

# MongoDB 설치
MongoDB를 설치하여 사용할 일이 생겨서 간단히 Docker에 MongoDB 띄우는 것을 정리한다.

<br>

# MongoDB 도커 컨테이너 사용
이번 장에선 간단히 도커 컨테이너에 MongoDB를 띄워본다.

```shell
# Mongo 이미지 Pull
$ docker pull mongo

# MongoDB 컨테이너 생성
$ docker run --name mongodb -v ~/data:/data/db -d -p 27017:27017 mongo

# MongoDB 컨테이너 접속
$ docker exec -it mongodb mongosh
```

Mongo 컨테이너에 접속후 아래와 같이 명렁어를 입력하여 결과가 출력된다면 성공적으로 Docker에 Mongo 컨테이너를 띄운 것이다.

```shell
test> db.runCommand( {hello: 1} )

{
  isWritablePrimary: true,
  topologyVersion: {
    processId: ObjectId("647efd5fcd08c199e090d7b0"),
    counter: Long("0")
  },
  maxBsonObjectSize: 16777216,
  maxMessageSizeBytes: 48000000,
  maxWriteBatchSize: 100000,
  localTime: ISODate("2023-06-06T10:07:21.106Z"),
  logicalSessionTimeoutMinutes: 30,
  connectionId: 5,
  minWireVersion: 0,
  maxWireVersion: 17,
  readOnly: false,
  ok: 1
}
```


