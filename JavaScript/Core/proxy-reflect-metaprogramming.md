# Proxy와 Reflect를 이용한 메타프로그래밍

## 개요

ES6에서 도입된 Proxy와 Reflect는 JavaScript의 메타프로그래밍 능력을 혁신적으로 확장시킨 기능들입니다. Proxy는 객체의 기본 연산을 가로채고 재정의할 수 있게 해주며, Reflect는 이러한 연산을 수행하는 표준화된 방법을 제공합니다.

## 탄생 배경

기존 JavaScript에서는 객체의 기본 동작을 커스터마이징하는 것이 제한적이었습니다. Proxy와 Reflect는 객체의 속성 접근, 할당, 열거, 함수 호출 등의 기본 연산을 완전히 제어할 수 있게 하여 진정한 메타프로그래밍을 가능하게 합니다.

## Proxy 기본 개념

### 1. Proxy 생성과 기본 트랩

```javascript
// 기본 Proxy 생성
const target = {
  name: "Alice",
  age: 30,
};

const handler = {
  get(target, property, receiver) {
    console.log(`Getting ${property}`);
    return Reflect.get(target, property, receiver);
  },

  set(target, property, value, receiver) {
    console.log(`Setting ${property} = ${value}`);
    return Reflect.set(target, property, value, receiver);
  },
};

const proxy = new Proxy(target, handler);

console.log(proxy.name); // Getting name, 'Alice'
proxy.age = 31; // Setting age = 31
console.log(proxy.age); // Getting age, 31

// 모든 트랩 종류
const allTrapsHandler = {
  // 속성 접근
  get(target, property, receiver) {
    console.log(`get: ${property}`);
    return Reflect.get(target, property, receiver);
  },

  // 속성 설정
  set(target, property, value, receiver) {
    console.log(`set: ${property} = ${value}`);
    return Reflect.set(target, property, value, receiver);
  },

  // 속성 존재 확인 (in 연산자)
  has(target, property) {
    console.log(`has: ${property}`);
    return Reflect.has(target, property);
  },

  // 속성 삭제
  deleteProperty(target, property) {
    console.log(`delete: ${property}`);
    return Reflect.deleteProperty(target, property);
  },

  // 속성 정의
  defineProperty(target, property, descriptor) {
    console.log(`defineProperty: ${property}`);
    return Reflect.defineProperty(target, property, descriptor);
  },

  // 속성 설명자 가져오기
  getOwnPropertyDescriptor(target, property) {
    console.log(`getOwnPropertyDescriptor: ${property}`);
    return Reflect.getOwnPropertyDescriptor(target, property);
  },

  // 속성 목록 가져오기
  ownKeys(target) {
    console.log("ownKeys");
    return Reflect.ownKeys(target);
  },

  // 프로토타입 가져오기
  getPrototypeOf(target) {
    console.log("getPrototypeOf");
    return Reflect.getPrototypeOf(target);
  },

  // 프로토타입 설정
  setPrototypeOf(target, prototype) {
    console.log("setPrototypeOf");
    return Reflect.setPrototypeOf(target, prototype);
  },

  // 객체 확장 가능 여부 확인
  isExtensible(target) {
    console.log("isExtensible");
    return Reflect.isExtensible(target);
  },

  // 객체 확장 방지
  preventExtensions(target) {
    console.log("preventExtensions");
    return Reflect.preventExtensions(target);
  },

  // 함수 호출
  apply(target, thisArg, argumentsList) {
    console.log("apply");
    return Reflect.apply(target, thisArg, argumentsList);
  },

  // 생성자 호출
  construct(target, argumentsList, newTarget) {
    console.log("construct");
    return Reflect.construct(target, argumentsList, newTarget);
  },
};
```

### 2. 속성 접근 제어

