# 클래스(Class)와 클래스 내부의 동작 원리 (Syntactic Sugar)

## 개요

ES6에서 도입된 클래스(Class) 문법은 JavaScript의 프로토타입 기반 상속을 더 직관적으로 작성할 수 있게 해주는 syntactic sugar입니다. 내부적으로는 여전히 프로토타입과 생성자 함수를 사용하지만, 클래스 기반 언어와 유사한 문법을 제공합니다.

## 탄생 배경

JavaScript 개발자들이 다른 객체지향 언어(Java, C#)에서 온 경우가 많아, 프로토타입 문법보다 클래스 문법을 선호했습니다. ES6 클래스는 기존 프로토타입 시스템의 복잡성을 숨기고 더 명확한 상속 구조를 제공합니다.

## 클래스 vs 생성자 함수 비교

### 전통적인 생성자 함수

```javascript
// ES5 스타일
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function () {
  return `Hello, I'm ${this.name}`;
};

Person.prototype.getAge = function () {
  return this.age;
};

Person.createAdult = function (name) {
  return new Person(name, 18);
};

const person1 = new Person("Alice", 25);
```

### ES6 클래스 문법

```javascript
// ES6 클래스
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    return `Hello, I'm ${this.name}`;
  }

  getAge() {
    return this.age;
  }

  static createAdult(name) {
    return new Person(name, 18);
  }
}

const person2 = new Person("Bob", 30);

// 내부적으로는 동일한 프로토타입 구조
console.log(typeof Person); // 'function'
console.log(Person.prototype.greet); // function
console.log(person2.__proto__ === Person.prototype); // true
```

## 클래스의 내부 동작 원리

### 1. 클래스 선언의 변환

```javascript
// 클래스 문법
class MyClass {
  constructor(value) {
    this.value = value;
  }

  method() {
    return this.value;
  }

  static staticMethod() {
    return "static";
  }
}

// 내부적으로 변환되는 코드 (개념적)
function MyClass(value) {
  this.value = value;
}

MyClass.prototype.method = function () {
  return this.value;
};

MyClass.staticMethod = function () {
  return "static";
};

// 클래스는 호이스팅되지 않음 (TDZ)
// console.log(MyClass); // ReferenceError
// class MyClass {} // 선언 후에야 사용 가능
```

### 2. 상속의 내부 동작

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Animal.call(this, name)과 유사
    this.breed = breed;
  }

  speak() {
    return `${this.name} barks`;
  }

  getBreed() {
    return this.breed;
  }
}

// 내부 프로토타입 체인 설정
console.log(Dog.prototype.__proto__ === Animal.prototype); // true
console.log(Dog.__proto__ === Animal); // true (정적 상속)

const dog = new Dog("Rex", "Labrador");
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
```

## 클래스의 고급 기능

### 1. Getter와 Setter

```javascript
class Rectangle {
  constructor(width, height) {
    this._width = width;
    this._height = height;
  }

  get width() {
    return this._width;
  }

  set width(value) {
    if (value <= 0) {
      throw new Error("Width must be positive");
    }
    this._width = value;
  }

  get area() {
    return this._width * this._height;
  }

  // setter만 있는 경우
  set dimensions({ width, height }) {
    this.width = width;
    this._height = height;
  }
}

const rect = new Rectangle(10, 5);
console.log(rect.area); // 50 (getter 호출)
rect.width = 20; // setter 호출
rect.dimensions = { width: 15, height: 8 }; // setter 호출
```

### 2. 정적 메서드와 프로퍼티

```javascript
class MathUtils {
  static PI = 3.14159;

  static add(a, b) {
    return a + b;
  }

  static multiply(a, b) {
    return a * b;
  }

  static get version() {
    return "1.0.0";
  }

  // 정적 블록 (ES2022)
  static {
    console.log("MathUtils class initialized");
    this.initialized = true;
  }
}

console.log(MathUtils.PI); // 3.14159
console.log(MathUtils.add(2, 3)); // 5
console.log(MathUtils.version); // '1.0.0'

// 인스턴스에서는 접근 불가
const utils = new MathUtils();
console.log(utils.PI); // undefined
```

### 3. Private 필드와 메서드 (ES2022)

