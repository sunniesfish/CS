# 타입 안전성과 TypeScript의 통합 전략

## 개요

타입 안전성은 프로그램이 실행되기 전에 타입 오류를 감지하여 런타임 에러를 방지하는 프로그래밍 패러다임입니다. TypeScript는 JavaScript에 정적 타입 검사를 도입하여 대규모 애플리케이션의 안정성과 유지보수성을 크게 향상시킵니다.

## 탄생 배경

JavaScript의 동적 타입 시스템은 유연성을 제공하지만, 대규모 프로젝트에서는 타입 관련 버그와 유지보수의 어려움을 야기합니다. TypeScript는 이러한 문제를 해결하면서도 JavaScript의 생산성을 유지하기 위해 Microsoft에서 개발되었습니다.

## 타입 안전성의 기본 개념

### 1. 정적 타입 vs 동적 타입

```typescript
// JavaScript (동적 타입)
function addJS(a, b) {
  return a + b;
}

console.log(addJS(5, 3)); // 8
console.log(addJS("5", "3")); // "53" - 의도하지 않은 동작
console.log(addJS(5, "3")); // "53" - 타입 강제 변환

// TypeScript (정적 타입)
function addTS(a: number, b: number): number {
  return a + b;
}

console.log(addTS(5, 3)); // 8
// console.log(addTS("5", "3")); // 컴파일 에러
// console.log(addTS(5, "3")); // 컴파일 에러

// 타입 가드를 통한 안전한 처리
function safeAdd(a: unknown, b: unknown): number | string {
  if (typeof a === "number" && typeof b === "number") {
    return a + b;
  }
  if (typeof a === "string" && typeof b === "string") {
    return a + b;
  }
  throw new Error("Invalid arguments: both must be numbers or strings");
}
```

### 2. 타입 추론과 명시적 타입

```typescript
// 타입 추론
let name = "Alice"; // string으로 추론
let age = 30; // number로 추론
let isActive = true; // boolean으로 추론

// 복잡한 타입 추론
const users = [
  { id: 1, name: "Alice", age: 30 },
  { id: 2, name: "Bob", age: 25 },
]; // { id: number; name: string; age: number; }[]로 추론

// 명시적 타입 선언
interface User {
  id: number;
  name: string;
  age: number;
  email?: string; // 선택적 속성
}

const user: User = {
  id: 1,
  name: "Alice",
  age: 30,
};

// 함수 타입 명시
type Calculator = (a: number, b: number) => number;

const multiply: Calculator = (a, b) => a * b;
const divide: Calculator = (a, b) => {
  if (b === 0) {
    throw new Error("Division by zero");
  }
  return a / b;
};
```

## TypeScript 핵심 기능

### 1. 인터페이스와 타입 별칭

```typescript
// 인터페이스 정의
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}

// 인터페이스 확장
interface DigitalProduct extends Product {
  downloadUrl: string;
  fileSize: number;
}

interface PhysicalProduct extends Product {
  weight: number;
  dimensions: {
    length: number;
    width: number;
    height: number;
  };
}

// 타입 별칭
type ProductId = number;
type ProductName = string;
type Price = number;

// 유니온 타입
type ProductType = DigitalProduct | PhysicalProduct;

// 인터섹션 타입
type ProductWithReviews = Product & {
  reviews: Review[];
  averageRating: number;
};

interface Review {
  id: number;
  userId: number;
  rating: number;
  comment: string;
  createdAt: Date;
}

// 조건부 타입
type ProductCategory<T> = T extends DigitalProduct
  ? "digital"
  : T extends PhysicalProduct
  ? "physical"
  : "unknown";

// 매핑된 타입
type PartialProduct = Partial<Product>; // 모든 속성을 선택적으로
type RequiredProduct = Required<Product>; // 모든 속성을 필수로
type ProductKeys = keyof Product; // 'id' | 'name' | 'price' | 'category' | 'inStock'
```

