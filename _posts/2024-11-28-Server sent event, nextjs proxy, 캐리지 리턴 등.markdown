---
layout: post
title: "[24.11.28] Server sent event, nextjs proxy, 캐리지 리턴 등"
date: 2024-11-28 00:00:00 +0900
categories: memo
---

### SSE란

- **server-sent-event**
- 클라이언트가 서버로부터 일방향 스트리밍 데이터를 받을 수 있도록 해주는 웹 기술이다. (서버가 데이터를 보내고 싶을 때 보낼 수 있다)
- https://developer.mozilla.org/ko/docs/Web/API/Server-sent_events

```jsx
var source = new EventSource("updates.cgi");
source.onmessage = function (event) {
  alert(event.data);
};
```

이런 식으로 서버로부터 data를 수신받을 수 있다.

그냥 EventSource를 쓰면 get 요청밖에 보낼 수가 없다. 우리는 post 요청을 해야했기 때문에, post 요청을 구현 할 수도 있겠지만 라이브러리를 썼다.

### fetchEventSource

- https://github.com/Azure/fetch-event-source
- microsoft에서 만들었다.

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

이런 식으로 post 요청을 보낼 수 있다.

AbortController를 사용하여 요청을 중지할 수도 있다.

### **cors에는 nextjs의 proxy 설정을 써보자**

- cors 해결하는 법
  - 서버에서 access-control-allow-origin를 허용해준다 (하지만 우리 회사에선 사용하지 않는 듯)
  - 프론트에서 proxy를 사용한다. https://nextjs.org/docs/pages/api-reference/next-config-js/rewrites ✅

```jsx
/** @type {import('next').NextConfig} */
const nextConfig = {
	...
  async rewrites() {
    return [
	    ...
      {
        source: `/api/:path*`,
        destination: `(api주소)/:path*`,
      },
    ];
  },
};

module.exports = nextConfig;

```

이런식으로 설정해주면 된다.

### postman

업무하다가 갑자기 Postman을 다룰 일이 있었다. 너무 오랜만에 켜봐서 허둥지둥 했는데 좀… 부끄러웠다 🙄

![스크린샷 2024-11-28 오후 6.00.03.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/7b11a208-c4b5-4782-923f-1e83de0d8a3e/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-28_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6.00.03.png)

### **캐리지 리턴**이란

**carriage return (`CR`)**

- 문자의 새 줄을 시작하는 데 쓰이는 [제어 문자](https://ko.wikipedia.org/wiki/%EC%A0%9C%EC%96%B4_%EB%AC%B8%EC%9E%90)나 그 구조를 가리킨다.
- 현재 커서를 해당 줄의 맨 앞으로 이동시키는 기능이다
- **문자 코드 값**: `\r` (ASCII 13, 0x0D)

캐리지리턴이 있으면 빠지지않는 **라인 피드(Line Feed, `LF`)**

- `\n` (ASCII 10)
- 커서를 다음 줄로 이동시킨다

**운영체제마다 줄바꿈 방식이 다르다.**

- **Windows**: `CR + LF` (`\r\n`)
- **Unix/Linux/macOS (현대)**: `LF` (`\n`)
