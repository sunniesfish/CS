# 호이스팅(Hoisting)과 TDZ(Temporal Dead Zone)

## 개요

호이스팅(Hoisting)은 JavaScript에서 변수와 함수 선언이 해당 스코프의 최상단으로 "끌어올려지는" 것처럼 동작하는 메커니즘입니다. TDZ(Temporal Dead Zone)는 let과 const로 선언된 변수가 선언되기 전까지 접근할 수 없는 구간을 의미합니다.

## 탄생 배경

JavaScript 엔진은 코드를 실행하기 전에 컴파일 단계를 거치며, 이 과정에서 변수와 함수 선언을 미리 처리합니다:

- **메모리 할당**: 실행 전 변수를 위한 메모리 공간 확보
- **스코프 결정**: 변수와 함수의 유효 범위 설정
- **성능 최적화**: 런타임에서의 변수 검색 시간 단축
- **에러 방지**: 선언되지 않은 변수 접근 시 명확한 에러 제공

## 호이스팅의 동작 원리

### 1. 변수 호이스팅

#### var 키워드의 호이스팅

```javascript
console.log(hoistedVar); // undefined (에러가 아님)
var hoistedVar = "I am hoisted";
console.log(hoistedVar); // 'I am hoisted'

// 실제 엔진의 처리 과정:
// var hoistedVar; // undefined로 초기화
// console.log(hoistedVar); // undefined
// hoistedVar = 'I am hoisted'; // 값 할당
// console.log(hoistedVar); // 'I am hoisted'
```

#### let과 const의 호이스팅

```javascript
// let 호이스팅
console.log(letVar); // ReferenceError: Cannot access 'letVar' before initialization
let letVar = "I am let";

// const 호이스팅
console.log(constVar); // ReferenceError: Cannot access 'constVar' before initialization
const constVar = "I am const";

// 실제로는 호이스팅되지만 TDZ에 있음
```

### 2. 함수 호이스팅

#### 함수 선언문의 호이스팅

```javascript
// 함수 호출이 선언보다 먼저 나와도 작동
console.log(hoistedFunction()); // 'I am hoisted function'

function hoistedFunction() {
  return "I am hoisted function";
}

// 엔진의 처리 과정:
// function hoistedFunction() { return 'I am hoisted function'; } // 전체가 호이스팅
// console.log(hoistedFunction()); // 함수 호출 가능
```

#### 함수 표현식의 호이스팅

```javascript
// var로 선언된 함수 표현식
console.log(varFunction); // undefined
console.log(varFunction()); // TypeError: varFunction is not a function

var varFunction = function () {
  return "I am function expression";
};

// let/const로 선언된 함수 표현식
console.log(letFunction); // ReferenceError: Cannot access 'letFunction' before initialization

let letFunction = function () {
  return "I am let function expression";
};
```

### 3. 클래스 호이스팅

```javascript
// 클래스는 호이스팅되지만 TDZ에 있음
console.log(MyClass); // ReferenceError: Cannot access 'MyClass' before initialization

class MyClass {
  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

// 클래스 표현식도 동일
console.log(MyClassExpression); // ReferenceError

const MyClassExpression = class {
  constructor(name) {
    this.name = name;
  }
};
```

## TDZ(Temporal Dead Zone) 심화

### 1. TDZ의 정확한 정의

```javascript
function demonstrateTDZ() {
  // TDZ 시작
  console.log(typeof myLet); // ReferenceError (TDZ 내에서)
  console.log(typeof myConst); // ReferenceError (TDZ 내에서)

  // 여전히 TDZ 내
  if (true) {
    // 블록 스코프 시작, 새로운 TDZ
    console.log(blockLet); // ReferenceError
    let blockLet = "block scoped";
    console.log(blockLet); // 'block scoped' - TDZ 종료
  }

  let myLet = "let value"; // TDZ 종료
  const myConst = "const value"; // TDZ 종료

  console.log(myLet); // 'let value'
  console.log(myConst); // 'const value'
}

demonstrateTDZ();
```

### 2. TDZ와 typeof 연산자