### 2. 제네릭과 고급 타입

```typescript
// 기본 제네릭
interface Repository<T> {
  findById(id: number): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(item: Omit<T, "id">): Promise<T>;
  update(id: number, item: Partial<T>): Promise<T>;
  delete(id: number): Promise<boolean>;
}

class ProductRepository implements Repository<Product> {
  private products: Product[] = [];

  async findById(id: number): Promise<Product | null> {
    return this.products.find((p) => p.id === id) || null;
  }

  async findAll(): Promise<Product[]> {
    return [...this.products];
  }

  async create(item: Omit<Product, "id">): Promise<Product> {
    const newProduct: Product = {
      id: Date.now(),
      ...item,
    };
    this.products.push(newProduct);
    return newProduct;
  }

  async update(id: number, item: Partial<Product>): Promise<Product> {
    const index = this.products.findIndex((p) => p.id === id);
    if (index === -1) {
      throw new Error(`Product with id ${id} not found`);
    }

    this.products[index] = { ...this.products[index], ...item };
    return this.products[index];
  }

  async delete(id: number): Promise<boolean> {
    const index = this.products.findIndex((p) => p.id === id);
    if (index === -1) {
      return false;
    }

    this.products.splice(index, 1);
    return true;
  }
}

// 제네릭 제약
interface Identifiable {
  id: number;
}

class GenericRepository<T extends Identifiable> {
  protected items: T[] = [];

  findById(id: number): T | undefined {
    return this.items.find((item) => item.id === id);
  }

  add(item: T): void {
    this.items.push(item);
  }
}

// 조건부 타입과 유틸리티 타입
type ApiResponse<T> =
  | {
      success: true;
      data: T;
    }
  | {
      success: false;
      error: string;
    };

// 타입 가드 함수
function isSuccessResponse<T>(
  response: ApiResponse<T>
): response is { success: true; data: T } {
  return response.success;
}

// 사용 예시
async function handleApiResponse<T>(response: ApiResponse<T>): Promise<T> {
  if (isSuccessResponse(response)) {
    return response.data; // 타입 안전
  } else {
    throw new Error(response.error);
  }
}
```

### 3. 데코레이터와 메타데이터

```typescript
// 클래스 데코레이터
function Entity(tableName: string) {
  return function <T extends { new (...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      tableName = tableName;
    };
  };
}

// 속성 데코레이터
function Column(options?: { nullable?: boolean; unique?: boolean }) {
  return function (target: any, propertyKey: string) {
    const columns = Reflect.getMetadata("columns", target) || [];
    columns.push({
      propertyKey,
      ...options,
    });
    Reflect.defineMetadata("columns", columns, target);
  };
}

// 메서드 데코레이터
function Validate(validationRules: ValidationRule[]) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      // 유효성 검사 로직
      for (const rule of validationRules) {
        if (!rule.validate(args[0])) {
          throw new Error(rule.message);
        }
      }

      return originalMethod.apply(this, args);
    };
  };
}

interface ValidationRule {
  validate(value: any): boolean;
  message: string;
}

// 사용 예시
@Entity("users")
class User {
  @Column({ unique: true })
  id!: number;

  @Column({ nullable: false })
  name!: string;

  @Column({ nullable: false, unique: true })
  email!: string;

  @Validate([
    {
      validate: (user: Partial<User>) => !!user.name && user.name.length > 0,
      message: "Name is required",
    },
    {
      validate: (user: Partial<User>) =>
        !!user.email && user.email.includes("@"),
      message: "Valid email is required",
    },
  ])
  create(userData: Omit<User, "id">): User {
    return {
      id: Date.now(),
      ...userData,
    };
  }
}
```

## 점진적 마이그레이션 전략

### 1. JavaScript에서 TypeScript로의 단계별 전환

