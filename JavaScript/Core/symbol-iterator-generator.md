# Symbol, Iterator, Generator의 원리 및 활용

## 개요

ES6에서 도입된 Symbol, Iterator, Generator는 JavaScript의 메타프로그래밍 능력을 크게 확장시킨 핵심 기능들입니다. 이들은 객체의 동작을 커스터마이징하고, 반복 가능한 데이터 구조를 만들며, 함수 실행을 일시정지하고 재개할 수 있는 강력한 도구를 제공합니다.

## 탄생 배경

JavaScript의 객체 시스템을 더욱 유연하게 만들고, 다른 언어의 반복자 패턴과 제너레이터 패턴을 도입하여 비동기 프로그래밍과 데이터 처리를 개선하기 위해 도입되었습니다.

## Symbol

### 1. Symbol의 기본 개념

```javascript
// Symbol 생성
const sym1 = Symbol();
const sym2 = Symbol("description");
const sym3 = Symbol("description");

console.log(sym1); // Symbol()
console.log(sym2); // Symbol(description)
console.log(sym2 === sym3); // false - 각 Symbol은 유일함

// Symbol 타입 확인
console.log(typeof sym1); // 'symbol'

// Symbol은 문자열로 변환되지 않음
try {
  console.log(sym1 + ""); // TypeError
} catch (error) {
  console.log("Symbol cannot be converted to string");
}

// 명시적 변환은 가능
console.log(sym1.toString()); // 'Symbol()'
console.log(String(sym1)); // 'Symbol()'
console.log(sym1.description); // undefined (sym1에는 description이 없음)
console.log(sym2.description); // 'description'
```

### 2. 전역 Symbol 레지스트리

```javascript
// 전역 Symbol 생성 및 검색
const globalSym1 = Symbol.for("app.id");
const globalSym2 = Symbol.for("app.id");

console.log(globalSym1 === globalSym2); // true - 같은 전역 Symbol

// Symbol key 찾기
console.log(Symbol.keyFor(globalSym1)); // 'app.id'
console.log(Symbol.keyFor(sym1)); // undefined (전역 Symbol이 아님)

// 전역 Symbol 활용 예시
const APP_CONFIG = {
  [Symbol.for("app.version")]: "1.0.0",
  [Symbol.for("app.name")]: "MyApp",
  regularProperty: "visible",
};

console.log(APP_CONFIG[Symbol.for("app.version")]); // '1.0.0'
```

### 3. Well-known Symbol

```javascript
// Symbol.iterator - 객체를 반복 가능하게 만듦
const iterableObj = {
  data: [1, 2, 3, 4, 5],

  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;

    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        } else {
          return { done: true };
        }
      },
    };
  },
};

// for...of 루프 사용 가능
for (const value of iterableObj) {
  console.log(value); // 1, 2, 3, 4, 5
}

// Symbol.toStringTag - 객체의 문자열 표현 커스터마이징
class CustomClass {
  get [Symbol.toStringTag]() {
    return "CustomClass";
  }
}

const custom = new CustomClass();
console.log(Object.prototype.toString.call(custom)); // '[object CustomClass]'

// Symbol.hasInstance - instanceof 연산자 동작 커스터마이징
class MyArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}

console.log([1, 2, 3] instanceof MyArray); // true
console.log({} instanceof MyArray); // false

// Symbol.toPrimitive - 원시값 변환 커스터마이징
const obj = {
  [Symbol.toPrimitive](hint) {
    console.log(`Converting to ${hint}`);
    switch (hint) {
      case "number":
        return 42;
      case "string":
        return "hello";
      case "default":
        return "default";
      default:
        throw new Error(`Unknown hint: ${hint}`);
    }
  },
};

console.log(+obj); // Converting to number, 42
console.log(`${obj}`); // Converting to string, 'hello'
console.log(obj + ""); // Converting to default, 'default'
```

### 4. Symbol을 이용한 프라이빗 속성

