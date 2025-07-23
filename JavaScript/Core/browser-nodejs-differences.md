# 브라우저 환경과 Node.js의 차이점

## 개요

JavaScript는 원래 브라우저에서 실행되도록 설계되었지만, Node.js의 등장으로 서버 사이드에서도 실행할 수 있게 되었습니다. 두 환경은 같은 JavaScript 언어를 사용하지만, 런타임 환경, API, 모듈 시스템 등에서 중요한 차이점들을 가지고 있습니다.

## 탄생 배경

브라우저 JavaScript는 웹 페이지의 상호작용을 위해 개발되었고, Node.js는 Ryan Dahl이 2009년 V8 엔진을 기반으로 서버 사이드 JavaScript 실행 환경을 만들면서 시작되었습니다. 이는 JavaScript를 풀스택 개발 언어로 만들어 개발자 생산성을 크게 향상시켰습니다.

## 런타임 환경의 차이

### 1. JavaScript 엔진과 런타임

```javascript
// 브라우저 환경 확인
if (typeof window !== "undefined") {
  console.log("브라우저 환경");
  console.log("엔진:", navigator.userAgent);
  console.log("전역 객체:", window);
}

// Node.js 환경 확인
if (typeof process !== "undefined") {
  console.log("Node.js 환경");
  console.log("버전:", process.version);
  console.log("플랫폼:", process.platform);
  console.log("전역 객체:", global);
}

// 범용 환경 감지
function getEnvironment() {
  // 브라우저
  if (typeof window !== "undefined" && typeof document !== "undefined") {
    return "browser";
  }

  // Web Worker
  if (typeof self !== "undefined" && typeof importScripts === "function") {
    return "webworker";
  }

  // Node.js
  if (
    typeof process !== "undefined" &&
    process.versions &&
    process.versions.node
  ) {
    return "node";
  }

  // Deno
  if (typeof Deno !== "undefined") {
    return "deno";
  }

  return "unknown";
}

console.log("현재 환경:", getEnvironment());

// 환경별 조건부 코드 실행
const environment = getEnvironment();

switch (environment) {
  case "browser":
    // 브라우저 전용 코드
    document.addEventListener("DOMContentLoaded", () => {
      console.log("DOM 로드 완료");
    });
    break;

  case "node":
    // Node.js 전용 코드
    const fs = require("fs");
    const path = require("path");
    console.log("현재 디렉토리:", process.cwd());
    break;

  case "webworker":
    // Web Worker 전용 코드
    self.addEventListener("message", (event) => {
      console.log("워커에서 메시지 수신:", event.data);
    });
    break;
}
```

### 2. 전역 객체의 차이

```javascript
// 브라우저의 전역 객체 (window)
if (typeof window !== "undefined") {
  // DOM 관련
  console.log("DOM 요소:", document.getElementById);
  console.log("이벤트 리스너:", addEventListener);

  // 브라우저 API
  console.log("로컬 스토리지:", localStorage);
  console.log("세션 스토리지:", sessionStorage);
  console.log("히스토리:", history);
  console.log("위치:", location);
  console.log("네비게이터:", navigator);

  // 타이머
  console.log("setTimeout:", setTimeout);
  console.log("setInterval:", setInterval);
  console.log("requestAnimationFrame:", requestAnimationFrame);

  // 네트워킹
  console.log("Fetch API:", fetch);
  console.log("XMLHttpRequest:", XMLHttpRequest);
  console.log("WebSocket:", WebSocket);
}

// Node.js의 전역 객체 (global)
if (typeof global !== "undefined") {
  // 프로세스 관련
  console.log("프로세스:", process);
  console.log("버퍼:", Buffer);
  console.log(
    "__dirname:",
    typeof __dirname !== "undefined" ? __dirname : "undefined"
  );
  console.log(
    "__filename:",
    typeof __filename !== "undefined" ? __filename : "undefined"
  );

  // 모듈 시스템
  console.log(
    "require:",
    typeof require !== "undefined" ? require : "undefined"
  );
  console.log("module:", typeof module !== "undefined" ? module : "undefined");
  console.log(
    "exports:",
    typeof exports !== "undefined" ? exports : "undefined"
  );

  // 타이머 (브라우저와 유사하지만 구현이 다름)
  console.log("setTimeout:", setTimeout);
  console.log("setInterval:", setInterval);
  console.log("setImmediate:", setImmediate); // Node.js 전용
  console.log("process.nextTick:", process.nextTick); // Node.js 전용
}

// 범용 전역 객체 접근
function getGlobalObject() {
  // 최신 표준
  if (typeof globalThis !== "undefined") {
    return globalThis;
  }

  // 브라우저
  if (typeof window !== "undefined") {
    return window;
  }

  // Node.js
  if (typeof global !== "undefined") {
    return global;
  }

  // Web Worker
  if (typeof self !== "undefined") {
    return self;
  }

  throw new Error("글로벌 객체를 찾을 수 없습니다");
}

const globalObj = getGlobalObject();
console.log("범용 전역 객체:", globalObj.constructor.name);
```