```typescript
// 1단계: JavaScript 파일에 JSDoc 주석 추가
/**
 * @param {number} a
 * @param {number} b
 * @returns {number}
 */
function add(a, b) {
  return a + b;
}

/**
 * @typedef {Object} User
 * @property {number} id
 * @property {string} name
 * @property {string} email
 * @property {boolean} isActive
 */

/**
 * @param {User[]} users
 * @returns {User[]}
 */
function getActiveUsers(users) {
  return users.filter((user) => user.isActive);
}

// 2단계: .js를 .ts로 변경하고 기본 타입 추가
function addTyped(a: number, b: number): number {
  return a + b;
}

interface UserInterface {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

function getActiveUsersTyped(users: UserInterface[]): UserInterface[] {
  return users.filter((user) => user.isActive);
}

// 3단계: 엄격한 타입 검사 활성화
// tsconfig.json에서 strict: true 설정

// 4단계: 고급 타입 기능 도입
type UserStatus = "active" | "inactive" | "pending";

interface EnhancedUser {
  id: number;
  name: string;
  email: string;
  status: UserStatus;
  createdAt: Date;
  updatedAt: Date;
}

class UserService {
  private users: EnhancedUser[] = [];

  createUser(
    userData: Omit<EnhancedUser, "id" | "createdAt" | "updatedAt">
  ): EnhancedUser {
    const now = new Date();
    const newUser: EnhancedUser = {
      id: Date.now(),
      ...userData,
      createdAt: now,
      updatedAt: now,
    };

    this.users.push(newUser);
    return newUser;
  }

  updateUserStatus(id: number, status: UserStatus): EnhancedUser | null {
    const user = this.users.find((u) => u.id === id);
    if (!user) {
      return null;
    }

    user.status = status;
    user.updatedAt = new Date();
    return user;
  }

  getUsersByStatus(status: UserStatus): EnhancedUser[] {
    return this.users.filter((user) => user.status === status);
  }
}
```

### 2. 라이브러리와 외부 모듈 타입 정의

```typescript
// 외부 라이브러리 타입 정의
declare module "custom-library" {
  export interface CustomLibraryOptions {
    apiKey: string;
    baseUrl?: string;
    timeout?: number;
  }

  export class CustomLibrary {
    constructor(options: CustomLibraryOptions);
    makeRequest<T>(endpoint: string, data?: any): Promise<T>;
    setApiKey(apiKey: string): void;
  }

  export default CustomLibrary;
}

// 글로벌 타입 확장
declare global {
  interface Window {
    customGlobal: {
      version: string;
      config: Record<string, any>;
    };
  }

  namespace NodeJS {
    interface ProcessEnv {
      NODE_ENV: "development" | "production" | "test";
      API_URL: string;
      DATABASE_URL: string;
    }
  }
}

// 모듈 증강 (Module Augmentation)
import "express";

declare module "express" {
  interface Request {
    user?: {
      id: number;
      email: string;
      role: string;
    };
  }
}

// 사용 예시
import express from "express";

const app = express();

app.use((req, res, next) => {
  // 인증 로직
  req.user = {
    id: 1,
    email: "user@example.com",
    role: "admin",
  };
  next();
});

app.get("/profile", (req, res) => {
  if (req.user) {
    res.json({
      id: req.user.id,
      email: req.user.email,
      role: req.user.role,
    });
  } else {
    res.status(401).json({ error: "Unauthorized" });
  }
});
```

## 실무 활용 패턴

### 1. API 클라이언트 타입 안전성

