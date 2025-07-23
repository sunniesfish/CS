# 비동기 처리 모델 (콜백, Promise, async/await, 병렬/직렬 처리)

## 개요

JavaScript는 싱글 스레드 언어이지만 비동기 처리를 통해 논블로킹 방식으로 작업을 수행할 수 있습니다. 콜백, Promise, async/await 등 다양한 패턴이 발전해왔으며, 각각의 장단점과 적절한 사용 시나리오가 있습니다.

## 콜백 패턴 (Callback Pattern)

### 1. 기본 콜백

```javascript
// 전통적인 콜백 패턴
function fetchData(callback) {
  setTimeout(() => {
    const data = { id: 1, name: "John" };
    callback(null, data); // error-first callback
  }, 1000);
}

fetchData((error, data) => {
  if (error) {
    console.error("Error:", error);
  } else {
    console.log("Data:", data);
  }
});

// Node.js 스타일 콜백
const fs = require("fs");
fs.readFile("file.txt", "utf8", (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

### 2. 콜백 헬 (Callback Hell)

```javascript
// 콜백 헬 예시
getUser(userId, (err, user) => {
  if (err) throw err;

  getProfile(user.id, (err, profile) => {
    if (err) throw err;

    getPreferences(profile.id, (err, preferences) => {
      if (err) throw err;

      updateUI(user, profile, preferences, (err) => {
        if (err) throw err;
        console.log("UI updated successfully");
      });
    });
  });
});

// 해결책: 함수 분리
function handleUser(err, user) {
  if (err) throw err;
  getProfile(user.id, handleProfile);
}

function handleProfile(err, profile) {
  if (err) throw err;
  getPreferences(profile.id, handlePreferences);
}

function handlePreferences(err, preferences) {
  if (err) throw err;
  updateUI(user, profile, preferences, handleUIUpdate);
}
```

## Promise 패턴

### 1. Promise 기본

```javascript
// Promise 생성
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = Math.random() > 0.5;
      if (success) {
        resolve({ id: 1, name: "John" });
      } else {
        reject(new Error("Failed to fetch data"));
      }
    }, 1000);
  });
}

// Promise 사용
fetchData()
  .then((data) => {
    console.log("Success:", data);
    return processData(data);
  })
  .then((processedData) => {
    console.log("Processed:", processedData);
  })
  .catch((error) => {
    console.error("Error:", error);
  })
  .finally(() => {
    console.log("Operation completed");
  });
```

### 2. Promise 체이닝

```javascript
// Promise 체이닝으로 콜백 헬 해결
function getUserData(userId) {
  return getUser(userId)
    .then((user) => {
      return getProfile(user.id).then((profile) => ({ user, profile }));
    })
    .then(({ user, profile }) => {
      return getPreferences(profile.id).then((preferences) => ({
        user,
        profile,
        preferences,
      }));
    })
    .then(({ user, profile, preferences }) => {
      return updateUI(user, profile, preferences);
    });
}

// 더 나은 방법: 플래트한 체이닝
function getUserDataFlat(userId) {
  let userData = {};

  return getUser(userId)
    .then((user) => {
      userData.user = user;
      return getProfile(user.id);
    })
    .then((profile) => {
      userData.profile = profile;
      return getPreferences(profile.id);
    })
    .then((preferences) => {
      userData.preferences = preferences;
      return updateUI(userData.user, userData.profile, userData.preferences);
    });
}
```

### 3. Promise 정적 메서드

```javascript
// Promise.all - 모든 Promise가 완료될 때까지 대기
const promises = [
  fetch("/api/users"),
  fetch("/api/posts"),
  fetch("/api/comments"),
];

Promise.all(promises)
  .then((responses) => Promise.all(responses.map((r) => r.json())))
  .then(([users, posts, comments]) => {
    console.log({ users, posts, comments });
  })
  .catch((error) => {
    console.error("하나라도 실패하면 전체 실패:", error);
  });

// Promise.allSettled - 모든 Promise 완료 대기 (실패 무관)
Promise.allSettled(promises).then((results) => {
  results.forEach((result, index) => {
    if (result.status === "fulfilled") {
      console.log(`Promise ${index} 성공:`, result.value);
    } else {
      console.log(`Promise ${index} 실패:`, result.reason);
    }
  });
});

// Promise.race - 가장 먼저 완료되는 Promise
const timeout = new Promise((_, reject) =>
  setTimeout(() => reject(new Error("Timeout")), 5000)
);

Promise.race([fetchData(), timeout])
  .then((data) => console.log("빠른 응답:", data))
  .catch((error) => console.error("타임아웃 또는 에러:", error));

// Promise.any - 가장 먼저 성공하는 Promise
Promise.any([
  fetch("/api/server1"),
  fetch("/api/server2"),
  fetch("/api/server3"),
])
  .then((response) => console.log("첫 번째 성공:", response))
  .catch((error) => console.error("모든 요청 실패:", error));
