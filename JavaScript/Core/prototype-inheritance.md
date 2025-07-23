# 프로토타입 기반 상속과 **proto**, prototype 체계

## 개요

JavaScript는 클래스 기반이 아닌 프로토타입 기반 상속을 사용하는 언어입니다. 모든 객체는 다른 객체를 참조하는 프로토타입 체인을 가지며, 이를 통해 속성과 메서드를 상속받습니다.

## 탄생 배경

JavaScript는 1995년 Brendan Eich가 10일 만에 설계한 언어로, 당시 Java의 인기에 영향을 받았지만 더 유연한 객체 시스템이 필요했습니다. 프로토타입 기반 상속은 Self 언어에서 영감을 받아 구현되었습니다.

## 핵심 개념

### 1. 프로토타입 체인 기본

```javascript
// 모든 객체는 프로토타입을 가짐
const obj = {};
console.log(obj.__proto__); // Object.prototype
console.log(obj.__proto__.__proto__); // null (체인의 끝)

// 배열의 프로토타입 체인
const arr = [];
console.log(arr.__proto__); // Array.prototype
console.log(arr.__proto__.__proto__); // Object.prototype
console.log(arr.__proto__.__proto__.__proto__); // null
```

### 2. prototype vs **proto**

```javascript
function Person(name) {
  this.name = name;
}

// Person.prototype: 함수의 prototype 프로퍼티
Person.prototype.greet = function () {
  return `Hello, I'm ${this.name}`;
};

const person1 = new Person("Alice");

// person1.__proto__: 객체의 프로토타입 참조
console.log(person1.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true
```

### 3. 프로토타입 체인 탐색

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function () {
  return `${this.name} makes a sound`;
};

function Dog(name, breed) {
  Animal.call(this, name); // 부모 생성자 호출
  this.breed = breed;
}

// 프로토타입 상속 설정
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function () {
  return `${this.name} barks`;
};

const dog = new Dog("Rex", "German Shepherd");

console.log(dog.bark()); // 'Rex barks' - Dog.prototype에서
console.log(dog.speak()); // 'Rex makes a sound' - Animal.prototype에서
console.log(dog.name); // 'Rex' - 인스턴스 프로퍼티
```

## Object.create와 프로토타입 설정

### 1. Object.create 활용

```javascript
// 프로토타입을 명시적으로 설정
const animal = {
  type: "Animal",
  speak() {
    return `${this.name} is a ${this.type}`;
  },
};

const dog = Object.create(animal);
dog.name = "Buddy";
dog.type = "Dog";

console.log(dog.speak()); // 'Buddy is a Dog'
console.log(dog.__proto__ === animal); // true

// null 프로토타입 객체 생성
const pureObject = Object.create(null);
console.log(pureObject.__proto__); // undefined
console.log(pureObject.toString); // undefined
```

### 2. 프로토타입 체인 수정

```javascript
const parent = {
  parentMethod() {
    return "parent method";
  },
};

const child = {
  childMethod() {
    return "child method";
  },
};

// 프로토타입 설정
Object.setPrototypeOf(child, parent);

console.log(child.childMethod()); // 'child method'
console.log(child.parentMethod()); // 'parent method' - 상속됨

// 프로토타입 확인
console.log(Object.getPrototypeOf(child) === parent); // true
```

## 고급 상속 패턴

### 1. 믹스인 패턴

```javascript
const Flyable = {
  fly() {
    return `${this.name} is flying`;
  },
};

const Swimmable = {
  swim() {
    return `${this.name} is swimming`;
  },
};

function Duck(name) {
  this.name = name;
}

// 여러 믹스인 적용
Object.assign(Duck.prototype, Flyable, Swimmable);

Duck.prototype.quack = function () {
  return `${this.name} says quack`;
};

const duck = new Duck("Donald");
console.log(duck.fly()); // 'Donald is flying'
console.log(duck.swim()); // 'Donald is swimming'
console.log(duck.quack()); // 'Donald says quack'
```

### 2. 팩토리 함수와 프로토타입

```javascript
function createVehicle(type) {
  const vehicle = Object.create(Vehicle.prototype);
  vehicle.type = type;
  return vehicle;
}

const Vehicle = {
  prototype: {
    start() {
      return `${this.type} is starting`;
    },

    stop() {
      return `${this.type} is stopping`;
    },
  },
};

const car = createVehicle("Car");
const bike = createVehicle("Bike");

console.log(car.start()); // 'Car is starting'
console.log(bike.start()); // 'Bike is starting'
```

## 프로토타입 체인 최적화

### 1. 성능 고려사항

```javascript
// 비효율적: 깊은 프로토타입 체인
function Level1() {}
function Level2() {}
function Level3() {}
function Level4() {}

Level2.prototype = Object.create(Level1.prototype);
Level3.prototype = Object.create(Level2.prototype);
Level4.prototype = Object.create(Level3.prototype);

// 효율적: 플래트한 구조
function OptimizedClass() {}
OptimizedClass.prototype = Object.assign(
  {},
  Level1.prototype,
  Level2.prototype,
  Level3.prototype,
  Level4.prototype
);
```

### 2. 프로퍼티 섀도잉

```javascript
const parent = {
  name: "Parent",
  greet() {
    return `Hello from ${this.name}`;
  },
};

const child = Object.create(parent);
child.name = "Child"; // 프로퍼티 섀도잉

console.log(child.name); // 'Child' - 자신의 프로퍼티
console.log(child.greet()); // 'Hello from Child' - 상속된 메서드, this는 child

// hasOwnProperty로 구분
console.log(child.hasOwnProperty("name")); // true
console.log(child.hasOwnProperty("greet")); // false
```

## ES6 클래스와 프로토타입

### 1. 클래스 문법의 내부 동작

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} speaks`;
  }

  static getSpecies() {
    return "Unknown species";
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }

  speak() {
    return `${this.name} barks`;
  }

  static getSpecies() {
    return "Canis lupus";
  }
}

// 내부적으로는 여전히 프로토타입 기반
console.log(Dog.prototype.__proto__ === Animal.prototype); // true
console.log(Dog.__proto__ === Animal); // true (정적 상속)

const dog = new Dog("Rex", "Labrador");
console.log(dog.__proto__ === Dog.prototype); // true
```