```typescript
// API 응답 타입 정의
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
}

interface PaginatedResponse<T> {
  items: T[];
  totalCount: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

// 엔드포인트별 타입 정의
interface UserEndpoints {
  "GET /users": {
    query?: {
      page?: number;
      limit?: number;
      status?: UserStatus;
    };
    response: ApiResponse<PaginatedResponse<EnhancedUser>>;
  };
  "GET /users/:id": {
    params: { id: number };
    response: ApiResponse<EnhancedUser>;
  };
  "POST /users": {
    body: Omit<EnhancedUser, "id" | "createdAt" | "updatedAt">;
    response: ApiResponse<EnhancedUser>;
  };
  "PUT /users/:id": {
    params: { id: number };
    body: Partial<Omit<EnhancedUser, "id" | "createdAt" | "updatedAt">>;
    response: ApiResponse<EnhancedUser>;
  };
  "DELETE /users/:id": {
    params: { id: number };
    response: ApiResponse<{ deleted: boolean }>;
  };
}

// 타입 안전한 API 클라이언트
class TypedApiClient {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async request<K extends keyof UserEndpoints, E = UserEndpoints[K]>(
    method: string,
    endpoint: K,
    options?: {
      params?: E extends { params: infer P } ? P : never;
      query?: E extends { query: infer Q } ? Q : never;
      body?: E extends { body: infer B } ? B : never;
    }
  ): Promise<E extends { response: infer R } ? R : never> {
    // URL 구성
    let url = `${this.baseUrl}${endpoint}`;

    // 경로 매개변수 대체
    if (options?.params) {
      for (const [key, value] of Object.entries(options.params)) {
        url = url.replace(`:${key}`, String(value));
      }
    }

    // 쿼리 매개변수 추가
    if (options?.query) {
      const queryString = new URLSearchParams(
        Object.entries(options.query).reduce((acc, [key, value]) => {
          if (value !== undefined) {
            acc[key] = String(value);
          }
          return acc;
        }, {} as Record<string, string>)
      ).toString();

      if (queryString) {
        url += `?${queryString}`;
      }
    }

    // 요청 옵션
    const requestOptions: RequestInit = {
      method,
      headers: {
        "Content-Type": "application/json",
      },
    };

    if (options?.body) {
      requestOptions.body = JSON.stringify(options.body);
    }

    const response = await fetch(url, requestOptions);
    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.error || `HTTP ${response.status}`);
    }

    return data;
  }

  // 편의 메서드
  async getUsers(query?: UserEndpoints["GET /users"]["query"]) {
    return this.request("GET", "GET /users", { query });
  }

  async getUser(id: number) {
    return this.request("GET", "GET /users/:id", { params: { id } });
  }

  async createUser(userData: UserEndpoints["POST /users"]["body"]) {
    return this.request("POST", "POST /users", { body: userData });
  }

  async updateUser(
    id: number,
    userData: UserEndpoints["PUT /users/:id"]["body"]
  ) {
    return this.request("PUT", "PUT /users/:id", {
      params: { id },
      body: userData,
    });
  }

  async deleteUser(id: number) {
    return this.request("DELETE", "DELETE /users/:id", { params: { id } });
  }
}
```

### 2. 상태 관리 타입 안전성

