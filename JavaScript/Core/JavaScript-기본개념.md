# JavaScript 기본개념

## 목차

- [[#주요 특징]]
- [[#핵심 개념]]
  - [[#핵심 개념#1. WeakMap과 WeakSet]]
  - [[#핵심 개념#2. 클로저(Closure)]]
  - [[#핵심 개념#3. this와 바인딩]]
- [[#성능 최적화]]
  - [[#성능 최적화#1. 이벤트 리스너 최적화]]

## 주요 특징

- **동적 타입 언어**
- **비동기 처리** (Event Loop, Call Stack, Web API, Task Queue)
- **객체 기반 언어**
- **클라이언트와 서버 양쪽에서 실행 가능**

## 핵심 개념

### 1. WeakMap과 WeakSet

- **WeakMap**: 객체만을 키로 사용, 약한 참조 유지
- **WeakSet**: 객체만 저장 가능, 자동 메모리 해제
- **Map과의 차이점**:
  - Map: 모든 값이 키로 사용 가능, 강한 참조
  - WeakMap: 객체만 키로 사용 가능, 약한 참조

### 2. 클로저(Closure)

- 함수가 생성될 때의 렉시컬 환경을 기억하는 특성
- 데이터 은닉에 활용
- 메모리 누수 주의점

**상세 설명:**

- **클로저란?** 함수가 자신이 생성될 때의 렉시컬 환경(Lexical Environment)을 기억하는 특성으로, 외부 함수의 변수에 접근할 수 있습니다.
- **데이터 은닉:** 외부에서 접근할 수 없는 변수를 만들어 민감한 데이터를 보호할 수 있습니다.

**예제 코드:**

```javascript
function createCounter() {
  let count = 0;
  return {
    increment: () => count++,
    getCount: () => count,
  };
}

const counter = createCounter();
console.log(counter.getCount()); // 0
counter.increment();
console.log(counter.getCount()); // 1
// count 변수는 외부에서 직접 접근 불가능
```

**메모리 누수 예시:**

```javascript
function init() {
  let largeArray = new Array(1000000);
  return function () {
    console.log(largeArray.length);
  };
}
let leak = init();
```

`largeArray`는 `leak`을 통해 계속 참조되기 때문에 메모리 해제가 안 됩니다.

**해결법:**

필요하지 않은 참조는 `null`로 설정하여 가비지 컬렉션이 수거할 수 있게 합니다.

```javascript
// 사용 완료 후
leak = null;
```

### 3. this와 바인딩

- **call**: 함수 호출과 동시에 this 설정
- **apply**: call과 유사하나 인수를 배열로 받음
- **bind**: this를 고정하고 새 함수 반환

**상세 설명:**

- `this`: 실행 컨텍스트에 따라 달라집니다 (전역, 함수, 객체, 클래스 등)
- `call`: 함수 호출과 동시에 `this`를 명시적으로 설정합니다
- `apply`: `call`과 비슷하지만, 인수를 배열로 받습니다
- `bind`: `this`를 고정하고 새로운 함수를 반환합니다

**예제 코드:**

```javascript
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}

const user = { name: "Alice" };

greet.call(user, "Hello"); // Hello, Alice
greet.apply(user, ["Hi"]); // Hi, Alice
const boundGreet = greet.bind(user);
boundGreet("Hey"); // Hey, Alice
```

## 성능 최적화

### 1. 이벤트 리스너 최적화

- **이벤트 위임 활용**
- **불필요한 리스너 제거**
- **스로틀링과 디바운싱 활용**
- **패시브 이벤트 리스너 사용**

**상세 설명:**

1. **이벤트 위임 (Event Delegation)**

   - 많은 자식 요소에 이벤트 리스너를 개별로 추가하지 않고, 상위 요소에 추가합니다.

   ```javascript
   // 비효율적인 방법
   document.querySelectorAll("li").forEach((item) => {
     item.addEventListener("click", handleClick);
   });

   // 효율적인 방법 (이벤트 위임)
   document.querySelector("ul").addEventListener("click", (e) => {
     if (e.target.tagName === "LI") {
       handleClick(e);
     }
   });
   ```

## 관련 문서

- [[JavaScript-메모리관리]]
- [[JavaScript-싱글스레드와-멀티스레드]]

#javascript #basics #closure #this #performance
