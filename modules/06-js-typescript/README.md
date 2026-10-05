# Module 6: JavaScript & TypeScript

## Learning Objectives

- Understand modern JavaScript (ES6+)
- Learn TypeScript fundamentals
- Work with async/await in JavaScript
- Use modules and modern tooling

## 6.1 Modern JavaScript (ES6+)

Coming from PHP, JavaScript has evolved significantly. Here's what's new:

### Let and Const

```javascript
// var is function-scoped (old way)
var x = 1;

// let is block-scoped (preferred for variables)
let count = 0;
count = count + 1;

// const is block-scoped and cannot be reassigned
const API_URL = "http://localhost:8000";
// API_URL = "other"; // Error!

// But objects/arrays can be mutated
const user = { name: "Alice" };
user.name = "Bob"; // OK
```

### Arrow Functions

```javascript
// Traditional function
function add(a, b) {
  return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// With body
const greet = (name) => {
  const message = `Hello, ${name}`;
  return message;
};

// In callbacks
const numbers = [1, 2, 3];
const doubled = numbers.map((n) => n * 2);
```

### Destructuring

```javascript
// Object destructuring
const user = { name: "Alice", age: 30, email: "alice@example.com" };
const { name, age } = user;
console.log(name); // "Alice"

// With rename
const { name: userName } = user;

// With defaults
const { role = "user" } = user;

// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest); // [3, 4, 5]

// In function parameters
function createUser({ name, email, role = "user" }) {
  return { name, email, role };
}
```

### Spread Operator

```javascript
// Array spread
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

// Object spread
const defaults = { theme: "light", lang: "en" };
const userPrefs = { theme: "dark" };
const settings = { ...defaults, ...userPrefs }; // { theme: "dark", lang: "en" }

// Function arguments
const numbers = [1, 2, 3];
Math.max(...numbers); // 3
```

### Template Literals

```javascript
const name = "Alice";
const age = 30;

// Template literal with expressions
const message = `Hello, ${name}! You are ${age} years old.`;

// Multi-line strings
const html = `
  <div class="user">
    <h2>${name}</h2>
    <p>Age: ${age}</p>
  </div>
`;
```

### Optional Chaining & Nullish Coalescing

```javascript
const user = {
  name: "Alice",
  address: {
    city: "NYC",
  },
};

// Optional chaining (?.)
const city = user.address?.city; // "NYC"
const zip = user.address?.zip; // undefined (no error)
const country = user.location?.country; // undefined (no error)

// Nullish coalescing (??)
const theme = user.theme ?? "light"; // "light" (only if null/undefined)
const count = user.count ?? 0; // 0

// Difference from ||
const value1 = 0 || "default"; // "default" (0 is falsy)
const value2 = 0 ?? "default"; // 0 (only null/undefined trigger default)
```

### Modules

```javascript
// Named exports (math.js)
export const PI = 3.14159;
export function add(a, b) {
  return a + b;
}

// Default export (User.js)
export default class User {
  constructor(name) {
    this.name = name;
  }
}

// Importing
import User from "./User.js"; // Default import
import { PI, add } from "./math.js"; // Named imports
import { add as sum } from "./math.js"; // Rename
import * as math from "./math.js"; // Namespace import
```

---

## 6.2 Async JavaScript

### Promises

```javascript
// Creating a Promise
const fetchUser = (id) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, name: "Alice" });
      } else {
        reject(new Error("Invalid ID"));
      }
    }, 1000);
  });
};

// Using Promises
fetchUser(1)
  .then((user) => console.log(user))
  .catch((error) => console.error(error))
  .finally(() => console.log("Done"));
```

### Async/Await

```javascript
// Async function
async function getUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error("User not found");
    }
    const user = await response.json();
    return user;
  } catch (error) {
    console.error("Error:", error);
    throw error;
  }
}

// Arrow function version
const getUser = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

// Parallel execution
const [user, posts] = await Promise.all([
  fetch("/api/user").then((r) => r.json()),
  fetch("/api/posts").then((r) => r.json()),
]);
```