```javascript
// Symbol을 이용한 프라이빗 속성 구현
const _private = Symbol("private");
const _secret = Symbol("secret");

class SecureClass {
  constructor(publicData, privateData) {
    this.public = publicData;
    this[_private] = privateData;
    this[_secret] = "top secret";
  }

  getPrivateData() {
    return this[_private];
  }

  getSecret() {
    return this[_secret];
  }
}

const secure = new SecureClass("public info", "private info");
console.log(secure.public); // 'public info'
console.log(secure.getPrivateData()); // 'private info'

// Symbol 속성은 일반적인 방법으로 접근 불가
console.log(Object.keys(secure)); // ['public']
console.log(Object.getOwnPropertyNames(secure)); // ['public']

// Symbol 속성 확인 방법
console.log(Object.getOwnPropertySymbols(secure)); // [Symbol(private), Symbol(secret)]
console.log(Reflect.ownKeys(secure)); // ['public', Symbol(private), Symbol(secret)]
```

## Iterator

### 1. Iterator 프로토콜

```javascript
// 기본 Iterator 구현
function createIterator(array) {
  let index = 0;

  return {
    next() {
      if (index < array.length) {
        return {
          value: array[index++],
          done: false,
        };
      } else {
        return {
          value: undefined,
          done: true,
        };
      }
    },
  };
}

const iterator = createIterator([1, 2, 3]);
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: undefined, done: true }

// Iterator를 사용하는 다양한 방법
const arr = [1, 2, 3, 4, 5];
const arrIterator = arr[Symbol.iterator]();

console.log(arrIterator.next().value); // 1
console.log(arrIterator.next().value); // 2

// 내장 Iterator들
const string = "hello";
const stringIterator = string[Symbol.iterator]();
console.log([...stringIterator]); // ['h', 'e', 'l', 'l', 'o']

const map = new Map([
  ["a", 1],
  ["b", 2],
]);
const mapIterator = map[Symbol.iterator]();
console.log([...mapIterator]); // [['a', 1], ['b', 2]]
```

### 2. 커스텀 Iterable 객체

```javascript
// 피보나치 수열 Iterator
class FibonacciSequence {
  constructor(max = Infinity) {
    this.max = max;
  }

  [Symbol.iterator]() {
    let current = 0;
    let next = 1;
    let count = 0;
    const max = this.max;

    return {
      next() {
        if (count < max) {
          const value = current;
          [current, next] = [next, current + next];
          count++;
          return { value, done: false };
        } else {
          return { done: true };
        }
      },
    };
  }
}

const fib = new FibonacciSequence(10);
console.log([...fib]); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// 무한 시퀀스도 가능 (take와 함께 사용)
function take(iterable, n) {
  const result = [];
  let count = 0;

  for (const value of iterable) {
    if (count >= n) break;
    result.push(value);
    count++;
  }

  return result;
}

const infiniteFib = new FibonacciSequence();
console.log(take(infiniteFib, 5)); // [0, 1, 1, 2, 3]

// Range Iterator
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    const step = this.step;

    return {
      next() {
        if ((step > 0 && current < end) || (step < 0 && current > end)) {
          const value = current;
          current += step;
          return { value, done: false };
        } else {
          return { done: true };
        }
      },
    };
  }
}

const range = new Range(1, 10, 2);
console.log([...range]); // [1, 3, 5, 7, 9]
```

### 3. Iterator 헬퍼 메서드

```javascript
// Iterator 유틸리티 클래스
class IteratorUtils {
  static map(iterable, mapFn) {
    return {
      [Symbol.iterator]() {
        const iterator = iterable[Symbol.iterator]();

        return {
          next() {
            const { value, done } = iterator.next();
            if (done) {
              return { done: true };
            }
            return { value: mapFn(value), done: false };
          },
        };
      },
    };
  }

  static filter(iterable, filterFn) {
    return {
      [Symbol.iterator]() {
        const iterator = iterable[Symbol.iterator]();

        return {
          next() {
            while (true) {
              const { value, done } = iterator.next();
              if (done) {
                return { done: true };
              }
              if (filterFn(value)) {
                return { value, done: false };
              }
            }
          },
        };
      },
    };
  }

  static take(iterable, n) {
    return {
      [Symbol.iterator]() {
        const iterator = iterable[Symbol.iterator]();
        let count = 0;

        return {
          next() {
            if (count >= n) {
              return { done: true };
            }
            count++;
            return iterator.next();
          },
        };
      },
    };
  }
}

// 사용 예시
const numbers = new Range(1, 100);
const evenSquares = IteratorUtils.take(
  IteratorUtils.map(
    IteratorUtils.filter(numbers, (x) => x % 2 === 0),
    (x) => x * x
  ),
  5
);

console.log([...evenSquares]); // [4, 16, 36, 64, 100]
```

