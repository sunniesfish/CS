# SSR, CORS, Critical CSS

## 1. SSR (서버 사이드 렌더링)

### Hydration

- 서버에서 렌더링된 HTML을 클라이언트 React DOM과 연결
- 정적 HTML에 동적 기능(이벤트 핸들러 등) 추가
- 서버와 클라이언트의 UI 상태 일치 필요

### SSR의 단점

1. 서버 부하 증가
2. 첫 번째 요청 지연 (TTFB 증가)
3. 복잡성 증가 (상태 동기화, 데이터 페칭)

## 2. CORS (Cross-Origin Resource Sharing)

### 기본 개념

- 다른 출처(origin)의 리소스 요청 제한/허용 메커니즘
- 서버가 HTTP 응답 헤더로 허용 범위 지정
- 브라우저가 요청을 허용/차단

### Preflight 요청

- OPTIONS 메서드로 실제 요청 가능 여부 확인
- 허용된 메서드, 헤더, 출처 확인
- Access-Control-Max-Age로 캐싱 제어

## 3. Critical CSS

### 개념

- 초기 렌더링에 필요한 최소 CSS를 HTML에 인라인 삽입
- 렌더링 차단 리소스 문제 감소
- 초기 로딩 시간 단축

### React 환경 구현

```javascript
useEffect(() => {
  const style = document.createElement("style");
  style.textContent = ".critical { color: red; }";
  document.head.appendChild(style);
}, []);
```

## 4. 최적화 기법

### 이미지 최적화

1. WebP, AVIF 포맷 사용

   - 높은 압축 효율
   - 품질 손실 최소화
   - SEO 개선

2. 폰트 최적화
   - font-display: swap 사용
   - 폰트 서브셋 생성
   - 브라우저 캐싱 활용

### 서비스 워커

1. 주요 기능

   - 리소스 캐싱
   - 푸시 알림
   - 백그라운드 동작
   - 네트워크 요청 제어

2. 한계
   - 브라우저 지원 제한
   - HTTPS 필요
   - DOM 접근 불가
   - 설치/업데이트 과정 필요

## 관련 문서

- [[React-Query-Hydration]]
- [[JavaScript-기본개념]]

#web #ssr #cors #optimization
