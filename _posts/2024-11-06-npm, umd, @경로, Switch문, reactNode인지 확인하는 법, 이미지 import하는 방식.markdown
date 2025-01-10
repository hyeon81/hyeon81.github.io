---
layout: post
title:  "[241106] npm,umd/@경로/Switch문/reactNode인지 확인하는 법/이미지 import하는 방식"
date:   2024-11-06 00:00:00 +0900
categories: memo
---

### npm / umd / dev build..

처음 본 라이브러리를 설치해보려는데, 라이브러리 docs install 가이드에 위와 같은 방법을 소개하고 있었다. npm이야 많이 사용해서 익히 알고 있지만, umd나 dev build는 잘 안쓰는 방식이라 검색해봤다.

- **npm**
    - `npm install [라이브러리]`
    - node package manager로 패키지를 다운받는 형식
    - 받은 패키지는 import나 require로 가져다 쓸 수 있다.
- **umd**
    - `<script src="[https://unpkg.com/react/umd/[라이브러리].min.js](https://unpkg.com/react/umd/react.production.min.js)"></script>`
    - Universal Module Definition
        - node js 뿐만 아니라 다양한 JavaScript 환경에서 사용할 수 있는 방식
        - HTML 파일에서 `<script>` 태그로 바로 사용하거나, Node.js에서 모듈로 불러오는 등 다양한 환경에서 호환된다 (node js가 아니어도 쓸 수 있다)
    - [unpkg.com](http://unpkg.com/)
        - npm에 등록된 패키지를 cdn으로 바로 활용 가능한 서비스
        - cdn: content delivery network. 콘텐츠를 빠르게 전달하기 위해 서버를 분산한 네트워크다.
- **dev build**
    - `git clone [https://github.com/](https://github.com/recharts/recharts.git)[라이브러리 주소]`
    - 이건 그냥 직접 git clone 받아서 사용하는 방식인듯

**[궁금한 것]**

umd를 검색하다보니 umd는 npm 보다는 commonJs, amd, es6 등 모듈 시스템으로 함께 언급되는 듯 하다. 이 부분에 대해서도 자세히 알고 싶었으나 항상 그러다가 TIL을 다 마무리 짓지 못하고 하루가 지나버린 전적이 있기 때문에 나중에 공부 하는 걸로

### @ 골뱅이로 시작하는 경로

프로젝트를 하다보니 @로 시작하는 경로를 자주 본다.  처음엔 이게 자동으로 설정된 건 줄 알았는데 (루트 디렉토리로…) 경로 별칭을 따로 지정해줬다는 걸 이제야 깨달았다.. 😓

보통 webpack이나 vite와 같은 모듈 번들러를 사용할 때 config 파일에서 설정할 수 있다고 한다. NEXTJS라면 이런 식으로…

```jsx
*{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["components/*"]
    }
  }
}*
```

```jsx
import Button from '@/components/button'
 
export default function HomePage() {
  return (
    <>
      <h1>Hello World</h1>
      <Button />
    </>
  )
}
```

- https://nextjs.org/docs/app/building-your-application/configuring/absolute-imports-and-module-aliases

*회사 프로젝트에서도 해당 설정이 되어있을텐데 이 밤에 회사 프로젝트를 까보긴 좀 그러니 회사가서 확인하는 걸로…*

### Switch문

- https://ko.javascript.info/switch
    - `switch`문은 하나 이상의 `case`문으로 구성됩니다. 대개 `default`문도 있지만, 이는 필수는 아닙니다

### 주석에 Todo를 작성하는 법

```jsx
/**
 * @todo 내용 
 */
```

vscode에서 작성하면 todo 부분의 색깔이 변해서 기분이 좋다

- https://jsdoc.app/tags-todo

### 해당 변수가 reactNode인지 확인하는 법

- `React.isValidElement(obj)`
- https://stackoverflow.com/questions/32635867/check-if-variable-is-react-node-or-array

*공용 컴포넌트에서 어떻게든 props 수를 줄여보려고 들어오는 값이 reactNode인지 아닌지에 따라 다르게 렌더링하려고 했는데 부질없는 일인 것 같아서  (그냥 들어오는 값의 타입이 다른 건 따로 받는 걸로…)그만 알아보기로 했다.*

### 이미지를 import 하는 방식

난 귀찮아서 매번 image 컴포넌트에 경로를 직접 입력했다. 덕분에 가끔 경로가 틀리거나 철자가 틀리면 이미지가 깨지는 곤란한 상황이 있었다.

회사 프로젝트에서는 이미지들을 한 파일에 모두 정의해놓고 import 하는 형식으로 관리하고 있다. 이렇게 하면 경로 관리도 되고 깔끔하다. 근데 한 파일에 담기는 이미지가 너무 많아져서 좀 분리했으면 좋겠다 (…)

```jsx
import React from 'react';
import { ReactComponent as Logo } from './logo.svg';

function MyComponent() {
  return (
    <div>
      <Logo width="100" height="100" />
    </div>
  );
}

export default MyComponent;

```

### **[짧은 일기]**

새 프로젝트 들어갔는데 chart를 그리는 작업이 필요해서 머리가 어질어질하다. (물론 훌륭한 선임들이 잘 짜주셔서 데이터에 맞게 수정만 하면 되긴함. 근데 그 코드 분석하는 데도 꽤 걸림 ㅠ)

오늘은 청년 공간 안 가고 집에서 til를 작성했더니 시간이 배로 든다. 춥더라도 청년 공간을 가자.