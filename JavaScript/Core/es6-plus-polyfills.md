# ES6+ 문법 변화 및 Polyfill 이해

## 개요

ES6(ES2015)부터 시작된 JavaScript의 현대적 문법 변화는 개발자 경험을 크게 개선했습니다. 하지만 구형 브라우저 호환성을 위해 Polyfill과 Transpilation 기술이 필요합니다. 이 문서는 ES6+ 주요 기능들과 이를 구형 환경에서 사용하기 위한 전략들을 다룹니다.

## 탄생 배경

JavaScript는 1995년 브렌던 아이크(Brendan Eich)가 10일 만에 개발한 언어로, 오랫동안 큰 변화 없이 사용되었습니다. ES6(2015)부터 매년 새로운 기능이 추가되면서 현대적인 프로그래밍 언어로 발전했지만, 브라우저 호환성 문제로 Polyfill과 Transpiler가 중요해졌습니다.

## ES6+ 주요 기능 변화

### 1. 변수 선언 (let, const)

```javascript
// ES5 - var의 문제점
function es5VariableIssues() {
  console.log("=== ES5 var의 문제점 ===");

  // 1. 함수 스코프 (블록 스코프 X)
  if (true) {
    var x = 1;
  }
  console.log(x); // 1 - 블록 밖에서도 접근 가능

  // 2. 호이스팅으로 인한 혼란
  console.log(y); // undefined (에러가 아님)
  var y = 2;

  // 3. 반복문에서의 클로저 문제
  for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log("var:", i), 100); // 3, 3, 3
  }
}

// ES6+ - let, const의 개선점
function es6VariableImprovements() {
  console.log("=== ES6+ let, const의 개선점 ===");

  // 1. 블록 스코프
  if (true) {
    let x = 1;
    const y = 2;
  }
  // console.log(x); // ReferenceError
  // console.log(y); // ReferenceError

  // 2. TDZ (Temporal Dead Zone)
  // console.log(z); // ReferenceError (호이스팅되지만 접근 불가)
  let z = 3;

  // 3. 반복문에서의 올바른 클로저
  for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log("let:", i), 200); // 0, 1, 2
  }

  // 4. const의 불변성 (참조 불변)
  const obj = { name: "Alice" };
  obj.name = "Bob"; // 가능 (객체 내용 변경)
  // obj = {}; // TypeError (참조 변경 불가)

  const arr = [1, 2, 3];
  arr.push(4); // 가능
  // arr = []; // TypeError

  console.log("const object:", obj);
  console.log("const array:", arr);
}

// Polyfill for let/const (제한적)
function letConstPolyfill() {
  // let/const는 문법적 기능이라 완전한 polyfill 불가
  // Babel 같은 transpiler가 필요

  // 대신 IIFE를 사용한 스코프 격리
  (function () {
    var _let1 = 1; // let x = 1을 var로 변환

    // 블록 스코프 시뮬레이션
    (function () {
      var _let2 = 2; // 내부 블록의 let
    })();

    // _let2는 여기서 접근 불가
  })();
}

es5VariableIssues();
es6VariableImprovements();
```

### 2. 화살표 함수 (Arrow Functions)

