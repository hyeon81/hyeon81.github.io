---
layout: post
title: "[241029] Organize Imports, useSWR에서 Multiple Arguments 받는 법"
date: 2024-10-29 00:00:00 +0900
categories: memo
---

_어째서 글 하나 쓰는데 이렇게 오래걸릴까… 매일매일 회고하는 게 목표인데 어렵다 .._

### Organize Imports

_오래된 회사 프로젝트는 쉽사리 코드를 못 건드리겠다. 그 중 하나가 import. 개인 프로젝트면 불필요한 import는 설정으로 싹 다 날려버릴 텐데,… 회사 프로젝트는 괜히 지웠다가 나중에 필요해지면 헷갈리기만 할 것 같아 그냥 내버려둔다…_

_근데 이번에 새로운 단기 프로젝트를 혼자 맡게 되었다. 그래서 맨날 불필요한 import 지우고 있다. eslint 였나 설정으로 save할 때마다 불필요한 import를 날리는 설정이 있었던 것 같은데 찾기 귀찮다._ **command + shift + p로 Organize Imports 선택해주면 해당 파일의 불필요한 import는 날아간다.**

### useSWR에서 **Multiple Arguments 받는 법**

```jsx
const { data: user } = useSWR(["/api/user", token], ([url, token]) =>
  fetchWithToken(url, token)
);
```

key를 배열 형태로 전달하여 fetcher에 argument를 전달하는 방법이다.

```jsx
const { data: orders } = useSWR({ url: "/api/orders", args: user }, fetcher);
```

문서를 보니까 object 형태로 던지는 것도 가능한가보다

### **HttpOnly Cookie는 getServerSideProps에서 접근을 못할까?**

정답을 찾지 못했다. 이건 내가 직접 서버를 파서 실험을 해봐야 알 듯…

일단 내 이전 경험으로서는 접근이 어려웠었다.

[오늘 본 글 들]

- https://codiving.kr/99
- https://swr.vercel.app/docs/arguments
- https://github.com/vercel/next.js/discussions/17677
