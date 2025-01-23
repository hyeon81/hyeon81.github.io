---
layout: post
title: "[25.01.21] dayjs에서 빼기 더하기, Discriminated Union, JS 숫자인지 아닌지 판단하기"
date: 2025-01-21 00:00:00 +0900
categories: memo
---

- dayjs에서 빼기 더하기

  - 빼기 subtract
    - `dayjs().subtract(7, 'year')`
  - 더하기 add
    - `dayjs().add(7, 'year')`

- **Discriminated Union**

  - 같은 타입 안에서도 특정 프로퍼티 값에 따라 세부 타입이 조금씩 달라지는 경우가 있는데 그럴 때 쓰면 유용하다

  ```jsx
  interface Square {
    kind: "square";
    size: number;
  }

  interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
  }
  type Shape = Square | Rectangle;
  ```

  - https://radlohead.gitbook.io/typescript-deep-dive/type-system/discriminated-unions

- 숫자인지 아닌지 판단하기

```jsx
!isNaN(Number(변수));
```

---

**_🖊️_ 짧은 일기**

한동안 위염과 감기 콜라보로 TIL는 커녕 TIL를 위한 메모도 하지 않았다. 생각해보니 원래 TIL를 쓰기 시작한 이유가 내 깃헙의 잔디가 너무 비어있길래 그걸 채우고 싶기 때문이었는데, TIL도 밀려 업로드를 못하니 잔디도 안심기는 불상사가 생겼다. 😢 밀린 TIL는 주말에 쓰고 오늘 TIL부터 제대로 써서 올려야겠다 (그래봤자 내일 회식이라 내일 TIL는 못 올릴 가능성이 크지만…)