```javascript
class BankAccount {
  #balance = 0; // private 필드
  #accountNumber; // private 필드

  constructor(initialBalance, accountNumber) {
    this.#balance = initialBalance;
    this.#accountNumber = accountNumber;
  }

  // private 메서드
  #validateAmount(amount) {
    return typeof amount === "number" && amount > 0;
  }

  // private static 메서드
  static #generateAccountNumber() {
    return Math.random().toString(36).substr(2, 9);
  }

  deposit(amount) {
    if (this.#validateAmount(amount)) {
      this.#balance += amount;
      return this.#balance;
    }
    throw new Error("Invalid amount");
  }

  withdraw(amount) {
    if (this.#validateAmount(amount) && this.#balance >= amount) {
      this.#balance -= amount;
      return this.#balance;
    }
    throw new Error("Invalid amount or insufficient funds");
  }

  get balance() {
    return this.#balance;
  }

  static createAccount(initialBalance) {
    return new BankAccount(initialBalance, this.#generateAccountNumber());
  }
}

const account = new BankAccount(1000, "ACC123");
console.log(account.balance); // 1000
account.deposit(500); // 1500
// console.log(account.#balance); // SyntaxError: Private field '#balance'
```

## 클래스 표현식과 동적 클래스

### 1. 클래스 표현식

```javascript
// 익명 클래스 표현식
const MyClass = class {
  constructor(value) {
    this.value = value;
  }

  getValue() {
    return this.value;
  }
};

// 명명된 클래스 표현식
const NamedClass = class InternalName {
  constructor() {
    console.log(InternalName.name); // 'InternalName'
  }

  getClassName() {
    return InternalName.name; // 내부에서만 접근 가능
  }
};

// 조건부 클래스 생성
function createClass(type) {
  if (type === "simple") {
    return class {
      constructor(value) {
        this.value = value;
      }
    };
  } else {
    return class {
      constructor(value) {
        this.value = value;
        this.timestamp = Date.now();
      }
    };
  }
}

const SimpleClass = createClass("simple");
const ComplexClass = createClass("complex");
```

### 2. 믹스인 패턴

```javascript
// 믹스인 함수
const Flyable = (Base) =>
  class extends Base {
    fly() {
      return `${this.name} is flying`;
    }
  };

const Swimmable = (Base) =>
  class extends Base {
    swim() {
      return `${this.name} is swimming`;
    }
  };

// 기본 클래스
class Animal {
  constructor(name) {
    this.name = name;
  }
}

// 믹스인 적용
class Duck extends Swimmable(Flyable(Animal)) {
  constructor(name) {
    super(name);
  }

  quack() {
    return `${this.name} says quack`;
  }
}

const duck = new Duck("Donald");
console.log(duck.fly()); // 'Donald is flying'
console.log(duck.swim()); // 'Donald is swimming'
console.log(duck.quack()); // 'Donald says quack'
```

## super 키워드의 동작

### 1. 생성자에서의 super

```javascript
class Parent {
  constructor(name) {
    this.name = name;
    console.log("Parent constructor");
  }
}

class Child extends Parent {
  constructor(name, age) {
    // super()는 반드시 this 사용 전에 호출
    super(name); // Parent.call(this, name)과 유사하지만 더 복잡
    this.age = age;
    console.log("Child constructor");
  }
}

// super() 없이는 에러
class InvalidChild extends Parent {
  constructor(name, age) {
    // this.age = age; // ReferenceError: Must call super constructor
    super(name);
    this.age = age;
  }
}
```

### 2. 메서드에서의 super

```javascript
class Parent {
  greet() {
    return "Hello from parent";
  }

  static staticGreet() {
    return "Static hello from parent";
  }
}

class Child extends Parent {
  greet() {
    const parentGreeting = super.greet(); // Parent.prototype.greet.call(this)
    return `${parentGreeting} and child`;
  }

  static staticGreet() {
    const parentGreeting = super.staticGreet(); // Parent.staticGreet()
    return `${parentGreeting} and child`;
  }
}

const child = new Child();
console.log(child.greet()); // 'Hello from parent and child'
console.log(Child.staticGreet()); // 'Static hello from parent and child'
```

## 클래스의 한계와 주의사항

### 1. 호이스팅 차이

```javascript
// 함수는 호이스팅됨
console.log(new MyFunction()); // 작동함

function MyFunction() {}

// 클래스는 호이스팅되지 않음
console.log(new MyClass()); // ReferenceError

class MyClass {}
```

### 2. 엄격 모드 자동 적용

```javascript
class StrictClass {
  constructor() {
    // 클래스 내부는 자동으로 strict mode
    undeclaredVariable = "error"; // ReferenceError
  }
}

function NonStrictFunction() {
  undeclaredVariable = "works"; // 전역 변수 생성 (non-strict mode)
}
```

