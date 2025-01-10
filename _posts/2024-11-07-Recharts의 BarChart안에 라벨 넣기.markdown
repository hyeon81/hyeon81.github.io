---
layout: post
title: "[241107] Recharts의 BarChart안에 라벨 넣기"
date: 2024-11-07 00:00:00 +0900
categories: memo
---

### Recharts의 BarChart안에 라벨 넣기

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d2d0348d-b394-49c9-b6fe-f9ad02f7f098/2837efcf-e830-40bb-87c5-7272f6993d2d/image.png)

지금 하고 있는 프로젝트에선 차트가 많이 들어가는데, 이를 구현할 때 recharts를 사용하고 있다. 이번엔 차트에 요소가 하나뿐인 bar chart 를 만들어야했는데, 디자인을 보니 차트 안에다가 값을 표기해야 했다. 요소가 하나뿐이니 그냥 css로 그려도 되겠지만, 뭐가 그렇게 어려울까 싶어 recharts로 만들기로 했다.

근데 어려웠다. 어떻게든 recharts를 잘 사용해서 값을 넣어보려고 했는데, 잘 되지 않아서 그냥 상위 컴포넌트에 position을 relatvie로 두고, 값에는 absolute를 넣어서 배치했다.

```jsx
import styled from "@emotion/styled";
import { Box } from "@mui/material";
import React from "react";
import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
} from "recharts";

function SimpleBarChart() {
  return (
    <Box position="relative">
      <ResponsiveContainer
        width="100%"
        height={90}
        style={{
          backgroundColor: "lightgray",
        }}
      >
        <BarChart
          margin={{ top: -2, left: 0, right: 0, bottom: -2 }}
          layout="vertical"
          data={[
            {
              name: "Example",
              value: 44, // 하나의 데이터 포인트만 존재
            },
          ]}
        >
          <XAxis type="number" hide domain={[0, 100]} />
          <YAxis type="category" dataKey="name" hide />
          <Bar dataKey="value" fill="skyblue" radius={[2, 2, 2, 2]} />
        </BarChart>
      </ResponsiveContainer>
      <CostLabel>
        <h1>라벨 입니다 라벨</h1>
      </CostLabel>
    </Box>
  );
}

export default SimpleBarChart;

const CostLabel = styled.div`
  position: absolute;
  top: 0;
  right: 0;
  height: 100%;
  line-height: 100px;
  margin-right: 20px;
`;
```

### **CSS Property를 Props로 받을 때 type**

React.CSSProperties

**[참고할만한 링크]**

https://ko.legacy.reactjs.org/docs/faq-styling.html
