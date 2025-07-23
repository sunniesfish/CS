# 모듈 시스템 (ESM vs CommonJS, 동적 import)

## 개요

JavaScript 모듈 시스템은 코드를 재사용 가능한 단위로 분리하고 관리하는 메커니즘입니다. 주요 시스템으로는 CommonJS(Node.js), AMD, UMD, 그리고 ES6 모듈(ESM)이 있으며, 각각 다른 환경과 요구사항에 맞춰 발전했습니다.

## 탄생 배경

초기 JavaScript는 브라우저에서만 실행되는 간단한 스크립트 언어였지만, Node.js의 등장과 대규모 애플리케이션 개발 필요성으로 인해 모듈 시스템이 필수가 되었습니다.

## CommonJS (Node.js 기본 모듈 시스템)

### 1. 기본 문법

```javascript
// math.js - 모듈 내보내기
const PI = 3.14159;

function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

// 단일 내보내기
module.exports = {
  PI,
  add,
  multiply,
};

// 또는 개별 내보내기
exports.PI = PI;
exports.add = add;
exports.multiply = multiply;
```

```javascript
// main.js - 모듈 가져오기
const math = require("./math");
const { add, multiply } = require("./math");

console.log(math.add(2, 3)); // 5
console.log(multiply(4, 5)); // 20

// 동적 require
const moduleName = "./math";
const dynamicMath = require(moduleName);
```

### 2. CommonJS의 특징

```javascript
// 동기적 로딩
console.log("Before require");
const fs = require("fs"); // 동기적으로 로드됨
console.log("After require");

// 모듈 캐싱
const math1 = require("./math");
const math2 = require("./math");
console.log(math1 === math2); // true - 같은 객체 참조

// 조건부 require
if (process.env.NODE_ENV === "development") {
  const devTools = require("./dev-tools");
  devTools.init();
}

// 런타임에 모듈 경로 결정
const moduleMap = {
  dev: "./dev-config",
  prod: "./prod-config",
};
const config = require(moduleMap[process.env.NODE_ENV]);
```

### 3. module.exports vs exports

```javascript
// 올바른 사용법
module.exports = {
  name: "MyModule",
  version: "1.0.0",
};

// exports는 module.exports의 참조
exports.helper = function () {
  return "helper function";
};

// 잘못된 사용법 - exports 재할당
exports = {
  name: "MyModule", // 이것은 작동하지 않음
};

// 올바른 방법
module.exports = {
  name: "MyModule",
};
```

## ES6 모듈 (ESM)

### 1. 기본 문법

```javascript
// math.mjs 또는 math.js (package.json에서 "type": "module")
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

// 기본 내보내기
export default function divide(a, b) {
  return a / b;
}

// 한 번에 내보내기
const subtract = (a, b) => a - b;
export { subtract, PI as MATH_PI };
```

```javascript
// main.mjs - 모듈 가져오기
import divide, { add, multiply, PI } from "./math.mjs";
import * as math from "./math.mjs";

console.log(add(2, 3)); // 5
console.log(divide(10, 2)); // 5
console.log(math.PI); // 3.14159

// 별칭 사용
import { PI as MATH_PI } from "./math.mjs";

// 재내보내기
export { add, multiply } from "./math.mjs";
export { default as divide } from "./math.mjs";
```

### 2. ESM의 특징

```javascript
// 정적 분석 가능
import { add } from "./math.mjs"; // 컴파일 타임에 결정

// 호이스팅
console.log(add(1, 2)); // 3 - import가 호이스팅됨
import { add } from "./math.mjs";

// Live binding
// counter.mjs
export let count = 0;
export function increment() {
  count++;
}

// main.mjs
import { count, increment } from "./counter.mjs";
console.log(count); // 0
increment();
console.log(count); // 1 - 실시간 업데이트
```

## 동적 import

### 1. 기본 사용법

```javascript
// 동적 import는 Promise를 반환
async function loadMath() {
  try {
    const math = await import("./math.mjs");
    console.log(math.add(2, 3)); // 5
  } catch (error) {
    console.error("Failed to load math module:", error);
  }
}

// 조건부 로딩
async function conditionalLoad() {
  if (window.innerWidth > 768) {
    const { default: DesktopComponent } = await import(
      "./DesktopComponent.mjs"
    );
    new DesktopComponent();
  } else {
    const { default: MobileComponent } = await import("./MobileComponent.mjs");
    new MobileComponent();
  }
}

// 런타임 모듈 경로
async function dynamicPath(moduleName) {
  const module = await import(`./modules/${moduleName}.mjs`);
  return module.default;
}
```

### 2. 고급 동적 import 패턴

