# 구조 분해 할당, 전개 연산자, Rest 파라미터

## 개요

ES6에서 도입된 구조 분해 할당(Destructuring Assignment), 전개 연산자(Spread Operator), Rest 파라미터는 JavaScript에서 데이터 조작과 함수 매개변수 처리를 더욱 간결하고 직관적으로 만들어주는 핵심 기능들입니다.

## 탄생 배경

복잡한 객체나 배열에서 필요한 값들을 추출하거나, 함수의 가변 매개변수를 처리하는 작업을 더 간편하게 하기 위해 도입되었습니다. Python, Ruby 등 다른 언어의 유사한 기능에서 영감을 받았습니다.

## 구조 분해 할당 (Destructuring Assignment)

### 1. 배열 구조 분해

```javascript
// 기본 배열 구조 분해
const numbers = [1, 2, 3, 4, 5];
const [first, second, third] = numbers;
console.log(first, second, third); // 1 2 3

// 일부 요소 건너뛰기
const [a, , c, , e] = numbers;
console.log(a, c, e); // 1 3 5

// 기본값 설정
const [x = 0, y = 0, z = 0] = [10, 20];
console.log(x, y, z); // 10 20 0

// 변수 교환
let num1 = 100;
let num2 = 200;
[num1, num2] = [num2, num1];
console.log(num1, num2); // 200 100

// 중첩 배열 구조 분해
const nested = [
  [1, 2],
  [3, 4],
  [5, 6],
];
const [[a1, a2], [b1, b2]] = nested;
console.log(a1, a2, b1, b2); // 1 2 3 4

// 나머지 요소 수집
const [head, ...tail] = numbers;
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]
```

### 2. 객체 구조 분해

```javascript
// 기본 객체 구조 분해
const person = {
  name: "Alice",
  age: 30,
  city: "Seoul",
  country: "Korea",
};

const { name, age, city } = person;
console.log(name, age, city); // Alice 30 Seoul

// 변수명 변경
const { name: fullName, age: years } = person;
console.log(fullName, years); // Alice 30

// 기본값 설정
const { name, age, profession = "Unknown" } = person;
console.log(profession); // Unknown

// 중첩 객체 구조 분해
const user = {
  id: 1,
  profile: {
    personal: {
      firstName: "John",
      lastName: "Doe",
    },
    contact: {
      email: "john@example.com",
      phone: "123-456-7890",
    },
  },
};

const {
  profile: {
    personal: { firstName, lastName },
    contact: { email },
  },
} = user;

console.log(firstName, lastName, email); // John Doe john@example.com

// 계산된 프로퍼티 이름
const key = "dynamicKey";
const obj = { [key]: "dynamic value", staticKey: "static value" };
const { [key]: dynamicValue, staticKey } = obj;
console.log(dynamicValue, staticKey); // dynamic value static value
```

### 3. 함수 매개변수 구조 분해

```javascript
// 객체 매개변수 구조 분해
function greetUser({ name, age, city = "Unknown" }) {
  console.log(`Hello ${name}, age ${age} from ${city}`);
}

greetUser({ name: "Bob", age: 25, city: "Busan" });
greetUser({ name: "Charlie", age: 35 }); // city는 'Unknown'

// 배열 매개변수 구조 분해
function processCoordinates([x, y, z = 0]) {
  console.log(`Position: x=${x}, y=${y}, z=${z}`);
}

processCoordinates([10, 20]); // z는 0
processCoordinates([5, 15, 25]);

// 복합 구조 분해
function analyzeUser({
  name,
  scores: [math, science, english],
  address: { city, country },
}) {
  console.log(`${name} from ${city}, ${country}`);
  console.log(`Scores: Math=${math}, Science=${science}, English=${english}`);
}

analyzeUser({
  name: "Alice",
  scores: [95, 87, 92],
  address: { city: "Seoul", country: "Korea" },
});
```

## 전개 연산자 (Spread Operator)

### 1. 배열 전개

```javascript
// 배열 복사
const original = [1, 2, 3];
const copy = [...original];
console.log(copy); // [1, 2, 3]
console.log(copy === original); // false (다른 참조)

// 배열 연결
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// 배열 중간에 요소 삽입
const numbers = [1, 2, 5, 6];
const inserted = [1, 2, 3, 4, 5, 6];
const result = [...numbers.slice(0, 2), 3, 4, ...numbers.slice(2)];
console.log(result); // [1, 2, 3, 4, 5, 6]

// 문자열을 배열로 변환
const str = "hello";
const chars = [...str];
console.log(chars); // ['h', 'e', 'l', 'l', 'o']

// Math 함수와 함께 사용
const nums = [3, 1, 4, 1, 5, 9, 2, 6];
console.log(Math.max(...nums)); // 9
console.log(Math.min(...nums)); // 1

// 배열 평탄화 (1레벨)
const nested = [
  [1, 2],
  [3, 4],
  [5, 6],
];
const flattened = [].concat(...nested);
console.log(flattened); // [1, 2, 3, 4, 5, 6]
```

