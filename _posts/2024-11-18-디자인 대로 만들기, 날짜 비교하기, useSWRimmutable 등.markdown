---
layout: post
title: "[24.11.18-24.11.22] 디자인 대로 만들기, 날짜 비교하기, useSWRimmutable 등"
date: 2024-11-18 00:00:00 +0900
categories: memo
---

22일까지 마감인 프로젝트에 18일에 투입되었다. 정말 일주일 내내 정신없이 코딩했다.

### 디자인 대로 차트 만들기

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/1219094b-a561-482f-a93a-ccabf474aa16/image.png)

```jsx
import React from "react";
import { BarChart, Bar, ResponsiveContainer, XAxis, LabelList } from "recharts";

const data = [
  {
    name: "2020",
    uv: 10000,
  },
  {
    name: "2021",
    uv: 3000,
  },
  {
    name: "2022",
    uv: 2000,
  },
  {
    name: "2023",
    uv: 2780,
  },
  {
    name: "2024",
    uv: 1890,
  },
];

export default function CostBarChart() {
  return (
    <ResponsiveContainer width="100%" height="100%">
      <BarChart data={data}>
        <XAxis dataKey="name" stroke="0" fontSize={FontSize.body2} />
        <Bar dataKey="uv" fill="#2F70D7" radius={[4, 4, 0, 0]} barSize={32} />
      </BarChart>
    </ResponsiveContainer>
  );
}
```

### 디자인 대로 달력 만들기

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/64a72295-e4a4-4c0b-869d-8ef946f8bb4d/image.png)

대충 이런 느낌의 달력을 만들어야 했는데, 이미 달력은 만들어져 있었고 내가 스타일만 고치면 됐었다. 달력은 react-datepicker를 사용하고 있었는데 마음대로 스타일링이 되지 않아서 힘들었다.

그래도 색깔 커스텀은 얼추 했다.

```jsx

  const CustomInput = forwardRef(({ value, onClick, className }: any, ref: any) => (
    <DateInput
      placeholder='조회 기간을 선택해 주세요'
      onClick={onClick}
      ref={ref}
      defaultValue={value}
    />
  ));

  CustomInput.displayName = 'Calendar';

  const getDayClassName = (d: any) => {
    const start = startDate?.setHours(0, 0, 0, 0);
    const end = endDate?.setHours(0, 0, 0, 0);
    const date = d.setHours(0, 0, 0, 0);
    if (start == date || end == date) return 'selectedDay';
    else if (start && start <= date && end && end >= date) return 'betweenDay';
    return 'unselectedDay';
  };

  return (
    <div css={calendarStyle}>
      <DatePicker
        ref={datePickerRef}
        selectsRange={true}
        startDate={startDate === null ? undefined : startDate}
        endDate={endDate === null ? undefined : endDate}
        onChange={(update) => {
          setDateRange(update);
        }}
        onCalendarClose={() => {
          if (!endDate) setDateRange([startDate, startDate]);
        }}
        dayClassName={getDayClassName}
        customInput={<CustomInput />}
        renderCustomHeader={CustomHeader}
        icon={
          <svg
            width='24'
            height='24'
            viewBox='0 0 24 24'
            fill='none'
            xmlns='http://www.w3.org/2000/svg'
          >
            <rect x='3' y='4' width='18' height='17' rx='2' stroke='#848A94' strokeWidth='1.5' />
            <path
              d='M9 14.2222L11.5 17L15 12'
              stroke='#848A94'
              strokeWidth='1.5'
              strokeLinecap='round'
              strokeLinejoin='round'
            />
            <path
              d='M3 6C3 4.89543 3.89543 4 5 4H19C20.1046 4 21 4.89543 21 6V8H3V6Z'
              fill='#848A94'
            />
            <rect x='16' y='2' width='2' height='4' rx='1' fill='#848A94' />
            <rect x='6' y='2' width='2' height='4' rx='1' fill='#848A94' />
          </svg>
        }
        toggleCalendarOnIconClick
        locale={ko}
        dateFormat='yyyy.MM.dd'
        shouldCloseOnSelect={false}
        showIcon
        withPortal
      >
        <FlexContainer>
          <Text>{startDate && getKorDate(startDate)}</TextDiv>
          <Text>{endDate && ` ~ ${getKorDate(endDate)}`}</TextDiv>
        </FlexContainer>
        <ButtonContainer>
          <MainButton
            titleText={'확인'}
            isActive={true}
            onClick={closeCalendar}
          />
        </ButtonContainer>
      </DatePicker>
    </div>
  );
}

```