## Generator

### 1. Generator 기본 개념

```javascript
// 기본 Generator 함수
function* simpleGenerator() {
  console.log("Generator started");
  yield 1;
  console.log("After first yield");
  yield 2;
  console.log("After second yield");
  yield 3;
  console.log("Generator finished");
  return "done";
}

const gen = simpleGenerator();
console.log(gen.next()); // Generator started, { value: 1, done: false }
console.log(gen.next()); // After first yield, { value: 2, done: false }
console.log(gen.next()); // After second yield, { value: 3, done: false }
console.log(gen.next()); // Generator finished, { value: 'done', done: true }

// Generator는 Iterator를 자동으로 구현
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const numGen = numberGenerator();
console.log([...numGen]); // [1, 2, 3]

// for...of 루프와 함께 사용
for (const value of numberGenerator()) {
  console.log(value); // 1, 2, 3
}
```

### 2. Generator의 고급 기능

```javascript
// yield*를 사용한 Generator 위임
function* innerGenerator() {
  yield "a";
  yield "b";
}

function* outerGenerator() {
  yield 1;
  yield* innerGenerator(); // 다른 Generator에 위임
  yield 2;
}

console.log([...outerGenerator()]); // [1, 'a', 'b', 2]

// 양방향 통신 - next()에 값 전달
function* communicatingGenerator() {
  const a = yield "First yield";
  console.log("Received:", a);

  const b = yield "Second yield";
  console.log("Received:", b);

  return a + b;
}

const commGen = communicatingGenerator();
console.log(commGen.next()); // { value: 'First yield', done: false }
console.log(commGen.next(10)); // Received: 10, { value: 'Second yield', done: false }
console.log(commGen.next(20)); // Received: 20, { value: 30, done: true }

// Generator에서 예외 처리
function* errorHandlingGenerator() {
  try {
    yield 1;
    yield 2;
    yield 3;
  } catch (error) {
    console.log("Caught error:", error.message);
    yield "error handled";
  }
}

const errorGen = errorHandlingGenerator();
console.log(errorGen.next()); // { value: 1, done: false }
console.log(errorGen.throw(new Error("Something went wrong")));
// Caught error: Something went wrong, { value: 'error handled', done: false }
```

### 3. 실용적인 Generator 활용

```javascript
// 무한 시퀀스 Generator
function* infiniteSequence(start = 0, step = 1) {
  let current = start;
  while (true) {
    yield current;
    current += step;
  }
}

// 필요한 만큼만 생성
const evenNumbers = infiniteSequence(0, 2);
console.log(evenNumbers.next().value); // 0
console.log(evenNumbers.next().value); // 2
console.log(evenNumbers.next().value); // 4

// ID Generator
function* idGenerator() {
  let id = 1;
  while (true) {
    yield `id_${id++}`;
  }
}

const getId = idGenerator();
console.log(getId.next().value); // 'id_1'
console.log(getId.next().value); // 'id_2'
console.log(getId.next().value); // 'id_3'

// 트리 순회 Generator
class TreeNode {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }

  // 중위 순회
  *inorderTraversal() {
    if (this.left) {
      yield* this.left.inorderTraversal();
    }
    yield this.value;
    if (this.right) {
      yield* this.right.inorderTraversal();
    }
  }

  // 전위 순회
  *preorderTraversal() {
    yield this.value;
    if (this.left) {
      yield* this.left.preorderTraversal();
    }
    if (this.right) {
      yield* this.right.preorderTraversal();
    }
  }
}

// 트리 생성
const root = new TreeNode(
  4,
  new TreeNode(2, new TreeNode(1), new TreeNode(3)),
  new TreeNode(6, new TreeNode(5), new TreeNode(7))
);

console.log([...root.inorderTraversal()]); // [1, 2, 3, 4, 5, 6, 7]
console.log([...root.preorderTraversal()]); // [4, 2, 1, 3, 6, 5, 7]
```

