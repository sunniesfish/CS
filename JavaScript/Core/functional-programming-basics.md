# 함수형 프로그래밍 기본 (Functional Programming Basics)

## 개요

함수형 프로그래밍은 함수를 일급 객체로 취급하고, 순수 함수, 불변성, 고차 함수 등의 개념을 통해 부작용을 최소화하고 예측 가능한 코드를 작성하는 패러다임입니다.

## 탄생 배경

복잡한 상태 관리와 부작용으로 인한 버그를 줄이고, 코드의 재사용성과 테스트 가능성을 높이기 위해 함수형 프로그래밍 개념이 JavaScript에 도입되었습니다.

## 순수 함수 (Pure Functions)

### 1. 순수 함수의 특징

```javascript
// 순수 함수 - 같은 입력에 대해 항상 같은 출력
function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

// 순수 함수 - 부작용 없음
function calculateCircleArea(radius) {
  return Math.PI * radius * radius;
}

// 비순수 함수 - 외부 상태에 의존
let counter = 0;
function impureIncrement() {
  return ++counter; // 외부 변수 변경
}

// 비순수 함수 - 같은 입력에 대해 다른 출력
function impureRandom() {
  return Math.random(); // 매번 다른 결과
}

// 순수 함수로 개선
function pureIncrement(value) {
  return value + 1;
}

function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    getValue: () => count,
  };
}
```

### 2. 순수 함수의 장점

```javascript
// 테스트 가능성
function sum(numbers) {
  return numbers.reduce((acc, num) => acc + num, 0);
}

// 예측 가능한 결과
console.log(sum([1, 2, 3])); // 항상 6
console.log(sum([1, 2, 3])); // 항상 6

// 메모이제이션 가능
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const memoizedSum = memoize(sum);
console.log(memoizedSum([1, 2, 3, 4])); // 계산됨
console.log(memoizedSum([1, 2, 3, 4])); // 캐시에서 반환
```

## 고차 함수 (Higher-Order Functions)

### 1. 함수를 매개변수로 받는 함수

```javascript
// 고차 함수 - 함수를 매개변수로 받음
function applyOperation(numbers, operation) {
  return numbers.map(operation);
}

function double(x) {
  return x * 2;
}

function square(x) {
  return x * x;
}

const numbers = [1, 2, 3, 4, 5];
console.log(applyOperation(numbers, double)); // [2, 4, 6, 8, 10]
console.log(applyOperation(numbers, square)); // [1, 4, 9, 16, 25]

// 조건부 실행
function conditionalExecute(condition, fn, ...args) {
  return condition ? fn(...args) : null;
}

const result = conditionalExecute(true, add, 5, 3); // 8
const noResult = conditionalExecute(false, add, 5, 3); // null
```

### 2. 함수를 반환하는 함수

```javascript
// 커링 (Currying)
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function (...nextArgs) {
        return curried.apply(this, args.concat(nextArgs));
      };
    }
  };
}

const curriedAdd = curry((a, b, c) => a + b + c);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6

// 부분 적용 (Partial Application)
function partial(fn, ...presetArgs) {
  return function (...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

const addFive = partial(add, 5);
console.log(addFive(3)); // 8

// 함수 조합 (Function Composition)
function compose(...functions) {
  return function (value) {
    return functions.reduceRight((acc, fn) => fn(acc), value);
  };
}

const addOne = (x) => x + 1;
const multiplyByTwo = (x) => x * 2;
const subtractThree = (x) => x - 3;

const composedFunction = compose(subtractThree, multiplyByTwo, addOne);
console.log(composedFunction(5)); // ((5 + 1) * 2) - 3 = 9
```

## map, filter, reduce 활용

### 1. map - 변환

```javascript
// 기본 map 사용
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((x) => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// 객체 배열 변환
const users = [
  { id: 1, name: "Alice", age: 25 },
  { id: 2, name: "Bob", age: 30 },
  { id: 3, name: "Charlie", age: 35 },
];

const userNames = users.map((user) => user.name);
console.log(userNames); // ['Alice', 'Bob', 'Charlie']

// 복잡한 변환
const enrichedUsers = users.map((user) => ({
  ...user,
  isAdult: user.age >= 18,
  displayName: `${user.name} (${user.age})`,
}));

// 체이닝과 함께 사용
const processedData = users
  .map((user) => ({ ...user, age: user.age + 1 }))
  .map((user) => ({ ...user, category: user.age >= 30 ? "senior" : "junior" }));
```

