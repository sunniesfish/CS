# 실행 컨텍스트와 스코프 체인 (Execution Context & Scope Chain)

## 개요

실행 컨텍스트(Execution Context)는 JavaScript 코드가 실행되는 환경을 추상화한 개념으로, 변수, 함수, 객체 등이 어떻게 접근되고 관리되는지를 정의합니다. 스코프 체인(Scope Chain)은 변수나 함수의 식별자를 해결하기 위한 메커니즘입니다.

## 탄생 배경

JavaScript가 설계될 때, 동적 타입 언어의 유연성을 제공하면서도 변수와 함수의 접근 범위를 체계적으로 관리할 필요가 있었습니다:

- **변수 충돌 방지**: 같은 이름의 변수가 다른 범위에서 사용될 때의 충돌 해결
- **메모리 효율성**: 필요 없는 변수들의 자동 해제
- **보안**: 외부에서 접근하면 안 되는 변수들의 은닉
- **코드 구조화**: 함수와 블록 단위의 논리적 분리

## 실행 컨텍스트의 구조

### 1. 실행 컨텍스트의 종류

#### 전역 실행 컨텍스트 (Global Execution Context)

```javascript
// 전역 실행 컨텍스트
var globalVar = "I am global";

function globalFunction() {
  console.log("Global function");
}

// 전역 객체에 바인딩됨
console.log(window.globalVar); // 브라우저에서 'I am global'
console.log(global.globalVar); // Node.js에서 'I am global'
```

#### 함수 실행 컨텍스트 (Function Execution Context)

```javascript
function outerFunction(x) {
  var outerVar = "I am outer";

  function innerFunction(y) {
    var innerVar = "I am inner";
    console.log(x, y, outerVar, innerVar);
  }

  return innerFunction;
}

const closure = outerFunction("outer param");
closure("inner param"); // 새로운 함수 실행 컨텍스트 생성
```

#### 평가 실행 컨텍스트 (Eval Execution Context)

```javascript
// eval로 생성되는 컨텍스트 (사용 권장하지 않음)
var evalVar = "global";

function testEval() {
  var evalVar = "local";
  eval("console.log(evalVar)"); // 'local' - 현재 스코프 사용
}
```

### 2. 실행 컨텍스트의 구성 요소

#### Variable Environment (변수 환경)

```javascript
function example() {
  // Variable Environment에 저장
  var hoistedVar; // undefined로 초기화
  let blockScoped; // TDZ 상태
  const constant = "value";

  function hoistedFunction() {
    return "I am hoisted";
  }
}
```

#### Lexical Environment (렉시컬 환경)

```javascript
function createCounter() {
  let count = 0;

  return {
    increment: function () {
      count++; // 렉시컬 환경의 count에 접근
      return count;
    },
    decrement: function () {
      count--; // 동일한 count 변수
      return count;
    },
    getCount: function () {
      return count; // 클로저로 보존된 count
    },
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount()); // 2
```

#### this 바인딩

```javascript
const obj = {
  name: "Object",
  method: function () {
    console.log(this.name); // 실행 컨텍스트의 this 바인딩

    function innerFunction() {
      console.log(this.name); // 전역 객체 또는 undefined (strict mode)
    }

    const arrowFunction = () => {
      console.log(this.name); // 렉시컬 this (obj.name)
    };

    innerFunction();
    arrowFunction();
  },
};

obj.method();
```

## 스코프 체인의 동작 원리

### 1. 스코프 체인 구성

```javascript
var globalScope = "global";

function outerFunction() {
  var outerScope = "outer";

  function middleFunction() {
    var middleScope = "middle";

    function innerFunction() {
      var innerScope = "inner";

      // 스코프 체인: inner → middle → outer → global
      console.log(innerScope); // 'inner' - 현재 스코프
      console.log(middleScope); // 'middle' - 상위 스코프
      console.log(outerScope); // 'outer' - 더 상위 스코프
      console.log(globalScope); // 'global' - 전역 스코프
    }

    return innerFunction;
  }

  return middleFunction;
}

const nested = outerFunction()();
nested();
```

### 2. 변수 해결 과정 (Variable Resolution)

```javascript
function demonstrateResolution() {
  var name = "function scope";

  if (true) {
    let name = "block scope"; // 블록 스코프

    function inner() {
      var name = "inner function"; // 함수 스코프
      console.log(name); // 'inner function'
    }

    inner();
    console.log(name); // 'block scope'
  }

  console.log(name); // 'function scope'
}

demonstrateResolution();
```

### 3. 스코프 체인과 클로저

```javascript
function createMultiplier(multiplier) {
  // 외부 함수의 렉시컬 환경

  return function (number) {
    // 내부 함수는 외부 함수의 스코프를 기억
    return number * multiplier; // multiplier는 클로저로 보존
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10 - multiplier=2가 보존됨
console.log(triple(5)); // 15 - multiplier=3이 보존됨

// 각각 독립적인 렉시컬 환경을 가짐
console.log(double.toString()); // multiplier 참조를 확인할 수 있음
```

## 실행 컨텍스트 스택 (Call Stack)

