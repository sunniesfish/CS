# this 바인딩 규칙 (This Binding Rules)

## 개요

JavaScript의 `this`는 함수가 호출되는 방식에 따라 동적으로 결정되는 실행 컨텍스트 참조입니다. 암시적, 명시적, new, 화살표 함수 등 다양한 바인딩 규칙이 존재하며, 이를 정확히 이해하는 것이 JavaScript 마스터의 핵심입니다.

## 탄생 배경

`this`는 객체지향 프로그래밍에서 현재 객체를 참조하기 위한 메커니즘으로, JavaScript에서는 함수형과 객체지향 패러다임을 모두 지원하기 위해 유연한 바인딩 시스템을 채택했습니다.

## 4가지 주요 바인딩 규칙

### 1. 기본 바인딩 (Default Binding)

```javascript
function defaultBinding() {
  console.log(this); // 전역 객체 (window/global) 또는 undefined (strict mode)
}

defaultBinding(); // 독립 함수 호출

// strict mode에서
("use strict");
function strictMode() {
  console.log(this); // undefined
}
strictMode();
```

### 2. 암시적 바인딩 (Implicit Binding)

```javascript
const obj = {
  name: "Object",
  method: function () {
    console.log(this.name); // 'Object' - obj가 this
  },
};

obj.method(); // 암시적 바인딩

// 바인딩 손실 (Implicit Binding Loss)
const detachedMethod = obj.method;
detachedMethod(); // undefined 또는 전역 객체의 name

// 중첩 객체에서의 암시적 바인딩
const nested = {
  name: "Nested",
  inner: {
    name: "Inner",
    method: function () {
      console.log(this.name); // 'Inner' - 마지막 호출 객체가 this
    },
  },
};

nested.inner.method(); // 'Inner'
```

### 3. 명시적 바인딩 (Explicit Binding)

#### call, apply, bind 메서드

```javascript
const person = {
  name: "Alice",
  age: 30,
};

function introduce(greeting, punctuation) {
  console.log(
    `${greeting}, I'm ${this.name}, ${this.age} years old${punctuation}`
  );
}

// call - 개별 인수 전달
introduce.call(person, "Hello", "!"); // "Hello, I'm Alice, 30 years old!"

// apply - 배열로 인수 전달
introduce.apply(person, ["Hi", "."]); // "Hi, I'm Alice, 30 years old."

// bind - 새로운 함수 생성
const boundIntroduce = introduce.bind(person, "Hey");
boundIntroduce("~"); // "Hey, I'm Alice, 30 years old~"
```

#### 하드 바인딩 (Hard Binding)

```javascript
function hardBinding() {
  const obj = { name: "Hard Bound" };

  function originalFunction() {
    console.log(this.name);
  }

  const hardBound = function () {
    originalFunction.call(obj); // 항상 obj로 바인딩
  };

  return hardBound;
}

const hardBoundFunction = hardBinding();
hardBoundFunction(); // 'Hard Bound'

// 다른 객체로 바인딩 시도해도 무시됨
const anotherObj = { name: "Another" };
hardBoundFunction.call(anotherObj); // 여전히 'Hard Bound'
```

### 4. new 바인딩 (New Binding)

```javascript
function Constructor(name, age) {
  this.name = name;
  this.age = age;
  this.introduce = function () {
    console.log(`I'm ${this.name}, ${this.age} years old`);
  };
}

const person1 = new Constructor("Bob", 25);
person1.introduce(); // "I'm Bob, 25 years old"

// new 바인딩의 내부 동작 시뮬레이션
function simulateNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype);
  const result = Constructor.apply(obj, args);
  return result instanceof Object ? result : obj;
}

const person2 = simulateNew(Constructor, "Carol", 28);
person2.introduce(); // "I'm Carol, 28 years old"
```

## 화살표 함수와 렉시컬 this

### 화살표 함수의 특별한 this 바인딩

```javascript
const arrowObj = {
  name: "Arrow Object",

  regularMethod: function () {
    console.log("Regular:", this.name); // 'Arrow Object'

    const arrowFunction = () => {
      console.log("Arrow:", this.name); // 'Arrow Object' - 렉시컬 this
    };

    function regularInner() {
      console.log("Inner:", this.name); // undefined 또는 전역
    }

    arrowFunction();
    regularInner();
  },

  arrowMethod: () => {
    console.log("Arrow Method:", this.name); // undefined - 전역 this
  },
};