```javascript
// ES5 함수 선언
function es5Functions() {
  console.log("=== ES5 함수 ===");

  var self = this;
  var numbers = [1, 2, 3, 4, 5];

  // 일반 함수
  var doubled = numbers.map(function (n) {
    return n * 2;
  });

  // this 바인딩 문제
  var obj = {
    name: "ES5 Object",
    greet: function () {
      setTimeout(function () {
        console.log("Hello from " + this.name); // undefined
      }, 100);
    },
    greetFixed: function () {
      var self = this;
      setTimeout(function () {
        console.log("Hello from " + self.name); // ES5 Object
      }, 100);
    },
  };

  console.log("Doubled:", doubled);
  obj.greet();
  obj.greetFixed();
}

// ES6+ 화살표 함수
function es6ArrowFunctions() {
  console.log("=== ES6+ 화살표 함수 ===");

  const numbers = [1, 2, 3, 4, 5];

  // 간결한 문법
  const doubled = numbers.map((n) => n * 2);
  const filtered = numbers.filter((n) => n % 2 === 0);
  const sum = numbers.reduce((acc, n) => acc + n, 0);

  // 다양한 화살표 함수 문법
  const simple = (x) => x * 2; // 매개변수 하나
  const multiple = (x, y) => x + y; // 매개변수 여러 개
  const noParams = () => "Hello"; // 매개변수 없음
  const block = (x) => {
    // 블록 문
    const result = x * 2;
    return result;
  };
  const objectReturn = (x) => ({ value: x }); // 객체 반환

  // this 바인딩 - 렉시컬 this
  const obj = {
    name: "ES6 Object",
    greet() {
      setTimeout(() => {
        console.log("Hello from " + this.name); // ES6 Object
      }, 200);
    },

    // 화살표 함수는 메서드로 부적합
    arrowMethod: () => {
      console.log("Arrow method this:", this); // 전역 객체
    },

    regularMethod() {
      console.log("Regular method this:", this.name); // ES6 Object
    },
  };

  console.log("Arrow function results:", { doubled, filtered, sum });
  console.log(
    "Function calls:",
    simple(5),
    multiple(3, 4),
    noParams(),
    block(6),
    objectReturn(7)
  );

  obj.greet();
  obj.arrowMethod();
  obj.regularMethod();
}

// 화살표 함수 Polyfill
function arrowFunctionPolyfill() {
  // 화살표 함수도 문법적 기능이라 완전한 polyfill 불가
  // Babel이 다음과 같이 변환:

  // ES6: const add = (a, b) => a + b;
  // ES5 변환:
  var add = function add(a, b) {
    return a + b;
  };

  // ES6: setTimeout(() => console.log(this.name), 100);
  // ES5 변환 (this 바인딩 유지):
  var self = this;
  setTimeout(function () {
    console.log(self.name);
  }, 100);

  console.log("Arrow function polyfill example:", add(2, 3));
}

es5Functions();
es6ArrowFunctions();
arrowFunctionPolyfill();
```

### 3. 템플릿 리터럴 (Template Literals)

```javascript
// ES5 문자열 처리
function es5StringHandling() {
  console.log("=== ES5 문자열 처리 ===");

  var name = "Alice";
  var age = 30;
  var city = "Seoul";

  // 문자열 연결
  var greeting =
    "Hello, my name is " + name + " and I am " + age + " years old.";

  // 여러 줄 문자열
  var multiLine = "This is line 1\n" + "This is line 2\n" + "This is line 3";

  // HTML 템플릿
  var htmlTemplate =
    '<div class="user">' +
    "<h2>" +
    name +
    "</h2>" +
    "<p>Age: " +
    age +
    "</p>" +
    "<p>City: " +
    city +
    "</p>" +
    "</div>";

  console.log(greeting);
  console.log(multiLine);
  console.log(htmlTemplate);
}

// ES6+ 템플릿 리터럴
function es6TemplateStrings() {
  console.log("=== ES6+ 템플릿 리터럴 ===");

  const name = "Bob";
  const age = 25;
  const city = "Busan";

  // 문자열 보간
  const greeting = `Hello, my name is ${name} and I am ${age} years old.`;

  // 여러 줄 문자열
  const multiLine = `This is line 1
