# 이벤트 루프, 콜 스택, 마이크로태스크 vs 매크로태스크

## 개요

JavaScript는 단일 스레드 언어지만 이벤트 루프(Event Loop)를 통해 비동기 작업을 효율적으로 처리합니다. 콜 스택(Call Stack), 마이크로태스크 큐(Microtask Queue), 매크로태스크 큐(Macrotask Queue)가 협력하여 JavaScript의 동시성을 구현합니다.

## 탄생 배경

웹 브라우저에서 사용자 인터페이스 응답성을 유지하면서 네트워크 요청, 타이머, DOM 이벤트 등을 처리하기 위해 이벤트 기반 비동기 모델이 필요했습니다. JavaScript는 이를 위해 이벤트 루프 메커니즘을 채택했습니다.

## 콜 스택 (Call Stack)

### 1. 콜 스택의 동작 원리

```javascript
function first() {
  console.log("First start");
  second();
  console.log("First end");
}

function second() {
  console.log("Second start");
  third();
  console.log("Second end");
}

function third() {
  console.log("Third");
}

// 실행 순서와 콜 스택 상태
first();

// Call Stack 변화:
// 1. [first]
// 2. [first, second]
// 3. [first, second, third]
// 4. [first, second] (third 완료)
// 5. [first] (second 완료)
// 6. [] (first 완료)
```

### 2. 스택 오버플로우

```javascript
function recursiveFunction(count) {
  console.log(`Recursion count: ${count}`);

  // 종료 조건 없는 재귀 (스택 오버플로우 발생)
  return recursiveFunction(count + 1);
}

// recursiveFunction(1); // RangeError: Maximum call stack size exceeded

// 안전한 재귀 (꼬리 재귀 최적화)
function safeRecursion(count, max = 1000) {
  if (count >= max) {
    return count;
  }

  console.log(`Safe recursion: ${count}`);
  return safeRecursion(count + 1, max);
}

console.log(safeRecursion(1, 5)); // 5
```

## 이벤트 루프 (Event Loop)

### 1. 이벤트 루프의 구조

```javascript
console.log("1: Synchronous");

setTimeout(() => {
  console.log("2: Macrotask (setTimeout)");
}, 0);

Promise.resolve().then(() => {
  console.log("3: Microtask (Promise)");
});

console.log("4: Synchronous");

// 출력 순서:
// 1: Synchronous
// 4: Synchronous
// 3: Microtask (Promise)
// 2: Macrotask (setTimeout)
```

### 2. 이벤트 루프 시각화

```javascript
function visualizeEventLoop() {
  console.log("=== Event Loop Visualization ===");

  // 1. Call Stack 실행
  console.log("Start");

  // 2. Macrotask Queue에 추가
  setTimeout(() => {
    console.log("Timeout 1");
  }, 0);

  // 3. Microtask Queue에 추가
  Promise.resolve()
    .then(() => {
      console.log("Promise 1");

      // 4. 중첩된 Microtask
      return Promise.resolve();
    })
    .then(() => {
      console.log("Promise 2");
    });

  // 5. 또 다른 Macrotask
  setTimeout(() => {
    console.log("Timeout 2");
  }, 0);

  // 6. 즉시 실행되는 Microtask
  queueMicrotask(() => {
    console.log("queueMicrotask");
  });

  console.log("End");
}

visualizeEventLoop();
// 출력:
// Start
// End
// Promise 1
// Promise 2
// queueMicrotask
// Timeout 1
// Timeout 2
```

## 마이크로태스크 vs 매크로태스크

### 1. 마이크로태스크 (Microtasks)

```javascript
// 마이크로태스크를 생성하는 방법들
function microtaskExamples() {
  console.log("=== Microtasks ===");

  // Promise.then
  Promise.resolve("Promise 1").then(console.log);

  // async/await
  (async () => {
    console.log("Async function start");
    await Promise.resolve();
    console.log("After await");
  })();

  // queueMicrotask
  queueMicrotask(() => {
    console.log("queueMicrotask");
  });

  // MutationObserver (브라우저)
  if (typeof MutationObserver !== "undefined") {
    const observer = new MutationObserver(() => {
      console.log("MutationObserver");
    });

    const div = document.createElement("div");
    observer.observe(div, { childList: true });
    div.appendChild(document.createElement("span"));
  }

  console.log("Microtask examples end");
}

microtaskExamples();
```