### 1. 스택의 동작

```javascript
function first() {
  console.log("First function start");
  second();
  console.log("First function end");
}

function second() {
  console.log("Second function start");
  third();
  console.log("Second function end");
}

function third() {
  console.log("Third function");
}

// 실행 순서:
first();
// Call Stack:
// 1. Global Execution Context
// 2. first() - pushed
// 3. second() - pushed
// 4. third() - pushed, then popped
// 5. second() - popped
// 6. first() - popped
// 7. Global - remains
```

### 2. 스택 오버플로우

```javascript
function recursiveFunction(n) {
  console.log(`Recursion level: ${n}`);

  if (n > 10000) {
    throw new Error("Maximum recursion reached");
  }

  return recursiveFunction(n + 1); // 스택에 계속 쌓임
}

// 주의: 실제로는 RangeError: Maximum call stack size exceeded
// recursiveFunction(1);

// 해결책: 꼬리 재귀 최적화 (일부 엔진에서만 지원)
function tailRecursive(n, accumulator = 0) {
  if (n === 0) return accumulator;
  return tailRecursive(n - 1, accumulator + n); // 꼬리 호출
}
```

## 렉시컬 스코핑 (Lexical Scoping)

### 1. 정적 스코핑의 특성

```javascript
var x = "global x";

function outer() {
  var x = "outer x";

  function inner() {
    console.log(x); // 'outer x' - 정의된 위치 기준
  }

  return inner;
}

function caller() {
  var x = "caller x";
  var innerFunc = outer();
  innerFunc(); // 'outer x' - 호출 위치가 아닌 정의 위치 기준
}

caller();
```

### 2. 동적 스코핑과의 비교

```javascript
// JavaScript는 렉시컬 스코핑을 사용
function lexicalExample() {
  var message = "lexical";

  function showMessage() {
    console.log(message); // 정의 시점의 스코프 사용
  }

  function callWithDifferentContext() {
    var message = "different context";
    showMessage(); // 여전히 'lexical' 출력
  }

  callWithDifferentContext();
}

lexicalExample();

// with 문은 동적 스코핑 유사 동작 (strict mode에서 금지)
// var obj = { x: 10 };
// with (obj) {
//     console.log(x); // obj.x를 참조
// }
```

## 블록 스코프와 함수 스코프

### 1. var vs let/const

```javascript
function scopeComparison() {
  console.log("=== Function Scope (var) ===");

  for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log("var i:", i), 100); // 3, 3, 3
  }

  console.log("=== Block Scope (let) ===");

  for (let j = 0; j < 3; j++) {
    setTimeout(() => console.log("let j:", j), 200); // 0, 1, 2
  }

  console.log("=== IIFE Pattern (var) ===");

  for (var k = 0; k < 3; k++) {
    (function (index) {
      setTimeout(() => console.log("IIFE k:", index), 300); // 0, 1, 2
    })(k);
  }
}

scopeComparison();
```

### 2. 블록 스코프 활용

```javascript
function blockScopeExample() {
  let message = "outer";

  if (true) {
    let message = "inner"; // 새로운 블록 스코프
    const constant = "block constant";

    console.log(message); // 'inner'

    {
      let message = "nested block"; // 더 깊은 블록 스코프
      console.log(message); // 'nested block'
    }

    console.log(message); // 'inner'
  }

  console.log(message); // 'outer'
  // console.log(constant); // ReferenceError
}

blockScopeExample();
```

## 모듈 스코프

### 1. ES6 모듈 스코프

```javascript
// moduleA.js
let modulePrivate = "private to module";
export let modulePublic = "public from module";

export function getPrivate() {
  return modulePrivate; // 모듈 스코프 내에서 접근 가능
}

// main.js
import { modulePublic, getPrivate } from "./moduleA.js";

console.log(modulePublic); // 'public from module'
console.log(getPrivate()); // 'private to module'
// console.log(modulePrivate); // ReferenceError
```

### 2. CommonJS 모듈 스코프

```javascript
// moduleB.js (Node.js)
(function (exports, require, module, __filename, __dirname) {
  // 모듈은 함수로 래핑되어 독립적인 스코프를 가짐
  let privateVar = "module private";

  function privateFunction() {
    return privateVar;
  }

  module.exports = {
    publicFunction: function () {
      return privateFunction(); // 모듈 스코프 내 접근
    },
  };
});
```

## 성능 최적화와 메모리 관리

### 1. 스코프 체인 최적화

```javascript
// 비효율적: 긴 스코프 체인
function inefficient() {
  return function () {
    return function () {
      return function () {
        return function () {
          return globalVariable; // 깊은 스코프 체인 탐색
        };
      };
    };
  };
}

// 효율적: 지역 변수 캐싱
function efficient() {
  const localGlobal = globalVariable; // 스코프 체인 탐색 최소화

  return function () {
    return function () {
      return function () {
        return function () {
          return localGlobal; // 지역 변수 접근
        };
      };
    };
  };
}
```

### 2. 메모리 누수 방지

