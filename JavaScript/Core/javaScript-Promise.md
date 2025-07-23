---
## 📦 JavaScript Promise: 핵심 개념 정리
---

### ✅ 1. Promise의 생성과 상태

```ts
const p = new Promise((resolve, reject) => {
  // executor 함수
});
```

- `new Promise(executor)`로 Promise 객체를 생성하면 즉시 `executor(resolve, reject)` 함수가 실행됩니다.
- 이 executor 함수는 비동기 처리의 **실행 컨텍스트**이며, 그 안에서 처리 결과에 따라 `resolve(value)` 또는 `reject(reason)`을 호출합니다.

#### 🔹 내부 상태

Promise는 3가지 상태를 가집니다:

- `pending` (초기 상태)
- `fulfilled` (`resolve`가 호출됨)
- `rejected` (`reject`가 호출됨)

---

### ✅ 2. resolve / reject

```ts
resolve("data");
reject("error");
```

- 이 함수들을 호출하면 Promise의 내부 상태가 \*\*`pending` → `fulfilled` 또는 `rejected`\*\*로 전이됩니다.
- 동시에 내부에 `[[PromiseResult]] = value` 형태로 결과 값을 저장합니다.
- 그 후, `then`, `catch` 등으로 등록된 후속 처리 함수가 **마이크로태스크 큐**에 등록됩니다.

---

### ✅ 3. then / catch의 실행

```ts
p.then((value) => {
  console.log(value);
}).catch((err) => {
  console.error(err);
});
```

- `then`, `catch`, `finally`는 `resolve`/`reject` 상태에 따라 실행될 **콜백을 등록**합니다.
- 실제 실행은 즉시 되지 않고, **마이크로태스크 큐에 등록**되어 **현재 실행 컨텍스트가 종료된 후** 실행됩니다.
- 따라서, `resolve`는 **동기적으로 호출**되지만, `then` 콜백의 실행은 \*\*비동기적 (마이크로태스크)\*\*입니다.

---

### ✅ 4. 마이크로태스크 큐

- `Promise`의 후속 작업 (`then`, `catch`)은 **Microtask Queue**에 등록됩니다.
- JavaScript의 이벤트 루프는:

  1. 현재 실행 중인 콜 스택이 비면,
  2. **Microtask Queue**를 먼저 실행하고,
  3. 그 후 **Task Queue** (setTimeout 등)를 실행합니다.

#### 📌 예:

```ts
console.log("start");

Promise.resolve("resolved").then(console.log);

console.log("end");
```

**출력 결과:**

```
start
end
resolved
```

---

### ✅ 5. resolve 값의 전달 흐름

```ts
resolve("someValue");
```

- 이 호출 시:

  - `[[PromiseResult]] = "someValue"`로 내부 값이 저장됨
  - `then`에 등록된 콜백이 마이크로태스크 큐에 들어감
  - 이 콜백이 실행될 때 `"someValue"`가 **파라미터로 전달됨**

---

### ✅ 6. Deferred 패턴 예시 (외부에서 resolve/reject)

```ts
let externalResolve!: (val: string) => void;

const deferred = new Promise<string>((resolve) => {
  externalResolve = resolve;
});

deferred.then((val) => console.log("결과:", val));

setTimeout(() => {
  externalResolve("외부에서 처리됨");
}, 1000);
```

- `executor` 내부의 `resolve`를 외부 변수에 저장해두고,
- 외부 타이밍에서 `resolve()`를 호출해 Promise를 해결

---

## 🧠 핵심 요약

| 요소              | 설명                                                                         |
| ----------------- | ---------------------------------------------------------------------------- |
| `executor`        | Promise 생성 시 실행되는 함수. 내부에서 `resolve/reject` 호출                |
| `resolve(value)`  | 상태를 `fulfilled`로 바꾸고, value를 then으로 넘김                           |
| `reject(reason)`  | 상태를 `rejected`로 바꾸고, reason을 catch로 넘김                            |
| `then(callback)`  | fulfilled 상태일 때 실행할 콜백 등록 (마이크로태스크 큐에 들어감)            |
| `catch(callback)` | rejected 상태일 때 실행할 콜백 등록                                          |
| `Promise` 상태    | `pending` → `fulfilled` 또는 `rejected` 한 번만 바뀔 수 있음                 |
| 실행 순서         | `resolve()`는 동기, but `then()` 콜백 실행은 마이크로태스크 큐를 통해 비동기 |

---