하지만 문제는 range 설정 시 선택된 range의 날짜들을 한 줄로 묶어서 색칠해주는 것이었다. 날짜 셀마다 gap이 서로 있었는데, 그 gap까지도 색칠을 해줘야했기 때문에… 결국 시간이 모자라서 나중에 작업하기로 했다.

chatgpt한테 물어보니
가상 요소(`::before`, `::after`)를 사용한 확장이나 음수 마진을 사용해보라는데 나중에 한번 다시 시도해봐야지

### 날짜 비교 코드

```jsx
function isDateInRange(date, startDate, endDate) {
  const targetDate = new Date(date).setHours(0, 0, 0, 0); // 비교를 위해 시간 부분 제거
  const start = new Date(startDate).setHours(0, 0, 0, 0);
  const end = new Date(endDate).setHours(0, 0, 0, 0);

  return targetDate >= start && targetDate <= end;
}

function isDateEqual(date, comparisonDate) {
  const targetDate = new Date(date).setHours(0, 0, 0, 0); // 시간 제거
  const compareDate = new Date(comparisonDate).setHours(0, 0, 0, 0);

  return targetDate === compareDate;
}

// 테스트
const testDate = "2024-11-22";
const rangeStart = "2024-11-20";
const rangeEnd = "2024-11-25";
const comparisonDate = "2024-11-22";

console.log(isDateInRange(testDate, rangeStart, rangeEnd)); // true
console.log(isDateEqual(testDate, comparisonDate)); // true
```

### sort로 날짜 순으로 정렬하기

```jsx
const customerList = useMemo(() => {
  const sortedList = [...filteredList]; // 원본 배열 복사
  const order = sortType === SORT_TYPE.OLDEST ? -1 : 1; // 정렬 방향 결정
  return sortedList.sort((a, b) =>
    dayjs(a.evaluation_date).isAfter(b.evaluation_date)
      ? order
      : dayjs(a.evaluation_date).isBefore(b.evaluation_date)
      ? -order
      : 0
  );
}, [sortType, filteredList]); // 의존성 배열에 filteredList 포함\
```

### useSWRimmutable

작업하다가 보인 useSWRimmutable. 뭐하는 친구일까

https://swr.vercel.app/docs/revalidation

useSWR은 페이지에 다시 초점을 맞추거나 재연결을 하거나 하면 자동으로 다시 재검증 한다 → 그걸 비활성화하는 게 `useSWRimmutable.`

리소스가 변경 불가능하다면 다시 검증하더라도 절대 변경되지 않게 하는 훅이다.

```jsx
useSWR(key, fetcher, {
  revalidateIfStale: false,
  revalidateOnFocus: false,
  revalidateOnReconnect: false,
});

// equivalent to
useSWRImmutable(key, fetcher);
```

그렇다고 한다.

왜 쓰는지는 알겠지만 (받아오는 데이터가 변할일이 거의 없었다) 작업하다보니 내가 하려던 작업엔 맞지 않아 (같은 api로 검색도 해야하고 필터링도 해야했음) 조용히 다시 useSWR로 바꾸었다…ㅎ

### You provided a `value` prop to a form field without an `onChange` handler

- https://stackoverflow.com/questions/43556212/failed-form-proptype-you-provided-a-value-prop-to-a-form-field-without-an-on

form 요소인 input의 value를 onChange 없이 다루지 말라는 얘기인 것 같다. 사용자 입력값을 다루지 않으려면 defaultValue를 쓰고 사용자 입력값을 다루려면 onChange를 함께 쓰자

```jsx
<input
  type="text"
  placeholder="Search..."
  value={keyword}
  onChange={(event) => inputChangedHandler(event)}
/>
```

### git remote update를 했는데 아무것도 업데이트 되지 않아

결국 원인을 찾지 못하고 다른 팀원분이 올려두신 커밋 아이디로 업데이트 했다…