### 2. 매크로태스크 (Macrotasks)

```javascript
// 매크로태스크를 생성하는 방법들
function macrotaskExamples() {
  console.log("=== Macrotasks ===");

  // setTimeout
  setTimeout(() => {
    console.log("setTimeout");
  }, 0);

  // setInterval
  const intervalId = setInterval(() => {
    console.log("setInterval");
    clearInterval(intervalId);
  }, 0);

  // setImmediate (Node.js)
  if (typeof setImmediate !== "undefined") {
    setImmediate(() => {
      console.log("setImmediate");
    });
  }

  // I/O operations (Node.js)
  if (typeof process !== "undefined") {
    process.nextTick(() => {
      console.log("process.nextTick (higher priority)");
    });
  }

  console.log("Macrotask examples end");
}

macrotaskExamples();
```

### 3. 실행 우선순위

```javascript
function priorityDemo() {
  console.log("=== Priority Demo ===");

  // 다양한 비동기 작업들
  setTimeout(() => console.log("1: setTimeout"), 0);

  Promise.resolve().then(() => console.log("2: Promise.then"));

  queueMicrotask(() => console.log("3: queueMicrotask"));

  if (typeof process !== "undefined") {
    process.nextTick(() => console.log("4: process.nextTick"));
  }

  Promise.resolve().then(() => {
    console.log("5: Promise.then with nested");
    queueMicrotask(() => console.log("6: nested queueMicrotask"));
  });

  console.log("7: Synchronous");

  // Node.js 출력 순서:
  // 7: Synchronous
  // 4: process.nextTick (가장 높은 우선순위)
  // 2: Promise.then
  // 3: queueMicrotask
  // 5: Promise.then with nested
  // 6: nested queueMicrotask
  // 1: setTimeout
}

priorityDemo();
```

## 실무에서의 이벤트 루프 활용

### 1. 논블로킹 처리

```javascript
// 블로킹 코드 (나쁜 예)
function blockingOperation() {
  const start = Date.now();
  while (Date.now() - start < 1000) {
    // 1초 동안 블로킹
  }
  console.log("Blocking operation completed");
}

// 논블로킹 코드 (좋은 예)
function nonBlockingOperation(callback) {
  setTimeout(() => {
    console.log("Non-blocking operation completed");
    callback();
  }, 1000);
}

// 큰 데이터 처리를 청크로 나누기
function processLargeData(data, chunkSize = 1000) {
  let index = 0;

  function processChunk() {
    const endIndex = Math.min(index + chunkSize, data.length);

    // 청크 처리
    for (let i = index; i < endIndex; i++) {
      // 데이터 처리 로직
      data[i] = data[i] * 2;
    }

    index = endIndex;

    if (index < data.length) {
      // 다음 청크를 비동기로 처리
      setTimeout(processChunk, 0);
    } else {
      console.log("All data processed");
    }
  }

  processChunk();
}

// 사용 예시
const largeArray = new Array(10000).fill(1);
processLargeData(largeArray);
```

### 2. 배치 DOM 업데이트

```javascript
// DOM 업데이트 최적화
class BatchedDOMUpdater {
  constructor() {
    this.updates = [];
    this.scheduled = false;
  }

  scheduleUpdate(element, property, value) {
    this.updates.push({ element, property, value });

    if (!this.scheduled) {
      this.scheduled = true;

      // 다음 프레임에서 일괄 업데이트
      requestAnimationFrame(() => {
        this.flushUpdates();
      });
    }
  }

  flushUpdates() {
    // 모든 업데이트를 한 번에 적용
    this.updates.forEach(({ element, property, value }) => {
      element.style[property] = value;
    });

    this.updates = [];
    this.scheduled = false;
  }
}

// 사용 예시
const updater = new BatchedDOMUpdater();

// 여러 DOM 업데이트가 한 번에 처리됨
// updater.scheduleUpdate(element1, 'left', '100px');
// updater.scheduleUpdate(element2, 'top', '200px');
```

### 3. 비동기 에러 처리