### 2. filter - 필터링

```javascript
// 기본 filter 사용
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const evenNumbers = numbers.filter((x) => x % 2 === 0);
console.log(evenNumbers); // [2, 4, 6, 8, 10]

// 객체 필터링
const adults = users.filter((user) => user.age >= 30);
console.log(adults); // Bob과 Charlie

// 복합 조건 필터링
function createFilter(criteria) {
  return function (item) {
    return Object.keys(criteria).every((key) => {
      const criterion = criteria[key];
      const value = item[key];

      if (typeof criterion === "function") {
        return criterion(value);
      }

      return value === criterion;
    });
  };
}

const seniorUsers = users.filter(
  createFilter({
    age: (age) => age >= 30,
    name: (name) => name.length > 3,
  })
);

// 중복 제거 필터
function uniqueBy(array, keyFn) {
  const seen = new Set();
  return array.filter((item) => {
    const key = keyFn(item);
    if (seen.has(key)) {
      return false;
    }
    seen.add(key);
    return true;
  });
}

const duplicateUsers = [...users, ...users];
const uniqueUsers = uniqueBy(duplicateUsers, (user) => user.id);
```

### 3. reduce - 축약

```javascript
// 기본 reduce 사용
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum); // 15

const product = numbers.reduce((acc, curr) => acc * curr, 1);
console.log(product); // 120

// 객체로 그룹화
const groupedUsers = users.reduce((acc, user) => {
  const ageGroup = user.age >= 30 ? "senior" : "junior";
  if (!acc[ageGroup]) {
    acc[ageGroup] = [];
  }
  acc[ageGroup].push(user);
  return acc;
}, {});

// 통계 계산
const stats = numbers.reduce(
  (acc, curr) => ({
    count: acc.count + 1,
    sum: acc.sum + curr,
    min: Math.min(acc.min, curr),
    max: Math.max(acc.max, curr),
  }),
  {
    count: 0,
    sum: 0,
    min: Infinity,
    max: -Infinity,
  }
);

// 플래튼 (flatten)
const nestedArrays = [
  [1, 2],
  [3, 4],
  [5, 6],
];
const flattened = nestedArrays.reduce((acc, curr) => acc.concat(curr), []);
console.log(flattened); // [1, 2, 3, 4, 5, 6]

// 파이프라인 구현
function pipe(...functions) {
  return function (value) {
    return functions.reduce((acc, fn) => fn(acc), value);
  };
}

const pipeline = pipe(
  (x) => x + 1,
  (x) => x * 2,
  (x) => x - 3
);

console.log(pipeline(5)); // ((5 + 1) * 2) - 3 = 9
```

## 함수형 프로그래밍 패턴

### 1. 함수 조합과 파이프라인

```javascript
// 함수 조합 유틸리티
const FP = {
  compose:
    (...fns) =>
    (value) =>
      fns.reduceRight((acc, fn) => fn(acc), value),
  pipe:
    (...fns) =>
    (value) =>
      fns.reduce((acc, fn) => fn(acc), value),
  curry: (fn) =>
    function curried(...args) {
      return args.length >= fn.length
        ? fn.apply(this, args)
        : (...nextArgs) => curried.apply(this, args.concat(nextArgs));
    },
};

// 데이터 처리 파이프라인
const processUserData = FP.pipe(
  (users) => users.filter((user) => user.age >= 18),
  (users) =>
    users.map((user) => ({
      ...user,
      displayName: `${user.name} (${user.age})`,
    })),
  (users) => users.sort((a, b) => a.age - b.age)
);

const processedUsers = processUserData(users);

// 재사용 가능한 함수들
const isAdult = (user) => user.age >= 18;
const addDisplayName = (user) => ({
  ...user,
  displayName: `${user.name} (${user.age})`,
});
const sortByAge = (users) => users.sort((a, b) => a.age - b.age);

const processUserDataReusable = FP.pipe(
  (users) => users.filter(isAdult),
  (users) => users.map(addDisplayName),
  sortByAge
);
```

### 2. 모나드 패턴 (Maybe/Optional)

