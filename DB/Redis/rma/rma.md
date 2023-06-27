# 목차

<br>

- [목차](#목차)
- [Redis Memory Analyzer (RMA)](#redis-memory-analyzer-rma)
- [RMA란?](#rma란)
- [RMA 설치](#rma-설치)
- [동작 과정](#동작-과정)
- [RMA에서 제공하는 통계](#rma에서-제공하는-통계)
  - [global](#global)
  - [scanner](#scanner)
  - [ram](#ram)
- [통계 내용은 완전 정확하진 않다.](#통계-내용은-완전-정확하진-않다)

<br>

# Redis Memory Analyzer (RMA)
오픈소스 계열인 Redis는 따로 모니터링 환경을 제공하지않는다. 

물론 `monitor` 명령어를 통해 실시간으로 요청을 모니터링할 수 있지만, 그외에 key 기반의 메모리 사용량 확인등의 모니터링 기능은 제공해주지않는다.

당연하기도한데 오픈소스 계열의 소프트웨어는 보통 기능적인 측면에 국한되며, 별도로 다양한 분석 기능이나 모니터링 기능을 서드파티를 통해 사용된다.

이번 글은 Redis의 key별 메모리 사용량을 분석할 때 좋은 [Redis Memory Analyzer](https://github.com/gamenet/redis-memory-analyzer) (이하 RMA)에 대해서 소개한다.

<br>

# RMA란?

RMA는 Redis의 메모리 프로파일러이며, RAM 병목 현상을 찾기 위해 실시간으로 key 공간을 스캔하고 패턴별로 RAM 사용 통계를 집계해주는 콘솔 도구이다.

<br>

"string", "hash", "list", "set", "zset"과 같은 모든 Redis 유형을 스캔하고 원하는 대로 일치하는 패턴을 통계낼 수도 있다.

<br>

RMA가 좋은 점은 패턴으로 key를 식별한다는 것이다. 

예를 들어, `user:100`, `user:101`와 같은 키가 존재할 경우 RMA는 일반적인 패턴인 `user:*`로 메모리 사용량을 분석해준다.

<br>

# RMA 설치

<br>

💁‍♂️ **사전 조건**

* python >= 3.5 and pip.
* [redis-py](https://github.com/redis/redis-py)

<br>

💁‍♂️ **RMA 설치**

```shell
> pip install rma
```

Github 저장소의 특정 버전을 설치하고싶은경우

```shell
> pip install git+https://github.com/gamenet/redis-memory-analyzer@v0.2.0
```

<br>

💁‍♂️ **RMA 구동 확인**

```shell
> rma --help

usage: rma [-h] [-s HOST] [-p PORT] [-a PASSWORD] [-d DB] [-m MATCH] [-l LIMIT]
           [-b BEHAVIOUR] [-t TYPES]

RMA is used to scan Redis key space in and aggregate memory usage statistic by
key patterns.

optional arguments:
  -h, --help                 show this help message and exit
  -s, --server HOST          Redis Server hostname. Defaults to 127.0.0.1
  -p, --port PORT            Redis Server port. Defaults to 6379
  -a, --password PASSWORD    Password to use when connecting to the server
  -d, --db DB                Database number, defaults to 0
  -m, --match MATCH          Keys pattern to match
  -l, --limit LIMIT          Get max key matched by pattern
  -b, --behaviour BEHAVIOUR  Specify application working mode. Allowed values
                             are all, scanner, ram, global
  -t, --type TYPES           Data types to include. Possible values are string,
                             hash, list, set. Multiple types can be provided. If
                             not specified, all data types will be returned.
                             Allowed values arestring, hash, list, set, zset
  -f --format TYPE           Output type format: json or text (by default)
  -x --separator SEPARATOR   Specify namespace separator. Default is ':'
```

help 명령의 응답으로 RAM가 어떤 기능을 지원하는지 간단히 살펴볼 수 있다.

<br>

# 동작 과정

RMA는 Type별로 여러가지 통계를 제공해준다. 모든 작업은 몇 단계를 거쳐 수행된다.

1. `SCAN` 명령을 통해 Redis Key를 순회하여 통계를 보여준다. 배치와 같이 LUA 스크립팅을 이용하여 주어진 패턴과 일치하는 각 키에 대한 통계를 처리한다.
2. Type별로 키를 분리하고 패턴을 일치시킨다.
3. 계속해서 조회된 주어진 데이터 세트를 대상으로 일정 동작 및 규칙을 실행한다. (limit, match 등등)
4. 주어진 보고된 출력 결과

<br>

# RMA에서 제공하는 통계
RMA는 세 가지 수준의 세부 정보를 지원한다.

<br>

## global
global은 Redis 서버의 전반적인 통계를 제공해준다.

> 통계 예시
```shell
> rma -p 6379 -b global

Match *: 100%|██████████████████████████████████████████████████████████████████████████████████████████████▉| 139450/139451 [00:01<00:00, 115322.26it/s]

Aggregating keys by pattern and type

Apply rules

Nodes

| Stat                               | Value          |
|:-----------------------------------|:---------------|
| info used_memory                   | 238576688      |
| info used_memory_human             | 227.52M        |
| info used_memory_rss               | 249810944      |
| info used_memory_rss_human         | 238.24M        |
| info used_memory_peak              | 238803320      |
| info used_memory_peak_human        | 227.74M        |
| info used_memory_peak_perc         | 99.91%         |
| info used_memory_overhead          | 8772496        |
| info used_memory_startup           | 894976         |
| info used_memory_dataset           | 229804192      |
| info used_memory_dataset_perc      | 96.69%         |
| info allocator_allocated           | 238919504      |
| info allocator_active              | 245628928      |
| info allocator_resident            | 249430016      |
| info total_system_memory           | 8231829504     |
| info total_system_memory_human     | 7.67G          |
| info used_memory_lua               | 1294336        |
| info used_memory_vm_eval           | 1294336        |
| info used_memory_lua_human         | 1.23M          |
| info used_memory_scripts_eval      | 504            |
| info number_of_cached_scripts      | 1              |
| info number_of_functions           | 0              |
| info number_of_libraries           | 0              |
| info used_memory_vm_functions      | 32768          |
| info used_memory_vm_total          | 1327104        |
| info used_memory_vm_total_human    | 1.27M          |
| info used_memory_functions         | 184            |
| info used_memory_scripts           | 688            |
| info used_memory_scripts_human     | 688B           |
| info maxmemory                     | 0              |
| info maxmemory_human               | 0B             |
| info maxmemory_policy              | noeviction     |
| info allocator_frag_ratio          | 1.03           |
| info allocator_frag_bytes          | 6709424        |
| info allocator_rss_ratio           | 1.02           |
| info allocator_rss_bytes           | 3801088        |
| info rss_overhead_ratio            | 1.0            |
| info rss_overhead_bytes            | 380928         |
| info mem_fragmentation_ratio       | 1.05           |
| info mem_fragmentation_bytes       | 11259144       |
| info mem_not_counted_for_evict     | 0              |
| info mem_replication_backlog       | 0              |
| info mem_total_replication_buffers | 0              |
| info mem_clients_slaves            | 0              |
| info mem_clients_normal            | 89832          |
| info mem_cluster_links             | 0              |
| info mem_aof_buffer                | 0              |
| info mem_allocator                 | jemalloc-5.2.1 |
| info active_defrag_running         | 0              |
| info lazyfree_pending_objects      | 0              |
| info lazyfreed_objects             | 0              |
| used hash-max-listpack-value       | 64             |
| used zset-max-listpack-value       | 64             |
| used proto-max-bulk-len            | 536870912      |
| used hash-max-listpack-entries     | 512            |
| used list-max-ziplist-size         | -2             |
| used set-max-intset-entries        | 512            |
| used list-max-listpack-size        | -2             |
| used active-defrag-max-scan-fields | 1000           |
| used hash-max-ziplist-value        | 64             |
| used zset-max-listpack-entries     | 128            |
| used hash-max-ziplist-entries      | 512            |
| used zset-max-ziplist-value        | 64             |
| used zset-max-ziplist-entries      | 128            |
| totalKeys                          | 139450         |
| redisKeySpaceOverhead              | 8388608        |
```

눈에 띄는 부분은 Redis Key에 대한 통계를 내준다는 것이다.

Key 데이터를 저장하기위해 Redis가 사용하는 메모리양과 이에 대한 오버헤드를 표시해준다.

그외에도 key count, 시스템 메모리, allocation 크기 등과 같은 통계를 제공한다.

<br>

## scanner
scanner는 Redis에 저장된 키 종류별로 통계를 내준다. (keyspace의 개요)

Redis에 어떠한 종류의 key가 저장되어 있는지 한눈에 볼 수 있어서 굉장히 유용하다.

> 실제 필자는 scanner를 가장 많이 사용한다.

```shell
> rma -p 6379 -b scanner

Match *: 100%|███████████████████████████████████████████████████████████████████████████████████████████████| 139450/139450 [00:01<00:00, 114962.42it/s]

Aggregating keys by pattern and type

Apply rules

Keys by types

| name                                                  |   count | type   | percent   |
|:------------------------------------------------------|--------:|:-------|:----------|
| Displayxxxxxxx:xxxxId:*:*:*                           |  132583 | string | 95.07%    |
| Displayxxxxxxx:Sample:xxxxxxxId:*                     |    3583 | string | 2.56%     |
| Displayxxxxxxxxxxxxxxxxxxxx:xxxxxxxId:*               |    1137 | string | 0.81%     |
| Displayxxxxxxxxxxxx:xxxxxxxId:*:labels                |    1055 | string | 0.75%     |
| Displayxxxxxxxxxxxxxxxxx:xxxxxxxId:*:*:*              |    1054 | string | 0.75%     |
| Displayxxxxxxxxxxxxxx:xxxxId:*                        |      16 | set    | 0.01%     |
| Displayxxxxxxxxxxxxxx:xxxxId:*:*:*                    |      15 | set    | 0.01%     |
| cache2:*:*                                            |       4 | string | 0.00%     |
| cache2:xxxx:*                                         |       1 | string | 0.00%     |
| EntityEventMessageStore                               |       1 | set    | 0.00%     |
```

<br>

## ram
Redis는 key와 모든 컬렉션 유형의 value를 dict 기반으로 RedisDB 내부 데이터 구조에 저장한다. (자세한 내용은 [RedisPlanet](https://collabnix.github.io/redisplanet/) 참고)

그리고 ram 통계는 keyspace 수준의 key와 value에 대한 메모리 사용량 통계를 해준다. 사용된 메모리, 실제 데이터 크기, 오버 헤드, 인코딩, TTL 등의 정보를 얻을 수 있다.

```shell
rma -p 6379 -b ram
Match *: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 139446/139446 [00:01<00:00, 116364.03it/s]

Aggregating keys by pattern and type

Apply rules

Keys statistic

Stat by <string>

| Match                                                 |   Count |    Useful |   Free |      Real |   Ratio | Encoding                     |   Min |   Max |     Avg |   TTL Min |   TTL Max |   TTL Avg |
|:------------------------------------------------------|--------:|----------:|-------:|----------:|--------:|:-----------------------------|------:|------:|--------:|----------:|----------:|----------:|
| Displayxxxxxxx:xxxxId:*:*:*                           |  132583 | 219294432 |      0 | 253235680 |    1.15 | raw [100.0%]                 |   800 |  3104 | 1654.02 |        -1 |      2955 |     -0.73 |
| Displayxxxxxxx:Sample:xxxxxxxId:*                     |    3583 |   5009120 |      0 |   5926368 |    1.18 | raw [100.0%]                 |   800 |  2848 | 1398.02 |        -1 |        -1 |     -1.00 |
| Displayxxxxxxxxxxxxxxxxxxxx:xxxxxxxId:*               |    1137 |    163728 |      0 |    254688 |    1.56 | raw [100.0%]                 |   144 |   144 |  144.00 |      2937 |      3012 |   2968.18 |
| Displayxxxxxxxxxxxx:xxxxxxxId:*:labels                |    1055 |    357856 |      0 |    426720 |    1.19 | raw [100.0%]                 |   288 |   800 |  339.20 |      2937 |      3013 |   2969.63 |
| Displayxxxxxxxxxxxxxxxxx:xxxxxxxId:*:*:*              |    1054 |    452480 |      0 |    546432 |    1.21 | raw [100.0%]                 |   288 |  2592 |  429.30 |      2925 |      3092 |   2998.23 |
| Displayxxxxxxxxxxxxxx:xxxxId:*:*:*:xxxxxxxxxxxxxxId:* |      12 |     10880 |      0 |     13952 |    1.28 | raw [100.0%]                 |   800 |  1312 |  906.67 |      2939 |      2955 |   2951.42 |
| Total:                                                |  139428 | 225289264 |      0 | 260404832 |    0.00 |                              |     0 |     0 |    0.00 |        -1 |     85723 |    nan    |

Stat by <set>

| Match                              |   Count |   Avg Count |   Value mem |   Real |   Ratio |   System* | Encoding           |   Total |   TTL Min |   TTL Max |   TTL Avg. |
|:-----------------------------------|--------:|------------:|------------:|-------:|--------:|----------:|:-------------------|--------:|----------:|----------:|-----------:|
| Displayxxxxxxxxxxxxxx:xxxxId:*     |      16 |        2.25 |        2469 |   4032 |    1.63 |      2368 | hashtable [100.0%] |    6400 |      2938 |      2955 |    2951.81 |
| Displayxxxxxxxxxxxxxx:xxxxId:*:*:* |      15 |        1.60 |        1646 |   2688 |    1.63 |      1856 | hashtable [100.0%] |    4544 |      2938 |      2955 |    2951.60 |
| Total:                             |      32 |        0.00 |        4822 |   7584 |    0.00 |      4416 |                    |   12000 |        -1 |      2955 |     nan    |
```

<br>

# 통계 내용은 완전 정확하진 않다.
[문서](https://github.com/gamenet/redis-memory-analyzer)에 따르면 RAM의 통계는 근사치이다.

일반적으로 보고된 메모리는 info가 보고한 것의 10% 이내여야한다고 말하고있으며, 아래 몇가지 메모리를 통계시 카운트하지 않는다고한다.

* Memory used by allocator metadata
* Memory used for pub/sub
* Redis process internals (like shared objects)