```typescript
// Redux 스타일 상태 관리
interface AppState {
  users: {
    items: EnhancedUser[];
    loading: boolean;
    error: string | null;
    currentPage: number;
    totalPages: number;
  };
  auth: {
    user: EnhancedUser | null;
    isAuthenticated: boolean;
    token: string | null;
  };
}

// 액션 타입 정의
type UserAction =
  | { type: "FETCH_USERS_REQUEST" }
  | {
      type: "FETCH_USERS_SUCCESS";
      payload: { users: EnhancedUser[]; totalPages: number };
    }
  | { type: "FETCH_USERS_FAILURE"; payload: { error: string } }
  | { type: "CREATE_USER_SUCCESS"; payload: { user: EnhancedUser } }
  | { type: "UPDATE_USER_SUCCESS"; payload: { user: EnhancedUser } }
  | { type: "DELETE_USER_SUCCESS"; payload: { id: number } }
  | { type: "SET_CURRENT_PAGE"; payload: { page: number } };

type AuthAction =
  | { type: "LOGIN_SUCCESS"; payload: { user: EnhancedUser; token: string } }
  | { type: "LOGOUT" }
  | { type: "UPDATE_PROFILE"; payload: { user: EnhancedUser } };

type AppAction = UserAction | AuthAction;

// 리듀서 타입 정의
type Reducer<S, A> = (state: S, action: A) => S;

const userReducer: Reducer<AppState["users"], UserAction> = (state, action) => {
  switch (action.type) {
    case "FETCH_USERS_REQUEST":
      return { ...state, loading: true, error: null };

    case "FETCH_USERS_SUCCESS":
      return {
        ...state,
        loading: false,
        items: action.payload.users,
        totalPages: action.payload.totalPages,
      };

    case "FETCH_USERS_FAILURE":
      return {
        ...state,
        loading: false,
        error: action.payload.error,
      };

    case "CREATE_USER_SUCCESS":
      return {
        ...state,
        items: [...state.items, action.payload.user],
      };

    case "UPDATE_USER_SUCCESS":
      return {
        ...state,
        items: state.items.map((user) =>
          user.id === action.payload.user.id ? action.payload.user : user
        ),
      };

    case "DELETE_USER_SUCCESS":
      return {
        ...state,
        items: state.items.filter((user) => user.id !== action.payload.id),
      };

    case "SET_CURRENT_PAGE":
      return {
        ...state,
        currentPage: action.payload.page,
      };

    default:
      return state;
  }
};

// 액션 크리에이터
const userActions = {
  fetchUsersRequest: (): UserAction => ({ type: "FETCH_USERS_REQUEST" }),

  fetchUsersSuccess: (
    users: EnhancedUser[],
    totalPages: number
  ): UserAction => ({
    type: "FETCH_USERS_SUCCESS",
    payload: { users, totalPages },
  }),

  fetchUsersFailure: (error: string): UserAction => ({
    type: "FETCH_USERS_FAILURE",
    payload: { error },
  }),

  createUserSuccess: (user: EnhancedUser): UserAction => ({
    type: "CREATE_USER_SUCCESS",
    payload: { user },
  }),

  updateUserSuccess: (user: EnhancedUser): UserAction => ({
    type: "UPDATE_USER_SUCCESS",
    payload: { user },
  }),

  deleteUserSuccess: (id: number): UserAction => ({
    type: "DELETE_USER_SUCCESS",
    payload: { id },
  }),

  setCurrentPage: (page: number): UserAction => ({
    type: "SET_CURRENT_PAGE",
    payload: { page },
  }),
};
```

### 3. 컴포넌트 Props 타입 정의

```typescript
// React 컴포넌트 타입 정의
interface UserListProps {
  users: EnhancedUser[];
  loading: boolean;
  error: string | null;
  currentPage: number;
  totalPages: number;
  onPageChange: (page: number) => void;
  onUserSelect: (user: EnhancedUser) => void;
  onUserDelete: (id: number) => void;
}

interface UserCardProps {
  user: EnhancedUser;
  onClick: (user: EnhancedUser) => void;
  onDelete: (id: number) => void;
  showActions?: boolean;
  className?: string;
}

// 제네릭 컴포넌트 타입
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
  loading?: boolean;
  error?: string | null;
  emptyMessage?: string;
  className?: string;
}

// 고차 컴포넌트 타입
interface WithLoadingProps {
  loading: boolean;
}

function withLoading<P extends object>(
  WrappedComponent: React.ComponentType<P>
): React.ComponentType<P & WithLoadingProps> {
  return function WithLoadingComponent(props: P & WithLoadingProps) {
    if (props.loading) {
      return <div>Loading...</div>;
    }

    const { loading, ...restProps } = props;
    return <WrappedComponent {...(restProps as P)} />;
  };
}

// 커스텀 훅 타입
interface UseApiOptions<T> {
  initialData?: T;
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
}

interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => Promise<void>;
}

function useApi<T>(
  apiCall: () => Promise<T>,
  dependencies: React.DependencyList = [],
  options: UseApiOptions<T> = {}
): UseApiResult<T> {
  const [data, setData] = React.useState<T | null>(options.initialData || null);
  const [loading, setLoading] = React.useState(false);
  const [error, setError] = React.useState<Error | null>(null);

  const fetchData = React.useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const result = await apiCall();
      setData(result);
      options.onSuccess?.(result);
    } catch (err) {
      const error = err instanceof Error ? err : new Error("Unknown error");
      setError(error);
      options.onError?.(error);
    } finally {
      setLoading(false);
    }
  }, dependencies);

  React.useEffect(() => {
    fetchData();
  }, [fetchData]);

  return {
    data,
    loading,
    error,
    refetch: fetchData,
  };
}
```