### 2. 객체 전개

```javascript
// 객체 복사
const originalObj = { a: 1, b: 2, c: 3 };
const copiedObj = { ...originalObj };
console.log(copiedObj); // { a: 1, b: 2, c: 3 }

// 객체 병합
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// 객체 속성 덮어쓰기
const base = { name: "John", age: 30, city: "Seoul" };
const updated = { ...base, age: 31, country: "Korea" };
console.log(updated); // { name: 'John', age: 31, city: 'Seoul', country: 'Korea' }

// 조건부 속성 추가
const includeEmail = true;
const userProfile = {
  name: "Alice",
  age: 25,
  ...(includeEmail && { email: "alice@example.com" }),
};
console.log(userProfile); // { name: 'Alice', age: 25, email: 'alice@example.com' }

// 중첩 객체 부분 업데이트 (얕은 복사 주의)
const user = {
  id: 1,
  profile: {
    name: "Bob",
    settings: { theme: "dark", notifications: true },
  },
};

// 잘못된 방법 - 중첩 객체는 여전히 참조 공유
const wrongUpdate = {
  ...user,
  profile: { ...user.profile, name: "Bobby" },
};

// 올바른 깊은 업데이트
const correctUpdate = {
  ...user,
  profile: {
    ...user.profile,
    name: "Bobby",
    settings: { ...user.profile.settings, theme: "light" },
  },
};
```

### 3. 함수 호출에서의 전개

```javascript
// 함수 인수로 배열 전개
function sum(a, b, c) {
  return a + b + c;
}

const numbers = [1, 2, 3];
console.log(sum(...numbers)); // 6

// apply() 메서드 대체
const arr = [1, 5, 3, 9, 2];
// 기존 방식
console.log(Math.max.apply(null, arr)); // 9
// 전개 연산자 사용
console.log(Math.max(...arr)); // 9

// 생성자 함수와 함께 사용
function Point(x, y, z) {
  this.x = x;
  this.y = y;
  this.z = z;
}

const coordinates = [10, 20, 30];
const point = new Point(...coordinates);
console.log(point); // Point { x: 10, y: 20, z: 30 }
```

## Rest 파라미터

### 1. 함수 매개변수에서의 Rest

```javascript
// 기본 Rest 파라미터
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3)); // 6
console.log(sum(1, 2, 3, 4, 5)); // 15

// 다른 매개변수와 함께 사용
function greetAll(greeting, ...names) {
  return names.map((name) => `${greeting}, ${name}!`).join(" ");
}

console.log(greetAll("Hello", "Alice", "Bob", "Charlie"));
// "Hello, Alice! Hello, Bob! Hello, Charlie!"

// arguments 객체와의 차이점
function oldWay() {
  // arguments는 유사 배열 객체
  const args = Array.from(arguments);
  return args.reduce((sum, num) => sum + num, 0);
}

function newWay(...numbers) {
  // Rest 파라미터는 실제 배열
  return numbers.reduce((sum, num) => sum + num, 0);
}

// Rest 파라미터는 화살표 함수에서도 사용 가능
const multiply = (...numbers) =>
  numbers.reduce((product, num) => product * num, 1);
console.log(multiply(2, 3, 4)); // 24
```

### 2. 구조 분해와 Rest 결합

```javascript
// 배열에서 첫 번째 요소와 나머지 분리
const [first, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest); // [2, 3, 4, 5]

// 객체에서 특정 속성과 나머지 분리
const { name, age, ...otherInfo } = {
  name: "Alice",
  age: 30,
  city: "Seoul",
  country: "Korea",
  profession: "Developer",
};

console.log(name); // Alice
console.log(age); // 30
console.log(otherInfo); // { city: 'Seoul', country: 'Korea', profession: 'Developer' }

// 함수에서 일부 매개변수와 나머지 처리
function processUser({ name, email, ...preferences }) {
  console.log(`User: ${name} (${email})`);
  console.log("Preferences:", preferences);
}

processUser({
  name: "Bob",
  email: "bob@example.com",
  theme: "dark",
  notifications: true,
  language: "ko",
});
```

## 고급 패턴과 활용 사례

### 1. 불변 데이터 업데이트