```javascript
// 모듈 캐시 관리
class ModuleLoader {
  constructor() {
    this.cache = new Map();
  }

  async load(modulePath) {
    if (this.cache.has(modulePath)) {
      return this.cache.get(modulePath);
    }

    try {
      const module = await import(modulePath);
      this.cache.set(modulePath, module);
      return module;
    } catch (error) {
      console.error(`Failed to load ${modulePath}:`, error);
      throw error;
    }
  }

  clearCache(modulePath) {
    this.cache.delete(modulePath);
  }
}

// 병렬 모듈 로딩
async function loadMultipleModules() {
  const [math, utils, helpers] = await Promise.all([
    import("./math.mjs"),
    import("./utils.mjs"),
    import("./helpers.mjs"),
  ]);

  return { math, utils, helpers };
}

// 폴백 로딩
async function loadWithFallback() {
  try {
    return await import("./modern-module.mjs");
  } catch (error) {
    console.warn("Modern module failed, loading legacy:", error);
    return await import("./legacy-module.mjs");
  }
}
```

## CommonJS vs ESM 비교

### 1. 문법 차이

```javascript
// CommonJS
const fs = require("fs");
const { readFile } = require("fs");
module.exports = { myFunction };

// ESM
import fs from "fs";
import { readFile } from "fs";
export { myFunction };
```

### 2. 로딩 방식

```javascript
// CommonJS - 동기적, 런타임
const math = require("./math"); // 즉시 실행

// ESM - 비동기적, 컴파일 타임
import math from "./math.mjs"; // 모듈 그래프 미리 구성
```

### 3. 순환 의존성 처리

```javascript
// CommonJS 순환 의존성
// a.js
console.log("a starting");
exports.done = false;
const b = require("./b.js");
console.log("in a, b.done =", b.done);
exports.done = true;
console.log("a done");

// b.js
console.log("b starting");
exports.done = false;
const a = require("./a.js");
console.log("in b, a.done =", a.done);
exports.done = true;
console.log("b done");

// ESM 순환 의존성 (더 예측 가능)
// a.mjs
import { b } from "./b.mjs";
export const a = "a";
console.log(b); // undefined (아직 초기화 안됨)

// b.mjs
import { a } from "./a.mjs";
export const b = "b";
console.log(a); // undefined
```

## 혼합 사용과 상호 운용성

### 1. ESM에서 CommonJS 사용

```javascript
// ESM에서 CommonJS 모듈 가져오기
import fs from "fs"; // 기본 가져오기만 가능
import { createRequire } from "module";

const require = createRequire(import.meta.url);
const lodash = require("lodash"); // CommonJS 모듈

// 동적 import로 CommonJS 사용
const lodash2 = await import("lodash");
```

### 2. CommonJS에서 ESM 사용

```javascript
// CommonJS에서 ESM 사용 (동적 import만 가능)
async function useESM() {
  const { add } = await import("./math.mjs");
  return add(2, 3);
}

// 최상위 await 사용 불가, 함수 내에서만
(async () => {
  const math = await import("./math.mjs");
  console.log(math.add(1, 2));
})();
```

## 모듈 번들러와의 관계

### 1. Webpack

```javascript
// webpack.config.js
module.exports = {
  entry: "./src/index.js",
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src"),
      utils: path.resolve(__dirname, "src/utils"),
    },
  },
  module: {
    rules: [
      {
        test: /\.m?js$/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"],
          },
        },
      },
    ],
  },
};

// 코드 분할
const LazyComponent = React.lazy(() => import("./LazyComponent"));

// 동적 import로 청크 분할
async function loadFeature() {
  const { feature } = await import(
    /* webpackChunkName: "feature" */ "./feature"
  );
  return feature;
}
```

### 2. Tree Shaking

```javascript
// utils.mjs - 여러 유틸리티 함수
export function add(a, b) {
  return a + b;
}
export function subtract(a, b) {
  return a - b;
}
export function multiply(a, b) {
  return a * b;
}
export function divide(a, b) {
  return a / b;
}

// main.mjs - 일부만 사용
import { add, multiply } from "./utils.mjs";

// 번들러가 subtract, divide는 제거 (tree shaking)
console.log(add(2, 3));
console.log(multiply(4, 5));
```

## 실무 패턴과 베스트 프랙티스

### 1. 모듈 구조 설계

```javascript
// 계층적 모듈 구조
// src/
//   modules/
//     auth/
//       index.mjs
//       login.mjs
//       logout.mjs
//     api/
//       index.mjs
//       client.mjs
//       endpoints.mjs

// auth/index.mjs - 배럴 패턴
export { login, logout } from "./auth.mjs";
export { validateToken } from "./validation.mjs";
export { default as AuthProvider } from "./AuthProvider.mjs";

// main.mjs
import { login, logout, AuthProvider } from "./modules/auth/index.mjs";
```

### 2. 환경별 모듈 로딩