## 모듈 시스템의 차이

### 1. CommonJS vs ES Modules

```javascript
// CommonJS (Node.js 기본)
// math.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

// 내보내기 방법들
module.exports = { add, subtract }; // 객체로 내보내기
// 또는
exports.add = add;
exports.subtract = subtract;

// 단일 함수 내보내기
module.exports = add;

// 사용하기
const math = require("./math");
console.log(math.add(2, 3)); // 5

const { add, subtract } = require("./math");
console.log(add(5, 3)); // 8

// ES Modules (브라우저, 최신 Node.js)
// math.mjs 또는 package.json에서 "type": "module" 설정
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

// 기본 내보내기
export default function multiply(a, b) {
  return a * b;
}

// 사용하기
import { add, subtract } from "./math.mjs";
import multiply from "./math.mjs";

console.log(add(2, 3)); // 5
console.log(multiply(4, 5)); // 20

// 동적 임포트 (브라우저와 Node.js 모두 지원)
async function loadMath() {
  const mathModule = await import("./math.mjs");
  console.log(mathModule.add(1, 2)); // 3
}

// 조건부 모듈 로딩
async function conditionalImport() {
  if (typeof window !== "undefined") {
    // 브라우저에서만 로드
    const { BrowserSpecificModule } = await import("./browser-module.js");
    return new BrowserSpecificModule();
  } else {
    // Node.js에서만 로드
    const { NodeSpecificModule } = await import("./node-module.js");
    return new NodeSpecificModule();
  }
}
```

### 2. 모듈 해결 방식의 차이

```javascript
// Node.js 모듈 해결
if (typeof require !== "undefined") {
  // 내장 모듈
  const fs = require("fs");
  const path = require("path");
  const http = require("http");

  // npm 패키지
  const express = require("express");
  const lodash = require("lodash");

  // 상대 경로
  const myModule = require("./my-module");
  const utils = require("../utils/helpers");

  // 절대 경로 (node_modules에서 찾기)
  const config = require("config");

  console.log("Node.js 모듈 경로들:", module.paths);
}

// 브라우저 모듈 해결 (ES Modules)
// HTML에서 script type="module" 사용
/*
<script type="module">
    // 상대 경로 (확장자 필수)
    import { utils } from './utils.js';
    import { api } from '../api/client.js';
    
    // 절대 URL
    import { library } from 'https://cdn.skypack.dev/library';
    
    // Import Maps 사용 (최신 브라우저)
    import { react } from 'react'; // import map에서 정의된 경로
</script>

<!-- Import Maps 정의 -->
<script type="importmap">
{
  "imports": {
    "react": "https://esm.sh/react@18",
    "react-dom": "https://esm.sh/react-dom@18"
  }
}
</script>
*/

// 범용 모듈 로더
class UniversalModuleLoader {
  static async loadModule(moduleName, options = {}) {
    const environment = getEnvironment();

    switch (environment) {
      case "node":
        if (options.dynamic) {
          return await import(moduleName);
        } else {
          return require(moduleName);
        }

      case "browser":
        return await import(moduleName);

      default:
        throw new Error(`지원하지 않는 환경: ${environment}`);
    }
  }

  static async loadConditionalModule(nodeModule, browserModule) {
    const environment = getEnvironment();

    try {
      if (environment === "node") {
        return await this.loadModule(nodeModule);
      } else {
        return await this.loadModule(browserModule);
      }
    } catch (error) {
      console.error(`모듈 로드 실패 (${environment}):`, error);
      throw error;
    }
  }
}

// 사용 예시
async function example() {
  try {
    // 환경별 조건부 모듈 로드
    const cryptoModule = await UniversalModuleLoader.loadConditionalModule(
      "crypto", // Node.js
      "https://cdn.skypack.dev/crypto-js" // 브라우저
    );

    console.log("암호화 모듈 로드 완료");
  } catch (error) {
    console.error("모듈 로드 실패:", error);
  }
}
```

## API의 차이점

### 1. 파일 시스템 접근