This is line 2
This is line 3`;

  // 표현식 사용
  const calculation = `The result is: ${10 + 20 * 3}`;
  const conditional = `Status: ${age >= 18 ? "Adult" : "Minor"}`;

  // HTML 템플릿
  const htmlTemplate = `
        <div class="user">
            <h2>${name}</h2>
            <p>Age: ${age}</p>
            <p>City: ${city}</p>
            <p>Category: ${age >= 30 ? "Senior" : "Junior"}</p>
        </div>
    `;

  // 중첩 템플릿
  const users = ["Alice", "Bob", "Charlie"];
  const userList = `
        <ul>
            ${users.map((user) => `<li>${user}</li>`).join("")}
        </ul>
    `;

  console.log(greeting);
  console.log(multiLine);
  console.log(calculation);
  console.log(conditional);
  console.log(htmlTemplate.trim());
  console.log(userList.trim());
}

// 태그된 템플릿 리터럴
function taggedTemplates() {
  console.log("=== 태그된 템플릿 리터럴 ===");

  // 기본 태그 함수
  function highlight(strings, ...values) {
    return strings.reduce((result, string, i) => {
      const value = values[i] ? `<mark>${values[i]}</mark>` : "";
      return result + string + value;
    }, "");
  }

  const name = "Alice";
  const score = 95;
  const result = highlight`Student ${name} scored ${score} points!`;
  console.log("Highlighted:", result);

  // 다국어 지원 태그
  function i18n(strings, ...values) {
    const translations = {
      Hello: "안녕하세요",
      Welcome: "환영합니다",
    };

    return strings.reduce((result, string, i) => {
      let value = values[i] || "";
      if (translations[value]) {
        value = translations[value];
      }
      return result + string + value;
    }, "");
  }

  const greeting = i18n`${"Hello"} ${name}, ${"Welcome"} to our site!`;
  console.log("Translated:", greeting);

  // 안전한 HTML 태그
  function safeHtml(strings, ...values) {
    const escapeHtml = (str) => {
      return String(str)
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#39;");
    };

    return strings.reduce((result, string, i) => {
      const value = values[i] ? escapeHtml(values[i]) : "";
      return result + string + value;
    }, "");
  }

  const userInput = '<script>alert("XSS")</script>';
  const safeOutput = safeHtml`User input: ${userInput}`;
  console.log("Safe HTML:", safeOutput);
}

// 템플릿 리터럴 Polyfill
function templateLiteralPolyfill() {
  // 간단한 템플릿 리터럴 polyfill
  function template(str, data) {
    return str.replace(/\$\{([^}]+)\}/g, function (match, key) {
      return data[key] || "";
    });
  }

  // 사용 예시
  var templateStr = "Hello, my name is ${name} and I am ${age} years old.";
  var data = { name: "Charlie", age: 35 };
  var result = template(templateStr, data);

  console.log("Template polyfill result:", result);

  // 더 고급 polyfill (함수 실행 지원)
  function advancedTemplate(str, context) {
    return str.replace(/\$\{([^}]+)\}/g, function (match, expression) {
      try {
        // 주의: eval 사용은 보안상 위험할 수 있음
        return Function(
          "context",
          `with(context) { return ${expression}; }`
        )(context);
      } catch (e) {
        return match;
      }
    });
  }

  var advancedStr =
    'Result: ${a + b}, Status: ${age >= 18 ? "Adult" : "Minor"}';
  var advancedData = { a: 10, b: 20, age: 25 };
  var advancedResult = advancedTemplate(advancedStr, advancedData);

  console.log("Advanced template result:", advancedResult);
}

es5StringHandling();
es6TemplateStrings();
taggedTemplates();
templateLiteralPolyfill();
```

### 4. 클래스 (Classes)

```javascript
// ES5 클래스 패턴
function es5ClassPattern() {
  console.log("=== ES5 클래스 패턴 ===");

  // 생성자 함수
  function Animal(name, species) {
    this.name = name;
    this.species = species;
  }

  // 프로토타입 메서드
  Animal.prototype.speak = function () {
    return this.name + " makes a sound";
  };

  Animal.prototype.getInfo = function () {
    return "Name: " + this.name + ", Species: " + this.species;
  };

  // 정적 메서드 시뮬레이션
  Animal.createDog = function (name) {
    return new Animal(name, "Dog");
  };

  // 상속
  function Dog(name, breed) {
    Animal.call(this, name, "Dog"); // 부모 생성자 호출
    this.breed = breed;
  }

  // 프로토타입 상속
  Dog.prototype = Object.create(Animal.prototype);
  Dog.prototype.constructor = Dog;

  // 메서드 오버라이드
  Dog.prototype.speak = function () {
    return this.name + " barks";
  };

  Dog.prototype.getBreed = function () {
    return this.breed;
  };

  // 사용 예시
  var animal = new Animal("Generic", "Unknown");
  var dog = new Dog("Buddy", "Golden Retriever");
  var staticDog = Animal.createDog("Max");

  console.log(animal.speak());
  console.log(animal.getInfo());
  console.log(dog.speak());
  console.log(dog.getInfo());
  console.log(dog.getBreed());
  console.log(staticDog.speak());
}

// ES6+ 클래스
function es6Classes() {
  console.log("=== ES6+ 클래스 ===");

  class Animal {
    constructor(name, species) {
      this.name = name;
      this.species = species;
    }

    speak() {
      return `${this.name} makes a sound`;
    }

    getInfo() {
      return `Name: ${this.name}, Species: ${this.species}`;
    }

    // 정적 메서드
    static createDog(name) {
      return new Animal(name, "Dog");
    }

    // Getter
    get displayName() {
      return `${this.name} the ${this.species}`;
    }

    // Setter
    set nickname(value) {
      this._nickname = value;
    }

    get nickname() {
      return this._nickname || this.name;
    }
  }

  class Dog extends Animal {
    constructor(name, breed) {
      super(name, "Dog"); // 부모 생성자 호출
      this.breed = breed;
    }

    speak() {
      return `${this.name} barks`;
    }

    getBreed() {
      return this.breed;
    }

    // 부모 메서드 호출
    getFullInfo() {
      return `${super.getInfo()}, Breed: ${this.breed}`;
    }

    // 정적 메서드 오버라이드
    static createPuppy(name, breed) {
      const puppy = new Dog(name, breed);
      puppy._age = "puppy";
      return puppy;
    }
  }

  // 사용 예시
  const animal = new Animal("Generic", "Unknown");
  const dog = new Dog("Buddy", "Golden Retriever");
  const staticDog = Animal.createDog("Max");
  const puppy = Dog.createPuppy("Luna", "Beagle");

  console.log(animal.speak());
  console.log(animal.getInfo());
  console.log(animal.displayName);

  console.log(dog.speak());
  console.log(dog.getFullInfo());
  console.log(dog.getBreed());

  dog.nickname = "Buddy Boy";
  console.log(dog.nickname);

  console.log(staticDog.speak());
  console.log(puppy.speak());
}

// 고급 클래스 기능 (ES2022+)
function advancedClassFeatures() {
  console.log("=== 고급 클래스 기능 ===");

  class ModernClass {
    // 퍼블릭 필드
    publicField = "I am public";

    // 프라이빗 필드 (ES2022)
    #privateField = "I am private";

    // 정적 퍼블릭 필드
    static staticPublicField = "Static public";

    // 정적 프라이빗 필드
    static #staticPrivateField = "Static private";

    constructor(value) {
      this.value = value;
    }

    // 프라이빗 메서드
    #privateMethod() {
      return "Private method result";
    }

    // 퍼블릭 메서드에서 프라이빗 접근
    getPrivateData() {
      return {
        field: this.#privateField,
        method: this.#privateMethod(),
      };
    }

    // 정적 프라이빗 메서드
    static #staticPrivateMethod() {
      return ModernClass.#staticPrivateField;
    }

    static getStaticPrivateData() {
      return ModernClass.#staticPrivateMethod();
    }

    // in 연산자로 프라이빗 필드 확인 (ES2022)
    hasPrivateField() {
      return #privateField in this;
    }
  }

  const instance = new ModernClass("test");

  console.log("Public field:", instance.publicField);
  console.log("Value:", instance.value);
  console.log("Private data:", instance.getPrivateData());
  console.log("Has private field:", instance.hasPrivateField());
  console.log("Static public field:", ModernClass.staticPublicField);
  console.log("Static private data:", ModernClass.getStaticPrivateData());

  // console.log(instance.#privateField); // SyntaxError
}

es5ClassPattern();
es6Classes();
advancedClassFeatures();
```

