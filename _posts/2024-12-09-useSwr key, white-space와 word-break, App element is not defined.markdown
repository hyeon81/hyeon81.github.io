---
layout: post
title: "[24.12.09] useSwr key, white-space와 word-break, App element is not defined"
date: 2024-12-09 00:00:00 +0900
categories: memo
---

### useSWR을 사용할 때 주의할 점

```jsx
export const useCurrentList = (body) => {
  const {
    data,
    error,
    mutate: mutateCurrentList,
  } = useSWR<ListGetRsp | undefined>(
    url,
    async (url) => {
        const rsp = await axios.post(...)
        if (rsp.error_code === 200) {
					return rsp.data;
      }
      return undefined;
    }
  );

  return {
    data,
    error,
    mutateCurrentList,
  };
};

```

useSWR은 key값이 변할 때마다 다시 패칭을 해오기 때문에 (key 값으로 보통 url을 넣는다) get method로 데이터를 가져올 때는 편하지만, post method로 데이터를 가져올 때는 조금 귀찮아지는 현상이 생긴다.

post로 보낼 때는 body값이 변하지 url을 변경하지 않기 때문에 key값으로 url만 넣게 되면 당연히 key가 변하지 않아 바로 패칭이 되지 않는다 😢

이런 당연한 얘기를 왜 적어놓냐면… 여태동안 했던 프로젝트에서 게시글 조회와 같은 api는 무조건 get method로 호출해서 문제 없이 작업했었다. 근데 지금 회사에선 모든 method가 post method라 위 내용을 까먹고 작업하다가 삽질을 좀 했기 때문이다….

고로 post로 보내는 데 값이 바뀔때마다 호출이 되어야 한다면 key값을 신경써줍시다..

### white-space, word-break

```jsx
  white-space: pre-wrap;
  word-break: break-all;
```

**css 작업하다보면 자주 만나는 프로퍼티**

- **white-space:** 요소 안의 공백을 어떻게 다룰 것인지 정한다.
  - 줄바꿈 여부와 줄바꿈 방법을 정한다 (단어를 분리하려면  [`overflow-wrap`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap), [`word-break`](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break) 또는 [`hyphens`](https://developer.mozilla.org/en-US/docs/Web/CSS/hyphens) 를 사용하라고 한다)
  - 기본값인 normal의 경우 연속 공백을 하나로 합치기 때문에 그러고 싶지 않을 때 pre, pre-wrap 같은 프로퍼티를 사용한다
  - 한 줄이 너무 길어도 줄 바꿈이 안되었으면 할 때에는 no-wrap을 사용한다 (저번에 이 프로퍼티 넣어놨다가 횡스크롤이 생겼는데 원인이 이것때문인지 모르고 한창 삽질했었다…^^)
  - https://developer.mozilla.org/en-US/docs/Web/CSS/white-space
- **word-break:** 텍스트가 콘텐츠 상자를 넘어갈때마다 줄바꿈을 표시할지 여부를 설정한다
  - `word-break: keep-all` 줄바꿈할 때 단어를 유지하면서 적용된다
  - `word-break: break-all` 줄바꿈할 때 단어를 유지하지 않으면서 적용된다
  - https://developer.mozilla.org/en-US/docs/Web/CSS/word-break

### Warning: react-modal: App element is not defined.

hydration-error-info.js:58 Warning: react-modal: App element is not defined. Please use `Modal.setAppElement(el)` or set `appElement={el}`. This is needed so screen readers don't see main content when modal is opened. It is not recommended, but you can opt-out by setting `ariaHideApp={false}`.

```jsx
    <ReactModal
      isOpen={isShow}
      className='modal'
      overlayClassName='overlay'
      **ariaHideApp={false}**
    >
      {children}
    </ReactModal>
```

warning에 나온대로 **`ariaHideApp={false}`**넣어서 warning을 지웠다 (추천하지 않는다곤 했지만…ㅎ)

근데 이런 warning은 왜 뜬걸까

https://reactcommunity.org/react-modal/accessibility/

### 어마어마한 ReactModalPortal

어느 날 그려진 dom을 보니 ReactModalPortal이 너무 많아서 이걸 어떻게 못하나 했는데

![스크린샷 2024-12-09 오후 5.33.39.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/f0ac6472-358c-4493-a319-6a32188abbf8/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.33.39.png)

https://github.com/reactjs/react-modal/issues/760

`This is not an issue.`

라고 한다.