```javascript
// 이벤트 루프에서의 에러 처리
function errorHandlingDemo() {
  // 동기 에러 - try/catch로 잡힘
  try {
    throw new Error("Synchronous error");
  } catch (error) {
    console.log("Caught sync error:", error.message);
  }

  // 비동기 에러 - try/catch로 잡히지 않음
  try {
    setTimeout(() => {
      throw new Error("Async error"); // 잡히지 않음
    }, 0);
  } catch (error) {
    console.log("This will not catch async error");
  }

  // 올바른 비동기 에러 처리
  setTimeout(() => {
    try {
      throw new Error("Async error in callback");
    } catch (error) {
      console.log("Caught async error:", error.message);
    }
  }, 0);

  // Promise 에러 처리
  Promise.reject(new Error("Promise error")).catch((error) => {
    console.log("Caught Promise error:", error.message);
  });

  // 전역 에러 핸들러
  if (typeof window !== "undefined") {
    window.addEventListener("error", (event) => {
      console.log("Global error handler:", event.error.message);
    });

    window.addEventListener("unhandledrejection", (event) => {
      console.log("Unhandled Promise rejection:", event.reason.message);
      event.preventDefault(); // 기본 에러 로깅 방지
    });
  }
}

errorHandlingDemo();
```

## 성능 최적화와 디버깅

### 1. 이벤트 루프 모니터링

```javascript
class EventLoopMonitor {
  constructor() {
    this.taskTimes = [];
    this.isMonitoring = false;
  }

  start() {
    this.isMonitoring = true;
    this.measureEventLoop();
  }

  stop() {
    this.isMonitoring = false;
    return this.getStats();
  }

  measureEventLoop() {
    if (!this.isMonitoring) return;

    const start = performance.now();

    setTimeout(() => {
      const end = performance.now();
      const delay = end - start;

      this.taskTimes.push(delay);

      // 지연이 16ms(60fps) 이상이면 경고
      if (delay > 16) {
        console.warn(`Event loop delay: ${delay.toFixed(2)}ms`);
      }

      this.measureEventLoop();
    }, 0);
  }

  getStats() {
    if (this.taskTimes.length === 0) return null;

    const sorted = this.taskTimes.sort((a, b) => a - b);
    const avg = this.taskTimes.reduce((a, b) => a + b) / this.taskTimes.length;

    return {
      average: avg.toFixed(2),
      median: sorted[Math.floor(sorted.length / 2)].toFixed(2),
      max: Math.max(...this.taskTimes).toFixed(2),
      samples: this.taskTimes.length,
    };
  }
}

// 사용 예시
const monitor = new EventLoopMonitor();
monitor.start();

setTimeout(() => {
  const stats = monitor.stop();
  console.log("Event loop stats:", stats);
}, 5000);
```

### 2. 메모리 누수 방지

```javascript
// 이벤트 루프와 관련된 메모리 누수 방지
class SafeAsyncHandler {
  constructor() {
    this.timeouts = new Set();
    this.intervals = new Set();
    this.destroyed = false;
  }

  setTimeout(callback, delay) {
    if (this.destroyed) return;

    const timeoutId = setTimeout(() => {
      this.timeouts.delete(timeoutId);
      if (!this.destroyed) {
        callback();
      }
    }, delay);

    this.timeouts.add(timeoutId);
    return timeoutId;
  }

  setInterval(callback, delay) {
    if (this.destroyed) return;

    const intervalId = setInterval(() => {
      if (!this.destroyed) {
        callback();
      }
    }, delay);

    this.intervals.add(intervalId);
    return intervalId;
  }

  destroy() {
    this.destroyed = true;

    // 모든 타이머 정리
    this.timeouts.forEach(clearTimeout);
    this.intervals.forEach(clearInterval);

    this.timeouts.clear();
    this.intervals.clear();
  }
}

// 사용 예시
const handler = new SafeAsyncHandler();

handler.setTimeout(() => {
  console.log("Safe timeout executed");
}, 1000);

// 컴포넌트 언마운트 시
// handler.destroy();
```

## Node.js vs 브라우저 차이점

### 1. Node.js 이벤트 루프 단계