## Polyfill의 개념과 구현

### 1. Polyfill의 기본 개념

```javascript
// Polyfill의 정의와 원리
function polyfillConcepts() {
  console.log("=== Polyfill 개념 ===");

  // Polyfill: 구형 브라우저에서 최신 기능을 사용할 수 있게 해주는 코드

  // 1. 기능 감지 (Feature Detection)
  function hasArrayIncludes() {
    return Array.prototype.includes !== undefined;
  }

  console.log("Array.includes 지원:", hasArrayIncludes());

  // 2. 조건부 Polyfill 로딩
  if (!hasArrayIncludes()) {
    console.log("Array.includes polyfill 로드 필요");
    loadArrayIncludesPolyfill();
  }

  // 3. Polyfill vs Transpilation
  // Polyfill: 런타임에 누락된 기능을 추가
  // Transpilation: 빌드 타임에 코드를 변환

  console.log("Polyfill 로드 완료");
}

// Array.includes Polyfill 구현
function loadArrayIncludesPolyfill() {
  if (!Array.prototype.includes) {
    Array.prototype.includes = function (searchElement, fromIndex) {
      "use strict";

      // 1. Let O be ? ToObject(this value).
      if (this == null) {
        throw new TypeError('"this" is null or not defined');
      }

      var o = Object(this);

      // 2. Let len be ? ToLength(? Get(O, "length")).
      var len = parseInt(o.length) || 0;

      // 3. If len is 0, return false.
      if (len === 0) {
        return false;
      }

      // 4. Let n be ? ToInteger(fromIndex).
      var n = parseInt(fromIndex) || 0;
      var k;

      // 5. If n ≥ 0, then
      if (n >= 0) {
        k = n;
      } else {
        // 6. Else n < 0,
        k = len + n;
        if (k < 0) {
          k = 0;
        }
      }

      // 7. Repeat, while k < len
      function sameValueZero(x, y) {
        return (
          x === y ||
          (typeof x === "number" &&
            typeof y === "number" &&
            isNaN(x) &&
            isNaN(y))
        );
      }

      for (; k < len; k++) {
        if (sameValueZero(o[k], searchElement)) {
          return true;
        }
      }

      // 8. Return false
      return false;
    };
  }
}

polyfillConcepts();

// 테스트
const testArray = [1, 2, 3, NaN, 5];
console.log("Array includes test:", testArray.includes(3)); // true
console.log("Array includes NaN test:", testArray.includes(NaN)); // true
```

