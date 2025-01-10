---
layout: post
title: "[24.12.03] 높이가 조정되는 textarea, Input 줄바꿈 이벤트, ref를 props로, 원하는 요소로 스크롤 내리기"
date: 2024-12-03 00:00:00 +0900
categories: memo
---

- **내용에 따라 높이가 조정되는 textarea 만들기**

이거야 말로 개 노다가를 했다.

```jsx
useEffect(() => {
  const scrollHeight = textareaRef.current?.scrollHeight;
  const clientHeight = textareaRef.current?.clientHeight;
  if (!scrollHeight || !clientHeight) return;
  if (clientHeight == scrollHeight) setTextareaHeight(scrollHeight - 27);
  else if (scrollHeight <= 44) setTextareaHeight(44);
  else if (scrollHeight <= 70) setTextareaHeight(70);
  else if (scrollHeight <= 97) setTextareaHeight(97);
  else setTextareaHeight(124);
}, [textareaRef.current?.scrollHeight]);
```

텍스트 높이에 따라 4줄까지는 자동으로 Textarea의 높이가 늘어나도록 만들어야 했다.

**원래 쓰려했던 방식:** 똑같은 Textarea를 만들어서 숨겨놓고 숨겨놓은 TEXTAREA의 높이를 재서 보여지는 Textarea의 Height에 반영하는 방식 (구글링하면 제일 많이 나오는) → 아무래도 복사한 Textarea를 dom의 어딘가엔 놓아야 하니 내키진 않았다.

그리하여 **하드코딩**하기로 했다.

textarea의 scrollHeight = 스크롤로 숨겨진 부분까지 포함하는 height

textarea의 clientHeight = 화면에서만 보이는 height

44, 70, 97, 124는 1줄, 2줄, 3줄, 4줄일때의 textarea의 높이다.

원래는 scrollHeight - clientHeight 한 값을 height로 넣으려고 했는데, 줄이 줄어든 것도 아닌데, 텍스트를 지울때마다 이상하게 조금씩 textarea가 줄어들어서 고정시키기로 했다.

`if (clientHeight == scrollHeight) setTextareaHeight(scrollHeight - 27);`

이 부분이 없으면 textarea의 줄이 줄어들었을 때 반영이 안 돼서 (clientHeight가 늘어난 상태니까) 줄이 줄어들때에는 반대로 한 단계씩 빼줬다.

- input 태그에서 ENTER 키 누를 시 이벤트 발생시키기

https://stackoverflow.com/questions/6014702/how-do-i-detect-shiftenter-and-generate-a-new-line-in-textarea

```jsx
const onKeyDownEnter = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
  if (e.key === "Enter" && !e.shiftKey) {
    onClickSubmitButton();
  }
};

return;
<TextInput
  ref={textareaRef}
  placeholder={placeholder}
  value={text}
  onChange={(e) => {
    setText(e.target.value);
  }}
  onKeyDown={(e) => onKeyDownEnter(e)}
  height={textareaHeight}
  disabled={disabled}
/>;
```

- white-space

```jsx
white-space: pre-wrap;
```

https://developer.mozilla.org/en-US/docs/Web/CSS/white-space

- **리액트 ref를 props로 전달하기**
  - forwardRef로 전달하기 https://ko.react.dev/reference/react/forwardRef
  - ref가 아닌 다른 이름으로 전달하기 ✅

귀찮아서 두 번째 방식으로 했다.

- **원하는 요소로 스크롤 내리기**

채팅 기능을 만들고 있는데 메세지가 새로 생성되면 그 메세지로 자동으로 스크롤이 갔으면 했다.

**첫번째 방법**

https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollTo

```jsx
const chatRef = useRef < HTMLDivElement > null;
const questionRef = useRef < HTMLDivElement > null;

useEffect(() => {
  if (!questionRef.current?.offsetTop) return;
  if (!questionRef.current?.offsetTop || !chatRef.current) return;
  const offsetTop = questionRef.current.offsetTop;
  chatRef.current.scrollTo({ top: offsetTop, behavior: "smooth" });
}, [currQuestion]);
```

useRef를 사용하여 스크롤이 향했으면 하는 dom 요소를 담아주고 scrollTo를 통해 해당 요소의 offsetTop(해당 요소의 Y축값을 가져옴) 으로 스크롤을 이동시킨다.

하지만 위의 방식은 offsetTop이 기준을 어디로 두느냐에 따라 달라진다. offsetTop은 offsetParent를 기준으로 계산하는데 offsetParent는 `a [containing block](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block#identifying_the_containing_block) for absolutely-positioned elements`를 의미하는 것으로 position이 정해져있는 (absolute나 relative…) 요소를 의미한다.

고로 저걸 사용하려면 스크롤이 페이지 자체에 생긴 경우, 혹은 스크롤이 생긴 container의 position이 absolute거나 relative 등이면 된다.

이거 공부하는 과정에서 scrollTop과 offsetTop, clientTop이 계속 헷갈려서 직접 만들어서 확인했다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/f109f719-d9c9-4365-8982-3a9f2772d940/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/ed01865c-545a-492b-85c6-8f92dccf3f5f/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/cc40771f-5e6e-4768-81b2-d43ad1207bca/image.png)

**두번째 방법**

사실 맨 처음에 쓰려했던 메소드였는데, 스크롤이 가장 가까운 부모뿐만 아니라 루트의 스크롤도 내려버리는 바람에 곤란했었다.

https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView

```jsx
  if (answerRef.current)
      answerRef.current?.scrollIntoView({
        behavior: 'smooth',
        block: 'nearest',
        inline: 'nearest',
      });
  }, [currQuestion, currAnswer]);

```

근데 nearest 를 넣어주니까 잘되었다. 다만 nearest는 이동할 때 요소의 모서리 중 더 가까운 곳으로 스크롤하겠다는 의미라 루트의 스크롤을 제어하는 옵션은 아닐텐데 왜 안 움직였는지 모르겠다. 다시 테스트 해봐야 할 듯… (항상 이렇게 회고하고 나서야 이상한 걸 깨닫는다)

어쨌든 내가 원한 건 질문-답변이 새로 생성되었을 때 해당 질문으로 스크롤하는 기능이었던 터라 잘 작동했다.

**📗 짧은 일기**

요즘 코딩하면서 재밌는 것: 디자인대로 ui 짜기, 코드 예쁘게 정리하기(남의 눈에도 예쁜지는 모름), 이벤트 연결하기

재미없는 것: 구현 잘 안되는 부분 (구글링 해도 잘 안 나오는) 4시간 넘게 잡고 있기… 별 거 아닌 것 같은데 잘 안 풀리면 그것만큼 열받는 게 없다.