```javascript
// Node.js - 파일 시스템 직접 접근 가능
if (typeof require !== "undefined") {
  const fs = require("fs").promises;
  const path = require("path");

  class NodeFileManager {
    static async readFile(filePath) {
      try {
        const content = await fs.readFile(filePath, "utf-8");
        return content;
      } catch (error) {
        console.error("파일 읽기 실패:", error);
        throw error;
      }
    }

    static async writeFile(filePath, content) {
      try {
        await fs.writeFile(filePath, content, "utf-8");
        console.log("파일 저장 완료:", filePath);
      } catch (error) {
        console.error("파일 쓰기 실패:", error);
        throw error;
      }
    }

    static async listDirectory(dirPath) {
      try {
        const files = await fs.readdir(dirPath);
        return files;
      } catch (error) {
        console.error("디렉토리 읽기 실패:", error);
        throw error;
      }
    }

    static async createDirectory(dirPath) {
      try {
        await fs.mkdir(dirPath, { recursive: true });
        console.log("디렉토리 생성 완료:", dirPath);
      } catch (error) {
        console.error("디렉토리 생성 실패:", error);
        throw error;
      }
    }

    static async getFileStats(filePath) {
      try {
        const stats = await fs.stat(filePath);
        return {
          size: stats.size,
          isFile: stats.isFile(),
          isDirectory: stats.isDirectory(),
          created: stats.birthtime,
          modified: stats.mtime,
          accessed: stats.atime,
        };
      } catch (error) {
        console.error("파일 정보 조회 실패:", error);
        throw error;
      }
    }
  }

  // 사용 예시
  async function nodeFileExample() {
    try {
      const content = await NodeFileManager.readFile("./package.json");
      console.log("Package.json 내용:", JSON.parse(content).name);

      const files = await NodeFileManager.listDirectory("./");
      console.log("현재 디렉토리 파일들:", files);
    } catch (error) {
      console.error("Node.js 파일 작업 실패:", error);
    }
  }
}

// 브라우저 - File API 사용 (사용자 상호작용 필요)
if (typeof window !== "undefined") {
  class BrowserFileManager {
    static async readUserFile() {
      return new Promise((resolve, reject) => {
        const input = document.createElement("input");
        input.type = "file";
        input.accept = ".txt,.json,.js";

        input.onchange = (event) => {
          const file = event.target.files[0];
          if (!file) {
            reject(new Error("파일이 선택되지 않았습니다"));
            return;
          }

          const reader = new FileReader();
          reader.onload = (e) => resolve(e.target.result);
          reader.onerror = () => reject(new Error("파일 읽기 실패"));
          reader.readAsText(file);
        };

        input.click();
      });
    }

    static downloadFile(content, filename, mimeType = "text/plain") {
      const blob = new Blob([content], { type: mimeType });
      const url = URL.createObjectURL(blob);

      const a = document.createElement("a");
      a.href = url;
      a.download = filename;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);

      URL.revokeObjectURL(url);
    }

    static async handleDragAndDrop(dropZoneElement) {
      return new Promise((resolve) => {
        dropZoneElement.addEventListener("dragover", (e) => {
          e.preventDefault();
          dropZoneElement.classList.add("drag-over");
        });

        dropZoneElement.addEventListener("dragleave", () => {
          dropZoneElement.classList.remove("drag-over");
        });

        dropZoneElement.addEventListener("drop", async (e) => {
          e.preventDefault();
          dropZoneElement.classList.remove("drag-over");

          const files = Array.from(e.dataTransfer.files);
          const fileContents = await Promise.all(
            files.map((file) => this.readFileContent(file))
          );

          resolve(fileContents);
        });
      });
    }

    static readFileContent(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) =>
          resolve({
            name: file.name,
            size: file.size,
            type: file.type,
            content: e.target.result,
          });
        reader.onerror = () => reject(new Error("파일 읽기 실패"));
        reader.readAsText(file);
      });
    }
  }

  // 사용 예시
  async function browserFileExample() {
    try {
      // 사용자가 파일을 선택하면 읽기
      const content = await BrowserFileManager.readUserFile();
      console.log("선택된 파일 내용:", content);

      // 파일 다운로드
      BrowserFileManager.downloadFile(
        JSON.stringify({ message: "Hello World" }, null, 2),
        "data.json",
        "application/json"
      );
    } catch (error) {
      console.error("브라우저 파일 작업 실패:", error);
    }
  }
}
```

### 2. 네트워킹의 차이