### 2. 주요 기능별 Polyfill 구현

```javascript
// Promise Polyfill (간단한 버전)
function promisePolyfill() {
  if (typeof Promise === "undefined") {
    window.Promise = function (executor) {
      var self = this;
      self.state = "pending";
      self.value = undefined;
      self.handlers = [];

      function resolve(result) {
        if (self.state === "pending") {
          self.state = "fulfilled";
          self.value = result;
          self.handlers.forEach(handle);
          self.handlers = null;
        }
      }

      function reject(error) {
        if (self.state === "pending") {
          self.state = "rejected";
          self.value = error;
          self.handlers.forEach(handle);
          self.handlers = null;
        }
      }

      function handle(handler) {
        if (self.state === "pending") {
          self.handlers.push(handler);
        } else {
          if (
            self.state === "fulfilled" &&
            typeof handler.onFulfilled === "function"
          ) {
            handler.onFulfilled(self.value);
          }
          if (
            self.state === "rejected" &&
            typeof handler.onRejected === "function"
          ) {
            handler.onRejected(self.value);
          }
        }
      }

      self.then = function (onFulfilled, onRejected) {
        return new Promise(function (resolve, reject) {
          handle({
            onFulfilled: function (result) {
              try {
                if (typeof onFulfilled === "function") {
                  resolve(onFulfilled(result));
                } else {
                  resolve(result);
                }
              } catch (ex) {
                reject(ex);
              }
            },
            onRejected: function (error) {
              try {
                if (typeof onRejected === "function") {
                  resolve(onRejected(error));
                } else {
                  reject(error);
                }
              } catch (ex) {
                reject(ex);
              }
            },
          });
        });
      };

      self.catch = function (onRejected) {
        return self.then(null, onRejected);
      };

      try {
        executor(resolve, reject);
      } catch (error) {
        reject(error);
      }
    };

    Promise.resolve = function (value) {
      return new Promise(function (resolve) {
        resolve(value);
      });
    };

    Promise.reject = function (error) {
      return new Promise(function (resolve, reject) {
        reject(error);
      });
    };

    Promise.all = function (promises) {
      return new Promise(function (resolve, reject) {
        var results = [];
        var remaining = promises.length;

        if (remaining === 0) {
          resolve(results);
          return;
        }

        function resolver(index) {
          return function (value) {
            results[index] = value;
            if (--remaining === 0) {
              resolve(results);
            }
          };
        }

        for (var i = 0; i < promises.length; i++) {
          Promise.resolve(promises[i]).then(resolver(i), reject);
        }
      });
    };
  }
}

// Object.assign Polyfill
function objectAssignPolyfill() {
  if (typeof Object.assign !== "function") {
    Object.assign = function (target) {
      "use strict";

      if (target == null) {
        throw new TypeError("Cannot convert undefined or null to object");
      }

      var to = Object(target);

      for (var index = 1; index < arguments.length; index++) {
        var nextSource = arguments[index];

        if (nextSource != null) {
          for (var nextKey in nextSource) {
            if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
              to[nextKey] = nextSource[nextKey];
            }
          }
        }
      }

      return to;
    };
  }
}

// Array.from Polyfill
function arrayFromPolyfill() {
  if (!Array.from) {
    Array.from = function (arrayLike, mapFn, thisArg) {
      "use strict";

      var C = this;
      var items = Object(arrayLike);

      if (arrayLike == null) {
        throw new TypeError(
          "Array.from requires an array-like object - not null or undefined"
        );
      }

      var mapFunction = arguments.length > 1 ? mapFn : void undefined;
      var T;

      if (typeof mapFunction !== "undefined") {
        if (typeof mapFunction !== "function") {
          throw new TypeError(
            "Array.from: when provided, the second argument must be a function"
          );
        }

        if (arguments.length > 2) {
          T = thisArg;
        }
      }

      var len = parseInt(items.length);
      var A = typeof C === "function" ? Object(new C(len)) : new Array(len);
      var k = 0;
      var kValue;

      while (k < len) {
        kValue = items[k];
        if (mapFunction) {
          A[k] =
            typeof T === "undefined"
              ? mapFunction(kValue, k)
              : mapFunction.call(T, kValue, k);
        } else {
          A[k] = kValue;
        }
        k += 1;
      }

      A.length = len;
      return A;
    };
  }
}

// String.startsWith Polyfill
function stringStartsWithPolyfill() {
  if (!String.prototype.startsWith) {
    String.prototype.startsWith = function (search, pos) {
      return this.substr(!pos || pos < 0 ? 0 : +pos, search.length) === search;
    };
  }
}

// 모든 polyfill 로드
function loadAllPolyfills() {
  promisePolyfill();
  objectAssignPolyfill();
  arrayFromPolyfill();
  stringStartsWithPolyfill();

  console.log("모든 polyfill 로드 완료");
}

loadAllPolyfills();

// Polyfill 테스트
function testPolyfills() {
  console.log("=== Polyfill 테스트 ===");

  // Promise 테스트
  Promise.resolve("Hello")
    .then((result) => console.log("Promise test:", result))
    .catch((error) => console.error("Promise error:", error));

  // Object.assign 테스트
  const target = { a: 1, b: 2 };
  const source = { b: 4, c: 5 };
  const result = Object.assign(target, source);
  console.log("Object.assign test:", result);

  // Array.from 테스트
  const arrayLike = { 0: "a", 1: "b", 2: "c", length: 3 };
  const array = Array.from(arrayLike);
  console.log("Array.from test:", array);

  // String.startsWith 테스트
  const str = "Hello World";
  console.log("String.startsWith test:", str.startsWith("Hello"));
}

testPolyfills();
```