### 2. super 키워드의 동작

```javascript
class Parent {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello, I'm ${this.name}`;
  }
}

class Child extends Parent {
  constructor(name, age) {
    super(name); // Parent 생성자 호출
    this.age = age;
  }

  greet() {
    const parentGreeting = super.greet(); // 부모 메서드 호출
    return `${parentGreeting} and I'm ${this.age} years old`;
  }
}

const child = new Child("Alice", 10);
console.log(child.greet()); // "Hello, I'm Alice and I'm 10 years old"
```

## 고급 프로토타입 기법

### 1. 프로토타입 기반 싱글톤

```javascript
const Singleton = (function () {
  let instance;

  function SingletonClass(data) {
    if (instance) {
      return instance;
    }

    this.data = data;
    this.timestamp = Date.now();

    instance = this;
    return instance;
  }

  SingletonClass.prototype.getData = function () {
    return this.data;
  };

  SingletonClass.prototype.getTimestamp = function () {
    return this.timestamp;
  };

  return SingletonClass;
})();

const s1 = new Singleton("first");
const s2 = new Singleton("second");

console.log(s1 === s2); // true
console.log(s1.getData()); // 'first'
```

### 2. 프로토타입 기반 옵저버 패턴

```javascript
function Observable() {
  this.observers = [];
}

Observable.prototype.addObserver = function (observer) {
  this.observers.push(observer);
};

Observable.prototype.removeObserver = function (observer) {
  const index = this.observers.indexOf(observer);
  if (index > -1) {
    this.observers.splice(index, 1);
  }
};

Observable.prototype.notify = function (data) {
  this.observers.forEach((observer) => {
    if (typeof observer.update === "function") {
      observer.update(data);
    }
  });
};

function Subject(name) {
  Observable.call(this);
  this.name = name;
  this.state = null;
}

Subject.prototype = Object.create(Observable.prototype);
Subject.prototype.constructor = Subject;