---

## 6.3 TypeScript Fundamentals

### Why TypeScript?

- Catch errors at compile time
- Better IDE support (autocomplete, refactoring)
- Self-documenting code
- Required for large Vue/React projects

### Basic Types

```typescript
// Primitives
let name: string = "Alice";
let age: number = 30;
let isActive: boolean = true;

// Arrays
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"];

// Objects
let user: { name: string; age: number } = {
  name: "Alice",
  age: 30,
};

// Union types
let id: string | number = "abc123";
id = 123; // Also valid

// Literal types
let status: "pending" | "approved" | "rejected" = "pending";
```

### Interfaces and Types

```typescript
// Interface
interface User {
  id: number;
  name: string;
  email: string;
  role?: string; // Optional
  readonly createdAt: Date; // Cannot be modified
}

// Type alias
type UserId = number | string;

// Extending interfaces
interface Employee extends User {
  department: string;
  salary: number;
}

// Intersection types
type AdminUser = User & { permissions: string[] };
```

### Functions

```typescript
// Function with types
function greet(name: string): string {
  return `Hello, ${name}`;
}

// Arrow function
const add = (a: number, b: number): number => a + b;

// Optional and default parameters
function createUser(name: string, role: string = "user", age?: number): User {
  return { name, role, age };
}

// Function type
type GreetFunction = (name: string) => string;
const sayHi: GreetFunction = (name) => `Hi, ${name}`;
```

### Generics

```typescript
// Generic function
function first<T>(items: T[]): T | undefined {
  return items[0];
}

const firstNumber = first([1, 2, 3]); // number
const firstString = first(["a", "b"]); // string

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Usage
interface User {
  id: number;
  name: string;
}

const response: ApiResponse<User> = {
  data: { id: 1, name: "Alice" },
  status: 200,
  message: "Success",
};

// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### Utility Types

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Partial - all properties optional
type UserUpdate = Partial<User>;

// Pick - select specific properties
type UserPreview = Pick<User, "id" | "name">;

// Omit - exclude specific properties
type UserResponse = Omit<User, "password">;

// Required - all properties required
type RequiredUser = Required<User>;

// Readonly - all properties readonly
type ReadonlyUser = Readonly<User>;

// Record - key-value mapping
type UserRoles = Record<string, string[]>;
```

---

## 6.4 TypeScript with APIs

```typescript
// API types
interface Job {
  id: number;
  title: string;
  company: string;
  salary_min: number | null;
  salary_max: number | null;
  is_remote: boolean;
  created_at: string;
}

interface JobListResponse {
  items: Job[];
  total: number;
  page: number;
  pages: number;
}

interface JobCreate {
  title: string;
  description: string;
  company_id: number;
  job_type: "full_time" | "part_time" | "contract";
  skills: string[];
}

// API client
async function fetchJobs(page = 1): Promise<JobListResponse> {
  const response = await fetch(`/api/jobs?page=${page}`);
  if (!response.ok) {
    throw new Error("Failed to fetch jobs");
  }
  return response.json();
}

async function createJob(job: JobCreate): Promise<Job> {
  const response = await fetch("/api/jobs", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(job),
  });
  if (!response.ok) {
    throw new Error("Failed to create job");
  }
  return response.json();
}
```

---

## Exercises

1. [Exercise 6.1: Modern JavaScript Practice](exercises/exercise-6.1.md)
2. [Exercise 6.2: TypeScript Types](exercises/exercise-6.2.md)
3. [Exercise 6.3: API Client](exercises/exercise-6.3.md)

## Assignment

[Assignment 6: TypeScript API Client](assignments/assignment-6.md)

## Quiz

[Module 6 Quiz](quiz/quiz-6.md)

---

## Summary

- ✅ Modern JavaScript: destructuring, spread, modules, async/await
- ✅ TypeScript basics: types, interfaces, functions
- ✅ Generics and utility types
- ✅ Type-safe API clients

**Next Module:** [Vue 3 Fundamentals](../07-vue-fundamentals/README.md)