### 3. 현대적 Polyfill 전략

```javascript
// 조건부 Polyfill 로딩
function conditionalPolyfillLoading() {
  console.log("=== 조건부 Polyfill 로딩 ===");

  // 기능 감지 객체
  const FeatureDetection = {
    promises: typeof Promise !== "undefined",
    fetch: typeof fetch !== "undefined",
    intersectionObserver: typeof IntersectionObserver !== "undefined",
    webComponents: typeof customElements !== "undefined",
    modules: "noModule" in document.createElement("script"),

    // 복합 기능 감지
    modernBrowser: function () {
      return this.promises && this.fetch && this.modules;
    },

    // 필요한 polyfill 목록 생성
    getRequiredPolyfills: function () {
      const required = [];

      if (!this.promises) required.push("promise");
      if (!this.fetch) required.push("fetch");
      if (!this.intersectionObserver) required.push("intersection-observer");
      if (!this.webComponents) required.push("web-components");

      return required;
    },
  };

  console.log("Feature support:", {
    promises: FeatureDetection.promises,
    fetch: FeatureDetection.fetch,
    intersectionObserver: FeatureDetection.intersectionObserver,
    webComponents: FeatureDetection.webComponents,
    modernBrowser: FeatureDetection.modernBrowser(),
  });

  const requiredPolyfills = FeatureDetection.getRequiredPolyfills();
  console.log("Required polyfills:", requiredPolyfills);

  return requiredPolyfills;
}

// 동적 Polyfill 로더
class PolyfillLoader {
  constructor() {
    this.loadedPolyfills = new Set();
    this.loadingPromises = new Map();
  }

  async loadPolyfill(name, url) {
    if (this.loadedPolyfills.has(name)) {
      return Promise.resolve();
    }

    if (this.loadingPromises.has(name)) {
      return this.loadingPromises.get(name);
    }

    const loadPromise = this.loadScript(url)
      .then(() => {
        this.loadedPolyfills.add(name);
        console.log(`Polyfill loaded: ${name}`);
      })
      .catch((error) => {
        console.error(`Failed to load polyfill ${name}:`, error);
        throw error;
      });

    this.loadingPromises.set(name, loadPromise);
    return loadPromise;
  }

  loadScript(url) {
    return new Promise((resolve, reject) => {
      const script = document.createElement("script");
      script.src = url;
      script.onload = resolve;
      script.onerror = reject;

      // 브라우저 환경이 아닌 경우 시뮬레이션
      if (typeof document === "undefined") {
        console.log(`Script load simulated: ${url}`);
        setTimeout(resolve, 100);
        return;
      }

      document.head.appendChild(script);
    });
  }

  async loadMultiple(polyfills) {
    const polyfillMap = {
      promise:
        "https://cdn.jsdelivr.net/npm/es6-promise@4/dist/es6-promise.auto.min.js",
      fetch:
        "https://cdn.jsdelivr.net/npm/whatwg-fetch@3.6.2/dist/fetch.umd.js",
      "intersection-observer":
        "https://cdn.jsdelivr.net/npm/intersection-observer@0.12.0/intersection-observer.js",
      "web-components":
        "https://cdn.jsdelivr.net/npm/@webcomponents/webcomponentsjs@2.6.0/webcomponents-bundle.js",
    };

    const loadPromises = polyfills.map((name) => {
      const url = polyfillMap[name];
      return url
        ? this.loadPolyfill(name, url)
        : Promise.reject(new Error(`Unknown polyfill: ${name}`));
    });

    try {
      await Promise.all(loadPromises);
      console.log("All polyfills loaded successfully");
    } catch (error) {
      console.error("Some polyfills failed to load:", error);
    }
  }
}

// Polyfill.io 스타일 동적 로딩
function polyfillIoStyleLoading() {
  console.log("=== Polyfill.io 스타일 로딩 ===");

  // 사용자 에이전트 기반 기능 감지 (간단한 버전)
  function getUserAgentFeatures() {
    const ua = typeof navigator !== "undefined" ? navigator.userAgent : "";

    return {
      isIE: ua.indexOf("MSIE") !== -1 || ua.indexOf("Trident") !== -1,
      isOldChrome: /Chrome\/([0-9]+)/.test(ua) && parseInt(RegExp.$1) < 60,
      isOldFirefox: /Firefox\/([0-9]+)/.test(ua) && parseInt(RegExp.$1) < 55,
      isOldSafari:
        /Version\/([0-9]+).*Safari/.test(ua) && parseInt(RegExp.$1) < 12,

      needsPolyfills: function () {
        return (
          this.isIE || this.isOldChrome || this.isOldFirefox || this.isOldSafari
        );
      },
    };
  }

  const browserInfo = getUserAgentFeatures();
  console.log("Browser info:", browserInfo);

  if (browserInfo.needsPolyfills()) {
    console.log("Loading polyfills for older browser");
    // 실제로는 polyfill.io URL을 생성하여 로드
    const polyfillUrl =
      "https://polyfill.io/v3/polyfill.min.js?features=es6,es2017,es2018";
    console.log("Polyfill URL:", polyfillUrl);
  } else {
    console.log("Modern browser detected, no polyfills needed");
  }
}

// 사용 예시
async function polyfillLoadingExample() {
  const requiredPolyfills = conditionalPolyfillLoading();

  if (requiredPolyfills.length > 0) {
    const loader = new PolyfillLoader();
    await loader.loadMultiple(requiredPolyfills);
  }

  polyfillIoStyleLoading();
}

polyfillLoadingExample();
```

