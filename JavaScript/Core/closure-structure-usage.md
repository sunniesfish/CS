# 클로저(Closure)의 구조와 용도

## 개요

클로저(Closure)는 함수가 선언될 때의 렉시컬 환경(Lexical Environment)을 기억하여, 함수가 실행되는 시점에 그 환경에 접근할 수 있게 하는 JavaScript의 핵심 개념입니다. 클로저는 데이터 은닉, 모듈 패턴, 콜백 함수 등 다양한 고급 프로그래밍 패턴의 기초가 됩니다.

## 탄생 배경

클로저는 함수형 프로그래밍 언어에서 유래된 개념으로, JavaScript에서는 다음과 같은 필요에 의해 중요해졌습니다:

- **데이터 캡슐화**: 전역 네임스페이스 오염 방지
- **상태 관리**: 함수 간 상태 공유 및 보존
- **모듈 패턴**: 프라이빗 변수와 메서드 구현
- **이벤트 핸들링**: 컨텍스트 정보 보존
- **함수형 프로그래밍**: 고차 함수와 커링 구현

## 클로저의 기본 구조

### 1. 기본적인 클로저

```javascript
function outerFunction(x) {
  // 외부 함수의 변수
  const outerVariable = x;

  // 내부 함수 (클로저)
  function innerFunction(y) {
    // 외부 함수의 변수에 접근
    return outerVariable + y;
  }

  // 내부 함수 반환
  return innerFunction;
}

// 클로저 생성
const closure = outerFunction(10);

// 클로저 실행 (outerVariable = 10이 보존됨)
console.log(closure(5)); // 15

// 다른 클로저 생성 (독립적인 환경)
const anotherClosure = outerFunction(20);
console.log(anotherClosure(5)); // 25
```

### 2. 클로저의 메모리 구조

```javascript
function createCounter() {
  let count = 0; // 프라이빗 변수

  return {
    increment: function () {
      count++; // 클로저로 count에 접근
      return count;
    },

    decrement: function () {
      count--; // 동일한 count 변수 공유
      return count;
    },

    getCount: function () {
      return count; // 읽기 전용 접근
    },
  };
}

const counter1 = createCounter();
const counter2 = createCounter();

console.log(counter1.increment()); // 1
console.log(counter1.increment()); // 2
console.log(counter2.increment()); // 1 (독립적인 count 변수)

console.log(counter1.getCount()); // 2
console.log(counter2.getCount()); // 1
```

### 3. 클로저와 스코프 체인

```javascript
const globalVar = "global";

function level1(param1) {
  const level1Var = "level1";

  function level2(param2) {
    const level2Var = "level2";

    function level3(param3) {
      const level3Var = "level3";

      // 모든 상위 스코프에 접근 가능
      return {
        global: globalVar,
        level1: level1Var,
        level2: level2Var,
        level3: level3Var,
        params: [param1, param2, param3],
      };
    }

    return level3;
  }

  return level2;
}

const deepClosure = level1("p1")("p2")("p3");
console.log(deepClosure);
// {
//   global: 'global',
//   level1: 'level1',
//   level2: 'level2',
//   level3: 'level3',
//   params: ['p1', 'p2', 'p3']
// }
```

## 클로저의 실용적 활용

### 1. 모듈 패턴 (Module Pattern)

```javascript
const BankAccount = (function () {
  // 프라이빗 변수들
  let balance = 0;
  let accountNumber = Math.random().toString(36).substr(2, 9);
  const transactions = [];

  // 프라이빗 함수들
  function validateAmount(amount) {
    return typeof amount === "number" && amount > 0;
  }

  function addTransaction(type, amount) {
    transactions.push({
      type,
      amount,
      timestamp: new Date(),
      balance: balance,
    });
  }

  // 퍼블릭 API 반환
  return {
    deposit: function (amount) {
      if (validateAmount(amount)) {
        balance += amount;
        addTransaction("deposit", amount);
        return balance;
      }
      throw new Error("Invalid amount");
    },

    withdraw: function (amount) {
      if (validateAmount(amount) && balance >= amount) {
        balance -= amount;
        addTransaction("withdraw", amount);
        return balance;
      }
      throw new Error("Invalid amount or insufficient funds");
    },

    getBalance: function () {
      return balance;
    },

    getAccountNumber: function () {
      return accountNumber;
    },

    getTransactionHistory: function () {
      return transactions.slice(); // 복사본 반환
    },
  };
})();

console.log(BankAccount.deposit(100)); // 100
console.log(BankAccount.withdraw(30)); // 70
console.log(BankAccount.getBalance()); // 70
console.log(BankAccount.getTransactionHistory());
```

### 2. 팩토리 함수 패턴

