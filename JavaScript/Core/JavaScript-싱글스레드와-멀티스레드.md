# JavaScript 싱글스레드와 멀티스레드

## 목차

- [[#Node.js의 동작 방식]]
  - [[#Node.js의 동작 방식#1. 단일 스레드 이벤트 루프]]
  - [[#Node.js의 동작 방식#2. 이벤트 루프 실행 과정]]
  - [[#Node.js의 동작 방식#3. Worker Threads (Node.js)]]
  - [[#Node.js의 동작 방식#4. Web Workers (브라우저)]]
- [[#모델 비교]]
  - [[#모델 비교#1. Non-Blocking I/O]]
  - [[#모델 비교#2. 멀티스레드]]
- [[#Web Workers vs Worker Threads]]
- [[#Node.js의 강점과 한계]]

## Node.js의 동작 방식

### 1. 단일 스레드 이벤트 루프

- **V8 엔진 기반**, 메인 이벤트 루프로 비동기 I/O 처리
- **이벤트 루프 단계**:
  1. Timers (setTimeout, setInterval)
  2. Pending Callbacks (I/O 콜백)
  3. Idle, Prepare (내부용)
  4. Poll (새로운 I/O 이벤트)
  5. Check (setImmediate)
  6. Close Callbacks

### 2. 이벤트 루프 실행 과정

- 각 단계는 **FIFO 큐** 형태로 콜백 실행
- 한 단계의 큐가 비거나 콜백 한도에 도달하면 다음 단계로 이동
- 반복적인 단계 전환으로 비동기 작업 처리

### 3. Worker Threads (Node.js)

- **CPU 집약적 작업**을 위한 API
- **Message Passing**으로 데이터 교환
- **SharedArrayBuffer**로 제한적 메모리 공유
- 백그라운드 스레드에서 JavaScript 코드 실행
- V8 인스턴스, 이벤트 루프, 메모리 독립적으로 관리

```javascript
const { Worker } = require("worker_threads");

const worker = new Worker(
  `
  const { parentPort } = require('worker_threads');
  // CPU 집약적 계산
  function fibonacci(n) {
    return n < 2 ? n : fibonacci(n-1) + fibonacci(n-2);
  }
  parentPort.on('message', (n) => {
    const result = fibonacci(n);
    parentPort.postMessage(result);
  });
`,
  { eval: true }
);

worker.on("message", (result) => {
  console.log("피보나치 결과:", result);
});

worker.postMessage(40);
```

### 4. Web Workers (브라우저)

- 브라우저에서 **백그라운드 작업** 수행
- **postMessage API**로 통신
- **UI 렌더링 방해 방지**
- DOM 직접 접근 불가
- 메인 스레드와 메시지 기반 통신

```javascript
// 메인 스레드
const worker = new Worker("worker.js");

worker.onmessage = function (e) {
  console.log("Worker 결과:", e.data);
};

worker.postMessage(40);

// worker.js
self.onmessage = function (e) {
  const result = fibonacci(e.data);
  self.postMessage(result);
};

function fibonacci(n) {
  return n < 2 ? n : fibonacci(n - 1) + fibonacci(n - 2);
}
```

## 모델 비교

### 1. Non-Blocking I/O

**장점**:

- 대기 없는 작업 처리
- 수천 개의 동시 연결 처리
- 컨텍스트 스위칭 비용 없음

**단점**:

- CPU 집약적 작업에 부적합
- 이벤트 루프 블로킹 위험
- 복잡한 병렬 처리의 어려움

### 2. 멀티스레드

**장점**:

- 병렬 작업 수행
- CPU 작업의 분산 처리
- CPU 코어 활용 극대화, 복잡한 동시성 구현

**단점**:

- 동기화 문제
- 컨텍스트 스위칭 오버헤드
- 복잡한 디버깅
- 경쟁 상태, 데드락, 스레드 관리 오버헤드

## Web Workers vs Worker Threads

| 특성        | Web Workers                            | Worker Threads                         |
| ----------- | -------------------------------------- | -------------------------------------- |
| 환경        | 브라우저                               | Node.js                                |
| 메모리 공유 | SharedArrayBuffer, MessageChannel 지원 | SharedArrayBuffer, MessageChannel 지원 |
| 주요 용도   | UI 블로킹 방지, 데이터 처리            | CPU 집약적 작업, 백엔드 병렬처리       |
| 데이터 통신 | postMessage, 메모리 전송/공유          | postMessage, 메모리 전송/공유          |

## Node.js의 강점과 한계

- **강점**:

  - I/O 바운드 작업에 효율적
  - 이벤트 루프 기반 비동기 처리
  - 메모리 효율성
  - 단일 언어로 풀스택 개발

- **한계**:
  - CPU 집약적 작업에 약점
  - 제한된 멀티코어 활용
  - 콜백 복잡성(해결법: Promise, async/await)
  - 오류 처리의 주의 필요

## 관련 문서

- [[JavaScript-기본개념]]
- [[JavaScript-메모리관리]]

#javascript #nodejs #webworker #multithreading #concurrency