## 실무 활용 사례

### 1. 데이터 스트리밍과 지연 평가

```javascript
// 대용량 데이터 처리를 위한 Generator
function* processLargeDataset(data) {
    for (let i = 0; i < data.length; i++) {
        // 복잡한 처리 시뮬레이션
        const processed = {
            id: data[i].id,
            processedAt: new Date(),
            result: data[i].value * 2
        };

        yield processed;

        // 메모리 압박을 피하기 위한 주기적 가비지 컬렉션 힌트
        if (i % 1000 === 0) {
            // 다른 작업에 제어권 양보
            await new Promise(resolve => setTimeout(resolve, 0));
        }
    }
}

// CSV 파일 파싱 Generator
function* parseCSV(csvString) {
    const lines = csvString.split('\n');
    const headers = lines[0].split(',');

    for (let i = 1; i < lines.length; i++) {
        if (lines[i].trim()) {
            const values = lines[i].split(',');
            const row = {};

            headers.forEach((header, index) => {
                row[header.trim()] = values[index]?.trim();
            });

            yield row;
        }
    }
}

// API 페이지네이션 Generator
async function* fetchAllPages(baseUrl, pageSize = 100) {
    let page = 1;
    let hasMore = true;

    while (hasMore) {
        try {
            const response = await fetch(`${baseUrl}?page=${page}&size=${pageSize}`);
            const data = await response.json();

            if (data.items && data.items.length > 0) {
                yield* data.items;
                page++;
                hasMore = data.hasMore;
            } else {
                hasMore = false;
            }
        } catch (error) {
            console.error(`Failed to fetch page ${page}:`, error);
            hasMore = false;
        }
    }
}

// 사용 예시
async function processAllData() {
    for await (const item of fetchAllPages('/api/data')) {
        console.log('Processing item:', item.id);
        // 각 아이템 처리
    }
}
```

### 2. 상태 머신 구현

```javascript
// Generator를 이용한 상태 머신
function* stateMachine(initialState) {
  let currentState = initialState;
  let action;

  while (true) {
    switch (currentState) {
      case "idle":
        action = yield { state: "idle", message: "Ready to start" };
        if (action === "start") {
          currentState = "running";
        }
        break;

      case "running":
        action = yield { state: "running", message: "Processing..." };
        if (action === "pause") {
          currentState = "paused";
        } else if (action === "stop") {
          currentState = "stopped";
        } else if (action === "complete") {
          currentState = "completed";
        }
        break;

      case "paused":
        action = yield { state: "paused", message: "Paused" };
        if (action === "resume") {
          currentState = "running";
        } else if (action === "stop") {
          currentState = "stopped";
        }
        break;

      case "completed":
        action = yield { state: "completed", message: "Task completed" };
        if (action === "reset") {
          currentState = "idle";
        }
        break;

      case "stopped":
        action = yield { state: "stopped", message: "Stopped" };
        if (action === "reset") {
          currentState = "idle";
        }
        break;
    }
  }
}

// 상태 머신 사용
const machine = stateMachine("idle");
console.log(machine.next().value); // { state: 'idle', message: 'Ready to start' }
console.log(machine.next("start").value); // { state: 'running', message: 'Processing...' }
console.log(machine.next("pause").value); // { state: 'paused', message: 'Paused' }
console.log(machine.next("resume").value); // { state: 'running', message: 'Processing...' }
```

### 3. 비동기 Generator와 AsyncIterator