```javascript
function createUser(name, role) {
  // 프라이빗 데이터
  let userData = {
    name: name,
    role: role,
    loginCount: 0,
    lastLogin: null,
    isActive: true,
  };

  // 프라이빗 메서드
  function validateRole(newRole) {
    const validRoles = ["admin", "user", "guest"];
    return validRoles.includes(newRole);
  }

  function log(action) {
    console.log(`[${new Date().toISOString()}] ${userData.name}: ${action}`);
  }

  // 퍼블릭 인터페이스
  return {
    getName: function () {
      return userData.name;
    },

    getRole: function () {
      return userData.role;
    },

    login: function () {
      if (!userData.isActive) {
        throw new Error("Account is deactivated");
      }

      userData.loginCount++;
      userData.lastLogin = new Date();
      log("logged in");

      return {
        success: true,
        loginCount: userData.loginCount,
      };
    },

    logout: function () {
      log("logged out");
    },

    changeRole: function (newRole) {
      if (validateRole(newRole)) {
        const oldRole = userData.role;
        userData.role = newRole;
        log(`role changed from ${oldRole} to ${newRole}`);
        return true;
      }
      return false;
    },

    deactivate: function () {
      userData.isActive = false;
      log("account deactivated");
    },

    getStats: function () {
      return {
        loginCount: userData.loginCount,
        lastLogin: userData.lastLogin,
        isActive: userData.isActive,
      };
    },
  };
}

const user = createUser("Alice", "admin");
console.log(user.login()); // { success: true, loginCount: 1 }
console.log(user.changeRole("user")); // true
console.log(user.getStats());
```

### 3. 이벤트 핸들러와 클로저

```javascript
function setupEventHandlers() {
  const buttons = document.querySelectorAll(".counter-button");

  buttons.forEach((button, index) => {
    // 각 버튼마다 독립적인 카운터 클로저 생성
    let count = 0;

    button.addEventListener("click", function () {
      count++; // 클로저로 보존된 count 변수
      button.textContent = `Clicked ${count} times`;

      // 컨텍스트 정보도 클로저로 보존
      console.log(`Button ${index} clicked ${count} times`);
    });
  });
}

// DOM이 로드된 후 실행
// document.addEventListener('DOMContentLoaded', setupEventHandlers);
```

### 4. 비동기 처리와 클로저

```javascript
function createAsyncProcessor() {
  let processingQueue = [];
  let isProcessing = false;

  function processNext() {
    if (processingQueue.length === 0) {
      isProcessing = false;
      return;
    }

    isProcessing = true;
    const { data, callback } = processingQueue.shift();

    // 비동기 작업 시뮬레이션
    setTimeout(() => {
      const result = data.toUpperCase(); // 간단한 처리
      callback(null, result);
      processNext(); // 다음 작업 처리
    }, Math.random() * 1000);
  }

  return {
    process: function (data, callback) {
      processingQueue.push({ data, callback });

      if (!isProcessing) {
        processNext();
      }
    },

    getQueueLength: function () {
      return processingQueue.length;
    },

    isProcessing: function () {
      return isProcessing;
    },
  };
}

const processor = createAsyncProcessor();

processor.process("hello", (err, result) => {
  console.log("Result 1:", result); // 'HELLO'
});

processor.process("world", (err, result) => {
  console.log("Result 2:", result); // 'WORLD'
});
```

## 고급 클로저 패턴

### 1. 커링 (Currying)

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function (...args2) {
        return curried.apply(this, args.concat(args2));
      };
    }
  };
}

// 사용 예시
const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6

// 부분 적용 활용
const addFive = curriedAdd(5);
const addFiveAndTwo = addFive(2);

console.log(addFiveAndTwo(3)); // 10
console.log(addFiveAndTwo(7)); // 14
```

### 2. 메모이제이션 (Memoization)

```javascript
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log("Cache hit for:", key);
      return cache.get(key);
    }

    console.log("Computing for:", key);
    const result = fn.apply(this, args);
    cache.set(key, result);

    return result;
  };
}

// 피보나치 함수 메모이제이션
const fibonacci = memoize(function (n) {
  if (n < 2) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10)); // 55 (캐시 미스들과 함께)
console.log(fibonacci(10)); // 55 (캐시 히트)
console.log(fibonacci(11)); // 89 (일부 캐시 히트)
```

### 3. 함수 조합 (Function Composition)

```javascript
function compose(...functions) {
  return function (value) {
    return functions.reduceRight((acc, fn) => fn(acc), value);
  };
}

function pipe(...functions) {
  return function (value) {
    return functions.reduce((acc, fn) => fn(acc), value);
  };
}

// 헬퍼 함수들
const addOne = (x) => x + 1;
const double = (x) => x * 2;
const square = (x) => x * x;

// 함수 조합 사용
const composedFunction = compose(square, double, addOne);
const pipedFunction = pipe(addOne, double, square);

