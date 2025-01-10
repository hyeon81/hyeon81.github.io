---
layout: post
title: "[241024] instanceof AxiosError/encrypt-storage/keyof/fnm"
date: 2024-10-24 00:00:00 +0900
categories: memo
---

### **error instanceof AxiosError**

프로젝트 코드를 보다 보니 대충 이런 코드가 있었다.

```jsx
import axios, { AxiosError } from 'axios';

async function fetchData() {
  try {
    const response = await axios.get('https://api.example.com/data');
    ...
  } catch (error) {
    **if (error instanceof AxiosError)** {
	    ...
    } else {
      ...
    }
  }
}

fetchData();
```

**`if (error instanceof AxiosError)`** 라는 코드를 처음보는 지라 이런 코드가 왜 있는 건지 의아했다.

**`if (error instanceof AxiosError)`란**

- **instanceof**
  - https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/instanceof
  - “`instanceof` 연산자는 `object`의 프로토타입 체인에 `constructor.prototype`이 존재하는지 판별합니다”
  - type도 object기 때문에 `error instanceof AxiosError` 라고 하면 error가 AxiosError에 속하는지 확인한다.
  - true 혹은 false를 반환한다.
- **AxiosError**
  ```jsx
  {
      "message": "Request failed with status code 404",
      "name": "AxiosError",
      "stack": "AxiosError: Request failed with status code 404\n    at settle (http://localhost:3000/_next/static/chunks/node_modules_cda541._.js:4438:16)\n    at XMLHttpRequest.onloadend (http://localhost:3000/_next/static/chunks/node_modules_cda541._.js:4508:174)\n    at Axios.request (http://localhost:3000/_next/static/chunks/node_modules_cda541._.js:5141:49)\n    at async testAxios (http://localhost:3000/_next/static/chunks/app_c7576d._.js:40:13)",
      "config": {
          "transitional": {
              "silentJSONParsing": true,
              "forcedJSONParsing": true,
              "clarifyTimeoutError": false
          },
          "adapter": [
              "xhr",
              "http",
              "fetch"
          ],
          "transformRequest": [
              null
          ],
          "transformResponse": [
              null
          ],
          "timeout": 0,
          "xsrfCookieName": "XSRF-TOKEN",
          "xsrfHeaderName": "X-XSRF-TOKEN",
          "maxContentLength": -1,
          "maxBodyLength": -1,
          "env": {},
          "headers": {
              "Accept": "application/json, text/plain, */*"
          },
          "method": "get",
          "url": "http://localhost:3000/api/hello"
      },
      "code": "ERR_BAD_REQUEST",
      "status": 404
  }
  ```
  axiosError를 구글링했더니 잘 안나오길래 그냥 console에 찍어봤다. 대충 요런 형태로 나온다.

따라서 **`if (error instanceof AxiosError)`** 문에서는 `axios` \*\*\*\*에서 발생한 에러일 경우만 실행하게 하는 코드다.

**왜 이런 코드가 필요한가?**

try..catch에서 catch로 잡히는 error는 unknown이다. 왜냐면? axios 에러 뿐만 아니라 throw로도 에러를 던질 수 있기 때문이다.

다만 나는 어떤 에러가 잡히든 console.error를 찍거나 에러 모달을 보여주는 형태로 처리했기 때문에 어떤 error가 잡히든 구별하지 않아서 위와 같은 코드가 낯설었던 것 같다.

비슷한 기능을 하는 함수로는 **`axios.isAxiosError(error)`** 가 있다.

**결론**

**`if (error instanceof AxiosError)`** 는 axios에서 일어난 에러만 처리할 때 사용한다.

**의문점**

사실 의문인 건 **`if (error instanceof AxiosError)`** 아래에 있는 \*_\*\* else 처리였다. 서버 API 호출 시 http status code엔 200이 뜨지만 response body에 서버에서 정의해둔 에러 코드가 담겨 오는 상황이었다. _(사실 서버에서 왜 이렇게 처리하는지도 궁금하다…)\* 근데 response body에 담긴 에러 코드가 error를 throw 하지 않을텐데 (axios error가 아니니까…?) 이상하게 catch문으로 계속 들어가는 것이다… 그 이유를 찾고 싶었지만 찾지 못했다. 나중에 다시 찾아봐야겠다.

**찾아본 링크**

https://ko.javascript.info/try-catch

---

### **encrypt-storage**

https://www.npmjs.com/package/encrypt-storage

이것도 프로젝트 코드를 보다가 발견했다.

서버에서 받아온 데이터를 특정 페이지에 넘겨줘야 할 때 사용하는 듯 했다.

**encrypt-storage란?**

`localStorage` and `sessionStorage` 에 암호화된 데이터를 저장한다.

그리고 get할 때 암호화를 풀어 원본 데이터를 제공하는 라이브러리다.

```jsx
//storage 생성, 어디다 저장할지 option에서 정할 수 있다.
export const encryptStorage = new EncryptStorage('secret-key-value', options);

//storage에 값 저장
encryptStorage.setItem('token', 'edbe38e0-748a-49c8-9f8f-b68f38dbe5a2');

//storage에 있는 값 가져오기
const value = encryptStorage.getItem<T = any>('token');

//storage에 있는 값 지우기
encryptStorage.removeItem('token');
```