Subject.prototype.setState = function (state) {
  this.state = state;
  this.notify({ name: this.name, state: this.state });
};
```

## 프로토타입 디버깅과 도구

### 1. 프로토타입 체인 시각화

```javascript
function visualizePrototypeChain(obj, depth = 0) {
  if (!obj || depth > 10) return; // 무한 루프 방지

  const indent = "  ".repeat(depth);
  const constructor = obj.constructor?.name || "Unknown";

  console.log(`${indent}${constructor} {`);

  // 자신의 프로퍼티만 표시
  Object.getOwnPropertyNames(obj).forEach((prop) => {
    if (prop !== "constructor") {
      const value = typeof obj[prop] === "function" ? "[Function]" : obj[prop];
      console.log(`${indent}  ${prop}: ${value}`);
    }
  });

  console.log(`${indent}}`);

  const proto = Object.getPrototypeOf(obj);
  if (proto && proto !== Object.prototype) {
    console.log(`${indent}↓ __proto__`);
    visualizePrototypeChain(proto, depth + 1);
  }
}

// 사용 예시
class A {}
class B extends A {}
const b = new B();
visualizePrototypeChain(b);
```

### 2. 프로토타입 검증 도구

```javascript
function validatePrototypeChain(obj, expectedChain) {
  let current = obj;

  for (let i = 0; i < expectedChain.length; i++) {
    const expected = expectedChain[i];

    if (!current) {
      throw new Error(`Chain ended prematurely at step ${i}`);
    }

    if (current.constructor !== expected) {
      throw new Error(
        `Expected ${expected.name}, got ${current.constructor?.name} at step ${i}`
      );
    }

    current = Object.getPrototypeOf(current);
  }

  return true;
}

// 사용 예시
class Animal {}
class Dog extends Animal {}
const dog = new Dog();

try {
  validatePrototypeChain(dog, [Dog, Animal, Object]);
  console.log("Prototype chain is valid");
} catch (error) {
  console.error("Prototype chain validation failed:", error.message);
}
```

## 최신 JavaScript와 프로토타입

### 1. Private 필드와 프로토타입

```javascript
class ModernClass {
  #privateField = "private";

  constructor(public) {
    this.public = public;
  }

  getPrivate() {
    return this.#privateField;
  }
}

class ExtendedClass extends ModernClass {
  constructor(public, additional) {
    super(public);
    this.additional = additional;
  }

  // private 필드는 상속되지 않음
  // getPrivate() {
  //     return this.#privateField; // Error
  // }
}
```

### 2. 프록시와 프로토타입

```javascript
const handler = {
  getPrototypeOf(target) {
    console.log("Getting prototype of", target);
    return Object.getPrototypeOf(target);
  },

  setPrototypeOf(target, prototype) {
    console.log("Setting prototype of", target, "to", prototype);
    return Object.setPrototypeOf(target, prototype);
  },
};

const obj = {};
const proxied = new Proxy(obj, handler);

Object.setPrototypeOf(proxied, Array.prototype); // 로그 출력
console.log(Object.getPrototypeOf(proxied)); // 로그 출력
```

## 성능과 메모리 최적화

### 1. 프로토타입 메서드 vs 인스턴스 메서드

```javascript
// 메모리 효율적: 프로토타입 메서드
function EfficientClass(name) {
  this.name = name;
}

EfficientClass.prototype.greet = function () {
  return `Hello, ${this.name}`;
};

// 메모리 비효율적: 인스턴스 메서드
function InefficientClass(name) {
  this.name = name;
  this.greet = function () {
    // 각 인스턴스마다 새로운 함수
    return `Hello, ${this.name}`;
  };
}

// 1000개 인스턴스 생성 시 메모리 사용량 차이 확인
const efficient = [];
const inefficient = [];

for (let i = 0; i < 1000; i++) {
  efficient.push(new EfficientClass(`User${i}`));
  inefficient.push(new InefficientClass(`User${i}`));
}
```

## 결론

JavaScript의 프로토타입 기반 상속은 유연하고 강력한 객체 지향 프로그래밍을 가능하게 합니다. `__proto__`와 `prototype`의 차이점을 이해하고, 프로토타입 체인의 동작 원리를 파악하면 더 효율적이고 유지보수하기 쉬운 코드를 작성할 수 있습니다. ES6 클래스 문법도 내부적으로는 프로토타입을 사용하므로, 이 개념을 정확히 이해하는 것이 JavaScript 마스터의 핵심입니다.