### 3. 클래스 필드와 메서드 바인딩

```javascript
class EventHandler {
  name = "Handler";

  // 일반 메서드 - this 바인딩 손실 가능
  handleClick() {
    console.log(`${this.name} clicked`);
  }

  // 화살표 함수 필드 - this 바인딩 유지
  handleClickBound = () => {
    console.log(`${this.name} clicked`);
  };
}

const handler = new EventHandler();

// 문제 상황
const detachedMethod = handler.handleClick;
detachedMethod(); // TypeError: Cannot read property 'name' of undefined

// 해결된 상황
const boundMethod = handler.handleClickBound;
boundMethod(); // 'Handler clicked'
```

## 고급 클래스 패턴

### 1. 추상 클래스 패턴

```javascript
class AbstractShape {
  constructor() {
    if (new.target === AbstractShape) {
      throw new Error("Cannot instantiate abstract class");
    }
  }

  // 추상 메서드 패턴
  getArea() {
    throw new Error("Abstract method must be implemented");
  }

  // 구현된 메서드
  describe() {
    return `Shape with area: ${this.getArea()}`;
  }
}

class Circle extends AbstractShape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  getArea() {
    return Math.PI * this.radius * this.radius;
  }
}

// const shape = new AbstractShape(); // Error
const circle = new Circle(5);
console.log(circle.describe()); // 'Shape with area: 78.54'
```

### 2. 싱글톤 패턴

```javascript
class Singleton {
  static #instance = null;

  constructor() {
    if (Singleton.#instance) {
      return Singleton.#instance;
    }

    this.data = {};
    this.timestamp = Date.now();

    Singleton.#instance = this;
  }

  static getInstance() {
    if (!Singleton.#instance) {
      Singleton.#instance = new Singleton();
    }
    return Singleton.#instance;
  }

  setData(key, value) {
    this.data[key] = value;
  }

  getData(key) {
    return this.data[key];
  }
}

const s1 = new Singleton();
const s2 = new Singleton();
const s3 = Singleton.getInstance();

console.log((s1 === s2) === s3); // true
```

## 성능과 최적화

### 1. 클래스 vs 팩토리 함수 성능

```javascript
// 클래스 방식
class ClassBased {
  constructor(value) {
    this.value = value;
  }

  getValue() {
    return this.value;
  }
}

// 팩토리 함수 방식
function createObject(value) {
  return {
    value,
    getValue() {
      return this.value;
    },
  };
}

// 성능 테스트
console.time("Class creation");
for (let i = 0; i < 100000; i++) {
  new ClassBased(i);
}
console.timeEnd("Class creation");

console.time("Factory creation");
for (let i = 0; i < 100000; i++) {
  createObject(i);
}
console.timeEnd("Factory creation");
```

### 2. 메모리 사용량 최적화

```javascript
class OptimizedClass {
  constructor(data) {
    // 필요한 데이터만 저장
    this.id = data.id;
    this.name = data.name;
    // 전체 data 객체를 저장하지 않음
  }

  // 메서드는 프로토타입에 정의 (메모리 공유)
  toString() {
    return `${this.name} (${this.id})`;
  }

  // 정적 메서드로 유틸리티 함수 제공
  static compare(a, b) {
    return a.id - b.id;
  }
}
```

## 클래스와 함수형 프로그래밍

### 1. 클래스의 함수형 활용

```javascript
class FunctionalClass {
  constructor(value) {
    this.value = value;
  }

  map(fn) {
    return new this.constructor(fn(this.value));
  }

  flatMap(fn) {
    const result = fn(this.value);
    return result instanceof this.constructor
      ? result
      : new this.constructor(result);
  }

  filter(predicate) {
    return predicate(this.value) ? this : new this.constructor(null);
  }

  static of(value) {
    return new this(value);
  }
}

const wrapped = FunctionalClass.of(5)
  .map((x) => x * 2)
  .map((x) => x + 1)
  .filter((x) => x > 10);

console.log(wrapped.value); // 11
```

## 결론

ES6 클래스는 JavaScript의 프로토타입 기반 상속을 더 직관적으로 표현하는 syntactic sugar입니다. 내부적으로는 여전히 프로토타입과 생성자 함수를 사용하지만, 더 명확한 문법과 private 필드, static 블록 등의 현대적 기능을 제공합니다. 클래스를 효과적으로 사용하려면 그 내부 동작 원리를 이해하고, 적절한 패턴을 선택하는 것이 중요합니다.