```javascript
// 비동기 Generator
async function* asyncDataGenerator() {
  const urls = [
    "https://api.example.com/data1",
    "https://api.example.com/data2",
    "https://api.example.com/data3",
  ];

  for (const url of urls) {
    try {
      // 실제로는 fetch를 사용하지만, 여기서는 시뮬레이션
      const data = await simulateAsyncFetch(url);
      yield data;
    } catch (error) {
      console.error(`Failed to fetch ${url}:`, error);
    }
  }
}

function simulateAsyncFetch(url) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ url, data: `Data from ${url}`, timestamp: Date.now() });
    }, Math.random() * 1000);
  });
}

// 비동기 Iterator 사용
async function processAsyncData() {
  for await (const data of asyncDataGenerator()) {
    console.log("Received:", data);
  }
}

// 병렬 처리를 위한 비동기 Generator
async function* parallelAsyncGenerator(urls, concurrency = 3) {
  const results = [];
  const inProgress = new Set();

  let index = 0;

  while (index < urls.length || inProgress.size > 0) {
    // 동시 실행 수만큼 요청 시작
    while (inProgress.size < concurrency && index < urls.length) {
      const url = urls[index++];
      const promise = simulateAsyncFetch(url)
        .then((result) => ({ result, url }))
        .catch((error) => ({ error, url }));

      inProgress.add(promise);

      promise.finally(() => {
        inProgress.delete(promise);
      });
    }

    // 가장 먼저 완료되는 요청 대기
    if (inProgress.size > 0) {
      const completed = await Promise.race(inProgress);

      if (completed.error) {
        console.error(`Error fetching ${completed.url}:`, completed.error);
      } else {
        yield completed.result;
      }
    }
  }
}
```

## 고급 패턴과 최적화

### 1. Symbol을 이용한 메타데이터 관리

```javascript
// 메타데이터 관리 시스템
const METADATA = Symbol("metadata");
const VALIDATORS = Symbol("validators");
const OBSERVERS = Symbol("observers");

class MetaClass {
  constructor() {
    this[METADATA] = new Map();
    this[VALIDATORS] = new Map();
    this[OBSERVERS] = new Set();
  }

  setMetadata(key, value) {
    this[METADATA].set(key, value);
    this.notifyObservers("metadata:set", { key, value });
  }

  getMetadata(key) {
    return this[METADATA].get(key);
  }

  addValidator(property, validator) {
    if (!this[VALIDATORS].has(property)) {
      this[VALIDATORS].set(property, []);
    }
    this[VALIDATORS].get(property).push(validator);
  }

  validate(property, value) {
    const validators = this[VALIDATORS].get(property) || [];
    return validators.every((validator) => validator(value));
  }

  addObserver(observer) {
    this[OBSERVERS].add(observer);
  }

  removeObserver(observer) {
    this[OBSERVERS].delete(observer);
  }

  notifyObservers(event, data) {
    this[OBSERVERS].forEach((observer) => {
      if (typeof observer === "function") {
        observer(event, data);
      }
    });
  }
}

// 사용 예시
const meta = new MetaClass();

meta.addObserver((event, data) => {
  console.log(`Event: ${event}`, data);
});

meta.addValidator("age", (value) => value >= 0 && value <= 150);
meta.addValidator("email", (value) => value.includes("@"));

console.log(meta.validate("age", 25)); // true
console.log(meta.validate("age", -5)); // false

meta.setMetadata("version", "1.0.0");
meta.setMetadata("author", "Developer");
```

### 2. Generator 기반 코루틴