arrowObj.regularMethod();
arrowObj.arrowMethod();
```

### 이벤트 핸들러에서의 화살표 함수

```javascript
class EventHandler {
  constructor(name) {
    this.name = name;
    this.count = 0;
  }

  // 일반 메서드 - this 바인딩 손실 가능
  regularHandler() {
    this.count++;
    console.log(`${this.name}: ${this.count}`);
  }

  // 화살표 함수 - this 바인딩 유지
  arrowHandler = () => {
    this.count++;
    console.log(`${this.name}: ${this.count}`);
  };

  setupEvents() {
    // 문제가 있는 바인딩
    // button.addEventListener('click', this.regularHandler); // this 손실
    // 해결 방법들
    // 1. bind 사용
    // button.addEventListener('click', this.regularHandler.bind(this));
    // 2. 화살표 함수 사용
    // button.addEventListener('click', this.arrowHandler);
    // 3. 래퍼 함수 사용
    // button.addEventListener('click', () => this.regularHandler());
  }
}
```

## 바인딩 우선순위

### 우선순위 규칙

```javascript
const obj = {
  name: "Object",
  method: function () {
    console.log(this.name);
  },
};

// 1. new 바인딩이 최우선
function Constructor() {
  this.name = "Constructor";
}
const boundConstructor = Constructor.bind(obj);
const instance = new boundConstructor(); // Constructor의 this, obj 무시

// 2. 명시적 바인딩이 암시적 바인딩보다 우선
obj.method.call({ name: "Explicit" }); // 'Explicit'

// 3. 암시적 바인딩이 기본 바인딩보다 우선
obj.method(); // 'Object'
```

## 실무 패턴과 해결책

### 1. 콜백 함수에서의 this 문제

```javascript
class Timer {
  constructor(name) {
    this.name = name;
    this.seconds = 0;
  }

  start() {
    // 문제: setTimeout 콜백에서 this 손실
    // setTimeout(function() {
    //     this.seconds++; // this는 전역 객체
    //     console.log(`${this.name}: ${this.seconds}s`);
    // }, 1000);

    // 해결책 1: 화살표 함수
    setTimeout(() => {
      this.seconds++;
      console.log(`${this.name}: ${this.seconds}s`);
    }, 1000);

    // 해결책 2: bind 사용
    // setTimeout(function() {
    //     this.seconds++;
    //     console.log(`${this.name}: ${this.seconds}s`);
    // }.bind(this), 1000);

    // 해결책 3: self 변수
    // const self = this;
    // setTimeout(function() {
    //     self.seconds++;
    //     console.log(`${self.name}: ${self.seconds}s`);
    // }, 1000);
  }
}
```

### 2. 메서드 체이닝과 this

```javascript
class Calculator {
  constructor(value = 0) {
    this.value = value;
  }

  add(n) {
    this.value += n;
    return this; // 메서드 체이닝을 위해 this 반환
  }

  multiply(n) {
    this.value *= n;
    return this;
  }

  subtract(n) {
    this.value -= n;
    return this;
  }

  result() {
    return this.value;
  }
}

const calc = new Calculator(10);
const result = calc.add(5).multiply(2).subtract(3).result(); // 27
console.log(result);
```

### 3. 믹스인 패턴에서의 this

```javascript
const Mixin = {
  init(name) {
    this.name = name;
    return this;
  },

  getName() {
    return this.name;
  },

  setName(name) {
    this.name = name;
    return this;
  },
};

function createObject() {
  const obj = Object.create(Mixin);
  return obj.init("Mixed Object");
}

