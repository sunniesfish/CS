# 에러 처리 패턴 (Error Handling Patterns)

## 개요

JavaScript의 에러 처리는 애플리케이션의 안정성과 사용자 경험에 직결되는 중요한 요소입니다. try-catch, error bubbling, 커스텀 에러 클래스 등 다양한 패턴을 통해 예외 상황을 효과적으로 관리할 수 있습니다.

## 탄생 배경

웹 애플리케이션의 복잡성이 증가하면서 런타임 에러, 네트워크 오류, 사용자 입력 오류 등 다양한 예외 상황을 체계적으로 처리할 필요가 생겼습니다. JavaScript는 이를 위해 다층적인 에러 처리 메커니즘을 제공합니다.

## 기본 에러 처리

### 1. try-catch-finally

```javascript
// 기본 try-catch 구문
function basicErrorHandling() {
  try {
    // 위험한 코드
    const result = riskyOperation();
    console.log("Success:", result);
  } catch (error) {
    // 에러 처리
    console.error("Error occurred:", error.message);
  } finally {
    // 항상 실행되는 코드
    console.log("Cleanup completed");
  }
}

function riskyOperation() {
  if (Math.random() > 0.5) {
    throw new Error("Random error occurred");
  }
  return "Operation successful";
}

// 중첩된 try-catch
function nestedErrorHandling() {
  try {
    try {
      throw new Error("Inner error");
    } catch (innerError) {
      console.log("Inner catch:", innerError.message);
      throw new Error("Outer error from inner catch");
    }
  } catch (outerError) {
    console.log("Outer catch:", outerError.message);
  }
}
```

### 2. 에러 타입별 처리

```javascript
function typeSpecificErrorHandling() {
  try {
    // 다양한 에러 발생 가능
    const operation = getRandomOperation();
    operation();
  } catch (error) {
    if (error instanceof TypeError) {
      console.error("Type Error:", error.message);
      // 타입 에러 특별 처리
    } else if (error instanceof ReferenceError) {
      console.error("Reference Error:", error.message);
      // 참조 에러 특별 처리
    } else if (error instanceof SyntaxError) {
      console.error("Syntax Error:", error.message);
      // 문법 에러 특별 처리
    } else {
      console.error("Unknown Error:", error.message);
      // 일반적인 에러 처리
    }
  }
}

function getRandomOperation() {
  const operations = [
    () => {
      throw new TypeError("Invalid type");
    },
    () => {
      throw new ReferenceError("Variable not defined");
    },
    () => {
      throw new SyntaxError("Invalid syntax");
    },
    () => {
      throw new Error("Generic error");
    },
  ];

  return operations[Math.floor(Math.random() * operations.length)];
}
```

## 커스텀 에러 클래스

### 1. 기본 커스텀 에러

```javascript
// 커스텀 에러 클래스 정의
class CustomError extends Error {
  constructor(message, code = "CUSTOM_ERROR") {
    super(message);
    this.name = this.constructor.name;
    this.code = code;
    this.timestamp = new Date().toISOString();

    // 스택 트레이스 정리 (V8 엔진)
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor);
    }
  }

  toJSON() {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      timestamp: this.timestamp,
      stack: this.stack,
    };
  }
}

// 특정 도메인별 에러 클래스
class ValidationError extends CustomError {
  constructor(field, value, message) {
    super(
      message || `Validation failed for field: ${field}`,
      "VALIDATION_ERROR"
    );
    this.field = field;
    this.value = value;
  }
}

class NetworkError extends CustomError {
  constructor(url, status, message) {
    super(message || `Network request failed: ${status}`, "NETWORK_ERROR");
    this.url = url;
    this.status = status;
  }
}

class BusinessLogicError extends CustomError {
  constructor(operation, reason, message) {
    super(message || `Business logic error in ${operation}`, "BUSINESS_ERROR");
    this.operation = operation;
    this.reason = reason;
  }
}
```

### 2. 에러 팩토리 패턴

