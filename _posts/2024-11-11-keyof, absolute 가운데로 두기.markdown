---
layout: post
title: "[241111] keyof, absolute 가운데로 두기"
date: 2024-11-11 00:00:00 +0900
categories: memo
---

### keyof

https://www.typescriptlang.org/ko/docs/handbook/2/keyof-types.html

객체의 키 값으로만 이루어진 타입을 만들고 싶을 때 사용한다.

객체의 키 값을 매칭하여 라벨을 붙여줘야한다거나 (테이블 같은..) 객체 안의 값만 선택해야할 때 유용하게 사용했다.

### absolute로 배치한 자식을 어떻게 부모의 가운데에 둘까

https://stackoverflow.com/questions/28455100/how-to-center-div-vertically-inside-of-absolutely-positioned-parent-div

나랑 딱 같은 고민을 한 사람을 Stackoverflow에서 발견

```jsx
.valign {
    position: absoulte;
    top: 50%;
    transform: translateY(-50%);
    /* vendor prefixes omitted due to brevity */
}
```

잘 먹혔던 해결법

어떻게 이게 가능할까

```jsx
{
    position: absolute;
    top: 0;
    right: 0;
    height: 100%;
    line-height: 100px;
}
```

그렇지만 난 귀찮아서 그냥 line-height를 부모높이만큼 설정해서 위치시켜버렸다.