```javascript
// 속성 접근 제어 예시
function createSecureObject(obj, allowedProperties = []) {
  return new Proxy(obj, {
    get(target, property) {
      if (property.startsWith("_")) {
        throw new Error(`Private property ${property} is not accessible`);
      }

      if (
        allowedProperties.length > 0 &&
        !allowedProperties.includes(property)
      ) {
        throw new Error(`Property ${property} is not allowed`);
      }

      return Reflect.get(target, property);
    },

    set(target, property, value) {
      if (property.startsWith("_")) {
        throw new Error(`Private property ${property} cannot be set`);
      }

      if (typeof property === "string" && property.includes("admin")) {
        throw new Error("Admin properties are read-only");
      }

      return Reflect.set(target, property, value);
    },

    has(target, property) {
      if (property.startsWith("_")) {
        return false; // 프라이빗 속성은 존재하지 않는 것처럼 처리
      }
      return Reflect.has(target, property);
    },

    ownKeys(target) {
      return Reflect.ownKeys(target).filter((key) => !key.startsWith("_"));
    },
  });
}

const user = {
  name: "Alice",
  _password: "secret123",
  adminRole: "superuser",
  email: "alice@example.com",
};

const secureUser = createSecureObject(user, ["name", "email"]);

console.log(secureUser.name); // 'Alice'
console.log("_password" in secureUser); // false
console.log(Object.keys(secureUser)); // ['name', 'adminRole', 'email']

try {
  console.log(secureUser._password); // Error: Private property _password is not accessible
} catch (error) {
  console.log(error.message);
}
```

## Reflect의 활용

### 1. Reflect 메서드들

```javascript
// Reflect 메서드 활용 예시
const obj = { a: 1, b: 2 };

// 속성 접근
console.log(Reflect.get(obj, "a")); // 1
console.log(Reflect.has(obj, "a")); // true

// 속성 설정
Reflect.set(obj, "c", 3);
console.log(obj.c); // 3

// 속성 삭제
Reflect.deleteProperty(obj, "b");
console.log("b" in obj); // false

// 속성 정의
Reflect.defineProperty(obj, "d", {
  value: 4,
  writable: false,
  enumerable: true,
  configurable: false,
});

console.log(obj.d); // 4
console.log(Reflect.getOwnPropertyDescriptor(obj, "d"));
// { value: 4, writable: false, enumerable: true, configurable: false }

// 객체 키 목록
console.log(Reflect.ownKeys(obj)); // ['a', 'c', 'd']

// 함수와 생성자 호출
function greet(name) {
  return `Hello, ${name}!`;
}

console.log(Reflect.apply(greet, null, ["World"])); // 'Hello, World!'

function Person(name) {
  this.name = name;
}

const person = Reflect.construct(Person, ["Alice"]);
console.log(person.name); // 'Alice'
```

### 2. Reflect와 Proxy의 조합

```javascript
// Reflect를 사용한 안전한 Proxy 구현
function createValidatedProxy(target, validators = {}) {
  return new Proxy(target, {
    set(target, property, value, receiver) {
      // 유효성 검사
      if (validators[property]) {
        const isValid = validators[property](value);
        if (!isValid) {
          throw new Error(`Invalid value for ${property}: ${value}`);
        }
      }

      // 기본 동작 수행
      return Reflect.set(target, property, value, receiver);
    },

    get(target, property, receiver) {
      const result = Reflect.get(target, property, receiver);

      // 함수인 경우 바인딩 유지
      if (typeof result === "function") {
        return result.bind(target);
      }

      return result;
    },

    defineProperty(target, property, descriptor) {
      console.log(`Defining property: ${property}`);
      return Reflect.defineProperty(target, property, descriptor);
    },
  });
}

// 사용 예시
const userValidators = {
  age: (value) => typeof value === "number" && value >= 0 && value <= 150,
  email: (value) => typeof value === "string" && value.includes("@"),
  name: (value) => typeof value === "string" && value.length > 0,
};

const validatedUser = createValidatedProxy({}, userValidators);

validatedUser.name = "Alice"; // OK
validatedUser.age = 25; // OK
validatedUser.email = "alice@example.com"; // OK

try {
  validatedUser.age = -5; // Error: Invalid value for age: -5
} catch (error) {
  console.log(error.message);
}
```

## 고급 메타프로그래밍 패턴

### 1. 동적 속성 생성

