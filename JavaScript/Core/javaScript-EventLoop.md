---
# 🧠 JavaScript 이벤트 루프: 구조와 동작 원리 완전 기술 정리
---

## 1. JavaScript 실행 모델 개요

- 자바스크립트는 **싱글 스레드 기반의 언어**이다.
- 실행 컨텍스트는 단 하나의 **Call Stack**에서 관리된다.
- 비동기 처리를 위해 **이벤트 루프(Event Loop)** + \*\*태스크 큐(Task Queues)\*\*를 이용한다.

---

## 2. Call Stack

- 자바스크립트의 실행 컨텍스트(스코프, 변수, this, 코드)를 저장하는 LIFO 구조.
- **함수가 호출되면 스택에 쌓이고, 리턴되면 스택에서 빠진다.**

### 실행 예시:

```js
function foo() {
  bar();
}

function bar() {
  console.log("Hi");
}

foo();
```

- 실행 흐름:

  - `foo()` 호출 → `foo`가 Call Stack에 push
  - `foo` 내부에서 `bar()` 호출 → `bar`가 Call Stack에 push
  - `bar`가 console 출력 후 종료 → `bar` pop
  - `foo` 종료 → `foo` pop
  - Call Stack 비게 됨

---

## 3. Event Loop

> 이벤트 루프는 "Call Stack이 비는 시점"을 감지하여 **Task Queue에서 대기 중인 작업을 실행할지 결정**하는 무한 루프이다.

동작 절차:

1. Call Stack이 비어 있는지 확인
2. **Microtask Queue가 비어 있지 않다면 → Microtask를 모두 실행**
3. Microtask가 모두 처리된 후 → **Macrotask Queue에서 하나 꺼내서 실행**
4. 다시 1번부터 반복

---

## 4. Task Queue 종류

### 📌 Macrotask Queue

- 일반적인 비동기 작업의 콜백이 등록됨.
- 대표 API:

  - `setTimeout`, `setInterval`
  - DOM 이벤트 (click, input 등)
  - `I/O` (파일, 네트워크)
  - `MessageChannel`, `setImmediate` (Node.js)

### 📌 Microtask Queue

- **이벤트 루프 사이클 중 Macrotask 이전에 처리**됨.
- 대표 API:

  - `Promise.then`, `.catch`, `.finally`
  - `MutationObserver`
  - `queueMicrotask` (명시적 마이크로태스크 삽입)

---

## 5. Queue 등록 vs 실행 시점

### 콜백이 Queue에 등록되는 시점

- Promise가 `resolve`되면 `.then()`에 등록된 콜백이 **즉시 실행되지 않고**, Microtask Queue에 **등록**된다.
- `setTimeout(..., 0)`도 등록 즉시 실행이 아님. Macrotask Queue에 들어가서 기다림.

### Queue에 등록되는 것은?

- **함수 객체(콜백)** 이다. 결과값이 아님.
- `Promise.resolve(42).then(v => console.log(v))`에서는

  - 42는 힙에 저장되고
  - `() => console.log(v)` 콜백이 Microtask Queue에 등록됨

---

## 6. Microtask Queue와 Macrotask Queue 실행 순서 예시

```js
console.log("start");

setTimeout(() => {
  console.log("macrotask");
}, 0);

Promise.resolve().then(() => {
  console.log("microtask");
});

console.log("end");
```

실행 순서:

1. `"start"` → Call Stack 실행
2. `setTimeout` 콜백 등록 → Macrotask Queue에 등록됨
3. `Promise.resolve()` 처리되며 `.then(...)` 콜백 등록 → Microtask Queue에 등록됨
4. `"end"` → Call Stack 실행
5. Call Stack 비면 → Microtask Queue 실행 → `"microtask"`
6. Microtask Queue 비면 → Macrotask Queue 실행 → `"macrotask"`

---

## 7. Microtask Queue의 "블로킹" 성질

- Microtask Queue는 **이벤트 루프 한 틱(tick)** 내에서 **모두 실행될 때까지 멈추지 않는다.**
- 하나라도 복잡한 계산이 있다면 다음 Macrotask나 사용자 입력, UI 업데이트도 지연된다.

```js
Promise.resolve().then(() => {
  while (true) {} // 무한 루프
});

setTimeout(() => {
  console.log("이건 절대 실행되지 않음");
}, 0);
```

- 위 코드에서 `while (true)`는 Microtask로 등록된 콜백에서 실행되므로
- **이벤트 루프는 Microtask를 다 처리하지 못하고 멈춤 → Macrotask는 절대 실행되지 않음**

---

## 8. Promise와 이벤트 루프의 상호작용

- `Promise.resolve()`는 **즉시 resolved 된 Promise 객체**를 생성한다.
- 이 Promise는 `.then()` 등록 시 **곧바로 Microtask Queue에 콜백을 넣는다**.

```js
Promise.resolve(42).then((v) => console.log(v));
```

- `42`는 힙 메모리에 저장된다.
- 콜백 `() => console.log(42)`가 Microtask Queue에 들어간다.
- Call Stack이 비면 이벤트 루프가 이 콜백을 Call Stack에 넣고 실행.

---

## 9. 메모리 구조에서의 역할

### 📍 Call Stack

- 함수 실행 컨텍스트 (Execution Context)

  - 로컬 변수, 매개변수, this 바인딩, 스코프 체인 등 저장
  - 휘발성 (LIFO)

### 📍 Heap

- 객체, 배열, 클로저, Promise 결과값 등 참조형 데이터 저장
- GC(Garbage Collector)의 관리 대상

### 📍 Queue (Micro/Macro)

- 콜백 함수 객체 참조 저장
- Heap에 있는 함수 참조값이 들어감

---

## 10. 싱글 스레드 한계와 이벤트 루프의 의의

- 이벤트 루프는 동시성(concurrency)은 제공하지만, \*\*병렬성(parallelism)\*\*은 제공하지 않음.
- 콜백 함수는 메인 스레드에서 실행되므로 **콜백 자체가 무거우면 전체 시스템이 느려진다.**

해결 방법:

1. **Web Worker / Worker Thread**

   - 별도 스레드에서 CPU 연산 처리

2. **작업 분할**

   - 무거운 계산을 `setTimeout`, `requestIdleCallback`, `queueMicrotask`로 나누기

---

## 11. 자주 혼동하는 개념 정리

| 개념                | 설명                                                                          |
| ------------------- | ----------------------------------------------------------------------------- |
| `Promise.resolve()` | 이미 resolve된 Promise 반환 (동기처럼 보이지만 실제 콜백은 비동기적으로 실행) |
| Microtask Queue     | 이벤트 루프 틱 안에서 모두 실행해야 함 (우선순위 높음)                        |
| Macrotask Queue     | Microtask Queue가 비워진 뒤에야 한 개씩 실행                                  |
| Queue 등록          | Promise의 `then` 등록 시 Promise 해결 후 콜백 함수가 Queue에 등록됨           |
| 메모리 위치         | 값은 Heap, 콜백 참조는 Queue, 실행은 Call Stack                               |

---
