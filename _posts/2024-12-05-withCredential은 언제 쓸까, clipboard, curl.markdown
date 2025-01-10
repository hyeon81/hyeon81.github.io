---
layout: post
title: "[24.12.05] withCredential은 언제 쓸까, clipboard, curl"
date: 2024-12-05 00:00:00 +0900
categories: memo
---

### withCredential은 언제 쓸까

withCredential은 **브라우저가 요청에 인증 정보를 포함할지 여부를 지정**할 때 사용된다.

**쓰는 이유?** `withCredentials`가 활성화되지 않으면, 브라우저는 Cross-Origin 요청에서 기본적으로 쿠키를 포함하지 않고 HTTP 인증 헤더(`Authorization`)를 포함하지 않는다.

그러므로 다른 도메인에 인증 정보를 담아 요청을 보낼 때, withCredentials 옵션을 붙여 인증정보를 포함하도록 해야한다.

```jsx
const xhr = new XMLHttpRequest();
xhr.open("GET", "http://example.com/", true);
xhr.withCredentials = true;
xhr.send(null);
```

https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials

### javascript 클립 보드 복사하기

```jsx
navigator.clipboard.writeText(text);
```

Clipboard = 시스템 클립보드의 컨텐츠에 읽기와 쓰기권한을 제공하는 web api

```jsx
button.addEventListener("click", () => writeClipboardText("<empty clipboard>"));

async function writeClipboardText(text) {
  try {
    await navigator.clipboard.writeText(text);
  } catch (error) {
    console.error(error.message);
  }
}
```

이런식으로 원하는 텍스트를 시스템의 클립보드에 저장할 수 있다.

https://developer.mozilla.org/en-US/docs/Web/API/Navigator/clipboard

### Curl

- https://curl.se/

브라우저가 아닌 곳에서 http 요청(물론 다른 요청도 가능!)을 보낼 때 많이 사용한다 (ex 터미널 같은 CLI)

보통 서버와의 통신을 확인하거나 테스트할 때 사용한다

```jsx
curl -X GET https://api.example.com/v1/users
```

(이번에 SSE 테스트하면서 서버 측 개발자분과 SSE 테스트할때 자주 사용해서 적어봤다)
