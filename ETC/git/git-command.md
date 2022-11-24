# 목차

<br>

# Git 명령어 모음
자주 사용하는 Git 명령어 정리

<br>

## Config 설정
```shell
# config 확인
$ git config --list

# 전역 config 확인
$ git config --global --list

# 전역 email, username 설정
$ git config --global user.email "{email}"
$ git config --global user.name "{username}"
```
* email, username 설정.
* `--global`빼면 해당 리포지토리 지역 설정.

<br>

## 커밋
```shell
# 스테이징 올리기
$ git add <파일>

# 커밋
$ git commit -m "<메시지>"

# 파일 하나하나 확인하며 스테이징 올릴지 체크.
$ git add -p

# (커밋 안된) 스에티징 변경 사항 재설정 - 전체
$ git reset HEAD

# (커밋 안된) 스테이징 변경 사항 재설정
$ git reset HEAD <파일>

# 마지막 커밋 수정 (hash 새로 생성됨)
$ git commit -m "<메시지>" --amend
```

<br>

## 이력 (그래프)
```shell
# 모든 이력 보기
$ git log

# 최근 n개만큼만 커밋 보기
$ git log -n

# 이틀 전까지의 커밋 로그 보기
$ git log --before="2days"

# 
$ git log --graph --all --oneline --decorate --date-order

$ git log --branches --decorate --graph --oneline
```

<br>

## 브랜치
```shell
# 브랜치 목록
$ git branch

# 브랜치 이동
$ git checkout <브랜치이름>

# 브랜치 생성 & 이동
$ git chcekout -b <브랜치이름>

# 브랜치 삭제
$ git branch -d <브랜치이름>
```

<br>

## 리모트
```shell
# 변경사항 원격 서버 업로드
$ git push <remote> <브랜치이름>

# 원격 저장소 리스트
$ git remote -v

# 원격 저장소 등록
$ git remote add <원격 저장소 이름> <원격 서버 주소>

# 원격 주소 삭제
$ git remote remove <원격 저장소 이름>
```

<br>

## 갱신 및 병합
```shell
# 원격 저장소로부터 현재 브랜치 pull
$ git pull

# 현재 브랜치에 다른 브랜치의 수정사항 병합 (현재 브랜치가 to, <다른 브랜치>가 from)
$ git merge <다른 브랜치>

# 선택하여 병합
$ git cherry-pick <커밋명>
```

<br>

## diff
```shell
# 변경 내용 비교
$ git diff <브랜치이름> <다른 브랜치 이름>
```


