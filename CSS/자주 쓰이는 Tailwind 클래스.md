# 자주 쓰이는 Tailwind CSS 클래스

## 레이아웃

### 디스플레이 (Display)

| Tailwind       | CSS                      | 설명                          |
| -------------- | ------------------------ | ----------------------------- |
| `flex`         | `display: flex;`         | 플렉스 컨테이너로 설정        |
| `inline-flex`  | `display: inline-flex;`  | 인라인 플렉스 컨테이너로 설정 |
| `grid`         | `display: grid;`         | 그리드 컨테이너로 설정        |
| `block`        | `display: block;`        | 블록 레벨 요소로 설정         |
| `inline-block` | `display: inline-block;` | 인라인 블록 요소로 설정       |
| `inline`       | `display: inline;`       | 인라인 요소로 설정            |
| `hidden`       | `display: none;`         | 요소를 숨김                   |

### 포지셔닝 (Position)

| Tailwind   | CSS                   | 설명                            |
| ---------- | --------------------- | ------------------------------- |
| `static`   | `position: static;`   | 기본 위치                       |
| `fixed`    | `position: fixed;`    | 뷰포트 기준 고정 위치           |
| `absolute` | `position: absolute;` | 가장 가까운 상대 위치 요소 기준 |
| `relative` | `position: relative;` | 원래 위치 기준                  |
| `sticky`   | `position: sticky;`   | 스크롤 시 고정 위치             |

### 플렉스 (Flex)

| Tailwind          | CSS                               | 설명                |
| ----------------- | --------------------------------- | ------------------- |
| `flex-row`        | `flex-direction: row;`            | 행 방향 배치        |
| `flex-col`        | `flex-direction: column;`         | 열 방향 배치        |
| `flex-wrap`       | `flex-wrap: wrap;`                | 여러 줄에 걸쳐 배치 |
| `flex-nowrap`     | `flex-wrap: nowrap;`              | 한 줄에 배치        |
| `justify-start`   | `justify-content: flex-start;`    | 주축 시작점 정렬    |
| `justify-center`  | `justify-content: center;`        | 주축 중앙 정렬      |
| `justify-end`     | `justify-content: flex-end;`      | 주축 끝점 정렬      |
| `justify-between` | `justify-content: space-between;` | 주축 양끝 정렬      |
| `items-start`     | `align-items: flex-start;`        | 교차축 시작점 정렬  |
| `items-center`    | `align-items: center;`            | 교차축 중앙 정렬    |
| `items-end`       | `align-items: flex-end;`          | 교차축 끝점 정렬    |

### 간격 (Spacing)

| Tailwind    | CSS                                        | 설명                   |
| ----------- | ------------------------------------------ | ---------------------- |
| `p-4`       | `padding: 1rem;`                           | 모든 방향 패딩 1rem    |
| `px-4`      | `padding-left: 1rem; padding-right: 1rem;` | 좌우 패딩 1rem         |
| `py-4`      | `padding-top: 1rem; padding-bottom: 1rem;` | 상하 패딩 1rem         |
| `pt-4`      | `padding-top: 1rem;`                       | 상단 패딩 1rem         |
| `m-4`       | `margin: 1rem;`                            | 모든 방향 마진 1rem    |
| `mx-4`      | `margin-left: 1rem; margin-right: 1rem;`   | 좌우 마진 1rem         |
| `my-4`      | `margin-top: 1rem; margin-bottom: 1rem;`   | 상하 마진 1rem         |
| `mt-4`      | `margin-top: 1rem;`                        | 상단 마진 1rem         |
| `-mx-4`     | `margin-left: -1rem; margin-right: -1rem;` | 음수 마진 (좌우)       |
| `space-x-4` | `margin-left: 1rem;` (2번째 요소부터)      | 형제 요소 간 수평 간격 |
| `space-y-4` | `margin-top: 1rem;` (2번째 요소부터)       | 형제 요소 간 수직 간격 |

## 크기 (Sizing)

### 너비 (Width)

| Tailwind   | CSS                  | 설명        |
| ---------- | -------------------- | ----------- |
| `w-full`   | `width: 100%;`       | 전체 너비   |
| `w-screen` | `width: 100vw;`      | 뷰포트 너비 |
| `w-auto`   | `width: auto;`       | 자동 너비   |
| `w-1/2`    | `width: 50%;`        | 절반 너비   |
| `w-1/3`    | `width: 33.333333%;` | 1/3 너비    |
| `w-2/3`    | `width: 66.666667%;` | 2/3 너비    |
| `w-1/4`    | `width: 25%;`        | 1/4 너비    |
| `w-3/4`    | `width: 75%;`        | 3/4 너비    |
| `w-4`      | `width: 1rem;`       | 1rem 너비   |

