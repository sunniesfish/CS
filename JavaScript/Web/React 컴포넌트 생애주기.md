# React 컴포넌트 생애주기

## 클래스 컴포넌트 vs 함수형 컴포넌트

### 클래스 컴포넌트의 생애주기

클래스 컴포넌트는 명시적인 생애주기 메서드를 통해 각 단계를 제어한다.

1. **마운트 단계 (Mounting)**

   ```typescript
   class MyComponent extends React.Component {
     constructor(props) {
       super(props);
       // 1. 초기 상태 설정
     }

     static getDerivedStateFromProps(props, state) {
       // 2. props로부터 state 도출
       return null;
     }

     render() {
       // 3. 컴포넌트 렌더링
       return <div />;
     }

     componentDidMount() {
       // 4. DOM에 마운트 완료 후 호출
       // 데이터 페칭, 구독 설정 등
     }
   }
   ```

2. **업데이트 단계 (Updating)**

   ```typescript
   class MyComponent extends React.Component {
     static getDerivedStateFromProps(props, state) {
       // 1. props 변경 시 state 도출
       return null;
     }

     shouldComponentUpdate(nextProps, nextState) {
       // 2. 리렌더링 여부 결정
       return true;
     }

     render() {
       // 3. 컴포넌트 리렌더링
       return <div />;
     }

     getSnapshotBeforeUpdate(prevProps, prevState) {
       // 4. DOM 업데이트 직전 호출
       return null;
     }

     componentDidUpdate(prevProps, prevState, snapshot) {
       // 5. DOM 업데이트 완료 후 호출
     }
   }
   ```

3. **언마운트 단계 (Unmounting)**
   ```typescript
   class MyComponent extends React.Component {
     componentWillUnmount() {
       // 컴포넌트 제거 직전 호출
       // 구독 해제, 타이머 정리 등
     }
   }
   ```

### 함수형 컴포넌트의 생애주기

함수형 컴포넌트는 Hooks를 통해 생애주기를 관리한다.

1. **마운트 단계**

   ```typescript
   function MyComponent() {
     // 1. 컴포넌트 함수 실행
     console.log("렌더링 시작");

     useEffect(() => {
       // 3. 마운트 완료 후 실행
       console.log("마운트됨");

       return () => {
         // 4. 언마운트 시 정리(cleanup) 함수 실행
         console.log("언마운트됨");
       };
     }, []); // 빈 의존성 배열: 마운트/언마운트 시에만 실행

     // 2. JSX 반환
     return <div />;
   }
   ```

2. **업데이트 단계**

   ```typescript
   function MyComponent({ data }) {
     const [state, setState] = useState(initialState);

     // 1. 의존성이 변경될 때마다 실행
     useEffect(() => {
       console.log("data 또는 state가 변경됨");
     }, [data, state]);

     // 2. 성능 최적화
     const memoizedValue = useMemo(() => {
       return expensiveComputation(data);
     }, [data]);

     // 3. 함수 메모이제이션
     const memoizedCallback = useCallback(() => {
       doSomething(state);
     }, [state]);

     return <div />;
   }
   ```

## 렌더링 사이클

### 1. 렌더 단계 (Render Phase)

```typescript
function ParentComponent() {
  // 1. 상태 변경 발생
  const [count, setCount] = useState(0);

  // 2. 컴포넌트 함수 실행
  console.log("렌더링 시작");

  // 3. Virtual DOM 생성
  return <ChildComponent count={count} />;
}
```

### 2. 커밋 단계 (Commit Phase)

```typescript
function ChildComponent({ count }) {
  useEffect(() => {
    // 2. DOM 업데이트 후 실행
    console.log("DOM 업데이트 완료");
  });

  // 1. 실제 DOM 업데이트
  return <div>{count}</div>;
}
```

## 최적화 기법

### 1. 메모이제이션

```typescript
// 1. 컴포넌트 메모이제이션
const MemoizedChild = React.memo(ChildComponent);

// 2. 값 메모이제이션
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

// 3. 함수 메모이제이션
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

### 2. 렌더링 최적화

```typescript
function OptimizedComponent({ data }) {
  // 1. 조건부 렌더링
  if (!data) return null;

  // 2. 리스트 렌더링 최적화
  return (
    <ul>
      {data.map((item) => (
        <MemoizedListItem key={item.id} item={item} />
      ))}
    </ul>
  );
}
```

## 주의사항

1. **부수 효과 관리**

   ```typescript
   function DataFetchingComponent() {
     useEffect(() => {
       let isMounted = true;

       async function fetchData() {
         const data = await api.getData();
         if (isMounted) {
           // 언마운트된 컴포넌트의 상태 업데이트 방지
           setData(data);
         }
       }

       fetchData();

       return () => {
         isMounted = false;
       };
     }, []);
   }
   ```

2. **의존성 배열 관리**

   ```typescript
   function DependencyExample({ onSubmit }) {
     // 잘못된 예
     useEffect(() => {
       // 매 렌더링마다 새로운 이벤트 리스너 등록
       document.addEventListener("click", onSubmit);
     }); // 의존성 배열 누락

     // 올바른 예
     useEffect(() => {
       document.addEventListener("click", onSubmit);
       return () => {
         document.removeEventListener("click", onSubmit);
       };
     }, [onSubmit]); // 명시적인 의존성 배열
   }
   ```

## 디버깅 팁

1. **렌더링 추적**

   ```typescript
   function DebuggingComponent() {
     console.log("컴포넌트 렌더링");

     useEffect(() => {
       console.log("마운트 또는 업데이트");
       return () => console.log("정리(cleanup)");
     });

     return <div />;
   }
   ```

2. **React DevTools 활용**
   - Components 탭: 컴포넌트 트리 구조 확인
   - Profiler 탭: 렌더링 성능 분석