| **Key**          | **Value**                                  |
| ---------------- | ------------------------------------------ |
| `@example:token` | `U2FsdGVkX1/2KEwOH+w4QaIcyq5521ZXB5pqw`... |

storage엔 이런식으로 저장된다고 한다.

**왜 쓰는걸까?**

기본적으로 어떤 데이터를 담을 때 state를 사용하지만 state는 새로고침하면 날아가니까, localStorage나 sessionStorage에 데이터를 담는 경우가 있다. 근데 둘 다 브라우저의 Application 탭에서 쉽게 확인이 가능하고 조작할 수 있기 때문에 이 라이브러리를 쓰는 게 아닌가 싶다.

하지만 공식 github readme에도 써져있다시피 암호화에 이용한 비밀키가 프론트엔드에 저장되어있기 때문에 충분히 찾고자하면 찾을 수 있다고 한다. 고로 완전히 안전하지 않다는 점을 주의해야 한다.

이런 코드를 보다보면 어떻게 데이터를 저장할지 고민하게 된다. React의 state에 담자니 새로고침하면 날아가고, localStorage나 sessionStorage에 담자니 application 탭에서 조작이 되니까 중요한 로직에 관여하는 값은 담기가 어렵다. .. … 근데 이 글 쓰면서 어떻게 저장하면 될지 깨닫게 되었다. 생각해보니까 서버에서 오는 값이 이미 암호화되어 오기 때문에 그걸 storage에 담아주면 될 것 같다. ㅎㅎ..

---

### Object key를 type으로 만드는법

**keyof** 연산자를 쓰면 된다.

_이걸 찾았던 이유. 서버에서 오는 데이터를 Table 형태로 그려야 했는데, 받아온 데이터는 Table엔 적절하지 않아 프론트에서 가공해야 했다. 그 때 쓴 게 keyof 연산자._

_가공하여 map을 돌리기 편한 형태로 데이터를 모아두는 것까지는 적절했는데, 거기서 더 나아가 데이터에 해당하는 데이터명(한글)을 한번에 묶어서 렌더링하고 싶어서 그 부분까지 매칭할 수 있도록 데이터를 다시 가공했다._

_그러다보니 데이터가 2중, 3중 for문을 돌게 되고, 이게 렌더링을 위한 데이터 가공인지 데이터 가공을 위한 데이터 가공인지 헷갈리기 시작했다.. 렌더링 부분을 순수하게 만들려고 하다보니 오히려 상위 컴포넌트에서 부차적인 액션이 많이 일어나서, 다른 사람이 내 코드를 보면 못 알아볼 것 같다는 생각이 들었다. (나도 어지러웠다) 그래서 그냥 다시 원상복귀하고, 적당히 추상화하는 쪽으로 작업했다._

---

### NODE 설치

https://nodejs.org/en/download/package-manager

- 맨날 node js 설치할 때 소스코드 받아서 설치했는데 이번엔 패키지 매니저 사용했다.

```powershell
# installs fnm (Fast Node Manager)
winget install Schniz.fnm

# configure fnm environment
fnm env --use-on-cd | Out-String | Invoke-Expression

# download and install Node.js
fnm use --install-if-missing 20

# verifies the right Node.js version is in the environment
node -v # should print `v20.18.0`

# verifies the right npm version is in the environment
npm -v # should print `10.8.2`
```

- 원래 chocolatey 사용했는데 fpm을 권장하길래 fpm으로 설치했다

```powershell
# Chocolatey is not a Node.js package manager.
# Please ensure it is already installed on your system.
# Follow official instructions at https://chocolatey.org/
# Chocolatey is not officially maintained by the Node.js project and might not support the v20.18.0 version of Node.js
```

- 대충 chocolatey는 node.js manager가 아니니 fnm 쓰라는 뜻
- https://chocolatey.org/
  - Chocolatey = 윈도우 소프트웨어 패키지 매니저
- https://github.com/Schniz/fnm
  - fnm = fast and simple node.js version manager
  - macOS, window, linux 모두에서 사용가능한 크로스 플랫폼 매니저라고 한다.
  - rust 로 쓰여졌다.

→ 근데 node.js package 가 왜 필요하지? 난 처음에 fnm이 npm같은 건 줄 알았는데 npm은 node.js 패키지를 관리하는 툴이고 fnm은 node.js 버전 관리에 중점을 둔 도구라고 한다.

→ 왜 node.js 버전을 관리를 manager까지 두어서 관리하는 걸까…? 생각했는데 프로젝트마다 필요한 node.js이 달라서 여러 버전을 사용하게 되는 경우가 많다고 한다. 프로젝트마다 node 버전을 간편하게 바꾸기 위해 사용한다고. _(난 그런 적이 딱히…)_

- 비슷한 기능을 하는 node manager로 nvm이 있다. fnm이 fast node manager니까 nvm보다 더 빠르겠지…?
  - https://github.com/nvm-sh/nvm