```

## async/await 패턴

### 1. 기본 사용법

```javascript
// async/await으로 Promise 간소화
async function getUserData(userId) {
  try {
    const user = await getUser(userId);
    const profile = await getProfile(user.id);
    const preferences = await getPreferences(profile.id);

    await updateUI(user, profile, preferences);
    console.log("UI updated successfully");

    return { user, profile, preferences };
  } catch (error) {
    console.error("Error in getUserData:", error);
    throw error; // 에러 재전파
  }
}

// async 함수는 항상 Promise 반환
getUserData(123)
  .then((data) => console.log("Final data:", data))
  .catch((error) => console.error("Final error:", error));
```

### 2. 병렬 vs 직렬 처리

```javascript
// 직렬 처리 (순차 실행)
async function serialProcessing() {
  console.time("Serial");

  const result1 = await fetchData(1); // 1초 대기
  const result2 = await fetchData(2); // 추가 1초 대기
  const result3 = await fetchData(3); // 추가 1초 대기

  console.timeEnd("Serial"); // 약 3초
  return [result1, result2, result3];
}

// 병렬 처리 (동시 실행)
async function parallelProcessing() {
  console.time("Parallel");

  const [result1, result2, result3] = await Promise.all([
    fetchData(1), // 동시 시작
    fetchData(2), // 동시 시작
    fetchData(3), // 동시 시작
  ]);

  console.timeEnd("Parallel"); // 약 1초
  return [result1, result2, result3];
}

// 혼합 처리 (일부는 병렬, 일부는 직렬)
async function mixedProcessing(userId) {
  // 1단계: 사용자 정보 가져오기
  const user = await getUser(userId);

  // 2단계: 프로필과 설정을 병렬로 가져오기
  const [profile, settings] = await Promise.all([
    getProfile(user.id),
    getSettings(user.id),
  ]);

  // 3단계: 위 정보를 바탕으로 추가 데이터 가져오기
  const additionalData = await getAdditionalData(profile.type, settings.theme);

  return { user, profile, settings, additionalData };
}
```

### 3. 에러 처리 패턴

```javascript
// 개별 에러 처리
async function individualErrorHandling() {
  let user, profile, preferences;

  try {
    user = await getUser(userId);
  } catch (error) {
    console.error("Failed to get user:", error);
    user = getDefaultUser();
  }

  try {
    profile = await getProfile(user.id);
  } catch (error) {
    console.error("Failed to get profile:", error);
    profile = getDefaultProfile();
  }

  try {
    preferences = await getPreferences(profile.id);
  } catch (error) {
    console.error("Failed to get preferences:", error);
    preferences = getDefaultPreferences();
  }

  return { user, profile, preferences };
}

// Promise.allSettled를 활용한 에러 처리
async function robustParallelProcessing(userIds) {
  const results = await Promise.allSettled(userIds.map((id) => getUser(id)));

  const successful = [];
  const failed = [];

  results.forEach((result, index) => {
    if (result.status === "fulfilled") {
      successful.push(result.value);
    } else {
      failed.push({
        userId: userIds[index],
        error: result.reason,
      });
    }
  });

  return { successful, failed };
}
```

## 고급 비동기 패턴

### 1. 재시도 로직

```javascript
async function retryAsync(fn, maxRetries = 3, delay = 1000) {
  let lastError;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      console.log(`Attempt ${i + 1} failed:`, error.message);

      if (i < maxRetries - 1) {
        await new Promise((resolve) => setTimeout(resolve, delay));
        delay *= 2; // 지수 백오프
      }
    }
  }

  throw lastError;
}

// 사용 예시
const unstableAPI = () => {
  if (Math.random() < 0.7) {
    throw new Error("API temporarily unavailable");
  }
  return Promise.resolve("Success!");
};

retryAsync(unstableAPI, 3, 500)
  .then((result) => console.log("Final result:", result))
  .catch((error) => console.error("All retries failed:", error));
```

### 2. 배치 처리

```javascript
async function batchProcess(items, batchSize = 5, concurrency = 2) {
  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    console.log(`Processing batch ${Math.floor(i / batchSize) + 1}`);

    // 배치 내에서 제한된 동시성으로 처리
    const batchPromises = batch.map(async (item) => {
      return await processItem(item);
    });

    // 동시성 제한
    const batchResults = await limitConcurrency(batchPromises, concurrency);
    results.push(...batchResults);

    // 배치 간 잠시 대기
    if (i + batchSize < items.length) {
      await new Promise((resolve) => setTimeout(resolve, 100));
    }
  }

  return results;
}

async function limitConcurrency(promises, limit) {
  const results = [];
  const executing = [];

  for (const promise of promises) {
    const p = promise.then((result) => {
      executing.splice(executing.indexOf(p), 1);
      return result;
    });

    results.push(p);
    executing.push(p);

    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }

  return Promise.all(results);
}
```

### 3. 스트림 처리

```javascript
// 비동기 이터레이터
class AsyncDataStream {
  constructor(data) {
    this.data = data;
    this.index = 0;
  }