```javascript
// Node.js - HTTP 서버/클라이언트
if (typeof require !== "undefined") {
  const http = require("http");
  const https = require("https");
  const url = require("url");

  class NodeNetworking {
    static createServer(port = 3000) {
      const server = http.createServer((req, res) => {
        const parsedUrl = url.parse(req.url, true);

        // CORS 헤더 설정
        res.setHeader("Access-Control-Allow-Origin", "*");
        res.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
        res.setHeader("Access-Control-Allow-Headers", "Content-Type");

        if (req.method === "OPTIONS") {
          res.writeHead(200);
          res.end();
          return;
        }

        // 라우팅
        switch (parsedUrl.pathname) {
          case "/":
            res.writeHead(200, { "Content-Type": "application/json" });
            res.end(JSON.stringify({ message: "Hello from Node.js!" }));
            break;

          case "/api/data":
            if (req.method === "GET") {
              res.writeHead(200, { "Content-Type": "application/json" });
              res.end(
                JSON.stringify({
                  data: [1, 2, 3, 4, 5],
                  timestamp: new Date().toISOString(),
                })
              );
            } else {
              res.writeHead(405, { "Content-Type": "text/plain" });
              res.end("Method Not Allowed");
            }
            break;

          default:
            res.writeHead(404, { "Content-Type": "text/plain" });
            res.end("Not Found");
        }
      });

      server.listen(port, () => {
        console.log(`서버가 포트 ${port}에서 실행 중입니다`);
      });

      return server;
    }

    static async makeRequest(url, options = {}) {
      return new Promise((resolve, reject) => {
        const protocol = url.startsWith("https") ? https : http;

        const req = protocol.request(url, options, (res) => {
          let data = "";

          res.on("data", (chunk) => {
            data += chunk;
          });

          res.on("end", () => {
            try {
              const response = {
                statusCode: res.statusCode,
                headers: res.headers,
                body: data,
              };
              resolve(response);
            } catch (error) {
              reject(error);
            }
          });
        });

        req.on("error", reject);

        if (options.body) {
          req.write(options.body);
        }

        req.end();
      });
    }

    static async fetchData(url) {
      try {
        const response = await this.makeRequest(url);
        return JSON.parse(response.body);
      } catch (error) {
        console.error("데이터 가져오기 실패:", error);
        throw error;
      }
    }
  }

  // 사용 예시
  function nodeNetworkExample() {
    // 서버 생성
    const server = NodeNetworking.createServer(3000);

    // 클라이언트 요청
    setTimeout(async () => {
      try {
        const data = await NodeNetworking.fetchData(
          "http://localhost:3000/api/data"
        );
        console.log("서버에서 받은 데이터:", data);
      } catch (error) {
        console.error("요청 실패:", error);
      }
    }, 1000);
  }
}

// 브라우저 - Fetch API, WebSocket
if (typeof window !== "undefined") {
  class BrowserNetworking {
    static async fetchData(url, options = {}) {
      try {
        const response = await fetch(url, {
          method: "GET",
          headers: {
            "Content-Type": "application/json",
            ...options.headers,
          },
          ...options,
        });

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }

        const contentType = response.headers.get("Content-Type");
        if (contentType && contentType.includes("application/json")) {
          return await response.json();
        } else {
          return await response.text();
        }
      } catch (error) {
        console.error("Fetch 요청 실패:", error);
        throw error;
      }
    }

    static async postData(url, data) {
      return this.fetchData(url, {
        method: "POST",
        body: JSON.stringify(data),
      });
    }

    static createWebSocket(url) {
      const ws = new WebSocket(url);

      ws.onopen = () => {
        console.log("WebSocket 연결 성공");
      };

      ws.onmessage = (event) => {
        console.log("WebSocket 메시지 수신:", event.data);
      };

      ws.onclose = (event) => {
        console.log("WebSocket 연결 종료:", event.code, event.reason);
      };

      ws.onerror = (error) => {
        console.error("WebSocket 에러:", error);
      };

      return ws;
    }

    static async uploadFile(url, file, onProgress = null) {
      return new Promise((resolve, reject) => {
        const formData = new FormData();
        formData.append("file", file);

        const xhr = new XMLHttpRequest();

        if (onProgress) {
          xhr.upload.onprogress = (event) => {
            if (event.lengthComputable) {
              const percentComplete = (event.loaded / event.total) * 100;
              onProgress(percentComplete);
            }
          };
        }

        xhr.onload = () => {
          if (xhr.status >= 200 && xhr.status < 300) {
            resolve(JSON.parse(xhr.responseText));
          } else {
            reject(new Error(`Upload failed: ${xhr.status}`));
          }
        };

        xhr.onerror = () => reject(new Error("Upload failed"));

        xhr.open("POST", url);
        xhr.send(formData);
      });
    }
  }

  // 사용 예시
  async function browserNetworkExample() {
    try {
      // REST API 호출
      const data = await BrowserNetworking.fetchData("/api/users");
      console.log("사용자 데이터:", data);

      // POST 요청
      const newUser = await BrowserNetworking.postData("/api/users", {
        name: "John Doe",
        email: "john@example.com",
      });
      console.log("생성된 사용자:", newUser);

      // WebSocket 연결
      const ws = BrowserNetworking.createWebSocket("ws://localhost:8080");

      // 파일 업로드 (파일 선택 후)
      const fileInput = document.createElement("input");
      fileInput.type = "file";
      fileInput.onchange = async (event) => {
        const file = event.target.files[0];
        if (file) {
          try {
            const result = await BrowserNetworking.uploadFile(
              "/api/upload",
              file,
              (progress) => console.log(`업로드 진행률: ${progress}%`)
            );
            console.log("업로드 완료:", result);
          } catch (error) {
            console.error("업로드 실패:", error);
          }
        }
      };
    } catch (error) {
      console.error("네트워크 작업 실패:", error);
    }
  }
}
```