```javascript
// 배열 불변 업데이트
class ImmutableArray {
  static add(array, item) {
    return [...array, item];
  }

  static remove(array, index) {
    return [...array.slice(0, index), ...array.slice(index + 1)];
  }

  static update(array, index, newItem) {
    return [...array.slice(0, index), newItem, ...array.slice(index + 1)];
  }

  static insert(array, index, item) {
    return [...array.slice(0, index), item, ...array.slice(index)];
  }
}

const numbers = [1, 2, 3, 4, 5];
console.log(ImmutableArray.add(numbers, 6)); // [1, 2, 3, 4, 5, 6]
console.log(ImmutableArray.remove(numbers, 2)); // [1, 2, 4, 5]
console.log(ImmutableArray.update(numbers, 1, 10)); // [1, 10, 3, 4, 5]

// 객체 불변 업데이트
class ImmutableObject {
  static set(obj, key, value) {
    return { ...obj, [key]: value };
  }

  static merge(obj, updates) {
    return { ...obj, ...updates };
  }

  static remove(obj, key) {
    const { [key]: removed, ...rest } = obj;
    return rest;
  }

  static updateNested(obj, path, value) {
    if (path.length === 1) {
      return { ...obj, [path[0]]: value };
    }

    const [head, ...tail] = path;
    return {
      ...obj,
      [head]: this.updateNested(obj[head], tail, value),
    };
  }
}

const user = { name: "Alice", age: 30, address: { city: "Seoul" } };
const updated = ImmutableObject.updateNested(
  user,
  ["address", "city"],
  "Busan"
);
console.log(updated); // { name: 'Alice', age: 30, address: { city: 'Busan' } }
```

### 2. 함수형 프로그래밍 패턴

```javascript
// 커링과 부분 적용
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function (...nextArgs) {
      return curried.apply(this, [...args, ...nextArgs]);
    };
  };
}

const add = curry((a, b, c) => a + b + c);
const addFive = add(5);
const addFiveAndThree = addFive(3);
console.log(addFiveAndThree(2)); // 10

// 함수 조합
const compose =
  (...fns) =>
  (value) =>
    fns.reduceRight((acc, fn) => fn(acc), value);
const pipe =
  (...fns) =>
  (value) =>
    fns.reduce((acc, fn) => fn(acc), value);

const addOne = (x) => x + 1;
const double = (x) => x * 2;
const square = (x) => x * x;

const composedFn = compose(square, double, addOne);
const pipedFn = pipe(addOne, double, square);

console.log(composedFn(3)); // ((3 + 1) * 2)² = 64
console.log(pipedFn(3)); // ((3 + 1) * 2)² = 64
```

### 3. 고급 구조 분해 패턴

```javascript
// 동적 구조 분해
function extractFields(obj, ...fields) {
  return fields.reduce((result, field) => {
    if (field in obj) {
      result[field] = obj[field];
    }
    return result;
  }, {});
}

const userData = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  age: 30,
  city: "Seoul",
};

console.log(extractFields(userData, "name", "email", "city"));
// { name: 'Alice', email: 'alice@example.com', city: 'Seoul' }

// 조건부 구조 분해
function processApiResponse(response) {
  const {
    data: {
      user: { name, email } = {},
      settings: { theme = "light", notifications = true } = {},
      ...otherData
    } = {},
  } = response;

  return {
    userName: name || "Unknown",
    userEmail: email || "No email",
    userTheme: theme,
    userNotifications: notifications,
    additionalData: otherData,
  };
}

// 재귀적 구조 분해
function flattenObject(obj, prefix = "") {
  return Object.keys(obj).reduce((acc, key) => {
    const newKey = prefix ? `${prefix}.${key}` : key;

    if (
      typeof obj[key] === "object" &&
      obj[key] !== null &&
      !Array.isArray(obj[key])
    ) {
      return { ...acc, ...flattenObject(obj[key], newKey) };
    }

    return { ...acc, [newKey]: obj[key] };
  }, {});
}

const nested = {
  user: {
    profile: {
      name: "Alice",
      age: 30,
    },
    settings: {
      theme: "dark",
    },
  },
};

console.log(flattenObject(nested));
// { 'user.profile.name': 'Alice', 'user.profile.age': 30, 'user.settings.theme': 'dark' }
```

## 실무 활용 사례

### 1. React 컴포넌트 props 처리

```javascript
// Props 구조 분해와 기본값
function UserCard({
  name,
  email,
  avatar = "/default-avatar.png",
  isOnline = false,
  ...otherProps
}) {
  return {
    name,
    email,
    avatar,
    isOnline,
    className: `user-card ${isOnline ? "online" : "offline"}`,
    ...otherProps,
  };
}

// 조건부 props 전달
function ConditionalProps({ showEmail, user }) {
  const cardProps = {
    name: user.name,
    avatar: user.avatar,
    ...(showEmail && { email: user.email }),
    ...(user.isVip && { badge: "VIP" }),
  };

  return cardProps;
}
```