```javascript
// Node.js 이벤트 루프 단계별 처리
function nodeEventLoopPhases() {
  console.log("=== Node.js Event Loop Phases ===");

  // Timer phase
  setTimeout(() => console.log("Timer phase"), 0);

  // I/O callbacks phase
  if (typeof setImmediate !== "undefined") {
    setImmediate(() => console.log("Check phase (setImmediate)"));
  }

  // Poll phase에서 처리되는 I/O
  if (typeof process !== "undefined") {
    const fs = require("fs");
    fs.readFile(__filename, () => {
      console.log("I/O callback");
    });
  }

  // Microtasks (각 단계 사이에서 실행)
  process.nextTick(() => console.log("nextTick 1"));
  Promise.resolve().then(() => console.log("Promise 1"));

  process.nextTick(() => console.log("nextTick 2"));
  Promise.resolve().then(() => console.log("Promise 2"));

  console.log("Synchronous");
}

if (typeof process !== "undefined") {
  nodeEventLoopPhases();
}
```

### 2. 브라우저 vs Node.js 우선순위

```javascript
function comparePlatforms() {
  console.log("=== Platform Comparison ===");

  setTimeout(() => console.log("setTimeout"), 0);

  if (typeof setImmediate !== "undefined") {
    // Node.js only
    setImmediate(() => console.log("setImmediate"));
  }

  if (typeof process !== "undefined") {
    // Node.js only - 가장 높은 우선순위
    process.nextTick(() => console.log("process.nextTick"));
  }

  Promise.resolve().then(() => console.log("Promise.then"));
  queueMicrotask(() => console.log("queueMicrotask"));

  if (typeof requestAnimationFrame !== "undefined") {
    // Browser only
    requestAnimationFrame(() => console.log("requestAnimationFrame"));
  }

  console.log("Synchronous");
}

comparePlatforms();
```

## 실제 사용 사례

### 1. 사용자 인터페이스 응답성

```javascript
// UI 응답성을 위한 작업 분할
class ResponsiveProcessor {
  constructor(maxExecutionTime = 16) {
    this.maxExecutionTime = maxExecutionTime;
  }

  async processArray(array, processor) {
    let index = 0;
    const results = [];

    while (index < array.length) {
      const startTime = performance.now();

      // 제한 시간 내에서 최대한 처리
      while (
        index < array.length &&
        performance.now() - startTime < this.maxExecutionTime
      ) {
        results.push(processor(array[index]));
        index++;
      }

      // UI 업데이트를 위한 시간 확보
      if (index < array.length) {
        await new Promise((resolve) => setTimeout(resolve, 0));
      }
    }

    return results;
  }
}

// 사용 예시
const processor = new ResponsiveProcessor();
const largeArray = new Array(100000).fill(0).map((_, i) => i);

processor
  .processArray(largeArray, (x) => x * x)
  .then((results) => {
    console.log(`Processed ${results.length} items`);
  });
```

### 2. 데이터 스트리밍

```javascript
// 이벤트 루프를 활용한 데이터 스트리밍
class DataStreamer {
  constructor() {
    this.subscribers = new Set();
  }

  subscribe(callback) {
    this.subscribers.add(callback);
    return () => this.subscribers.delete(callback);
  }

  async streamData(data, chunkSize = 10) {
    for (let i = 0; i < data.length; i += chunkSize) {
      const chunk = data.slice(i, i + chunkSize);

      // 각 구독자에게 데이터 전송
      this.subscribers.forEach((callback) => {
        // 다음 틱에서 실행하여 논블로킹 보장
        queueMicrotask(() => callback(chunk));
      });

      // 이벤트 루프에 제어권 반환
      await new Promise((resolve) => setTimeout(resolve, 0));
    }

    // 스트리밍 완료 알림
    this.subscribers.forEach((callback) => {
      queueMicrotask(() => callback(null)); // null은 완료 신호
    });
  }
}

// 사용 예시
const streamer = new DataStreamer();

const unsubscribe = streamer.subscribe((chunk) => {
  if (chunk === null) {
    console.log("Streaming completed");
  } else {
    console.log(`Received chunk of ${chunk.length} items`);
  }
});

const bigData = new Array(1000).fill(0).map((_, i) => i);
streamer.streamData(bigData);
```

## 결론

이벤트 루프, 콜 스택, 마이크로태스크/매크로태스크의 이해는 JavaScript의 비동기 처리 메커니즘을 완전히 파악하는 핵심입니다. 이를 통해 성능 최적화, 사용자 경험 향상, 메모리 누수 방지 등 실무에서 중요한 문제들을 해결할 수 있습니다.