```javascript
// 전역 스코프에서의 typeof
console.log(typeof undeclaredVariable); // 'undefined' (에러 없음)
console.log(typeof declaredLater); // ReferenceError (TDZ)

let declaredLater = "now declared";

// 함수 스코프에서의 TDZ
function typeofInTDZ() {
  // 매개변수도 TDZ 영향을 받음
  console.log(typeof param); // ReferenceError
  let param = "parameter";
}

// typeofInTDZ();
```

### 3. TDZ와 기본 매개변수

```javascript
// 기본 매개변수에서의 TDZ
function defaultParams(a = b, b = "default") {
  // b가 아직 초기화되지 않았으므로 ReferenceError
  return [a, b];
}

// console.log(defaultParams()); // ReferenceError

// 올바른 순서
function correctDefaultParams(a = "default", b = a) {
  return [a, b]; // a는 이미 초기화되었으므로 사용 가능
}

console.log(correctDefaultParams()); // ['default', 'default']
console.log(correctDefaultParams("custom")); // ['custom', 'custom']
```

### 4. TDZ와 구조 분해 할당

```javascript
// 구조 분해 할당에서의 TDZ
function destructuringTDZ() {
  console.log(x, y); // ReferenceError: Cannot access 'x' before initialization

  let [x, y] = [1, 2];
  console.log(x, y); // 1, 2
}

// 객체 구조 분해에서의 TDZ
function objectDestructuringTDZ() {
  console.log(name, age); // ReferenceError

  const { name, age } = { name: "John", age: 30 };
  console.log(name, age); // 'John', 30
}
```

## 실제 코드에서의 호이스팅 패턴

### 1. 함수 선언 vs 함수 표현식

```javascript
// 함수 선언문 - 완전히 호이스팅됨
function declaration() {
  return "declaration";
}

// 함수 표현식 - 변수만 호이스팅됨
var expression = function () {
  return "expression";
};

// 화살표 함수 - 변수만 호이스팅됨
const arrow = () => {
  return "arrow";
};

// 실제 사용 예시
console.log(declaration()); // 'declaration' - 작동함
// console.log(expression()); // TypeError: expression is not a function
// console.log(arrow()); // TypeError: arrow is not a function
```

### 2. 조건부 함수 선언

```javascript
// 조건부 함수 선언 (권장하지 않음)
if (true) {
  function conditionalFunction() {
    return "conditional";
  }
}

// 더 나은 방법: 함수 표현식 사용
let betterConditionalFunction;
if (true) {
  betterConditionalFunction = function () {
    return "better conditional";
  };
}

// 또는 const 사용 (블록 스코프)
if (true) {
  const blockScopedFunction = function () {
    return "block scoped";
  };
  console.log(blockScopedFunction()); // 'block scoped'
}
// console.log(blockScopedFunction()); // ReferenceError
```

### 3. 루프에서의 호이스팅

```javascript
// var를 사용한 루프 (호이스팅 문제)
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log("var i:", i); // 3, 3, 3 (모두 3)
  }, 100);
}

// let을 사용한 루프 (블록 스코프)
for (let j = 0; j < 3; j++) {
  setTimeout(() => {
    console.log("let j:", j); // 0, 1, 2 (각각 다른 값)
  }, 200);
}

// IIFE로 해결하는 전통적 방법
for (var k = 0; k < 3; k++) {
  (function (index) {
    setTimeout(() => {
      console.log("IIFE k:", index); // 0, 1, 2
    }, 300);
  })(k);
}
```

## 호이스팅 관련 함정과 해결책

### 1. 중복 선언 문제

```javascript
// var의 중복 선언 허용 (문제가 될 수 있음)
var duplicateVar = "first";
console.log(duplicateVar); // 'first'

var duplicateVar = "second"; // 재선언 허용
console.log(duplicateVar); // 'second'

function duplicateVar() {
  // 함수로도 재선언 가능
  return "function";
}

console.log(duplicateVar); // 'second' (변수가 함수를 덮어씀)

// let/const는 중복 선언 금지
let uniqueLet = "first";
// let uniqueLet = 'second'; // SyntaxError: Identifier 'uniqueLet' has already been declared
```