```javascript
function createHandler() {
  const largeData = new Array(1000000).fill("data");

  return function handler(event) {
    // largeData가 클로저로 인해 메모리에 유지됨
    console.log("Handler called");
    // largeData를 실제로 사용하지 않으면 메모리 낭비
  };
}

// 개선된 버전
function createOptimizedHandler() {
  const largeData = new Array(1000000).fill("data");
  const necessaryData = largeData.slice(0, 10); // 필요한 부분만 추출

  return function handler(event) {
    console.log("Handler called");
    // 필요한 데이터만 클로저로 유지
    return necessaryData;
  };
}

// 명시적 정리
function createCleanableHandler() {
  let largeData = new Array(1000000).fill("data");

  const handler = function (event) {
    if (largeData) {
      console.log("Handler with data");
    }
  };

  handler.cleanup = function () {
    largeData = null; // 메모리 해제
  };

  return handler;
}
```

## 디버깅과 개발 도구 활용

### 1. 스코프 체인 시각화

```javascript
function debugScope() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";

    // Chrome DevTools에서 Scope 패널 확인 가능
    debugger; // 브레이크포인트 설정

    console.log("Debugging scope chain");
  }

  inner();
}

// debugScope();
```

### 2. 스코프 분석 도구

```javascript
// 스코프 체인 분석을 위한 헬퍼 함수
function analyzeScopeChain() {
  const analysis = [];

  function addToAnalysis(scopeName, variables) {
    analysis.push({
      scope: scopeName,
      variables: variables,
      timestamp: Date.now(),
    });
  }

  // 전역 스코프 분석
  addToAnalysis("global", Object.keys(window || global));

  function outerFunction() {
    const outerVar = "outer";
    addToAnalysis("outer", ["outerVar"]);

    function innerFunction() {
      const innerVar = "inner";
      addToAnalysis("inner", ["innerVar"]);

      return analysis;
    }

    return innerFunction;
  }

  return outerFunction()();
}

console.log(analyzeScopeChain());
```

## 실제 활용 사례

### 1. 모듈 패턴

```javascript
const MyModule = (function () {
  // 프라이빗 변수와 함수
  let privateCounter = 0;
  const privateArray = [];

  function privateFunction() {
    return "This is private";
  }

  // 퍼블릭 API 반환
  return {
    increment: function () {
      privateCounter++;
    },

    decrement: function () {
      privateCounter--;
    },

    getCount: function () {
      return privateCounter;
    },

    addItem: function (item) {
      privateArray.push(item);
    },

    getItems: function () {
      return privateArray.slice(); // 복사본 반환
    },
  };
})();

MyModule.increment();
console.log(MyModule.getCount()); // 1
// console.log(privateCounter); // ReferenceError
```

### 2. 팩토리 패턴

```javascript
function createUser(name, role) {
  // 프라이빗 변수
  let userData = {
    name: name,
    role: role,
    loginCount: 0,
    lastLogin: null,
  };

  // 프라이빗 메서드
  function validateRole(role) {
    const validRoles = ["admin", "user", "guest"];
    return validRoles.includes(role);
  }

  function updateLoginStats() {
    userData.loginCount++;
    userData.lastLogin = new Date();
  }

  // 퍼블릭 API
  return {
    getName: function () {
      return userData.name;
    },

    getRole: function () {
      return userData.role;
    },

    login: function () {
      updateLoginStats();
      return `${userData.name} logged in`;
    },

    getStats: function () {
      return {
        loginCount: userData.loginCount,
        lastLogin: userData.lastLogin,
      };
    },

    changeRole: function (newRole) {
      if (validateRole(newRole)) {
        userData.role = newRole;
        return true;
      }
      return false;
    },
  };
}

const user = createUser("Alice", "admin");
console.log(user.login()); // 'Alice logged in'
console.log(user.getStats()); // { loginCount: 1, lastLogin: ... }
```

## 최신 동향과 미래

### 1. Private Fields (ES2022)

```javascript
class ModernClass {
  // 프라이빗 필드
  #privateField = "private";
  #privateMethod() {
    return "private method";
  }

  constructor(value) {
    this.#privateField = value;
  }

  getPrivate() {
    return this.#privateField;
  }

  callPrivateMethod() {
    return this.#privateMethod();
  }
}

const instance = new ModernClass("secret");
console.log(instance.getPrivate()); // 'secret'
// console.log(instance.#privateField); // SyntaxError
```

### 2. Top-level await

```javascript
// ES2022: 모듈 레벨에서 await 사용 가능
// 모듈 스코프에서 직접 비동기 작업 수행
const data = await fetch("/api/data").then((r) => r.json());

export const processedData = data.map((item) => ({
  ...item,
  processed: true,
}));
```

## 결론

실행 컨텍스트와 스코프 체인은 JavaScript의 핵심 동작 원리로, 변수 접근, 메모리 관리, 클로저, 모듈 패턴 등 모든 고급 개념의 기초가 됩니다. 이를 정확히 이해하면 더 효율적이고 안전한 JavaScript 코드를 작성할 수 있으며, 복잡한 디버깅 상황에서도 문제를 신속하게 해결할 수 있습니다.