### 높이 (Height)

| Tailwind   | CSS              | 설명        |
| ---------- | ---------------- | ----------- |
| `h-full`   | `height: 100%;`  | 전체 높이   |
| `h-screen` | `height: 100vh;` | 뷰포트 높이 |
| `h-auto`   | `height: auto;`  | 자동 높이   |
| `h-4`      | `height: 1rem;`  | 1rem 높이   |

### 최대/최소 크기

| Tailwind     | CSS                 | 설명            |
| ------------ | ------------------- | --------------- |
| `max-w-sm`   | `max-width: 24rem;` | 최대 너비 24rem |
| `max-w-md`   | `max-width: 28rem;` | 최대 너비 28rem |
| `max-w-lg`   | `max-width: 32rem;` | 최대 너비 32rem |
| `max-w-full` | `max-width: 100%;`  | 최대 너비 100%  |
| `min-h-0`    | `min-height: 0px;`  | 최소 높이 0     |
| `min-w-0`    | `min-width: 0px;`   | 최소 너비 0     |

## 타이포그래피

### 글꼴 크기 (Font Size)

| Tailwind    | CSS                                          | 설명             |
| ----------- | -------------------------------------------- | ---------------- |
| `text-xs`   | `font-size: 0.75rem; line-height: 1rem;`     | 아주 작은 텍스트 |
| `text-sm`   | `font-size: 0.875rem; line-height: 1.25rem;` | 작은 텍스트      |
| `text-base` | `font-size: 1rem; line-height: 1.5rem;`      | 기본 텍스트      |
| `text-lg`   | `font-size: 1.125rem; line-height: 1.75rem;` | 큰 텍스트        |
| `text-xl`   | `font-size: 1.25rem; line-height: 1.75rem;`  | 더 큰 텍스트     |
| `text-2xl`  | `font-size: 1.5rem; line-height: 2rem;`      | 매우 큰 텍스트   |

### 글꼴 두께 (Font Weight)

| Tailwind        | CSS                 | 설명             |
| --------------- | ------------------- | ---------------- |
| `font-thin`     | `font-weight: 100;` | 가는 글꼴        |
| `font-normal`   | `font-weight: 400;` | 일반 글꼴        |
| `font-medium`   | `font-weight: 500;` | 중간 두께 글꼴   |
| `font-semibold` | `font-weight: 600;` | 약간 두꺼운 글꼴 |
| `font-bold`     | `font-weight: 700;` | 두꺼운 글꼴      |

### 텍스트 정렬 (Text Align)

| Tailwind       | CSS                    | 설명        |
| -------------- | ---------------------- | ----------- |
| `text-left`    | `text-align: left;`    | 왼쪽 정렬   |
| `text-center`  | `text-align: center;`  | 중앙 정렬   |
| `text-right`   | `text-align: right;`   | 오른쪽 정렬 |
| `text-justify` | `text-align: justify;` | 양쪽 정렬   |

### 행간 (Line Height)

| Tailwind         | CSS                  | 설명      |
| ---------------- | -------------------- | --------- |
| `leading-none`   | `line-height: 1;`    | 행간 없음 |
| `leading-tight`  | `line-height: 1.25;` | 좁은 행간 |
| `leading-normal` | `line-height: 1.5;`  | 보통 행간 |
| `leading-loose`  | `line-height: 2;`    | 넓은 행간 |

## 배경 (Background)

### 배경색 (Background Color)

| Tailwind         | CSS                                   | 설명              |
| ---------------- | ------------------------------------- | ----------------- |
| `bg-white`       | `background-color: rgb(255 255 255);` | 흰색 배경         |
| `bg-black`       | `background-color: rgb(0 0 0);`       | 검은색 배경       |
| `bg-red-500`     | `background-color: rgb(239 68 68);`   | 빨간색 배경 (500) |
| `bg-blue-500`    | `background-color: rgb(59 130 246);`  | 파란색 배경 (500) |
| `bg-green-500`   | `background-color: rgb(34 197 94);`   | 초록색 배경 (500) |
| `bg-transparent` | `background-color: transparent;`      | 투명 배경         |

### 배경 위치 (Background Position)

| Tailwind    | CSS                            | 설명      |
| ----------- | ------------------------------ | --------- |
| `bg-center` | `background-position: center;` | 중앙 위치 |
| `bg-top`    | `background-position: top;`    | 상단 위치 |
| `bg-bottom` | `background-position: bottom;` | 하단 위치 |