  async *[Symbol.asyncIterator]() {
    while (this.index < this.data.length) {
      // 비동기 처리 시뮬레이션
      await new Promise((resolve) => setTimeout(resolve, 100));
      yield this.data[this.index++];
    }
  }
}

// 사용법
async function processStream() {
  const stream = new AsyncDataStream([1, 2, 3, 4, 5]);

  for await (const item of stream) {
    console.log("Processing:", item);
    // 각 아이템 처리
  }
}

// 비동기 제너레이터
async function* fetchPages(baseUrl, maxPages = 10) {
  for (let page = 1; page <= maxPages; page++) {
    try {
      const response = await fetch(`${baseUrl}?page=${page}`);
      const data = await response.json();

      if (data.items.length === 0) break;

      yield data.items;
    } catch (error) {
      console.error(`Failed to fetch page ${page}:`, error);
      break;
    }
  }
}

// 사용
async function loadAllData() {
  for await (const pageItems of fetchPages("/api/items")) {
    console.log(`Loaded ${pageItems.length} items`);
    // 각 페이지 처리
  }
}
```

## 성능 최적화

### 1. 메모이제이션과 캐싱

```javascript
class AsyncCache {
  constructor(ttl = 60000) {
    // 1분 TTL
    this.cache = new Map();
    this.ttl = ttl;
  }

  async get(key, fetchFn) {
    const cached = this.cache.get(key);

    if (cached && Date.now() - cached.timestamp < this.ttl) {
      return cached.data;
    }

    // 동시 요청 방지
    if (cached && cached.promise) {
      return cached.promise;
    }

    const promise = fetchFn()
      .then((data) => {
        this.cache.set(key, {
          data,
          timestamp: Date.now(),
          promise: null,
        });
        return data;
      })
      .catch((error) => {
        this.cache.delete(key);
        throw error;
      });

    this.cache.set(key, {
      data: null,
      timestamp: Date.now(),
      promise,
    });

    return promise;
  }

  clear() {
    this.cache.clear();
  }
}

// 사용
const cache = new AsyncCache(30000);

async function getCachedUserData(userId) {
  return cache.get(`user:${userId}`, () => fetchUserFromAPI(userId));
}
```

### 2. 요청 중복 제거

```javascript
class RequestDeduplicator {
  constructor() {
    this.pending = new Map();
  }

  async request(key, requestFn) {
    if (this.pending.has(key)) {
      return this.pending.get(key);
    }

    const promise = requestFn().finally(() => {
      this.pending.delete(key);
    });

    this.pending.set(key, promise);
    return promise;
  }
}

const deduplicator = new RequestDeduplicator();

// 동일한 사용자 데이터 요청이 중복되지 않음
function getUser(userId) {
  return deduplicator.request(`user:${userId}`, () =>
    fetch(`/api/users/${userId}`).then((r) => r.json())
  );
}
```

## 에러 처리와 디버깅

### 1. 상세한 에러 정보

```javascript
class AsyncError extends Error {
  constructor(message, operation, originalError = null) {
    super(message);
    this.name = "AsyncError";
    this.operation = operation;
    this.originalError = originalError;
    this.timestamp = new Date().toISOString();
  }
}

async function safeAsyncOperation(operation, data) {
  try {
    return await operation(data);
  } catch (error) {
    throw new AsyncError(
      `Failed to execute ${operation.name}`,
      operation.name,
      error
    );
  }
}

// 사용
try {
  await safeAsyncOperation(complexAsyncTask, userData);
} catch (error) {
  if (error instanceof AsyncError) {
    console.error(`Operation: ${error.operation}`);
    console.error(`Time: ${error.timestamp}`);
    console.error(`Original error:`, error.originalError);
  }
}
```

### 2. 비동기 스택 트레이스

```javascript
// async_hooks를 사용한 고급 디버깅 (Node.js)
const async_hooks = require("async_hooks");

const asyncResourceMap = new Map();

const hook = async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    asyncResourceMap.set(asyncId, {
      type,
      triggerAsyncId,
      stack: new Error().stack,
    });
  },
  destroy(asyncId) {
    asyncResourceMap.delete(asyncId);
  },
});

hook.enable();

// 비동기 작업 추적
async function trackedAsyncFunction() {
  const asyncId = async_hooks.executionAsyncId();
  console.log(`Current async ID: ${asyncId}`);

  const resource = asyncResourceMap.get(asyncId);
  if (resource) {
    console.log(`Async resource type: ${resource.type}`);
  }

  return new Promise((resolve) => {
    setTimeout(() => {
      console.log("Async operation completed");
      resolve("done");
    }, 1000);
  });
}
```

## 결론

JavaScript의 비동기 처리는 콜백에서 시작하여 Promise, async/await으로 발전하면서 더욱 직관적이고 강력해졌습니다. 각 패턴의 특성을 이해하고, 병렬/직렬 처리를 적절히 조합하며, 에러 처리와 성능 최적화를 고려하는 것이 효과적인 비동기 프로그래밍의 핵심입니다.