```javascript
// 동적 속성 생성 Proxy
function createDynamicObject(generator) {
  return new Proxy(
    {},
    {
      get(target, property) {
        if (!(property in target)) {
          // 속성이 존재하지 않으면 동적으로 생성
          target[property] = generator(property);
        }
        return Reflect.get(target, property);
      },

      has(target, property) {
        // 모든 속성이 존재하는 것처럼 처리
        return true;
      },

      ownKeys(target) {
        // 실제로 접근된 속성들만 반환
        return Reflect.ownKeys(target);
      },
    }
  );
}

// 수학 함수 객체
const mathFunctions = createDynamicObject((property) => {
  switch (property) {
    case "square":
      return (x) => x * x;
    case "cube":
      return (x) => x * x * x;
    case "double":
      return (x) => x * 2;
    default:
      return (x) => x;
  }
});

console.log(mathFunctions.square(5)); // 25
console.log(mathFunctions.cube(3)); // 27
console.log(mathFunctions.double(10)); // 20

// 다국어 메시지 객체
const messages = createDynamicObject((key) => {
  const translations = {
    hello: { en: "Hello", ko: "안녕하세요", ja: "こんにちは" },
    goodbye: { en: "Goodbye", ko: "안녕히 가세요", ja: "さようなら" },
    thank_you: { en: "Thank you", ko: "감사합니다", ja: "ありがとう" },
  };

  return translations[key] || { en: key, ko: key, ja: key };
});

console.log(messages.hello.ko); // '안녕하세요'
console.log(messages.goodbye.en); // 'Goodbye'
```

### 2. 메서드 체이닝과 플루언트 인터페이스

```javascript
// 플루언트 인터페이스 구현
function createFluentAPI(initialValue = null) {
  const state = {
    value: initialValue,
    operations: [],
  };

  return new Proxy(
    {},
    {
      get(target, property) {
        if (property === "value") {
          // 최종 값 반환
          return state.operations.reduce((acc, op) => op(acc), state.value);
        }

        if (property === "reset") {
          return () => {
            state.operations = [];
            return target;
          };
        }

        if (property === "set") {
          return (value) => {
            state.value = value;
            return target;
          };
        }

        // 동적 메서드 생성
        return (...args) => {
          state.operations.push(createOperation(property, args));
          return target; // 체이닝을 위해 자기 자신 반환
        };
      },
    }
  );
}

function createOperation(name, args) {
  const operations = {
    add: (value) => value + args[0],
    multiply: (value) => value * args[0],
    divide: (value) => value / args[0],
    power: (value) => Math.pow(value, args[0]),
    filter: (array) => array.filter(args[0]),
    map: (array) => array.map(args[0]),
    reduce: (array) => array.reduce(args[0], args[1]),
  };

  return operations[name] || ((value) => value);
}

// 수치 계산 체이닝
const calculator = createFluentAPI(10);
const result = calculator.add(5).multiply(2).divide(3).power(2).value;

console.log(result); // ((10 + 5) * 2 / 3) ^ 2 = 100

// 배열 처리 체이닝
const arrayProcessor = createFluentAPI([1, 2, 3, 4, 5]);
const processed = arrayProcessor
  .filter((x) => x % 2 === 0)
  .map((x) => x * x)
  .reduce((sum, x) => sum + x, 0).value;

console.log(processed); // (2^2 + 4^2) = 20
```

### 3. 객체 관찰자 패턴