## 이벤트 루프의 차이점

### 1. 태스크 큐의 차이

```javascript
// 브라우저의 이벤트 루프
if (typeof window !== "undefined") {
  console.log("=== 브라우저 이벤트 루프 테스트 ===");

  setTimeout(() => console.log("1: setTimeout (매크로태스크)"), 0);

  Promise.resolve().then(() => console.log("2: Promise (마이크로태스크)"));

  queueMicrotask(() => console.log("3: queueMicrotask"));

  requestAnimationFrame(() => console.log("4: requestAnimationFrame"));

  setTimeout(() => console.log("5: setTimeout 2"), 0);

  console.log("6: 동기 코드");

  // 예상 출력 순서:
  // 6: 동기 코드
  // 2: Promise (마이크로태스크)
  // 3: queueMicrotask
  // 4: requestAnimationFrame (다음 프레임)
  // 1: setTimeout (매크로태스크)
  // 5: setTimeout 2
}

// Node.js의 이벤트 루프
if (typeof process !== "undefined") {
  console.log("=== Node.js 이벤트 루프 테스트 ===");

  setTimeout(() => console.log("1: setTimeout (타이머)"), 0);

  setImmediate(() => console.log("2: setImmediate (체크)"));

  process.nextTick(() => console.log("3: process.nextTick (최우선)"));

  Promise.resolve().then(() => console.log("4: Promise (마이크로태스크)"));

  setTimeout(() => console.log("5: setTimeout 2"), 0);

  queueMicrotask(() => console.log("6: queueMicrotask"));

  console.log("7: 동기 코드");

  // 예상 출력 순서 (Node.js):
  // 7: 동기 코드
  // 3: process.nextTick (최우선)
  // 4: Promise (마이크로태스크)
  // 6: queueMicrotask
  // 1: setTimeout (타이머)
  // 5: setTimeout 2
  // 2: setImmediate (체크)

  // Node.js 이벤트 루프 단계별 설명
  function explainNodeEventLoop() {
    console.log("\n=== Node.js 이벤트 루프 단계 ===");

    // 1. Timer phase
    setTimeout(() => {
      console.log("Timer phase: setTimeout 실행");

      // 2. I/O callbacks phase (시스템 에러 등)

      // 3. Idle, prepare phase (내부적으로 사용)

      // 4. Poll phase (새로운 I/O 이벤트 가져오기)
      const fs = require("fs");
      fs.readFile(__filename, () => {
        console.log("Poll phase: 파일 읽기 완료");

        // 5. Check phase
        setImmediate(() => {
          console.log("Check phase: setImmediate 실행");
        });

        // 6. Close callbacks phase
        const server = require("http").createServer();
        server.close(() => {
          console.log("Close callbacks phase: 서버 종료");
        });
      });
    }, 0);

    // 각 단계 사이에 마이크로태스크 실행
    process.nextTick(() => {
      console.log("마이크로태스크: process.nextTick");
    });

    Promise.resolve().then(() => {
      console.log("마이크로태스크: Promise.then");
    });
  }

  explainNodeEventLoop();
}

// 범용 이벤트 루프 유틸리티
class EventLoopUtils {
  static defer(callback) {
    if (typeof process !== "undefined" && process.nextTick) {
      // Node.js: 가장 높은 우선순위
      process.nextTick(callback);
    } else if (typeof queueMicrotask !== "undefined") {
      // 브라우저/Node.js: 마이크로태스크
      queueMicrotask(callback);
    } else if (typeof Promise !== "undefined") {
      // 폴백: Promise.then
      Promise.resolve().then(callback);
    } else {
      // 최후의 폴백: setTimeout
      setTimeout(callback, 0);
    }
  }

  static schedule(callback) {
    if (typeof setImmediate !== "undefined") {
      // Node.js: setImmediate
      setImmediate(callback);
    } else if (typeof requestAnimationFrame !== "undefined") {
      // 브라우저: requestAnimationFrame
      requestAnimationFrame(callback);
    } else {
      // 폴백: setTimeout
      setTimeout(callback, 0);
    }
  }

  static async measureEventLoopLag() {
    const start = Date.now();

    return new Promise((resolve) => {
      this.defer(() => {
        const lag = Date.now() - start;
        resolve(lag);
      });
    });
  }

  static createEventLoopMonitor(interval = 1000) {
    let monitoring = true;

    const monitor = async () => {
      if (!monitoring) return;

      const lag = await this.measureEventLoopLag();
      console.log(`이벤트 루프 지연: ${lag}ms`);

      if (lag > 100) {
        console.warn("⚠️ 이벤트 루프 지연이 높습니다!");
      }

      setTimeout(monitor, interval);
    };

    monitor();

    return () => {
      monitoring = false;
    };
  }
}

// 사용 예시
const stopMonitoring = EventLoopUtils.createEventLoopMonitor(2000);

// 5초 후 모니터링 중지
setTimeout(() => {
  stopMonitoring();
  console.log("이벤트 루프 모니터링 중지");
}, 10000);
```

