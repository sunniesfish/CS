# React Query와 Hydration

## RSC (React Server Components)

### 개요

- 서버에서 데이터 미리 패칭
- 렌더링된 HTML을 클라이언트로 전달
- 클라이언트는 HTML 표시 후 상호작용 가능

## initialData vs Hydrate

### initialData

- useQuery의 첫 렌더링 시에만 초기값 설정
- 서버 데이터를 초기 상태로 설정
- 추가 네트워크 요청으로 데이터 갱신

### Hydrate

- 서버에서 패칭한 데이터를 클라이언트로 전달
- 전역 쿼리 캐시에 데이터 복원
- 초기 렌더링 시 즉시 데이터 사용

## 렌더링 타이밍 및 데이터 흐름

### initialData 방식

1. 서버 데이터 패칭
   - 데이터 직렬화하여 클라이언트 전달
2. 첫 번째 렌더링
   - initialData로 서버 데이터 설정
   - 추가 네트워크 요청 없이 렌더링
3. 네트워크 요청
   - 렌더링 후 서버에 요청하여 데이터 갱신
4. 데이터 갱신
   - 클라이언트 데이터 변경 시 서버 요청

### Hydrate 방식

1. 서버 데이터 패칭
   - dehydrate로 데이터 직렬화
   - 클라이언트로 전달
2. 첫 번째 렌더링
   - Hydrate로 쿼리 캐시 복원
   - 즉시 데이터 사용 및 렌더링
3. 네트워크 요청
   - 캐시된 데이터 사용으로 요청 최소화
4. 데이터 갱신
   - 새로운 데이터 요청 시에만 네트워크 요청

## 비교표

| 특징             | initialData               | Hydrate                 |
| ---------------- | ------------------------- | ----------------------- |
| 데이터 패칭 위치 | 서버 → initialData로 전달 | 서버에서 패칭 후 직렬화 |
| 첫 렌더링        | 초기값 설정 후 렌더링     | 패칭 데이터 즉시 사용   |
| 네트워크 요청    | 첫 렌더링 후 발생         | 최소화                  |
| 후속 요청        | 클라이언트에서 요청       | 캐시 데이터 우선 사용   |
| 초기 렌더링 성능 | 빠름, 후속 요청 있음      | 매우 빠름, 요청 최소화  |
| 적용 범위        | 개별 useQuery             | 전역 상태 동기화        |

## 선택 기준

- initialData: 클라이언트 사이드 초기값 설정, 미리 로드되지 않은 데이터
- Hydrate: 서버 데이터 즉시 사용, 초기 렌더링 최적화, 중복 요청 방지

## 관련 문서

- [[SSR-CORS-CriticalCSS]]
- [[JavaScript-기본개념]]

#react #query #hydration #performance