console.log(composedFunction(3)); // square(double(addOne(3))) = square(8) = 64
console.log(pipedFunction(3)); // square(double(addOne(3))) = square(8) = 64
```

### 4. 상태 머신 패턴

```javascript
function createStateMachine(initialState, transitions) {
  let currentState = initialState;
  const listeners = [];

  function notifyListeners(oldState, newState, event) {
    listeners.forEach((listener) => {
      listener(oldState, newState, event);
    });
  }

  return {
    getState: function () {
      return currentState;
    },

    transition: function (event) {
      const stateTransitions = transitions[currentState];

      if (stateTransitions && stateTransitions[event]) {
        const oldState = currentState;
        currentState = stateTransitions[event];
        notifyListeners(oldState, currentState, event);
        return true;
      }

      return false; // 유효하지 않은 전환
    },

    addListener: function (listener) {
      listeners.push(listener);
    },

    removeListener: function (listener) {
      const index = listeners.indexOf(listener);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    },
  };
}

// 사용 예시: 간단한 트래픽 라이트
const trafficLight = createStateMachine("red", {
  red: { next: "green" },
  green: { next: "yellow" },
  yellow: { next: "red" },
});

trafficLight.addListener((oldState, newState, event) => {
  console.log(
    `Traffic light changed from ${oldState} to ${newState} (event: ${event})`
  );
});

console.log(trafficLight.getState()); // 'red'
trafficLight.transition("next"); // red -> green
trafficLight.transition("next"); // green -> yellow
trafficLight.transition("next"); // yellow -> red
```

## 클로저와 메모리 관리

### 1. 메모리 누수 방지

```javascript
// 문제가 있는 클로저 (메모리 누수)
function createLeakyFunction() {
  const largeData = new Array(1000000).fill("data");
  const smallData = "small";

  return function () {
    // largeData를 사용하지 않지만 클로저로 인해 메모리에 유지됨
    return smallData;
  };
}

// 개선된 버전
function createOptimizedFunction() {
  const largeData = new Array(1000000).fill("data");
  const smallData = "small";

  // 필요한 데이터만 추출
  const processedData = smallData;

  return function () {
    return processedData; // 큰 데이터는 GC될 수 있음
  };
}

// 명시적 정리가 필요한 경우
function createCleanableFunction() {
  let largeData = new Array(1000000).fill("data");

  const fn = function () {
    return largeData ? largeData.length : 0;
  };

  fn.cleanup = function () {
    largeData = null; // 명시적 해제
  };

  return fn;
}

const cleanable = createCleanableFunction();
console.log(cleanable()); // 1000000
cleanable.cleanup(); // 메모리 해제
console.log(cleanable()); // 0
```

### 2. WeakMap을 이용한 프라이빗 데이터

```javascript
const privateData = new WeakMap();

function createSecureObject(name) {
  // WeakMap을 이용한 프라이빗 데이터 저장
  privateData.set(this, {
    name: name,
    secret: Math.random().toString(36),
    accessCount: 0,
  });

  return {
    getName: function () {
      const data = privateData.get(this);
      data.accessCount++;
      return data.name;
    },

    getAccessCount: function () {
      return privateData.get(this).accessCount;
    },

    // secret은 외부에서 접근 불가
    toString: function () {
      const data = privateData.get(this);
      return `SecureObject(${data.name})`;
    },
  };
}

const secure = createSecureObject.call({}, "test");
console.log(secure.getName()); // 'test'
console.log(secure.getAccessCount()); // 1
console.log(secure.toString()); // 'SecureObject(test)'
```

## 클로저 디버깅과 최적화

### 1. 클로저 시각화

```javascript
function analyzeClosures() {
  const analysis = [];

  function createAnalyzableFunction(name) {
    const creationTime = Date.now();
    let callCount = 0;

    const fn = function (...args) {
      callCount++;
      const callTime = Date.now();

      analysis.push({
        functionName: name,
        creationTime: creationTime,
        callTime: callTime,
        callCount: callCount,
        args: args,
      });

      return `${name} called ${callCount} times`;
    };

    fn.getAnalysis = () => analysis.filter((a) => a.functionName === name);

    return fn;
  }

  return {
    create: createAnalyzableFunction,
    getFullAnalysis: () => analysis,
  };
}

const analyzer = analyzeClosures();
const fn1 = analyzer.create("function1");
const fn2 = analyzer.create("function2");

fn1("arg1");
fn2("arg2");
fn1("arg3");

