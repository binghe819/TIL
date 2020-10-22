# 목차









# Branch & Conflict

소프트웨어 개발할 때에 개발자들은 동일한 소스코드를 함께 공유하고 다루게 된다.

여러 사람이 동일한 소스코드를 기반으로 서로 다른 작업을 할 때에는 각각 서로 다른 버전의 코드가 만들어 질 수 밖에 없다.

이럴 때 사용하는 것이 바로 브랜치이다.



## 1 Branch란

🤔 브랜치란

* **브랜치는 독립적으로 어떤 작업을 진행하기 위한 개념이다.**
* 또한, **이렇게 만들어진 브랜치는 다른 브랜치와 병합(Merge)함으로써, 작업한 내용을 다시 새로운 하나의 브랜치로 모을 수 있다.**



## 2 Branch 기본 사용법

> 실습을 위한 준비 -> 아래와 같이 txt파일 하나 만들고 커밋을 해주면 된다.
>
> <img src="image/image-20201022033358660.png" />



### 2-1 Branch 생성 및 조회

```shell
// Branch 만들기
git branch {branchname}

// Branch 목록보기
git branch
```

🙋‍♂️ 조회 내용

* `*`이 현재 가리키고 있는 브랜치이다. `HEAD`라고도 한다.

![image-20201022034915675](image/image-20201022034915675.png)

> `git log --all --graph --oneline` -> 브랜치 작업을 같이 보여준다.





### 2-2 Branch 전환하기

```shell
// Branch 전환
git check {branchname};
```



#### 실습 (Branch 생성 및 전환)

* master 브랜치에서 새로운 커밋을 한다. (`master work 4`)

  ![image-20201022035255130](image/image-20201022035255130.png)

  * master를 제외한 다른 브랜치는 여전히 `work 3`에 위치한다.

* master 브랜치에서 apple브랜치로 전환해보자. (`git checkout apple`)

  ![image-20201022035752917](image/image-20201022035752917.png)

  * HEAD가 apple을 가리키는 것을 볼 수 있다.

    ![image-20201022035818880](image/image-20201022035818880.png)

  * 또한, master 브랜치에서 커밋했던 `master work 4`의 대한 내용(`work.txt`의 마지막 문장)이 사라졌다.

  * 즉, `work 3`의 커밋 상태로 돌아간 것이다.

* 이제 apple브랜치에서 새로운 커밋을 해보자. (`apple work 4`)

  ![image-20201022040436405](image/image-20201022040436405.png)

  * 브랜치가 나눠져서 따로 커밋이 되는 것을 볼 수 있다.
    * `master`의 부모 `work 3`
    * `apple`의 부모 `work 3`



## 3 브랜치 병합



