### 2. API 데이터 처리

```javascript
// API 응답 정규화
function normalizeApiResponse(response) {
  const {
    data: {
      users = [],
      pagination: { page = 1, totalPages = 1, hasNext = false } = {},
      filters = {},
    } = {},
  } = response;

  return {
    users: users.map(({ id, name, email, profile = {} }) => ({
      id,
      name,
      email,
      avatar: profile.avatar || "/default-avatar.png",
      isActive: profile.isActive || false,
    })),
    pagination: { page, totalPages, hasNext },
    activeFilters: filters,
  };
}

// 에러 처리와 구조 분해
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    const {
      success,
      data: { user, permissions = [] } = {},
      error,
    } = await response.json();

    if (!success) {
      throw new Error(error?.message || "Failed to fetch user data");
    }

    return { user, permissions };
  } catch (error) {
    console.error("API Error:", error.message);
    return { user: null, permissions: [], error: error.message };
  }
}
```

### 3. 설정 관리와 병합

```javascript
// 설정 객체 병합
class ConfigManager {
  constructor(defaultConfig = {}) {
    this.defaultConfig = defaultConfig;
  }

  mergeConfig(userConfig = {}) {
    return this.deepMerge(this.defaultConfig, userConfig);
  }

  deepMerge(target, source) {
    const result = { ...target };

    for (const key in source) {
      if (
        source[key] &&
        typeof source[key] === "object" &&
        !Array.isArray(source[key])
      ) {
        result[key] = this.deepMerge(result[key] || {}, source[key]);
      } else {
        result[key] = source[key];
      }
    }

    return result;
  }

  // 특정 섹션만 업데이트
  updateSection(section, updates) {
    return {
      ...this.defaultConfig,
      [section]: {
        ...this.defaultConfig[section],
        ...updates,
      },
    };
  }
}

const configManager = new ConfigManager({
  api: {
    baseUrl: "https://api.example.com",
    timeout: 5000,
    retries: 3,
  },
  ui: {
    theme: "light",
    language: "en",
  },
});

const userConfig = {
  api: { timeout: 10000 },
  ui: { theme: "dark" },
};

console.log(configManager.mergeConfig(userConfig));
```

## 성능 고려사항

### 1. 얕은 복사 vs 깊은 복사

```javascript
// 얕은 복사의 함정
const original = {
  name: "Alice",
  hobbies: ["reading", "swimming"],
  address: { city: "Seoul", country: "Korea" },
};

const shallow = { ...original };
shallow.hobbies.push("coding"); // 원본도 변경됨!
console.log(original.hobbies); // ['reading', 'swimming', 'coding']

// 안전한 깊은 복사
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj.getTime());
  if (obj instanceof Array) return obj.map((item) => deepClone(item));

  const cloned = {};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }
  return cloned;
}

const deep = deepClone(original);
deep.hobbies.push("gaming");
console.log(original.hobbies); // ['reading', 'swimming', 'coding'] (변경되지 않음)
```

### 2. 메모리 효율성

```javascript
// 대용량 배열 처리 시 주의사항
function processLargeArray(largeArray) {
  // 비효율적 - 전체 배열 복사
  const processed = [...largeArray].map((item) => item * 2);

  // 효율적 - 필요한 부분만 처리
  const processedChunk = largeArray.slice(0, 1000).map((item) => item * 2);

  return processedChunk;
}

// 객체 병합 최적화
function efficientMerge(target, ...sources) {
  // 변경이 없으면 원본 반환
  if (sources.length === 0) return target;

  let hasChanges = false;
  const result = {};

  // 먼저 target 복사
  for (const key in target) {
    result[key] = target[key];
  }

  // sources 병합
  for (const source of sources) {
    for (const key in source) {
      if (result[key] !== source[key]) {
        result[key] = source[key];
        hasChanges = true;
      }
    }
  }

  return hasChanges ? result : target;
}
```

## 결론

구조 분해 할당, 전개 연산자, Rest 파라미터는 JavaScript에서 데이터 조작을 더 간결하고 읽기 쉽게 만들어주는 강력한 도구들입니다. 이들을 적절히 활용하면 불변성을 유지하면서도 효율적인 코드를 작성할 수 있으며, 함수형 프로그래밍 패턴과 결합하여 더욱 견고한 애플리케이션을 구축할 수 있습니다.