```javascript
// Observable 객체 구현
function createObservable(target) {
  const observers = new Set();

  const observable = new Proxy(target, {
    set(target, property, value, receiver) {
      const oldValue = target[property];
      const result = Reflect.set(target, property, value, receiver);

      if (result && oldValue !== value) {
        // 변경 알림
        observers.forEach((observer) => {
          observer({
            type: "set",
            property,
            value,
            oldValue,
            target: observable,
          });
        });
      }

      return result;
    },

    deleteProperty(target, property) {
      const oldValue = target[property];
      const result = Reflect.deleteProperty(target, property);

      if (result) {
        observers.forEach((observer) => {
          observer({
            type: "delete",
            property,
            oldValue,
            target: observable,
          });
        });
      }

      return result;
    },
  });

  // 관찰자 관리 메서드 추가
  observable.observe = function (callback) {
    observers.add(callback);
    return () => observers.delete(callback); // unsubscribe 함수 반환
  };

  observable.unobserve = function (callback) {
    observers.delete(callback);
  };

  return observable;
}

// 사용 예시
const user = createObservable({
  name: "Alice",
  age: 30,
});

const unsubscribe = user.observe((change) => {
  console.log(
    `${change.type}: ${change.property} changed from ${change.oldValue} to ${change.value}`
  );
});

user.name = "Bob"; // set: name changed from Alice to Bob
user.age = 31; // set: age changed from 30 to 31
delete user.age; // delete: age changed from 31 to undefined

unsubscribe(); // 관찰 중지
user.name = "Charlie"; // 알림 없음
```

## 실무 활용 사례

### 1. API 클라이언트 자동 생성

```javascript
// RESTful API 클라이언트 자동 생성
function createAPIClient(baseUrl, options = {}) {
  const defaultOptions = {
    headers: { "Content-Type": "application/json" },
    ...options,
  };

  return new Proxy(
    {},
    {
      get(target, property) {
        if (property in target) {
          return target[property];
        }

        // HTTP 메서드 처리
        const httpMethods = ["get", "post", "put", "delete", "patch"];
        if (httpMethods.includes(property)) {
          return (path, data = null, customOptions = {}) => {
            const url = `${baseUrl}${path}`;
            const options = {
              method: property.toUpperCase(),
              ...defaultOptions,
              ...customOptions,
            };

            if (data && ["post", "put", "patch"].includes(property)) {
              options.body = JSON.stringify(data);
            }

            return fetch(url, options).then((response) => {
              if (!response.ok) {
                throw new Error(
                  `HTTP ${response.status}: ${response.statusText}`
                );
              }
              return response.json();
            });
          };
        }

        // 리소스별 메서드 생성
        return new Proxy(
          {},
          {
            get(resourceTarget, method) {
              return (...args) => {
                const resourcePath = `/${property}`;

                switch (method) {
                  case "list":
                    return target.get(resourcePath);
                  case "get":
                    return target.get(`${resourcePath}/${args[0]}`);
                  case "create":
                    return target.post(resourcePath, args[0]);
                  case "update":
                    return target.put(`${resourcePath}/${args[0]}`, args[1]);
                  case "delete":
                    return target.delete(`${resourcePath}/${args[0]}`);
                  default:
                    throw new Error(`Unknown method: ${method}`);
                }
              };
            },
          }
        );
      },
    }
  );
}

// 사용 예시
const api = createAPIClient("https://api.example.com");

// 직접 HTTP 메서드 사용
api.get("/users").then((users) => console.log(users));
api.post("/users", { name: "Alice", email: "alice@example.com" });

// 리소스별 메서드 사용
api.users.list().then((users) => console.log(users));
api.users.create({ name: "Bob", email: "bob@example.com" });
api.users.get(123).then((user) => console.log(user));
api.users.update(123, { name: "Bobby" });
api.users.delete(123);
```

### 2. 데이터 검증 시스템