## 성능과 최적화의 차이

### 1. 메모리 관리

```javascript
// Node.js 메모리 관리
if (typeof process !== "undefined") {
  class NodeMemoryManager {
    static getMemoryUsage() {
      const usage = process.memoryUsage();
      return {
        rss: Math.round(usage.rss / 1024 / 1024) + "MB", // Resident Set Size
        heapTotal: Math.round(usage.heapTotal / 1024 / 1024) + "MB",
        heapUsed: Math.round(usage.heapUsed / 1024 / 1024) + "MB",
        external: Math.round(usage.external / 1024 / 1024) + "MB",
        arrayBuffers: Math.round(usage.arrayBuffers / 1024 / 1024) + "MB",
      };
    }

    static monitorMemory(interval = 5000) {
      const monitor = () => {
        const usage = this.getMemoryUsage();
        console.log("메모리 사용량:", usage);

        // 메모리 사용량이 높으면 경고
        const heapUsed = parseInt(usage.heapUsed);
        if (heapUsed > 500) {
          // 500MB 이상
          console.warn("⚠️ 높은 메모리 사용량 감지!");

          // 가비지 컬렉션 강제 실행 (--expose-gc 플래그 필요)
          if (global.gc) {
            global.gc();
            console.log("가비지 컬렉션 실행됨");
          }
        }
      };

      const intervalId = setInterval(monitor, interval);
      monitor(); // 즉시 한 번 실행

      return () => clearInterval(intervalId);
    }

    static createLargeObject() {
      // 메모리 사용량 테스트용
      return new Array(1000000).fill("test data");
    }
  }

  // 사용 예시
  const stopMemoryMonitoring = NodeMemoryManager.monitorMemory();

  // 메모리 사용량 증가 시뮬레이션
  const largeObjects = [];
  for (let i = 0; i < 10; i++) {
    largeObjects.push(NodeMemoryManager.createLargeObject());
  }

  setTimeout(() => {
    stopMemoryMonitoring();
    console.log("메모리 모니터링 중지");
  }, 30000);
}

// 브라우저 메모리 관리
if (typeof window !== "undefined") {
  class BrowserMemoryManager {
    static getMemoryInfo() {
      if ("memory" in performance) {
        const memory = performance.memory;
        return {
          usedJSHeapSize:
            Math.round(memory.usedJSHeapSize / 1024 / 1024) + "MB",
          totalJSHeapSize:
            Math.round(memory.totalJSHeapSize / 1024 / 1024) + "MB",
          jsHeapSizeLimit:
            Math.round(memory.jsHeapSizeLimit / 1024 / 1024) + "MB",
        };
      } else {
        return { message: "Memory API not supported" };
      }
    }

    static observeMemory() {
      if ("PerformanceObserver" in window) {
        const observer = new PerformanceObserver((list) => {
          for (const entry of list.getEntries()) {
            if (entry.entryType === "memory") {
              console.log("메모리 정보:", entry);
            }
          }
        });

        try {
          observer.observe({ entryTypes: ["memory"] });
        } catch (error) {
          console.log("Memory observation not supported");
        }

        return observer;
      }
    }

    static measureDOMMemory() {
      if ("measureUserAgentSpecificMemory" in performance) {
        return performance.measureUserAgentSpecificMemory();
      } else {
        return Promise.reject(
          new Error("measureUserAgentSpecificMemory not supported")
        );
      }
    }

    static createMemoryLeak() {
      // 메모리 누수 예제 (피해야 할 패턴)
      const leakyElements = [];

      for (let i = 0; i < 1000; i++) {
        const element = document.createElement("div");
        element.innerHTML = `Element ${i}`;

        // 이벤트 리스너 추가 (정리하지 않으면 메모리 누수)
        element.addEventListener("click", function () {
          console.log(`Clicked element ${i}`);
        });

        leakyElements.push(element);
      }

      return leakyElements;
    }

    static cleanupMemoryLeak(elements) {
      // 메모리 누수 정리
      elements.forEach((element) => {
        element.removeEventListener("click", null);
        element.remove();
      });
      elements.length = 0;
    }
  }

  // 사용 예시
  console.log("브라우저 메모리 정보:", BrowserMemoryManager.getMemoryInfo());

  const memoryObserver = BrowserMemoryManager.observeMemory();

  // DOM 메모리 측정 (Chrome에서만 지원)
  BrowserMemoryManager.measureDOMMemory()
    .then((result) => console.log("DOM 메모리:", result))
    .catch((error) => console.log("DOM 메모리 측정 불가:", error.message));
}
```