```javascript
// Maybe 모나드 구현
class Maybe {
  constructor(value) {
    this.value = value;
  }

  static of(value) {
    return new Maybe(value);
  }

  static nothing() {
    return new Maybe(null);
  }

  isNothing() {
    return this.value === null || this.value === undefined;
  }

  map(fn) {
    return this.isNothing() ? Maybe.nothing() : Maybe.of(fn(this.value));
  }

  flatMap(fn) {
    return this.isNothing() ? Maybe.nothing() : fn(this.value);
  }

  filter(predicate) {
    return this.isNothing() || !predicate(this.value) ? Maybe.nothing() : this;
  }

  getOrElse(defaultValue) {
    return this.isNothing() ? defaultValue : this.value;
  }
}

// Maybe 사용 예시
function safeDivide(a, b) {
  return b === 0 ? Maybe.nothing() : Maybe.of(a / b);
}

const result = Maybe.of(10)
  .flatMap((x) => safeDivide(x, 2))
  .map((x) => x * 3)
  .filter((x) => x > 10)
  .getOrElse(0);

console.log(result); // 15

// 안전한 속성 접근
function safeGet(obj, path) {
  return path.reduce((current, key) => {
    return current.flatMap((val) =>
      val && val[key] !== undefined ? Maybe.of(val[key]) : Maybe.nothing()
    );
  }, Maybe.of(obj));
}

const user = {
  profile: {
    address: {
      street: "123 Main St",
    },
  },
};

const street = safeGet(user, ["profile", "address", "street"]).getOrElse(
  "No address"
);
console.log(street); // '123 Main St'
```

### 3. 함수형 데이터 구조

```javascript
// 불변 리스트 구현
class ImmutableList {
  constructor(items = []) {
    this.items = [...items];
    Object.freeze(this.items);
    Object.freeze(this);
  }

  static of(...items) {
    return new ImmutableList(items);
  }

  map(fn) {
    return new ImmutableList(this.items.map(fn));
  }

  filter(predicate) {
    return new ImmutableList(this.items.filter(predicate));
  }

  reduce(fn, initial) {
    return this.items.reduce(fn, initial);
  }

  append(item) {
    return new ImmutableList([...this.items, item]);
  }

  prepend(item) {
    return new ImmutableList([item, ...this.items]);
  }

  concat(other) {
    return new ImmutableList([...this.items, ...other.items]);
  }

  take(n) {
    return new ImmutableList(this.items.slice(0, n));
  }

  drop(n) {
    return new ImmutableList(this.items.slice(n));
  }

  toArray() {
    return [...this.items];
  }
}

// 사용 예시
const list = ImmutableList.of(1, 2, 3, 4, 5);
const result = list
  .map((x) => x * 2)
  .filter((x) => x > 5)
  .append(12)
  .take(3);

console.log(result.toArray()); // [6, 8, 10]
console.log(list.toArray()); // [1, 2, 3, 4, 5] - 원본 불변
```

## 실무 적용 사례

### 1. 데이터 변환 파이프라인

```javascript
// API 응답 데이터 처리
const apiResponse = {
  users: [
    {
      id: 1,
      first_name: "John",
      last_name: "Doe",
      birth_year: 1990,
      active: true,
    },
    {
      id: 2,
      first_name: "Jane",
      last_name: "Smith",
      birth_year: 1985,
      active: false,
    },
    {
      id: 3,
      first_name: "Bob",
      last_name: "Johnson",
      birth_year: 1995,
      active: true,
    },
  ],
};

// 함수형 변환 파이프라인
const transformUserData = FP.pipe(
  (data) => data.users,
  (users) => users.filter((user) => user.active),
  (users) =>
    users.map((user) => ({
      id: user.id,
      fullName: `${user.first_name} ${user.last_name}`,
      age: new Date().getFullYear() - user.birth_year,
      isAdult: new Date().getFullYear() - user.birth_year >= 18,
    })),
  (users) => users.sort((a, b) => a.age - b.age)
);

const transformedData = transformUserData(apiResponse);
console.log(transformedData);

// 재사용 가능한 변환 함수들
const extractUsers = (data) => data.users;
const filterActive = (users) => users.filter((user) => user.active);
const normalizeUser = (user) => ({
  id: user.id,
  fullName: `${user.first_name} ${user.last_name}`,
  age: new Date().getFullYear() - user.birth_year,
  isAdult: new Date().getFullYear() - user.birth_year >= 18,
});
const sortByAge = (users) => users.sort((a, b) => a.age - b.age);

const reusableTransform = FP.pipe(
  extractUsers,
  filterActive,
  (users) => users.map(normalizeUser),
  sortByAge
);
```

### 2. 비동기 함수형 프로그래밍

