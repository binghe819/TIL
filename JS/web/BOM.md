# BOM

<br>

<br>

## BOM을 알아야 하는 이유
* 유저에게 경고창을 띄우고 싶은 경우
* 유저의 yes or no와 같은 선택에 따라 다른 응답을 보여주고 싶은 경우
* 유저가 브라우저 창을 닫기 전에 정말 떠날 것인지 확인하고 싶은 경우
* 유저가 접속한 환경을 알고 싶은 경우
* 현재 url 위치 및, 접속 history를 알고 싶은 경우

여러 ui를 직접 만들고 자바스크립트로 직접 제어할 수도 있지만, 브라우저에서 제공하는 기본 API를 사용한다면 빠르고 간단하게 프로토타이핑을 할 수 있다.

<br>

## BOM이란
* BOM(Browser Object Model)이란
  * **웹 브라우저 환경의 다양한 기능을 객체처럼 다루는 모델이다.**
  * 대부분의 브라우저에서 구현은 되어있지만, **정의된 표준이 없어 브라우저 제작사마다 세부사항이 다르고 다소 한정적이라는 특징이 있다.**
* BOM의 역할
  * 웹 브라우저의 버튼, URL 주소 입력창, 타이틀 바 등 웹브라우저 윈도우 및 웹페이지의 일부분을 제어할 수 있게끔 하는 것이다.
  * `window`객체를 통해 접근이 가능하다.
* 대표적인 BOM 객체
  * window: Global Context. 브라우저 창 객체
  * screen: 사용자 환경의 디스플레이 정보 객체
  * location: 현재 페이지의 url을 다루는 객체
  * navigator: 웹브라우저 및 브라우저 환경 정보 객체
  * history: 현재의 브라우저가 접근했던 URL history

<br>

## BOM API

<br>

### alert
```js
window.alert([message]);
alert([message]);
```
* 확인 버튼을 가지며 메시지를 지정할 수 있는 경고 대화 상자를 띄운다.

<br>

### prompt
```js
// 입력값이 없으면 null을 반환
result = window.prompt([message], [default]);

// 예시
let sign = prompt('당신의 별자리는 무엇입니까?);
if (sign.toLowerCase() === '전갈자리') {
    alert('와! 저도 전갈자리에요!');
}
```
* message: 사용자에게 보여줄 문자열. 프롬프트 창에 표시할 메시지가 없으면 생략할 수 있다.
* default: 텍스트 입력 필드에 기본으로 채워 넣을 문자열.
* 기본 반환값은 문자열이다!

<br>

### confirm
```js
result = window.confirm(message);
```
* message: 경고 대화 상자에 표시할 텍스트 문자열
* 반환 값: 확인(true)또는 취소(false);

<br>

# 참고
* 우테코 BOM 강의

