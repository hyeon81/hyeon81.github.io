---
layout: post
title: "[25.01.23] fetchEventSource에서 abortController가 안 먹힘, useEffect의 언마운트 시점"
date: 2025-01-23 00:00:00 +0900
categories: memo
---

- fetchEventSource에서 abortController로 abort()를 하려했는데 정상적인 응답에서는 잘 멈췄지만 정상적이지 않은 (정확히 말하면 서버가 처리를 제대로 안해서 무한 루프를 돌던 응답…) 응답에선 멈추지 않아서 너무 공포스러웠다. (아무리 abort를 해도 멈추지 않아…)

  - 서버에서 무한루프를 돌때에는 event 타입을 error로 주겠다고 그래서 throw new Error()를 던졌다. (try-catch로 감싸주는 걸 잊지말자)

  ```jsx
  onMessage: (e) => {
            if (e) {
              const { data, event } = e;
              if (event == 'error' || event == 'recursion_error') {
                setAnswerStatus(AnswerStatus.ERROR);
                throw new Error('error');
  	           ...
            }
          },
  ```

- https://github.com/Azure/fetch-event-source/issues/24

- useEffect의 언마운트 시점
  - cleanup 함수로 특정 페이지를 벗어났을 때 session storage에 저장해둔 데이터를 지우는 코드를 넣었는데, 이상하게 새로고침 할 때는 작동되지 않았다.
  - 새로고침하면 dom에서 해당 컴포넌트가 제거되니까 당연히 cleanup 함수가 작동되어야 하는 거 아닌가? 🧐 생각했는데, 새로고침을 하게 되면 react 애플리케이션이 뭘 하기도 전에 초기화되기 때문에 실행되지 않는 게 맞는듯… (어찌보면 당연)
  - 어차피 탭닫으면 session storage 값은 사라지니 그냥 놔두기로 했다. 꼭 지워야 한다면 beforeunload를 사용해도 될 것 같다.

📝 짧은 일기

- useSWR에 fetcher에 argument를 넘겨줄때, 타입 설정하기 힘들다. 퇴근할 시간이라 타입은 못 고치도 퇴근했는데 내일 다시 봐야겠다.
- https://velog.io/@na_0_i/useState-%EC%A7%80%EC%98%A5%EC%97%90%EC%84%9C-%EB%B2%97%EC%96%B4%EB%82%98%EA%B8%B0with-useReducer
- 빨리 작업하려다보니 useState를 남발하게 되는데, 너무 많은 상태가 끼어들다보니 나중가선 너무 머리가 아팠다. useState를 줄여봐야겠다.
- 백엔드에서 주는 값을 어떻게 하면 예쁘게 가공할 수 있을까...? 솔직한 마음으로는 백엔드에서 좀 더 정제된 데이터를 던져줬으면 좋겠다... (입사 6개월차가 마음속으로 생각했다.)
- 매일매일 TIL 꼼꼼하게 기록하는 사람은 뭐하는 사람일까? 난 이렇게 잡스러운(?) 내용만 적는데도 시간이 금방 지나가서 운동 가야한다...