## 성능 최적화와 빌드 설정

### 1. TypeScript 컴파일러 최적화

```json
// tsconfig.json 최적화 설정
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],

    // 타입 검사 최적화
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,

    // 성능 최적화
    "skipLibCheck": true,
    "incremental": true,
    "tsBuildInfoFile": "./node_modules/.cache/typescript/tsbuildinfo",

    // 출력 설정
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,

    // 모듈 해결
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@types/*": ["src/types/*"]
    },

    // 실험적 기능
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,

    // 엄격한 검사
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts", "**/*.spec.ts"]
}
```

### 2. 타입 전용 임포트와 번들 최적화

```typescript
// 타입 전용 임포트
import type { EnhancedUser, UserStatus } from "./types/user";
import type { ApiResponse } from "./types/api";

// 런타임 임포트
import { UserService } from "./services/UserService";
import { validateEmail } from "./utils/validation";

// 조건부 타입으로 번들 크기 최적화
type ImportedModule<T> = T extends "development"
  ? typeof import("./dev-tools")
  : never;

// 지연 로딩을 위한 타입 정의
interface LazyComponent<P = {}> {
  (): Promise<{ default: React.ComponentType<P> }>;
}

const LazyUserList: LazyComponent<UserListProps> = () =>
  import("./components/UserList");

// Tree shaking을 위한 유틸리티 타입
export type { EnhancedUser, UserStatus } from "./types/user";
export { UserService } from "./services/UserService";

// 타입 가드를 통한 런타임 최적화
function isEnhancedUser(obj: any): obj is EnhancedUser {
  return (
    obj &&
    typeof obj.id === "number" &&
    typeof obj.name === "string" &&
    typeof obj.email === "string" &&
    typeof obj.status === "string" &&
    obj.createdAt instanceof Date &&
    obj.updatedAt instanceof Date
  );
}

// 타입 안전한 환경 변수 처리
interface EnvironmentVariables {
  NODE_ENV: "development" | "production" | "test";
  API_URL: string;
  DATABASE_URL: string;
  JWT_SECRET: string;
}

function getEnvVar<K extends keyof EnvironmentVariables>(
  key: K
): EnvironmentVariables[K] {
  const value = process.env[key];
  if (!value) {
    throw new Error(`Environment variable ${key} is not defined`);
  }
  return value as EnvironmentVariables[K];
}

// 사용 예시
const apiUrl = getEnvVar("API_URL"); // string 타입 보장
const nodeEnv = getEnvVar("NODE_ENV"); // 'development' | 'production' | 'test' 타입 보장
```

## 테스팅과 타입 안전성

### 1. 타입 안전한 테스트 작성