```javascript
// config/index.mjs
const environment = process.env.NODE_ENV || 'development';

const config = await import(`./config.${environment}.mjs`);

export default config.default;

// 또는 정적 분석을 위해
let config;
switch (environment) {
    case 'production':
        config = await import('./config.production.mjs');
        break;
    case 'staging':
        config = await import('./config.staging.mjs');
        break;
    default:
        config = await import('./config.development.mjs');
}

export default config.default;
```

### 3. 플러그인 시스템

```javascript
// 플러그인 로더
class PluginLoader {
  constructor() {
    this.plugins = new Map();
  }

  async loadPlugin(name, path) {
    try {
      const plugin = await import(path);

      if (typeof plugin.default === "function") {
        this.plugins.set(name, new plugin.default());
      } else {
        this.plugins.set(name, plugin.default);
      }

      console.log(`Plugin ${name} loaded successfully`);
    } catch (error) {
      console.error(`Failed to load plugin ${name}:`, error);
    }
  }

  async loadPluginsFromConfig(config) {
    const loadPromises = config.plugins.map(({ name, path }) =>
      this.loadPlugin(name, path)
    );

    await Promise.allSettled(loadPromises);
  }

  getPlugin(name) {
    return this.plugins.get(name);
  }
}
```

## 성능 최적화

### 1. 모듈 지연 로딩

```javascript
// 라우트 기반 코드 분할
const routes = {
  "/home": () => import("./pages/Home.mjs"),
  "/about": () => import("./pages/About.mjs"),
  "/contact": () => import("./pages/Contact.mjs"),
};

async function loadPage(path) {
  const loader = routes[path];
  if (loader) {
    const module = await loader();
    return module.default;
  }
  throw new Error(`Route ${path} not found`);
}

// 기능별 지연 로딩
class FeatureManager {
  constructor() {
    this.features = new Map();
  }

  async loadFeature(name) {
    if (this.features.has(name)) {
      return this.features.get(name);
    }

    const feature = await import(`./features/${name}.mjs`);
    this.features.set(name, feature.default);
    return feature.default;
  }
}
```

### 2. 모듈 프리로딩

```javascript
// 중요한 모듈 프리로드
const criticalModules = ["./auth.mjs", "./api-client.mjs", "./utils.mjs"];

// 백그라운드에서 프리로드
criticalModules.forEach((module) => {
  import(module).catch(console.error);
});

// 사용자 상호작용 예측 기반 프리로드
document.addEventListener("mouseover", (event) => {
  if (event.target.matches("[data-preload]")) {
    const modulePath = event.target.dataset.preload;
    import(modulePath).catch(console.error);
  }
});
```

## 디버깅과 개발 도구

### 1. 모듈 의존성 추적

```javascript
// 개발 모드에서만 실행되는 의존성 추적
if (process.env.NODE_ENV === 'development') {
    const originalImport = globalThis.__import__ || import;

    globalThis.__import__ = function(specifier) {
        console.log(`Loading module: ${specifier}`);
        return originalImport(specifier);
    };
}

// 모듈 로드 시간 측정
async function timedImport(modulePath) {
    const start = performance.now();
    const module = await import(modulePath);
    const end = performance.now();

    console.log(`Module ${modulePath} loaded in ${end - start}ms`);
    return module;
}
```

### 2. 모듈 상태 검증

```javascript
// 모듈 무결성 검증
class ModuleValidator {
  static validate(module, expectedInterface) {
    const missing = [];

    for (const key of expectedInterface) {
      if (!(key in module)) {
        missing.push(key);
      }
    }

    if (missing.length > 0) {
      throw new Error(`Module missing required exports: ${missing.join(", ")}`);
    }

    return true;
  }
}

// 사용 예시
const mathModule = await import("./math.mjs");
ModuleValidator.validate(mathModule, ["add", "subtract", "multiply", "divide"]);
```

## 미래 전망과 새로운 기능

### 1. Import Maps

```html
<!-- HTML에서 모듈 경로 매핑 -->
<script type="importmap">
  {
    "imports": {
      "lodash": "/node_modules/lodash/lodash.js",
      "react": "/node_modules/react/index.js"
    }
  }
</script>

<script type="module">
  import _ from "lodash";
  import React from "react";
</script>
```

### 2. Top-level await

```javascript
// 모듈 최상위에서 await 사용 가능
const config = await fetch("/config.json").then((r) => r.json());
const database = await import("./database.mjs");

await database.connect(config.dbUrl);

export const api = database.createAPI();
```

## 결론

JavaScript 모듈 시스템은 CommonJS에서 시작하여 ESM으로 발전하면서 더욱 강력하고 유연해졌습니다. 동적 import는 코드 분할과 지연 로딩을 가능하게 하여 성능 최적화에 큰 도움을 줍니다. 각 시스템의 특성을 이해하고 적절한 상황에서 올바른 모듈 시스템을 선택하는 것이 중요합니다.
