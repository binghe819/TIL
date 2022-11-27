# 목차

<br>

- [목차](#목차)
- [개요](#개요)
  - [VSCODE](#vscode)
  - [Git](#git)
  - [iterm2](#iterm2)
  - [JAVA](#java)

<br>

# 개요
컴퓨터 바꿀 때마다 세팅하는데 이곳저곳 찾다보니.. 머리아파서 정리해둔 자료.

> 맥을 기준으로 작성된 자료입니다.

<br>

## VSCODE

<details>
  <summary>VSCODE</summary>
  
  ---
  
  * 플러그인
    * Atom One Dark Theme
    * Auto Close Tag
    * Beautify
    * ESLint
    * Live Server
    * Markdown All in One
    * Markdown Preview Enhance
    * Material Icon Theme
    * vscode-icons
  * Snippet
    * [Snippet 개념과 설명](https://jojoldu.tistory.com/491)
    * Snippet 트리거 설정 -> `Keyboard Shortcuts`에서 `Trigger Suggest`를 `Option + Space`로 수정
  
    ```json
    <!-- 많이 사용하는 Snippet -->
    {
        // Place your snippets for markdown here. Each snippet is defined under a snippet name and has a prefix, body and 
        // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
        // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
        // same ids are connected.
        // Example:
        // "Print to console": {
        // 	"prefix": "log",
        // 	"body": [
        // 		"console.log('$1');",
        // 		"$2"
        // 	],
        // 	"description": "Log output to console"
        // }
        "img": {
            "prefix": ["img"],
            "body": [
                "<p align=\"center\"><img src=\"./image/\"> </p>"
            ],
            "description": "이미지 스니펫"
        },
        "toggle": {
            "prefix": ["to"],
            "body": [
                "<details>",
                "  <summary>토글</summary>",
                "  ",
                "  ---",
                "  ",
                "  ",
                "  ",
                "  ---",
                "</details>"
            ],
            "description": "토글 스니펫"
        }
    }
    ```
  
  ---
</details>

<br>

## Git

<details>
  <summary>Git</summary>
  
  ---
  
  * [Git 설치 및 설정](https://investechnews.com/2021/06/14/mac-git-setting/)
    * `user.name`과 `user.email` 꼭 설정해주자.
  
  ---
</details>

<br>

## iterm2

<details>
  <summary>iterm2</summary>
  
  ---
  
  * [iterm2 및 zsh 설치 및 기본 커스텀](https://ooeunz.tistory.com/21)
  * [유용한 플러그인 및 설정방법 - 여기 나오는 방법이 좋음](https://2dowon.netlify.app/etc/use-iterm2/)
  
  ---
</details>

<details>
  <summary>iterm2 커서이동 기본값 수정</summary>
  
  ---
  
  * https://ifuwanna.tistory.com/300
  
  ---
</details>

<br>

## JAVA
<details>
  <summary>JAVA</summary>
  
  ---
  
  * [Amazon Open JDK - Correto 8](https://docs.aws.amazon.com/ko_kr/corretto/latest/corretto-8-ug/macos-install.html)
  * 
  
  ---
</details>

<br>