```javascript
// 에러 팩토리 클래스
class ErrorFactory {
  static createValidationError(field, value, constraints) {
    const message = `Field '${field}' with value '${value}' violates constraints: ${constraints.join(
      ", "
    )}`;
    return new ValidationError(field, value, message);
  }

  static createNetworkError(url, response) {
    const message = `Request to ${url} failed with status ${response.status}: ${response.statusText}`;
    return new NetworkError(url, response.status, message);
  }

  static createBusinessError(operation, details) {
    const message = `Business rule violation in ${operation}: ${details}`;
    return new BusinessLogicError(operation, details, message);
  }

  static createFromHttpStatus(status, url, data) {
    switch (status) {
      case 400:
        return new ValidationError("request", data, "Bad Request");
      case 401:
        return new CustomError("Unauthorized access", "AUTH_ERROR");
      case 403:
        return new CustomError("Forbidden operation", "PERMISSION_ERROR");
      case 404:
        return new CustomError("Resource not found", "NOT_FOUND_ERROR");
      case 500:
        return new CustomError("Internal server error", "SERVER_ERROR");
      default:
        return new NetworkError(url, status, `HTTP ${status} error`);
    }
  }
}

// 사용 예시
function validateUser(userData) {
  if (!userData.email) {
    throw ErrorFactory.createValidationError("email", userData.email, [
      "required",
    ]);
  }

  if (!userData.email.includes("@")) {
    throw ErrorFactory.createValidationError("email", userData.email, [
      "valid email format",
    ]);
  }

  if (userData.age < 18) {
    throw ErrorFactory.createBusinessError(
      "user_registration",
      "User must be 18 or older"
    );
  }
}
```

## 에러 버블링과 전파

### 1. 에러 전파 메커니즘

```javascript
// 에러 버블링 예시
class DataProcessor {
  constructor() {
    this.errorHandlers = new Map();
  }

  // 에러 핸들러 등록
  onError(errorType, handler) {
    if (!this.errorHandlers.has(errorType)) {
      this.errorHandlers.set(errorType, []);
    }
    this.errorHandlers.get(errorType).push(handler);
  }

  // 에러 전파
  propagateError(error) {
    const handlers = this.errorHandlers.get(error.constructor.name) || [];
    const genericHandlers = this.errorHandlers.get("Error") || [];

    // 특정 에러 타입 핸들러 먼저 실행
    [...handlers, ...genericHandlers].forEach((handler) => {
      try {
        handler(error);
      } catch (handlerError) {
        console.error("Error in error handler:", handlerError);
      }
    });
  }

  async processData(data) {
    try {
      const validated = this.validateData(data);
      const processed = await this.transformData(validated);
      return await this.saveData(processed);
    } catch (error) {
      this.propagateError(error);
      throw error; // 에러를 다시 던져서 상위로 전파
    }
  }

  validateData(data) {
    if (!data || typeof data !== "object") {
      throw new ValidationError("data", data, "Data must be an object");
    }
    return data;
  }

  async transformData(data) {
    // 변환 중 에러 발생 가능
    if (data.shouldFail) {
      throw new BusinessLogicError(
        "data_transformation",
        "Transformation not allowed"
      );
    }
    return { ...data, processed: true };
  }

  async saveData(data) {
    // 저장 중 네트워크 에러 발생 가능
    if (Math.random() > 0.7) {
      throw new NetworkError("/api/save", 500, "Database connection failed");
    }
    return { ...data, saved: true };
  }
}

// 사용 예시
const processor = new DataProcessor();

// 에러 핸들러 등록
processor.onError("ValidationError", (error) => {
  console.log(`Validation failed: ${error.field} - ${error.message}`);
});

processor.onError("NetworkError", (error) => {
  console.log(`Network issue: ${error.url} returned ${error.status}`);
});

processor.onError("Error", (error) => {
  console.log(`Generic error handler: ${error.message}`);
});
```

### 2. 계층적 에러 처리