### 2. 함수 내부 var 호이스팅 문제

```javascript
function hoistingTrap() {
  if (false) {
    var neverExecuted = "never";
  }

  console.log(neverExecuted); // undefined (호이스팅됨)

  for (var i = 0; i < 1; i++) {
    var loopVar = "loop";
  }

  console.log(i); // 1 (함수 스코프)
  console.log(loopVar); // 'loop' (함수 스코프)
}

hoistingTrap();

// 해결책: let/const 사용
function noHoistingTrap() {
  if (false) {
    let neverExecuted = "never";
  }

  // console.log(neverExecuted); // ReferenceError

  for (let i = 0; i < 1; i++) {
    let loopVar = "loop";
  }

  // console.log(i); // ReferenceError
  // console.log(loopVar); // ReferenceError
}
```

### 3. this 바인딩과 호이스팅

```javascript
var globalThis = "global";

const obj = {
  name: "object",

  regularFunction: function () {
    console.log(this.name); // 'object'

    // 내부 함수에서의 this (호이스팅과 관련)
    function innerFunction() {
      console.log(this.name); // undefined (strict mode) 또는 'global'
    }

    innerFunction();
  },

  arrowFunction: function () {
    console.log(this.name); // 'object'

    const innerArrow = () => {
      console.log(this.name); // 'object' (렉시컬 this)
    };

    innerArrow();
  },
};

obj.regularFunction();
obj.arrowFunction();
```

## 실무에서의 베스트 프랙티스

### 1. 변수 선언 가이드라인

```javascript
// ❌ 피해야 할 패턴
function badPractice() {
  console.log(x); // undefined
  if (true) {
    var x = 1;
  }
  console.log(x); // 1
}

// ✅ 권장하는 패턴
function goodPractice() {
  let x; // 명시적 선언

  if (true) {
    x = 1; // 할당
  }

  console.log(x); // 1
}

// ✅ 더 나은 패턴
function bestPractice() {
  const condition = true;

  if (condition) {
    const x = 1; // 블록 스코프, 불변성
    console.log(x); // 1
  }
}
```

### 2. 함수 선언 가이드라인

```javascript
// ✅ 함수 선언문 - 호이스팅 활용
function utilityFunction() {
  return "utility";
}

// ✅ 함수 표현식 - 조건부 할당
const conditionalFunction = condition
  ? function () {
      return "true case";
    }
  : function () {
      return "false case";
    };

// ✅ 화살표 함수 - 간단한 함수
const simpleFunction = (x, y) => x + y;

// ✅ 메서드 정의
const obj = {
  method() {
    // ES6 메서드 단축 구문
    return "method";
  },
};
```

### 3. 모듈 패턴에서의 호이스팅 활용

```javascript
// 모듈 패턴에서 호이스팅 활용
const MyModule = (function () {
  // 프라이빗 변수 (호이스팅됨)
  var privateVar = "private";

  // 프라이빗 함수 (호이스팅됨)
  function privateFunction() {
    return privateVar;
  }

  // 퍼블릭 API
  return {
    getPrivate: function () {
      return privateFunction(); // 호이스팅된 함수 사용
    },

    setPrivate: function (value) {
      privateVar = value; // 호이스팅된 변수 사용
    },
  };
})();

console.log(MyModule.getPrivate()); // 'private'
MyModule.setPrivate("updated");
console.log(MyModule.getPrivate()); // 'updated'
```

## 디버깅과 도구 활용

### 1. 호이스팅 시각화

```javascript
function visualizeHoisting() {
  console.log("=== 실행 순서 ===");
  console.log("1. 함수 실행 시작");

  console.log("2. hoistedVar:", hoistedVar); // undefined
  console.log("3. hoistedFunc:", hoistedFunc()); // 'I am hoisted'

  var hoistedVar = "now assigned";

  function hoistedFunc() {
    return "I am hoisted";
  }

  console.log("4. hoistedVar:", hoistedVar); // 'now assigned'
  console.log("5. 함수 실행 종료");
}

visualizeHoisting();
```