## 빌드 도구와 Transpilation

### 1. Babel 설정과 사용

```javascript
// Babel 설정 예시 (.babelrc 또는 babel.config.js)
const babelConfig = {
  // 프리셋 설정
  presets: [
    [
      "@babel/preset-env",
      {
        // 타겟 브라우저 설정
        targets: {
          browsers: ["> 1%", "last 2 versions", "not ie <= 8"],
        },

        // 사용량 기반 polyfill 포함
        useBuiltIns: "usage",
        corejs: 3,

        // 모듈 변환 설정
        modules: false, // Webpack에서 처리하도록 설정
      },
    ],
    "@babel/preset-react", // React 사용 시
    "@babel/preset-typescript", // TypeScript 사용 시
  ],

  // 플러그인 설정
  plugins: [
    "@babel/plugin-proposal-class-properties",
    "@babel/plugin-proposal-optional-chaining",
    "@babel/plugin-proposal-nullish-coalescing-operator",

    // 환경별 플러그인
    process.env.NODE_ENV === "development" && "react-hot-loader/babel",
  ].filter(Boolean),

  // 환경별 설정
  env: {
    test: {
      presets: [["@babel/preset-env", { targets: { node: "current" } }]],
    },
    production: {
      plugins: [
        "transform-remove-console",
        ["transform-remove-debugger", { removeDebugger: true }],
      ],
    },
  },
};

// Babel 변환 예시
function babelTransformationExample() {
  console.log("=== Babel 변환 예시 ===");

  // ES6+ 코드 (변환 전)
  const es6Code = `
        // 화살표 함수
        const add = (a, b) => a + b;
        
        // 클래스
        class Person {
            constructor(name) {
                this.name = name;
            }
            
            greet = () => {
                return \`Hello, \${this.name}!\`;
            }
        }
        
        // 구조 분해 할당
        const { name, age = 30 } = person;
        
        // 전개 연산자
        const newArray = [...oldArray, newItem];
        
        // async/await
        async function fetchData() {
            try {
                const response = await fetch('/api/data');
                return await response.json();
            } catch (error) {
                console.error('Error:', error);
            }
        }
        
        // 옵셔널 체이닝 (ES2020)
        const userName = user?.profile?.name;
        
        // Nullish coalescing (ES2020)
        const defaultName = userName ?? 'Anonymous';
    `;

  // ES5 변환 결과 (Babel 출력 시뮬레이션)
  const es5Code = `
        "use strict";
        
        // 화살표 함수 -> 일반 함수
        var add = function add(a, b) {
            return a + b;
        };
        
        // 클래스 -> 생성자 함수
        function Person(name) {
            var _this = this;
            this.name = name;
            
            this.greet = function() {
                return "Hello, " + _this.name + "!";
            };
        }
        
        // 구조 분해 할당 -> 개별 할당
        var name = person.name;
        var age = person.age === undefined ? 30 : person.age;
        
        // 전개 연산자 -> concat
        var newArray = oldArray.concat([newItem]);
        
        // async/await -> Promise 체인
        function fetchData() {
            return regeneratorRuntime.async(function fetchData$(_context) {
                while (1) {
                    switch (_context.prev = _context.next) {
                        case 0:
                            _context.prev = 0;
                            _context.next = 3;
                            return regeneratorRuntime.awrap(fetch('/api/data'));
                        case 3:
                            response = _context.sent;
                            _context.next = 6;
                            return regeneratorRuntime.awrap(response.json());
                        case 6:
                            return _context.abrupt("return", _context.sent);
                        case 9:
                            _context.prev = 9;
                            _context.t0 = _context["catch"](0);
                            console.error('Error:', _context.t0);
                        case 12:
                        case "end":
                            return _context.stop();
                    }
                }
            }, null, null, [[0, 9]]);
        }
        
        // 옵셔널 체이닝 -> 조건부 접근
        var userName = user != null && user.profile != null ? user.profile.name : void 0;
        
        // Nullish coalescing -> 조건부 할당
        var defaultName = userName !== null && userName !== void 0 ? userName : 'Anonymous';
    `;

  console.log("ES6+ 원본 코드 길이:", es6Code.length);
  console.log("ES5 변환 코드 길이:", es5Code.length);
  console.log(
    "변환으로 인한 크기 증가:",
    ((es5Code.length / es6Code.length - 1) * 100).toFixed(1) + "%"
  );
}

babelTransformationExample();
```

### 2. 웹팩과 폴리필 최적화

```javascript
// Webpack 설정 예시
const webpackConfig = {
  entry: {
    // 메인 앱
    app: "./src/index.js",

    // 폴리필 분리
    polyfills: "./src/polyfills.js",
  },

  output: {
    path: "./dist",
    filename: "[name].[contenthash].js",
    chunkFilename: "[name].[contenthash].chunk.js",
  },

  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: [
              [
                "@babel/preset-env",
                {
                  useBuiltIns: "entry",
                  corejs: 3,
                },
              ],
            ],
          },
        },
      },
    ],
  },

  optimization: {
    splitChunks: {
      chunks: "all",
      cacheGroups: {
        // 폴리필 청크 분리
        polyfills: {
          test: /[\\/]node_modules[\\/](core-js|regenerator-runtime)/,
          name: "polyfills",
          chunks: "all",
          priority: 20,
        },

        // 벤더 청크 분리
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          chunks: "all",
          priority: 10,
        },
      },
    },
  },

  plugins: [
    // HTML 플러그인으로 조건부 스크립트 로딩
    new HtmlWebpackPlugin({
      template: "./src/index.html",
      templateParameters: {
        modernScript: '<script type="module" src="app.js"></script>',
        legacyScript: '<script nomodule src="app.legacy.js"></script>',
      },
    }),
  ],
};

// 폴리필 파일 예시 (src/polyfills.js)
const polyfillsContent = `
// Core-js polyfills
import 'core-js/stable';
import 'regenerator-runtime/runtime';

// 조건부 폴리필
if (!window.IntersectionObserver) {
    import('intersection-observer');
}

if (!window.fetch) {
    import('whatwg-fetch');
}

// 커스텀 폴리필
import './polyfills/custom-polyfills.js';
`;

console.log("Webpack 폴리필 설정 완료");
```

## 결론

ES6+ 문법의 도입은 JavaScript 개발 경험을 크게 개선했지만, 브라우저 호환성을 위해 Polyfill과 Transpilation이 필수적입니다. 현대적인 개발에서는 Babel, Webpack 등의 도구를 활용하여 자동화된 변환과 최적화된 폴리필 로딩을 구현하는 것이 중요합니다. 또한 번들 크기 최적화와 성능을 고려하여 필요한 폴리필만 선택적으로 로드하는 전략을 수립해야 합니다.