```javascript
// Promise 체이닝을 함수형으로
const asyncPipe =
  (...fns) =>
  (value) =>
    fns.reduce((acc, fn) => acc.then(fn), Promise.resolve(value));

const fetchUser = (id) => fetch(`/api/users/${id}`).then((res) => res.json());

const enrichUser = (user) => ({
  ...user,
  displayName: `${user.firstName} ${user.lastName}`,
  isVip: user.orders > 10,
});

const validateUser = (user) => {
  if (!user.email) {
    throw new Error("User email is required");
  }
  return user;
};

const processUser = asyncPipe(fetchUser, enrichUser, validateUser);

// 사용
processUser(123)
  .then((user) => console.log("Processed user:", user))
  .catch((error) => console.error("Error:", error));

// 병렬 처리
const parallel = (...promises) => Promise.all(promises);

const fetchUserProfile = asyncPipe(
  (userId) =>
    parallel(
      fetchUser(userId),
      fetch(`/api/users/${userId}/orders`).then((res) => res.json()),
      fetch(`/api/users/${userId}/preferences`).then((res) => res.json())
    ),
  ([user, orders, preferences]) => ({
    ...user,
    orders,
    preferences,
    totalOrders: orders.length,
  })
);
```

### 3. 상태 관리 (Redux 스타일)

```javascript
// 함수형 상태 관리
const createReducer =
  (initialState, handlers) =>
  (state = initialState, action) =>
    handlers[action.type] ? handlers[action.type](state, action) : state;

// 불변 업데이트 헬퍼
const updateObject = (oldObject, newValues) =>
  Object.assign({}, oldObject, newValues);

const updateItemInArray = (array, itemId, updateItemCallback) =>
  array.map((item) => (item.id !== itemId ? item : updateItemCallback(item)));

// 사용자 리듀서
const userReducer = createReducer([], {
  ADD_USER: (state, action) => [...state, action.payload],

  UPDATE_USER: (state, action) =>
    updateItemInArray(state, action.payload.id, (user) =>
      updateObject(user, action.payload.updates)
    ),

  DELETE_USER: (state, action) =>
    state.filter((user) => user.id !== action.payload.id),

  SET_USERS: (state, action) => action.payload,
});

// 액션 크리에이터
const actionCreators = {
  addUser: (user) => ({ type: "ADD_USER", payload: user }),
  updateUser: (id, updates) => ({
    type: "UPDATE_USER",
    payload: { id, updates },
  }),
  deleteUser: (id) => ({ type: "DELETE_USER", payload: { id } }),
  setUsers: (users) => ({ type: "SET_USERS", payload: users }),
};
```

## 성능 최적화

### 1. 지연 평가 (Lazy Evaluation)

```javascript
// 지연 평가 구현
class LazySequence {
  constructor(iterable) {
    this.iterable = iterable;
  }

  static of(iterable) {
    return new LazySequence(iterable);
  }

  map(fn) {
    return new LazySequence(this._map(fn));
  }

  *_map(fn) {
    for (const item of this.iterable) {
      yield fn(item);
    }
  }

  filter(predicate) {
    return new LazySequence(this._filter(predicate));
  }

  *_filter(predicate) {
    for (const item of this.iterable) {
      if (predicate(item)) {
        yield item;
      }
    }
  }

  take(n) {
    return new LazySequence(this._take(n));
  }

  *_take(n) {
    let count = 0;
    for (const item of this.iterable) {
      if (count >= n) break;
      yield item;
      count++;
    }
  }

  toArray() {
    return Array.from(this.iterable);
  }

  forEach(fn) {
    for (const item of this.iterable) {
      fn(item);
    }
  }
}

// 지연 평가로 성능 최적화
const largeNumbers = Array.from({ length: 1000000 }, (_, i) => i);

// 즉시 평가 (메모리 많이 사용)
const eagerResult = largeNumbers
  .map((x) => x * 2)
  .filter((x) => x % 4 === 0)
  .slice(0, 10);

// 지연 평가 (메모리 효율적)
const lazyResult = LazySequence.of(largeNumbers)
  .map((x) => x * 2)
  .filter((x) => x % 4 === 0)
  .take(10)
  .toArray();
```

## 결론

함수형 프로그래밍은 JavaScript에서 더 예측 가능하고 테스트하기 쉬운 코드를 작성할 수 있게 해줍니다. 순수 함수, 고차 함수, map/filter/reduce 등의 개념을 활용하면 복잡한 데이터 변환과 상태 관리를 효율적으로 처리할 수 있습니다.