```typescript
// 테스트 유틸리티 타입
type MockFunction<T extends (...args: any[]) => any> = jest.MockedFunction<T>;

interface TestUser extends EnhancedUser {
  id: number;
  name: string;
  email: string;
  status: UserStatus;
  createdAt: Date;
  updatedAt: Date;
}

// 테스트 데이터 팩토리
class TestDataFactory {
  static createUser(overrides: Partial<TestUser> = {}): TestUser {
    const now = new Date();
    return {
      id: Math.floor(Math.random() * 1000),
      name: "Test User",
      email: "test@example.com",
      status: "active",
      createdAt: now,
      updatedAt: now,
      ...overrides,
    };
  }

  static createUsers(
    count: number,
    overrides: Partial<TestUser> = {}
  ): TestUser[] {
    return Array.from({ length: count }, (_, index) =>
      this.createUser({
        id: index + 1,
        name: `Test User ${index + 1}`,
        email: `test${index + 1}@example.com`,
        ...overrides,
      })
    );
  }
}

// 타입 안전한 모킹
interface MockUserService {
  findById: MockFunction<UserService["findById"]>;
  findAll: MockFunction<UserService["findAll"]>;
  create: MockFunction<UserService["create"]>;
  update: MockFunction<UserService["update"]>;
  delete: MockFunction<UserService["delete"]>;
}

function createMockUserService(): MockUserService {
  return {
    findById: jest.fn(),
    findAll: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  };
}

// 테스트 케이스
describe("UserService", () => {
  let userService: UserService;
  let mockUserService: MockUserService;

  beforeEach(() => {
    userService = new UserService();
    mockUserService = createMockUserService();
  });

  describe("findById", () => {
    it("should return user when found", async () => {
      // Arrange
      const testUser = TestDataFactory.createUser();
      mockUserService.findById.mockResolvedValue(testUser);

      // Act
      const result = await mockUserService.findById(testUser.id);

      // Assert
      expect(result).toEqual(testUser);
      expect(mockUserService.findById).toHaveBeenCalledWith(testUser.id);
    });

    it("should return null when user not found", async () => {
      // Arrange
      mockUserService.findById.mockResolvedValue(null);

      // Act
      const result = await mockUserService.findById(999);

      // Assert
      expect(result).toBeNull();
    });
  });

  describe("create", () => {
    it("should create user with valid data", async () => {
      // Arrange
      const userData = {
        name: "New User",
        email: "new@example.com",
        status: "active" as UserStatus,
      };
      const createdUser = TestDataFactory.createUser(userData);
      mockUserService.create.mockResolvedValue(createdUser);

      // Act
      const result = await mockUserService.create(userData);

      // Assert
      expect(result).toEqual(createdUser);
      expect(mockUserService.create).toHaveBeenCalledWith(userData);
    });
  });
});

// 타입 테스트 (컴파일 타임)
// 이 코드들은 컴파일 에러가 발생해야 함을 테스트
type AssertEqual<T, U> = T extends U ? (U extends T ? true : false) : false;
type AssertTrue<T extends true> = T;

// 타입 테스트 예시
type Test1 = AssertTrue<
  AssertEqual<UserStatus, "active" | "inactive" | "pending">
>;
type Test2 = AssertTrue<
  AssertEqual<
    keyof EnhancedUser,
    "id" | "name" | "email" | "status" | "createdAt" | "updatedAt"
  >
>;

// 런타임 타입 테스트
describe("Type Guards", () => {
  it("should correctly identify EnhancedUser", () => {
    const validUser = TestDataFactory.createUser();
    const invalidUser = { id: 1, name: "Test" }; // 필수 속성 누락

    expect(isEnhancedUser(validUser)).toBe(true);
    expect(isEnhancedUser(invalidUser)).toBe(false);
    expect(isEnhancedUser(null)).toBe(false);
    expect(isEnhancedUser(undefined)).toBe(false);
  });
});
```

## 결론

TypeScript를 통한 타입 안전성 확보는 현대 JavaScript 개발의 필수 요소가 되었습니다. 정적 타입 검사, 인터페이스 정의, 제네릭 활용, 그리고 점진적 마이그레이션 전략을 통해 코드의 안정성과 유지보수성을 크게 향상시킬 수 있습니다. 특히 대규모 프로젝트에서는 타입 시스템이 개발자 간의 소통을 개선하고, 버그를 사전에 방지하며, 리팩토링을 안전하게 수행할 수 있게 해줍니다.