### 2. TDZ 디버깅

```javascript
function debugTDZ() {
  console.log("=== TDZ 디버깅 ===");

  try {
    console.log(letVar); // TDZ 에러 발생
  } catch (error) {
    console.log("TDZ Error:", error.message);
  }

  let letVar = "initialized";
  console.log("After initialization:", letVar);
}

debugTDZ();
```

### 3. ESLint 규칙 활용

```javascript
// ESLint 규칙 예시
/* eslint no-use-before-define: "error" */
/* eslint no-var: "error" */
/* eslint prefer-const: "error" */

// 이 코드는 ESLint 에러를 발생시킴
// console.log(notYetDeclared); // no-use-before-define
// var oldStyle = 'avoid'; // no-var
// let shouldBeConst = 'constant'; // prefer-const

// 올바른 코드
const properConstant = "constant";
let mutableVariable = "mutable";

function properFunction() {
  return "proper";
}

console.log(properFunction());
```

## 성능 고려사항

### 1. 호이스팅과 메모리 사용

```javascript
// 메모리 효율적인 패턴
function memoryEfficient() {
  // 필요할 때만 변수 선언
  if (someCondition()) {
    const heavyObject = createHeavyObject();
    return processHeavyObject(heavyObject);
  }

  return null;
}

// 비효율적인 패턴 (호이스팅으로 인한 메모리 낭비)
function memoryInefficient() {
  var heavyObject; // 항상 메모리 할당

  if (someCondition()) {
    heavyObject = createHeavyObject();
    return processHeavyObject(heavyObject);
  }

  return null; // heavyObject는 여전히 메모리에 있음
}

function someCondition() {
  return Math.random() > 0.5;
}
function createHeavyObject() {
  return new Array(1000000);
}
function processHeavyObject(obj) {
  return obj.length;
}
```

### 2. 함수 호이스팅과 성능

```javascript
// 성능 최적화된 패턴
const OptimizedModule = (function () {
  // 자주 사용되는 함수는 호이스팅 활용
  function frequentlyUsed() {
    return "frequent";
  }

  // 가끔 사용되는 함수는 지연 로딩
  let rarelyUsed;

  return {
    frequent: frequentlyUsed,

    rare: function () {
      if (!rarelyUsed) {
        rarelyUsed = function () {
          return "rare and complex computation";
        };
      }
      return rarelyUsed();
    },
  };
})();
```

## 최신 JavaScript와 호이스팅

### 1. ES2020+ 기능들

```javascript
// Optional Chaining과 호이스팅
function modernFeatures() {
  console.log(obj?.property); // undefined (호이스팅된 obj)

  var obj = {
    property: "value",
    nested: {
      deep: "deep value",
    },
  };

  console.log(obj?.nested?.deep); // 'deep value'
}

modernFeatures();

// Nullish Coalescing과 호이스팅
function nullishCoalescing() {
  console.log(value ?? "default"); // undefined ?? 'default' = 'default'

  let value = null;
  console.log(value ?? "default"); // null ?? 'default' = 'default'

  value = "actual value";
  console.log(value ?? "default"); // 'actual value'
}

nullishCoalescing();
```

### 2. 모듈에서의 호이스팅

```javascript
// ES6 모듈에서의 호이스팅
// export된 함수는 호이스팅됨
export function exportedFunction() {
  return "exported";
}

// import된 바인딩은 TDZ 적용
console.log(importedFunction()); // 작동함 (호이스팅)

// 하지만 let/const export는 TDZ 적용
// console.log(exportedLet); // ReferenceError
export let exportedLet = "exported let";

// default export도 호이스팅 고려
export default function defaultFunction() {
  return "default";
}
```

## 결론

호이스팅과 TDZ는 JavaScript의 독특한 특성으로, 이를 정확히 이해하면 예측 가능하고 안전한 코드를 작성할 수 있습니다. 현대 JavaScript 개발에서는 let과 const를 사용하여 TDZ의 보호를 받으며, ESLint 같은 도구로 호이스팅 관련 문제를 사전에 방지하는 것이 좋은 실천 방법입니다.