### 배경 반복 (Background Repeat)

| Tailwind       | CSS                             | 설명           |
| -------------- | ------------------------------- | -------------- |
| `bg-repeat`    | `background-repeat: repeat;`    | 배경 반복      |
| `bg-no-repeat` | `background-repeat: no-repeat;` | 배경 반복 없음 |
| `bg-repeat-x`  | `background-repeat: repeat-x;`  | 가로 방향 반복 |

### 배경 크기 (Background Size)

| Tailwind     | CSS                         | 설명                      |
| ------------ | --------------------------- | ------------------------- |
| `bg-cover`   | `background-size: cover;`   | 컨테이너 채우기           |
| `bg-contain` | `background-size: contain;` | 컨테이너 내에 완전히 표시 |
| `bg-auto`    | `background-size: auto;`    | 원본 크기                 |

## 테두리 (Border)

### 테두리 너비 (Border Width)

| Tailwind   | CSS                         | 설명                 |
| ---------- | --------------------------- | -------------------- |
| `border`   | `border-width: 1px;`        | 모든 방향 1px 테두리 |
| `border-0` | `border-width: 0px;`        | 테두리 없음          |
| `border-2` | `border-width: 2px;`        | 모든 방향 2px 테두리 |
| `border-t` | `border-top-width: 1px;`    | 상단 테두리          |
| `border-r` | `border-right-width: 1px;`  | 우측 테두리          |
| `border-b` | `border-bottom-width: 1px;` | 하단 테두리          |
| `border-l` | `border-left-width: 1px;`   | 좌측 테두리          |

### 테두리 색상 (Border Color)

| Tailwind             | CSS                               | 설명          |
| -------------------- | --------------------------------- | ------------- |
| `border-transparent` | `border-color: transparent;`      | 투명 테두리   |
| `border-black`       | `border-color: rgb(0 0 0);`       | 검은색 테두리 |
| `border-white`       | `border-color: rgb(255 255 255);` | 흰색 테두리   |
| `border-red-500`     | `border-color: rgb(239 68 68);`   | 빨간색 테두리 |
| `border-blue-500`    | `border-color: rgb(59 130 246);`  | 파란색 테두리 |

### 테두리 스타일 (Border Style)

| Tailwind        | CSS                     | 설명               |
| --------------- | ----------------------- | ------------------ |
| `border-solid`  | `border-style: solid;`  | 실선 테두리        |
| `border-dashed` | `border-style: dashed;` | 파선 테두리        |
| `border-dotted` | `border-style: dotted;` | 점선 테두리        |
| `border-none`   | `border-style: none;`   | 테두리 스타일 없음 |

### 테두리 반경 (Border Radius)

| Tailwind       | CSS                                                                  | 설명                 |
| -------------- | -------------------------------------------------------------------- | -------------------- |
| `rounded-none` | `border-radius: 0px;`                                                | 각진 모서리          |
| `rounded`      | `border-radius: 0.25rem;`                                            | 기본 둥근 모서리     |
| `rounded-md`   | `border-radius: 0.375rem;`                                           | 중간 둥근 모서리     |
| `rounded-lg`   | `border-radius: 0.5rem;`                                             | 큰 둥근 모서리       |
| `rounded-full` | `border-radius: 9999px;`                                             | 완전히 둥근 모서리   |
| `rounded-t`    | `border-top-left-radius: 0.25rem; border-top-right-radius: 0.25rem;` | 상단 모서리만 둥글게 |

## 그림자 (Shadow)

| Tailwind      | CSS                                                                               | 설명        |
| ------------- | --------------------------------------------------------------------------------- | ----------- |
| `shadow-sm`   | `box-shadow: 0 1px 2px 0 rgb(0 0 0 / 0.05);`                                      | 작은 그림자 |
| `shadow`      | `box-shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);`      | 기본 그림자 |
| `shadow-md`   | `box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);`   | 중간 그림자 |
| `shadow-lg`   | `box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);` | 큰 그림자   |
| `shadow-none` | `box-shadow: 0 0 #0000;`                                                          | 그림자 없음 |

## 효과 (Effects)

### 불투명도 (Opacity)

| Tailwind      | CSS             | 설명      |
| ------------- | --------------- | --------- |
| `opacity-0`   | `opacity: 0;`   | 완전 투명 |
| `opacity-50`  | `opacity: 0.5;` | 반투명    |
| `opacity-100` | `opacity: 1;`   | 불투명    |

### 투명도 (Transition)