```javascript
// 계층적 에러 처리 시스템
class ErrorBoundary {
  constructor(name) {
    this.name = name;
    this.parent = null;
    this.children = [];
    this.handlers = new Map();
  }

  addChild(child) {
    child.parent = this;
    this.children.push(child);
    return child;
  }

  addHandler(errorType, handler) {
    this.handlers.set(errorType, handler);
  }

  handleError(error, source) {
    console.log(
      `[${this.name}] Handling error from ${source?.name || "unknown"}`
    );

    // 현재 레벨에서 처리 시도
    const handler =
      this.handlers.get(error.constructor.name) || this.handlers.get("Error");

    if (handler) {
      try {
        const handled = handler(error, source);
        if (handled) {
          console.log(`[${this.name}] Error handled successfully`);
          return true;
        }
      } catch (handlerError) {
        console.error(`[${this.name}] Handler failed:`, handlerError);
      }
    }

    // 부모로 에러 전파
    if (this.parent) {
      console.log(
        `[${this.name}] Bubbling error to parent: ${this.parent.name}`
      );
      return this.parent.handleError(error, this);
    }

    // 최상위에서 처리되지 않은 에러
    console.error(`[${this.name}] Unhandled error:`, error);
    return false;
  }

  execute(operation) {
    try {
      return operation();
    } catch (error) {
      this.handleError(error, this);
      throw error; // 필요에 따라 재던지기
    }
  }
}

// 계층적 구조 설정
const appBoundary = new ErrorBoundary("Application");
const serviceBoundary = appBoundary.addChild(new ErrorBoundary("Service"));
const dataBoundary = serviceBoundary.addChild(new ErrorBoundary("Data"));

// 각 레벨별 핸들러 설정
appBoundary.addHandler("Error", (error) => {
  console.log("Application level: Logging error and showing user message");
  return true; // 에러 처리됨
});

serviceBoundary.addHandler("NetworkError", (error) => {
  console.log("Service level: Retrying network request");
  return false; // 에러 처리 실패, 상위로 전파
});

dataBoundary.addHandler("ValidationError", (error) => {
  console.log("Data level: Fixing validation issue");
  return true; // 에러 처리됨
});
```

## 비동기 에러 처리

### 1. Promise 에러 처리

```javascript
// Promise 체인에서의 에러 처리
class AsyncErrorHandler {
  static async handlePromiseChain() {
    try {
      const data = await this.fetchData()
        .then((data) => this.validateData(data))
        .then((data) => this.processData(data))
        .catch((error) => {
          // 체인 중간에서 에러 처리
          if (error instanceof ValidationError) {
            console.log("Validation error in chain:", error.message);
            return { error: true, message: error.message };
          }
          throw error; // 다른 에러는 재던지기
        });

      return data;
    } catch (error) {
      console.error("Final error handler:", error);
      throw error;
    }
  }

  static async handleParallelPromises() {
    try {
      // 병렬 실행 중 일부 실패 허용
      const results = await Promise.allSettled([
        this.fetchUserData(),
        this.fetchOrderData(),
        this.fetchProductData(),
      ]);

      const successful = results.filter((r) => r.status === "fulfilled");
      const failed = results.filter((r) => r.status === "rejected");

      if (failed.length > 0) {
        console.warn(`${failed.length} operations failed:`, failed);
      }

      return successful.map((r) => r.value);
    } catch (error) {
      console.error("Parallel operation error:", error);
      throw error;
    }
  }

  static fetchData() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (Math.random() > 0.5) {
          resolve({ id: 1, name: "Test Data" });
        } else {
          reject(new NetworkError("/api/data", 500));
        }
      }, 1000);
    });
  }

  static validateData(data) {
    if (!data.name) {
      throw new ValidationError("name", data.name, "Name is required");
    }
    return data;
  }

  static processData(data) {
    return { ...data, processed: true };
  }

  static fetchUserData() {
    return Promise.resolve({ user: "John" });
  }

  static fetchOrderData() {
    return Promise.reject(new Error("Order service unavailable"));
  }

  static fetchProductData() {
    return Promise.resolve({ products: ["A", "B"] });
  }
}
```

### 2. async/await 에러 처리