console.log(analyzer.getFullAnalysis());
```

### 2. 성능 측정

```javascript
function benchmarkClosures() {
  const iterations = 1000000;

  // 클로저 사용
  function withClosure() {
    let count = 0;
    return function () {
      return ++count;
    };
  }

  // 객체 사용
  function withObject() {
    return {
      count: 0,
      increment: function () {
        return ++this.count;
      },
    };
  }

  // 클래스 사용
  class WithClass {
    constructor() {
      this.count = 0;
    }

    increment() {
      return ++this.count;
    }
  }

  // 벤치마크 실행
  console.time("Closure");
  const closureFn = withClosure();
  for (let i = 0; i < iterations; i++) {
    closureFn();
  }
  console.timeEnd("Closure");

  console.time("Object");
  const objectFn = withObject();
  for (let i = 0; i < iterations; i++) {
    objectFn.increment();
  }
  console.timeEnd("Object");

  console.time("Class");
  const classFn = new WithClass();
  for (let i = 0; i < iterations; i++) {
    classFn.increment();
  }
  console.timeEnd("Class");
}

// benchmarkClosures();
```

## 실무에서의 클로저 활용 사례

### 1. React Hooks 스타일 구현

```javascript
function createReactLikeHooks() {
  let currentIndex = 0;
  let hooks = [];

  function useState(initialValue) {
    const hookIndex = currentIndex++;

    if (hooks[hookIndex] === undefined) {
      hooks[hookIndex] = initialValue;
    }

    const setState = (newValue) => {
      hooks[hookIndex] =
        typeof newValue === "function" ? newValue(hooks[hookIndex]) : newValue;
    };

    return [hooks[hookIndex], setState];
  }

  function useEffect(effect, dependencies) {
    const hookIndex = currentIndex++;
    const prevDeps = hooks[hookIndex];

    const hasChanged =
      !prevDeps || dependencies.some((dep, i) => dep !== prevDeps[i]);

    if (hasChanged) {
      hooks[hookIndex] = dependencies;
      effect();
    }
  }

  function resetHooks() {
    currentIndex = 0;
  }

  return { useState, useEffect, resetHooks };
}

// 사용 예시
const { useState, useEffect, resetHooks } = createReactLikeHooks();

function MyComponent() {
  resetHooks(); // 컴포넌트 렌더링 시작

  const [count, setCount] = useState(0);
  const [name, setName] = useState("React");

  useEffect(() => {
    console.log(`Count changed to ${count}`);
  }, [count]);

  return {
    increment: () => setCount((c) => c + 1),
    setName: setName,
    getState: () => ({ count, name }),
  };
}

const component = MyComponent();
console.log(component.getState()); // { count: 0, name: 'React' }
component.increment();
console.log(component.getState()); // { count: 1, name: 'React' }
```

### 2. 플러그인 시스템

```javascript
function createPluginSystem() {
  const plugins = new Map();
  const hooks = new Map();

  return {
    registerPlugin: function (name, plugin) {
      if (plugins.has(name)) {
        throw new Error(`Plugin ${name} already registered`);
      }

      plugins.set(name, plugin);

      // 플러그인 초기화 (클로저로 시스템 접근)
      if (typeof plugin.init === "function") {
        plugin.init({
          addHook: (hookName, callback) => {
            if (!hooks.has(hookName)) {
              hooks.set(hookName, []);
            }
            hooks.get(hookName).push(callback);
          },

          removeHook: (hookName, callback) => {
            if (hooks.has(hookName)) {
              const callbacks = hooks.get(hookName);
              const index = callbacks.indexOf(callback);
              if (index > -1) {
                callbacks.splice(index, 1);
              }
            }
          },
        });
      }
    },

    executeHook: function (hookName, ...args) {
      if (hooks.has(hookName)) {
        const results = [];
        for (const callback of hooks.get(hookName)) {
          try {
            const result = callback(...args);
            results.push(result);
          } catch (error) {
            console.error(`Hook ${hookName} error:`, error);
          }
        }
        return results;
      }
      return [];
    },

    getPlugin: function (name) {
      return plugins.get(name);
    },

    listPlugins: function () {
      return Array.from(plugins.keys());
    },
  };
}

// 플러그인 예시
const pluginSystem = createPluginSystem();

const loggerPlugin = {
  init: function (system) {
    system.addHook("beforeAction", (action) => {
      console.log(`[Logger] Before action: ${action}`);
    });

    system.addHook("afterAction", (action, result) => {
      console.log(`[Logger] After action: ${action}, result: ${result}`);
    });
  },
};

pluginSystem.registerPlugin("logger", loggerPlugin);

// 사용
pluginSystem.executeHook("beforeAction", "save");
pluginSystem.executeHook("afterAction", "save", "success");
```

## 결론

클로저는 JavaScript의 가장 강력하고 유용한 기능 중 하나로, 데이터 은닉, 모듈화, 함수형 프로그래밍, 비동기 처리 등 다양한 고급 패턴의 기초가 됩니다. 클로저를 올바르게 이해하고 활용하면 더 안전하고 유지보수하기 쉬운 코드를 작성할 수 있지만, 메모리 누수에 주의하고 성능을 고려한 사용이 필요합니다.
