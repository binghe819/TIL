# 목차

<br>

- [목차](#목차)
- [리눅스 명령어 모음](#리눅스-명령어-모음)
- [특정 프로세스 상태 확인.](#특정-프로세스-상태-확인)
- [메모리](#메모리)
- [디스크](#디스크)

<br>

# 리눅스 명령어 모음
자주 사용하는 명령어 기록용.

<br>

# 특정 프로세스 상태 확인.

<br>

<details>
  <summary>특정 프로세스 사용령 확인.</summary>
  
  ---
  
  * ps 명령어로 확인.
    * `ps -eo user,pid,ppid,rss,size,vsize,pmem,pcpu,time,cmd --sort -rss | head -n 11`: 현재 실행중인 프로세스의 메모리 사용량으로 정렬후 PID와 각종 메모리 사용률 및 프로세스 상세하게 출력.
  * top 명령어로 확인.
    * `top` 명령어로 프로세스 상태창 들어간후, `Shift + m`으로 메모리 사용량순 정렬.
    * `top -d 1 | egrep "{PID}|systemd"`: 1초마다 특정 프로세스의 메모리 사용량 체크. (cpu도 같이 확인가능)
  
  ---
</details>

<br>

# 메모리

<br>

<details>
  <summary>메모리 용량 확인 - free</summary>
  
  ---
  
  * `free`: 시스템의 전체 (total) 여유 (availabel) 메모리등의 정보를 보여줌.
    * KB단위로 메모리 크기를 보여준다.
  * `free - h`: 메모리 크기를 더 읽기 좋은 단위로 변환해서 보여준다.
  
  ---
</details>

<details>
  <summary>메모리 용량 확인 - meminfo</summary>
  
  ---
  
  * `cat /proc/meminfo`: 메모리 정보 확인.
  * `cat /proc/meminfo | grep MemTotal`: 전체 메모리 용량 확인.

  > 자주 사용되지않는 명령어. 실제론 `free`를 많이 사용한다. 간혹 메모리의 정보를 확인하기위해 사용함.
  
  ---
</details>

<br>

# 디스크

<br>

<details>
  <summary>디스크 남은 용량 확인 - df (disk free)</summary>
  
  ---
  
  * `df`: 디스크 남은 용량 확인
  * `df -k`: KB단위로 남은 용량 확인
  * `df -m`: MB단위로 남은 용량 확인
  * `df -h`: GB단위로 남은 용량을 깔끔하게 확인.
  * `df .`: 현재 위치한 디렉토리가 포함된 파티션의 남은 용량 확인

  ---
</details>

<details>
  <summary>디스크 사용량 확인 - du (disk usage)</summary>
  
  ---
  
  * `du`: 현재 디렉토리에서 서브 디렉토리까지 용량 확인
  * `du -a`: 현재 디렉토리의 사용량 파일 단위로 출력
  * `du -s`: 총 사용량
  * `du -h`: du 명령어 깔끔하게 확인.
  * `du -h --max-depth=1`: 한단계 서브 디렉토리 기준으로 du -h 명령확인. (가장 유용)
  * `du -sh *`: 한단계 서브 디렉토리 기준으로 확인.
  
  ---
</details>

<br>