```javascript
// async/await 패턴에서의 에러 처리
class AsyncAwaitErrorHandler {
  async processWithRetry(operation, maxRetries = 3) {
    let lastError;

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        console.log(`Attempt ${attempt}/${maxRetries}`);
        const result = await operation();
        return result;
      } catch (error) {
        lastError = error;

        if (error instanceof NetworkError && attempt < maxRetries) {
          console.log(`Network error, retrying in ${attempt * 1000}ms...`);
          await this.delay(attempt * 1000);
          continue;
        }

        if (error instanceof ValidationError) {
          console.error("Validation error, no retry needed:", error.message);
          break;
        }

        console.log(`Attempt ${attempt} failed:`, error.message);
      }
    }

    throw new Error(
      `Operation failed after ${maxRetries} attempts: ${lastError.message}`
    );
  }

  async processWithFallback(primaryOperation, fallbackOperation) {
    try {
      return await primaryOperation();
    } catch (primaryError) {
      console.warn(
        "Primary operation failed, trying fallback:",
        primaryError.message
      );

      try {
        return await fallbackOperation();
      } catch (fallbackError) {
        throw new Error(
          `Both primary and fallback operations failed. ` +
            `Primary: ${primaryError.message}, Fallback: ${fallbackError.message}`
        );
      }
    }
  }

  async processWithTimeout(operation, timeoutMs = 5000) {
    const timeoutPromise = new Promise((_, reject) => {
      setTimeout(() => {
        reject(new Error(`Operation timed out after ${timeoutMs}ms`));
      }, timeoutMs);
    });

    try {
      return await Promise.race([operation(), timeoutPromise]);
    } catch (error) {
      if (error.message.includes("timed out")) {
        throw new CustomError("Operation timeout", "TIMEOUT_ERROR");
      }
      throw error;
    }
  }

  delay(ms) {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}
```

## 전역 에러 처리

### 1. 전역 에러 핸들러

```javascript
// 전역 에러 처리 시스템
class GlobalErrorHandler {
  constructor() {
    this.errorLog = [];
    this.errorCallbacks = [];
    this.setupGlobalHandlers();
  }

  setupGlobalHandlers() {
    // 동기 에러 처리 (브라우저)
    if (typeof window !== "undefined") {
      window.addEventListener("error", (event) => {
        this.handleError({
          type: "javascript",
          message: event.message,
          filename: event.filename,
          lineno: event.lineno,
          colno: event.colno,
          error: event.error,
          timestamp: new Date().toISOString(),
        });
      });

      // Promise rejection 처리
      window.addEventListener("unhandledrejection", (event) => {
        this.handleError({
          type: "unhandled_promise_rejection",
          reason: event.reason,
          timestamp: new Date().toISOString(),
        });

        // 기본 에러 로깅 방지
        event.preventDefault();
      });
    }

    // Node.js 환경
    if (typeof process !== "undefined") {
      process.on("uncaughtException", (error) => {
        this.handleError({
          type: "uncaught_exception",
          error: error,
          timestamp: new Date().toISOString(),
        });

        // 프로세스 종료 전 정리 작업
        this.cleanup();
        process.exit(1);
      });

      process.on("unhandledRejection", (reason, promise) => {
        this.handleError({
          type: "unhandled_rejection",
          reason: reason,
          promise: promise,
          timestamp: new Date().toISOString(),
        });
      });
    }
  }

  handleError(errorInfo) {
    // 에러 로깅
    this.errorLog.push(errorInfo);

    // 에러 정보 출력
    console.error("Global Error Handler:", errorInfo);

    // 등록된 콜백 실행
    this.errorCallbacks.forEach((callback) => {
      try {
        callback(errorInfo);
      } catch (callbackError) {
        console.error("Error in error callback:", callbackError);
      }
    });

    // 에러 보고 (외부 서비스)
    this.reportError(errorInfo);
  }

  onError(callback) {
    this.errorCallbacks.push(callback);
  }

  async reportError(errorInfo) {
    try {
      // 외부 에러 보고 서비스로 전송
      // await fetch('/api/errors', {
      //     method: 'POST',
      //     headers: { 'Content-Type': 'application/json' },
      //     body: JSON.stringify(errorInfo)
      // });

      console.log("Error reported to external service");
    } catch (reportingError) {
      console.error("Failed to report error:", reportingError);
    }
  }

  getErrorLog() {
    return this.errorLog.slice(); // 복사본 반환
  }

  clearErrorLog() {
    this.errorLog = [];
  }

  cleanup() {
    console.log("Performing cleanup before exit...");
    // 리소스 정리, 연결 종료 등
  }
}

// 전역 인스턴스 생성
const globalErrorHandler = new GlobalErrorHandler();

// 에러 콜백 등록
globalErrorHandler.onError((errorInfo) => {
  if (errorInfo.type === "unhandled_promise_rejection") {
    console.log("Handling unhandled promise rejection");
  }
});
```

