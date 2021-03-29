# fetch

<br>

- [fetch](#fetch)
  - [1 fetch란](#1-fetch란)
  - [2 fetch를 통한 HTTP 통신](#2-fetch를-통한-http-통신)

<br>

## 1 fetch란
```js
const promise = fetch(url [, options])
```
* 사전적 의미
  * 가지고 오다. 불러 오다.
* fetch함수
  * XMLHttpRequest 객체와 마찬가지로 HTTP 요청 전송 기능을 제공하는 클라이언트 사이드 Web API.
  * IE를 제외한 대부분의 모던 브라우저에서 제공한다.
* fetch함수는 Promise 객체를 반환한다.
  * 정확히는 HTTP 응답을 나타내는 Response 객체를 래핑한 Promise를 반환한다.

```js
const url = 'https://jsonplaceholder.typicode.com';

fetch(`${url}/todos/1`)
  .then(response => console.log(response)); // Response 객체를 반환한다.
```

<br>

## 2 fetch를 통한 HTTP 통신
```js
const request = {
    get(url) {
        return fetch(url);
    },
    post(url, payload) {
        return fetch(url, {
            method: 'POST',
            headers: { 'content-type': 'application/json' },
            body: JSON.stringify(payload)
        });
    },
    patch(url, payload) {
        return fetch(url, {
            method: 'PATCH',
            headers: { 'content-type': 'application/json'},
            body: JSON.stringify(payload)
        });
    },
    delete(url) {
        return fetch(url, { method: 'DELETE' });
    }
};
```
```js
// GET 요청
const url = "https://jsonplaceholder.typicode.com";

request.get(`${url}/todos/1`)
  .then(response => response.json())
  .then(todos => console.log(todos))
  .catch(e => console.error(e));

// POST 요청
request.post(`${url}/todos`, {
    userId: 1,
    title: 'JavaScript',
    completed: false
}).then(response => response.json())
  .then(todos => console.log(todos))
  .catch(e => console.error(e));
```

<br>

**40x 에러 처리**
```js
// 40x 에러로 인해 요청에 실패한 경우에도 응답 본문을 반환한다.
// 그럴땐 다음과 같이 하자
fetch('https://jsonplaceholder.typicode.com/posts/1')
  .then(data => {
    if (!data.ok) {
      throw new Error(data.status)
    }
    return data.json()
  })
  .then(post => {
    console.log(post.title)
  })
  .catch(error => {
    console.log(error)
  })
```