const mixedObj = createObject();
console.log(mixedObj.getName()); // 'Mixed Object'
```

## 고급 this 패턴

### 1. 동적 this 바인딩

```javascript
const dynamicBinder = {
  contexts: {
    user: { name: "User Context", role: "user" },
    admin: { name: "Admin Context", role: "admin" },
    guest: { name: "Guest Context", role: "guest" },
  },

  execute(contextName, fn) {
    const context = this.contexts[contextName];
    if (context && typeof fn === "function") {
      return fn.call(context);
    }
    throw new Error("Invalid context or function");
  },
};

function showInfo() {
  console.log(`Name: ${this.name}, Role: ${this.role}`);
}

dynamicBinder.execute("user", showInfo); // "Name: User Context, Role: user"
dynamicBinder.execute("admin", showInfo); // "Name: Admin Context, Role: admin"
```

### 2. this 캐싱과 성능 최적화

```javascript
class PerformantClass {
  constructor() {
    this.data = [];
    this.cache = new Map();

    // 자주 사용되는 메서드는 바인딩 캐시
    this.boundMethod = this.expensiveMethod.bind(this);
  }

  expensiveMethod(key) {
    if (this.cache.has(key)) {
      return this.cache.get(key);
    }

    const result = this.computeExpensiveValue(key);
    this.cache.set(key, result);
    return result;
  }

  computeExpensiveValue(key) {
    // 복잡한 계산 시뮬레이션
    return key * Math.random() * 1000;
  }

  setupEventListeners() {
    // 바인딩된 메서드 재사용
    document.addEventListener("click", this.boundMethod);
  }
}
```

## 디버깅과 도구

### 1. this 디버깅 헬퍼

```javascript
function debugThis(label) {
  return function (...args) {
    console.group(`Debug: ${label}`);
    console.log("this:", this);
    console.log("arguments:", args);
    console.log("constructor:", this.constructor?.name);
    console.groupEnd();

    // 원래 함수가 있다면 실행
    if (typeof this.originalMethod === "function") {
      return this.originalMethod.apply(this, args);
    }
  };
}

const debugObj = {
  name: "Debug Object",
  method: debugThis("method call"),
  originalMethod: function () {
    return `Hello from ${this.name}`;
  },
};

debugObj.method(); // this 정보 출력
```

### 2. this 바인딩 검증

```javascript
function validateThis(expectedType, methodName) {
  return function (target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args) {
      if (expectedType && !(this instanceof expectedType)) {
        throw new Error(
          `${methodName} must be called on ${expectedType.name} instance`
        );
      }

      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class ValidatedClass {
  constructor(name) {
    this.name = name;
  }

  @validateThis(ValidatedClass, "getName")
  getName() {
    return this.name;
  }
}
```

## 최신 JavaScript와 this

### 1. 클래스 필드와 this

```javascript
class ModernClass {
  // 클래스 필드 (자동으로 바인딩됨)
  name = "Modern";

  // 화살표 함수 필드 (this 자동 바인딩)
  arrowMethod = () => {
    return this.name;
  };

  // 일반 메서드
  regularMethod() {
    return this.name;
  }

  // 정적 메서드 (this는 클래스 자체)
  static staticMethod() {
    return this.name; // 'ModernClass'
  }
}

const modern = new ModernClass();
const detached = modern.arrowMethod;
console.log(detached()); // 'Modern' - 바인딩 유지
```

### 2. 프록시와 this

```javascript
const proxyHandler = {
  get(target, property) {
    if (typeof target[property] === "function") {
      return function (...args) {
        console.log(`Calling ${property} with:`, args);
        return target[property].apply(target, args);
      };
    }
    return target[property];
  },
};

const obj = {
  name: "Proxied Object",
  greet() {
    return `Hello from ${this.name}`;
  },
};

const proxied = new Proxy(obj, proxyHandler);
console.log(proxied.greet()); // 로깅과 함께 실행
```

## 결론

JavaScript의 `this` 바인딩은 복잡하지만 강력한 기능으로, 4가지 주요 규칙(기본, 암시적, 명시적, new)과 화살표 함수의 렉시컬 바인딩을 이해하면 예측 가능한 코드를 작성할 수 있습니다. 현대 JavaScript에서는 화살표 함수와 클래스 필드를 활용하여 this 관련 문제를 더 쉽게 해결할 수 있습니다.