### 2. 에러 복구 전략

```javascript
// 에러 복구 및 자동 치유 시스템
class ErrorRecoverySystem {
  constructor() {
    this.recoveryStrategies = new Map();
    this.circuitBreakers = new Map();
  }

  addRecoveryStrategy(errorType, strategy) {
    this.recoveryStrategies.set(errorType, strategy);
  }

  async executeWithRecovery(operation, context = {}) {
    try {
      return await operation();
    } catch (error) {
      console.log("Error occurred, attempting recovery:", error.message);

      const strategy = this.recoveryStrategies.get(error.constructor.name);
      if (strategy) {
        try {
          const recovered = await strategy(error, context);
          if (recovered.success) {
            console.log("Error recovered successfully");
            return recovered.result;
          }
        } catch (recoveryError) {
          console.error("Recovery failed:", recoveryError);
        }
      }

      // 복구 실패 시 원본 에러 재던지기
      throw error;
    }
  }

  createCircuitBreaker(name, threshold = 5, resetTimeout = 60000) {
    const breaker = {
      failureCount: 0,
      lastFailureTime: null,
      state: "CLOSED", // CLOSED, OPEN, HALF_OPEN
    };

    this.circuitBreakers.set(name, breaker);
    return breaker;
  }

  async executeWithCircuitBreaker(name, operation) {
    const breaker = this.circuitBreakers.get(name);
    if (!breaker) {
      throw new Error(`Circuit breaker ${name} not found`);
    }

    // OPEN 상태 확인
    if (breaker.state === "OPEN") {
      const timeSinceLastFailure = Date.now() - breaker.lastFailureTime;
      if (timeSinceLastFailure < 60000) {
        // resetTimeout
        throw new Error(`Circuit breaker ${name} is OPEN`);
      } else {
        breaker.state = "HALF_OPEN";
      }
    }

    try {
      const result = await operation();

      // 성공 시 회로 차단기 리셋
      if (breaker.state === "HALF_OPEN") {
        breaker.state = "CLOSED";
        breaker.failureCount = 0;
      }

      return result;
    } catch (error) {
      breaker.failureCount++;
      breaker.lastFailureTime = Date.now();

      if (breaker.failureCount >= 5) {
        // threshold
        breaker.state = "OPEN";
      }

      throw error;
    }
  }
}

// 복구 전략 설정
const recoverySystem = new ErrorRecoverySystem();

recoverySystem.addRecoveryStrategy("NetworkError", async (error, context) => {
  console.log("Attempting network error recovery...");

  // 캐시된 데이터 사용
  if (context.cache && context.cache.has(context.key)) {
    return {
      success: true,
      result: context.cache.get(context.key),
    };
  }

  // 대체 엔드포인트 시도
  if (context.fallbackUrl) {
    try {
      const response = await fetch(context.fallbackUrl);
      return {
        success: true,
        result: await response.json(),
      };
    } catch (fallbackError) {
      return { success: false };
    }
  }

  return { success: false };
});
```

## 결론

JavaScript의 에러 처리는 단순한 try-catch를 넘어서 체계적인 에러 분류, 전파, 복구 메커니즘을 포함합니다. 커스텀 에러 클래스, 에러 버블링, 비동기 에러 처리, 전역 에러 핸들링 등을 적절히 조합하여 견고하고 사용자 친화적인 애플리케이션을 구축할 수 있습니다.