```javascript
// 스키마 기반 데이터 검증
class ValidationSchema {
  constructor(schema) {
    this.schema = schema;
  }

  validate(data) {
    const errors = [];

    for (const [key, rules] of Object.entries(this.schema)) {
      const value = data[key];

      if (rules.required && (value === undefined || value === null)) {
        errors.push(`${key} is required`);
        continue;
      }

      if (value !== undefined && rules.type && typeof value !== rules.type) {
        errors.push(`${key} must be of type ${rules.type}`);
      }

      if (rules.min !== undefined && value < rules.min) {
        errors.push(`${key} must be at least ${rules.min}`);
      }

      if (rules.max !== undefined && value > rules.max) {
        errors.push(`${key} must be at most ${rules.max}`);
      }

      if (rules.pattern && !rules.pattern.test(value)) {
        errors.push(`${key} does not match required pattern`);
      }
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }
}

function createValidatedModel(schema) {
  const validator = new ValidationSchema(schema);
  const data = {};

  return new Proxy(data, {
    set(target, property, value) {
      // 단일 속성 검증
      const singleFieldSchema = { [property]: schema[property] };
      const validation = validator.validate({ [property]: value });

      if (!validation.isValid) {
        throw new Error(`Validation failed: ${validation.errors.join(", ")}`);
      }

      return Reflect.set(target, property, value);
    },

    get(target, property) {
      if (property === "validate") {
        return () => validator.validate(target);
      }

      if (property === "toJSON") {
        return () => ({ ...target });
      }

      return Reflect.get(target, property);
    },
  });
}

// 사용 예시
const userSchema = {
  name: { type: "string", required: true },
  age: { type: "number", required: true, min: 0, max: 150 },
  email: {
    type: "string",
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  },
};

const user = createValidatedModel(userSchema);

user.name = "Alice"; // OK
user.age = 30; // OK
user.email = "alice@example.com"; // OK

console.log(user.validate()); // { isValid: true, errors: [] }

try {
  user.age = -5; // Error: Validation failed: age must be at least 0
} catch (error) {
  console.log(error.message);
}
```

### 3. 캐싱 시스템

```javascript
// 지능형 캐싱 시스템
function createCachedProxy(target, options = {}) {
  const cache = new Map();
  const {
    ttl = 60000, // 기본 TTL: 1분
    maxSize = 100,
    computeKey = (property, args) => `${property}:${JSON.stringify(args)}`,
  } = options;

  return new Proxy(target, {
    get(target, property) {
      const originalMethod = Reflect.get(target, property);

      if (typeof originalMethod !== "function") {
        return originalMethod;
      }

      return function (...args) {
        const cacheKey = computeKey(property, args);
        const cached = cache.get(cacheKey);

        // 캐시 히트 확인
        if (cached && Date.now() - cached.timestamp < ttl) {
          console.log(`Cache hit for ${cacheKey}`);
          return cached.value;
        }

        // 캐시 미스 - 실제 메서드 호출
        console.log(`Cache miss for ${cacheKey}`);
        const result = originalMethod.apply(this, args);

        // 캐시 크기 관리
        if (cache.size >= maxSize) {
          const firstKey = cache.keys().next().value;
          cache.delete(firstKey);
        }

        // 결과 캐싱
        cache.set(cacheKey, {
          value: result,
          timestamp: Date.now(),
        });

        return result;
      };
    },
  });
}

// 데이터베이스 시뮬레이션
class Database {
  constructor() {
    this.data = new Map([
      [1, { id: 1, name: "Alice", email: "alice@example.com" }],
      [2, { id: 2, name: "Bob", email: "bob@example.com" }],
      [3, { id: 3, name: "Charlie", email: "charlie@example.com" }],
    ]);
  }

  findById(id) {
    // 실제 데이터베이스 조회 시뮬레이션 (지연)
    console.log(`Querying database for user ${id}...`);
    return this.data.get(id);
  }

  findByEmail(email) {
    console.log(`Searching database for email ${email}...`);
    return Array.from(this.data.values()).find((user) => user.email === email);
  }
}

// 캐싱이 적용된 데이터베이스
const cachedDb = createCachedProxy(new Database(), {
  ttl: 5000, // 5초 TTL
  maxSize: 50,
});

// 사용 예시
console.log(cachedDb.findById(1)); // Cache miss, 데이터베이스 조회
console.log(cachedDb.findById(1)); // Cache hit, 캐시에서 반환
console.log(cachedDb.findByEmail("bob@example.com")); // Cache miss
console.log(cachedDb.findByEmail("bob@example.com")); // Cache hit
```

## 성능 고려사항과 최적화

### 1. Proxy 성능 최적화

