---
layout: post
title: "[24.12.04] Dispatch<SetStateAction<타입>>, abortController"
date: 2024-12-04 00:00:00 +0900
categories: memo
---

### setState 타입

useState를 쓰다보면 종종 state와 setState를 props로 넘길 때가 있다. 그럴 때 setState 타입을 `React.Dispatch<React.SetStateAction<타입>>`

이렇게 설정한다.

React.Dispatch<ACTION>

### abortController

https://developer.mozilla.org/en-US/docs/Web/API/AbortController

클라이언트 측에서 SSE를 받는 코드를 짜고 있었는데, 예시에 abortController로 SSE를 중단하는 부분이 있었다.

AbortController란 웹 request를 원할 때 중단할 수 있는 컨트롤러 오브젝트다

[`AbortController.abort()`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController/abort)

이런식으로 호출하면 지금 보내는 요청이 중단된다.

```jsx
async function fetchVideo() {
  controller = new AbortController();
  const signal = controller.signal;

  try {
    const response = await fetch(url, { signal });
    console.log("Download complete", response);
    // process response further
  } catch (err) {
    console.error(`Download error: ${err.message}`);
  }
}
```

예시를 보니 이런식으로 요청에 붙여넣어서 중단하고 싶을 때 중단하나 보다.

```jsx
const ctrl = new AbortController();
fetchEventSource("/api/sse", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    foo: "bar",
  }),
  signal: ctrl.signal,
});
```

fetchEventSource를 사용할 때는 이런식으로 요청에 붙인 뒤 이후 `ctrl[.abort()](https://developer.mozilla.org/en-US/docs/Web/API/AbortController/abort)`를 호출하여 중단하고 싶을 때 중단시켰다.

### npm run build는 뭐고 npm run dev는 뭔 차이일까

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/3b9e6b5c-f569-46c7-9a72-48896aeb4042/image.png)