```javascript
// 코루틴 스케줄러
class CoroutineScheduler {
  constructor() {
    this.tasks = [];
    this.running = false;
  }

  spawn(generatorFunction, ...args) {
    const generator = generatorFunction(...args);
    this.tasks.push({
      id: Date.now() + Math.random(),
      generator,
      status: "ready",
    });

    if (!this.running) {
      this.run();
    }
  }

  async run() {
    this.running = true;

    while (this.tasks.length > 0) {
      for (let i = this.tasks.length - 1; i >= 0; i--) {
        const task = this.tasks[i];

        try {
          const { value, done } = task.generator.next();

          if (done) {
            this.tasks.splice(i, 1);
            console.log(`Task ${task.id} completed`);
          } else if (value instanceof Promise) {
            // 비동기 작업 대기
            task.status = "waiting";
            value
              .then((result) => {
                task.generator.next(result);
                task.status = "ready";
              })
              .catch((error) => {
                task.generator.throw(error);
              });
          }
        } catch (error) {
          console.error(`Task ${task.id} failed:`, error);
          this.tasks.splice(i, 1);
        }
      }

      // 다른 작업에 제어권 양보
      await new Promise((resolve) => setTimeout(resolve, 10));
    }

    this.running = false;
  }
}

// 코루틴 함수 예시
function* fetchAndProcess(url) {
  console.log(`Starting to fetch ${url}`);

  try {
    const response = yield fetch(url);
    const data = yield response.json();

    console.log(`Processing data from ${url}`);

    // 데이터 처리 시뮬레이션
    for (let i = 0; i < data.length; i++) {
      yield new Promise((resolve) => setTimeout(resolve, 100));
      console.log(`Processed item ${i + 1}/${data.length}`);
    }

    return `Completed processing ${url}`;
  } catch (error) {
    console.error(`Error processing ${url}:`, error);
    throw error;
  }
}

// 사용 예시
const scheduler = new CoroutineScheduler();
scheduler.spawn(fetchAndProcess, "https://api.example.com/data1");
scheduler.spawn(fetchAndProcess, "https://api.example.com/data2");
```

### 3. 성능 최적화 패턴

```javascript
// 지연 평가를 위한 Iterator 체이닝
class LazyIterator {
    constructor(iterable) {
        this.iterable = iterable;
    }

    static from(iterable) {
        return new LazyIterator(iterable);
    }

    map(fn) {
        const iterable = this.iterable;
        return new LazyIterator({
            *[Symbol.iterator]() {
                for (const item of iterable) {
                    yield fn(item);
                }
            }
        });
    }

    filter(predicate) {
        const iterable = this.iterable;
        return new LazyIterator({
            *[Symbol.iterator]() {
                for (const item of iterable) {
                    if (predicate(item)) {
                        yield item;
                    }
                }
            }
        });
    }

    take(n) {
        const iterable = this.iterable;
        return new LazyIterator({
            *[Symbol.iterator]() {
                let count = 0;
                for (const item of iterable) {
                    if (count >= n) break;
                    yield item;
                    count++;
                }
            }
        });
    }

    toArray() {
        return [...this.iterable];
    }

    forEach(fn) {
        for (const item of this.iterable) {
            fn(item);
        }
    }
}

// 메모이제이션을 위한 Symbol 키
const MEMO_CACHE = Symbol('memoCache');

function memoizeGenerator(generatorFn) {
    return function* (...args) {
        const key = JSON.stringify(args);

        if (!this[MEMO_CACHE]) {
            this[MEMO_CACHE] = new Map();
        }

        if (this[MEMO_CACHE].has(key)) {
            yield* this[MEMO_CACHE].get(key);
            return;
        }

        const results = [];
        const generator = generatorFn.apply(this, args);

        for (const value of generator) {
            results.push(value);
            yield value;
        }

        this[MEMO_CACHE].set(key, results);
    };
}

// 사용 예시
class FibonacciGenerator {
    *generate = memoizeGenerator(function* (n) {
        let a = 0, b = 1;
        for (let i = 0; i < n; i++) {
            yield a;
            [a, b] = [b, a + b];
        }
    });
}

const fibGen = new FibonacciGenerator();
console.log([...fibGen.generate(10)]); // 계산됨
console.log([...fibGen.generate(10)]); // 캐시에서 반환
```

## 결론

Symbol, Iterator, Generator는 JavaScript의 메타프로그래밍 능력을 크게 확장시키는 강력한 도구들입니다. Symbol을 통해 객체의 동작을 세밀하게 제어하고, Iterator로 커스텀 반복 패턴을 구현하며, Generator로 함수 실행을 제어하고 비동기 프로그래밍을 개선할 수 있습니다. 이들을 적절히 활용하면 더욱 유연하고 효율적인 JavaScript 코드를 작성할 수 있습니다.