### 2. 성능 측정의 차이

```javascript
// 범용 성능 측정 도구
class UniversalPerformanceMeasurer {
  static now() {
    if (typeof performance !== "undefined" && performance.now) {
      return performance.now();
    } else if (typeof process !== "undefined" && process.hrtime) {
      const [seconds, nanoseconds] = process.hrtime();
      return seconds * 1000 + nanoseconds / 1000000;
    } else {
      return Date.now();
    }
  }

  static measure(name, fn) {
    const start = this.now();

    try {
      const result = fn();

      if (result && typeof result.then === "function") {
        // Promise 처리
        return result.finally(() => {
          const duration = this.now() - start;
          console.log(`⏱️ ${name}: ${duration.toFixed(2)}ms`);
        });
      } else {
        const duration = this.now() - start;
        console.log(`⏱️ ${name}: ${duration.toFixed(2)}ms`);
        return result;
      }
    } catch (error) {
      const duration = this.now() - start;
      console.log(`❌ ${name} (에러): ${duration.toFixed(2)}ms`);
      throw error;
    }
  }

  static async measureAsync(name, asyncFn) {
    const start = this.now();

    try {
      const result = await asyncFn();
      const duration = this.now() - start;
      console.log(`⏱️ ${name}: ${duration.toFixed(2)}ms`);
      return result;
    } catch (error) {
      const duration = this.now() - start;
      console.log(`❌ ${name} (에러): ${duration.toFixed(2)}ms`);
      throw error;
    }
  }

  static benchmark(name, fn, iterations = 1000) {
    const times = [];

    for (let i = 0; i < iterations; i++) {
      const start = this.now();
      fn();
      const duration = this.now() - start;
      times.push(duration);
    }

    const total = times.reduce((sum, time) => sum + time, 0);
    const average = total / iterations;
    const min = Math.min(...times);
    const max = Math.max(...times);

    console.log(`📊 ${name} 벤치마크 (${iterations}회):`);
    console.log(`   평균: ${average.toFixed(4)}ms`);
    console.log(`   최소: ${min.toFixed(4)}ms`);
    console.log(`   최대: ${max.toFixed(4)}ms`);
    console.log(`   총합: ${total.toFixed(2)}ms`);

    return { average, min, max, total };
  }
}

// Node.js 전용 성능 측정
if (typeof process !== "undefined") {
  class NodePerformanceMeasurer extends UniversalPerformanceMeasurer {
    static measureCPU(name, fn) {
      const startUsage = process.cpuUsage();
      const startTime = process.hrtime.bigint();

      const result = fn();

      const endTime = process.hrtime.bigint();
      const endUsage = process.cpuUsage(startUsage);

      const wallTime = Number(endTime - startTime) / 1000000; // 나노초를 밀리초로
      const cpuTime = (endUsage.user + endUsage.system) / 1000; // 마이크로초를 밀리초로

      console.log(`🖥️ ${name}:`);
      console.log(`   벽시계 시간: ${wallTime.toFixed(2)}ms`);
      console.log(`   CPU 시간: ${cpuTime.toFixed(2)}ms`);
      console.log(`   사용자 시간: ${(endUsage.user / 1000).toFixed(2)}ms`);
      console.log(`   시스템 시간: ${(endUsage.system / 1000).toFixed(2)}ms`);

      return result;
    }

    static profileMemory(name, fn) {
      const startMemory = process.memoryUsage();
      const result = fn();
      const endMemory = process.memoryUsage();

      console.log(`🧠 ${name} 메모리 사용량:`);
      console.log(
        `   RSS 변화: ${(
          (endMemory.rss - startMemory.rss) /
          1024 /
          1024
        ).toFixed(2)}MB`
      );
      console.log(
        `   Heap 변화: ${(
          (endMemory.heapUsed - startMemory.heapUsed) /
          1024 /
          1024
        ).toFixed(2)}MB`
      );

      return result;
    }
  }

  // 사용 예시
  NodePerformanceMeasurer.measureCPU("CPU 집약적 작업", () => {
    let sum = 0;
    for (let i = 0; i < 1000000; i++) {
      sum += Math.sqrt(i);
    }
    return sum;
  });

  NodePerformanceMeasurer.profileMemory("메모리 집약적 작업", () => {
    const largeArray = new Array(1000000)
      .fill(0)
      .map((_, i) => ({ id: i, data: `item ${i}` }));
    return largeArray.length;
  });
}

// 브라우저 전용 성능 측정
if (typeof window !== "undefined") {
  class BrowserPerformanceMeasurer extends UniversalPerformanceMeasurer {
    static measurePaint(name, fn) {
      performance.mark(`${name}-start`);

      const result = fn();

      performance.mark(`${name}-end`);
      performance.measure(name, `${name}-start`, `${name}-end`);

      const measure = performance.getEntriesByName(name)[0];
      console.log(`🎨 ${name}: ${measure.duration.toFixed(2)}ms`);

      // 성능 엔트리 정리
      performance.clearMarks(`${name}-start`);
      performance.clearMarks(`${name}-end`);
      performance.clearMeasures(name);

      return result;
    }

    static observePerformance() {
      if ("PerformanceObserver" in window) {
        const observer = new PerformanceObserver((list) => {
          for (const entry of list.getEntries()) {
            console.log(`📈 ${entry.entryType}:`, entry);
          }
        });

        observer.observe({
          entryTypes: [
            "navigation",
            "resource",
            "paint",
            "largest-contentful-paint",
          ],
        });

        return observer;
      }
    }

    static getWebVitals() {
      const vitals = {};

      // First Contentful Paint
      const fcpEntry = performance.getEntriesByName(
        "first-contentful-paint"
      )[0];
      if (fcpEntry) {
        vitals.fcp = fcpEntry.startTime;
      }

      // Largest Contentful Paint
      const lcpEntries = performance.getEntriesByType(
        "largest-contentful-paint"
      );
      if (lcpEntries.length > 0) {
        vitals.lcp = lcpEntries[lcpEntries.length - 1].startTime;
      }

      // Navigation timing
      const navigation = performance.getEntriesByType("navigation")[0];
      if (navigation) {
        vitals.domContentLoaded =
          navigation.domContentLoadedEventEnd -
          navigation.domContentLoadedEventStart;
        vitals.loadComplete =
          navigation.loadEventEnd - navigation.loadEventStart;
      }

      return vitals;
    }
  }

  // 사용 예시
  const performanceObserver = BrowserPerformanceMeasurer.observePerformance();

  BrowserPerformanceMeasurer.measurePaint("DOM 조작", () => {
    const fragment = document.createDocumentFragment();
    for (let i = 0; i < 1000; i++) {
      const div = document.createElement("div");
      div.textContent = `Item ${i}`;
      fragment.appendChild(div);
    }
    document.body.appendChild(fragment);
  });

  setTimeout(() => {
    console.log("Web Vitals:", BrowserPerformanceMeasurer.getWebVitals());
  }, 2000);
}

// 공통 사용 예시
UniversalPerformanceMeasurer.benchmark("배열 생성", () => {
  return new Array(10000).fill(0).map((_, i) => i * 2);
});

UniversalPerformanceMeasurer.measureAsync("비동기 작업", async () => {
  await new Promise((resolve) => setTimeout(resolve, 100));
  return "완료";
});
```

## 결론

브라우저와 Node.js 환경은 같은 JavaScript 언어를 사용하지만, 런타임 환경, API, 모듈 시스템, 이벤트 루프 등에서 중요한 차이점들을 가지고 있습니다. 개발자는 이러한 차이점을 이해하고 각 환경에 적합한 코드를 작성해야 하며, 필요에 따라 범용 코드를 작성하여 두 환경에서 모두 동작할 수 있도록 해야 합니다. 현대의 JavaScript 개발에서는 이러한 환경 간의 차이를 추상화하는 도구들이 많이 발달해 있어, 개발자가 더 쉽게 크로스 플랫폼 애플리케이션을 개발할 수 있게 되었습니다.