| Tailwind            | CSS                                                                                                                                                                                      | 설명            |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| `transition`        | `transition-property: all; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms;`                                                                        | 기본 전환       |
| `transition-colors` | `transition-property: color, background-color, border-color, text-decoration-color, fill, stroke; transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); transition-duration: 150ms;` | 색상 전환       |
| `duration-300`      | `transition-duration: 300ms;`                                                                                                                                                            | 0.3초 지속 시간 |
| `ease-in`           | `transition-timing-function: cubic-bezier(0.4, 0, 1, 1);`                                                                                                                                | 천천히 시작     |
| `ease-out`          | `transition-timing-function: cubic-bezier(0, 0, 0.2, 1);`                                                                                                                                | 천천히 끝냄     |

### 변환 (Transform)

| Tailwind        | CSS                            | 설명              |
| --------------- | ------------------------------ | ----------------- |
| `scale-50`      | `transform: scale(.5);`        | 50% 크기로 축소   |
| `scale-100`     | `transform: scale(1);`         | 원래 크기         |
| `scale-150`     | `transform: scale(1.5);`       | 150% 크기로 확대  |
| `rotate-45`     | `transform: rotate(45deg);`    | 45도 회전         |
| `translate-x-4` | `transform: translateX(1rem);` | X축으로 1rem 이동 |
| `translate-y-4` | `transform: translateY(1rem);` | Y축으로 1rem 이동 |

## 인터랙티브 (Interactive)

### 커서 (Cursor)

| Tailwind             | CSS                    | 설명        |
| -------------------- | ---------------------- | ----------- |
| `cursor-auto`        | `cursor: auto;`        | 자동 커서   |
| `cursor-default`     | `cursor: default;`     | 기본 커서   |
| `cursor-pointer`     | `cursor: pointer;`     | 포인터 커서 |
| `cursor-wait`        | `cursor: wait;`        | 대기 커서   |
| `cursor-not-allowed` | `cursor: not-allowed;` | 금지 커서   |

### 포인터 이벤트 (Pointer Events)

| Tailwind              | CSS                     | 설명               |
| --------------------- | ----------------------- | ------------------ |
| `pointer-events-none` | `pointer-events: none;` | 포인터 이벤트 무시 |
| `pointer-events-auto` | `pointer-events: auto;` | 기본 포인터 이벤트 |

### 사용자 선택 (User Select)

| Tailwind      | CSS                  | 설명                     |
| ------------- | -------------------- | ------------------------ |
| `select-none` | `user-select: none;` | 텍스트 선택 불가         |
| `select-text` | `user-select: text;` | 텍스트 선택 가능         |
| `select-all`  | `user-select: all;`  | 한 번에 모든 텍스트 선택 |

## 반응형 (Responsive)

모든 Tailwind 클래스는 다음 반응형 접두사와 함께 사용할 수 있습니다:

| 접두사 | 기본 중단점 | 설명              |
| ------ | ----------- | ----------------- |
| `sm:`  | 640px       | 작은 화면 이상    |
| `md:`  | 768px       | 중간 화면 이상    |
| `lg:`  | 1024px      | 큰 화면 이상      |
| `xl:`  | 1280px      | 더 큰 화면 이상   |
| `2xl:` | 1536px      | 매우 큰 화면 이상 |

예) `md:flex` → 중간 화면 이상에서 `display: flex;` 적용

## 유틸리티

### 오버플로우 (Overflow)

| Tailwind           | CSS                  | 설명                      |
| ------------------ | -------------------- | ------------------------- |
| `overflow-auto`    | `overflow: auto;`    | 필요시 스크롤바 표시      |
| `overflow-hidden`  | `overflow: hidden;`  | 넘치는 내용 숨김          |
| `overflow-visible` | `overflow: visible;` | 넘치는 내용 표시          |
| `overflow-scroll`  | `overflow: scroll;`  | 항상 스크롤바 표시        |
| `overflow-x-auto`  | `overflow-x: auto;`  | 가로 방향 필요시 스크롤바 |
| `overflow-y-auto`  | `overflow-y: auto;`  | 세로 방향 필요시 스크롤바 |

### Z-인덱스 (Z-Index)

| Tailwind | CSS              | 설명          |
| -------- | ---------------- | ------------- |
| `z-0`    | `z-index: 0;`    | z-인덱스 0    |
| `z-10`   | `z-index: 10;`   | z-인덱스 10   |
| `z-50`   | `z-index: 50;`   | z-인덱스 50   |
| `z-auto` | `z-index: auto;` | 자동 z-인덱스 |