```javascript
// 성능 최적화된 Proxy 구현
function createOptimizedProxy(target, handlers) {
  // 자주 사용되는 속성들을 미리 캐싱
  const cache = new Map();
  const hotProperties = new Set();

  return new Proxy(target, {
    get(target, property, receiver) {
      // 핫 속성 추적
      if (hotProperties.has(property)) {
        if (cache.has(property)) {
          return cache.get(property);
        }
      }

      const result = handlers.get
        ? handlers.get(target, property, receiver)
        : Reflect.get(target, property, receiver);

      // 함수가 아닌 값들을 캐싱
      if (typeof result !== "function") {
        cache.set(property, result);
        hotProperties.add(property);
      }

      return result;
    },

    set(target, property, value, receiver) {
      // 캐시 무효화
      cache.delete(property);

      return handlers.set
        ? handlers.set(target, property, value, receiver)
        : Reflect.set(target, property, value, receiver);
    },
  });
}

// 배치 처리를 위한 Proxy
function createBatchProxy(target, batchSize = 10) {
  const pendingOperations = [];
  let batchTimeout = null;

  return new Proxy(target, {
    set(target, property, value) {
      pendingOperations.push({ type: "set", property, value });

      if (pendingOperations.length >= batchSize) {
        processBatch();
      } else if (!batchTimeout) {
        batchTimeout = setTimeout(processBatch, 100);
      }

      return true;
    },
  });

  function processBatch() {
    if (batchTimeout) {
      clearTimeout(batchTimeout);
      batchTimeout = null;
    }

    console.log(`Processing batch of ${pendingOperations.length} operations`);

    pendingOperations.forEach((op) => {
      if (op.type === "set") {
        Reflect.set(target, op.property, op.value);
      }
    });

    pendingOperations.length = 0;
  }
}
```

### 2. 메모리 관리

```javascript
// 메모리 효율적인 Proxy 구현
function createMemoryEfficientProxy(target, options = {}) {
  const { weakRefs = true, cleanup = true, cleanupInterval = 60000 } = options;

  const metadata = weakRefs ? new WeakMap() : new Map();
  let cleanupTimer;

  if (cleanup && cleanupInterval > 0) {
    cleanupTimer = setInterval(() => {
      performCleanup();
    }, cleanupInterval);
  }

  const proxy = new Proxy(target, {
    get(target, property) {
      const value = Reflect.get(target, property);

      // 메타데이터 추적
      if (!metadata.has(target)) {
        metadata.set(target, {
          accessCount: new Map(),
          lastAccessed: new Map(),
        });
      }

      const meta = metadata.get(target);
      meta.accessCount.set(property, (meta.accessCount.get(property) || 0) + 1);
      meta.lastAccessed.set(property, Date.now());

      return value;
    },

    set(target, property, value) {
      const result = Reflect.set(target, property, value);

      // 큰 객체에 대한 약한 참조 사용
      if (weakRefs && typeof value === "object" && value !== null) {
        // WeakRef 사용으로 메모리 누수 방지
        const weakValue = new WeakRef(value);
        return Reflect.set(target, property, weakValue);
      }

      return result;
    },
  });

  function performCleanup() {
    if (!cleanup || !metadata.has(target)) return;

    const meta = metadata.get(target);
    const now = Date.now();
    const staleThreshold = 300000; // 5분

    // 오래된 메타데이터 정리
    for (const [property, lastAccessed] of meta.lastAccessed) {
      if (now - lastAccessed > staleThreshold) {
        meta.accessCount.delete(property);
        meta.lastAccessed.delete(property);
      }
    }
  }

  // 정리 메서드 제공
  proxy.cleanup = () => {
    if (cleanupTimer) {
      clearInterval(cleanupTimer);
    }
    metadata.delete(target);
  };

  return proxy;
}
```

## 결론

Proxy와 Reflect는 JavaScript의 메타프로그래밍을 위한 강력하고 유연한 도구입니다. 이들을 통해 객체의 기본 동작을 완전히 제어하고, 동적인 API 생성, 데이터 검증, 캐싱, 관찰자 패턴 등 다양한 고급 패턴을 구현할 수 있습니다. 다만 성능과 메모리 사용량을 고려하여 신중하게 사용해야 하며, 복잡한 로직보다는 명확하고 이해하기 쉬운 구현을 선호해야 합니다.
