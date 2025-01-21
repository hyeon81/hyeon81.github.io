---
layout: post
title: "[24.12.13] async/await, Nextjs에서 SSE가 작동안함, CSS Animation, keyframe"
date: 2024-12-13 00:00:00 +0900
categories: memo
---

### async/await 어디까지 붙여야 할까

→ 기다릴거면 다 붙이자

https://ko.javascript.info/task/async-from-regular

근데 왜 종종 Promise를 쓸 때 async/await 를 안 붙여도 이상함을 느끼지 못한 때가 있을까…?

안 기다려도 되는 상황이었나보지…

### Nextjs에서 SSE로 이벤트를 받는데, 이벤트가 바로 오는게 아니라 뭉쳐서 온다

분명 서버에서 SSE로 보내주고 있고 나도 SSE로 받는데, 여러 STREAM이 기다렸다가 한번에 오는 문제가 있었다.

처음엔 이게 fetchEventSource 문제인줄 알았는데, 같은 코드를 vite에 넣고 돌렸더니 멀쩡하게 나눠서 왔다.

아무래도 NextJS 문제인 것 같아 찾아보다가,

https://github.com/vercel/next.js/discussions/48427#discussioncomment-6029799

이런 글을 보게 되었다. 이 이슈에서는 NextJS의 api routes에서 나와 똑같이 sse가 작동하지 않는다는 내용이 있었다. 아무래도 Nextjs에서 gzip으로 압축해서 우리에게 응답을 주기 때문인 게 아닐까 싶다.

지금 프로젝트에서는 api route를 사용하고 있지는 않지만, cors 문제를 해결하려고 nextjs의 proxy를 사용하고 있기 때문에 어떻게든 nextjs를 거쳐서 요청과 응답을 주고 받는 구조다.

그래서 config에서 compress 설정을 끄니까 의도한대로 SSE가 실시간으로 왔다. 하지만 compress 설정을 끄는 건 옳지 않다고 생각하기 때문에 다른 방법을 찾아봐야할 것 같다. 제일 먼저 하고 싶은 방법은 proxy를 사용하지 않고 요청을 보내는 건데, 이건 서버에서 처리해줘야한다.

아무래도 신입이다보니까 이런 부분에서 적극적으로 의견을 내기 힘들다… 흑흑 😢

- https://nextjs.org/docs/app/api-reference/config/next-config-js/compress

### CSS Animation, keyframe

로딩 시 세 개의 원이 서서히 밝아졌다 어두어졌다를 반복하는 애니메이션을 만들고 싶었다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/f2d43424-2abc-48e8-8e1b-65b57a3bb542/image.png)

이런 느낌의…

그래서

🔵⚪⚫ ← 컴포넌트1

⚫🔵⚪ ← 컴포넌트2

⚪⚫🔵 ← 컴포넌트3

이렇게 컴포넌트 세 개를 두고 겹친 뒤, animation 효과를 넣어 번갈아가면서 자연스럽게 전환되도록 하여, 원의 색이 자연스럽게 변하는 것처럼 작업했다.

https://developer.mozilla.org/ko/docs/Web/CSS/animation

```
import styled from '@emotion/styled';

const DotSpinner = ({ color }: { color: string[] }) => {
  return (
    <svg width='50' height='50' viewBox='0 0 50 50' fill='none' xmlns='http://www.w3.org/2000/svg'>
      <circle cx='6' cy='25' r='6' fill={color?.[0] ?? '#EAECEF'} />
      <circle cx='44' cy='25' r='6' fill={color?.[1] ?? '#CBCED2'} />
      <circle cx='25' cy='25' r='7' fill={color?.[2] ?? '#848A94'} />
    </svg>
  );
};

export default function LoadingChatAnimation() {
  const color1 = '#EAECEF';
  const color2 = '#CBCED2';
  const color3 = '#848A94';
  const colors = [
    [color1, color2, color3],
    [color2, color3, color1],
    [color3, color1, color2],
  ];

  return (
    <Container>
      <Transition delay='0s'>
        <DotSpinner color={colors[0]} />
      </Transition>
      <Transition delay='0.5s'>
        <DotSpinner color={colors[1]} />
      </Transition>
      <Transition delay='0.5s'>
        <DotSpinner color={colors[2]} />
      </Transition>
    </Container>
  );
}

const Container = styled.div`
  position: relative;
  height: 50px;
`;

const Transition = styled.div<{ delay: string }>`
  position: absolute;
  top: 0;
  left: 0;

  @keyframes fade {
    0% {
      opacity: 0;
    }
    50% {
      opacity: 1;
    }
    100% {
      opacity: 0;
    }
  }

  animation: fade 1.5s infinite ease-in-out;
  animation-delay: ${(props) => props.delay};
`;

```

@keyframe으로 fade 애니메이션을 정의해줬다.

opacity를 조절하여 나타났다가 사라졌다 하는 애니메이션이다.

animation 속성을 사용해 fade 애니메이션이 지속시간 1.5초동안 실행되고 무한반복하도록 했다.

그리고 각 컴포넌트마다 0.5씩 딜레이를 주어 순차적으로 나타나게 만들었다.

사실 그냥 색깔만 바꿨어도 괜찮았을듯… 그리고 css animation은 자주 만져보질 못해서 더 공부를 해야할 것 같다.

- https://webclub.tistory.com/621
