# TypeScript Functions - Day 2 Complete Guide 🚀

Let's dive deep into functions, arrow functions, and generics with **real-world, production-grade patterns**.

---

## Table of Contents
1. [Function Basics](#function-basics)
2. [Function Types & Signatures](#function-types--signatures)
3. [Arrow Functions](#arrow-functions)
4. [Optional & Default Parameters](#optional--default-parameters)
5. [Rest Parameters](#rest-parameters)
6. [Function Overloading](#function-overloading)
7. [Generics - Deep Dive](#generics---deep-dive)
8. [Generic Constraints](#generic-constraints)
9. [Advanced Generic Patterns](#advanced-generic-patterns)
10. [Real-World Examples](#real-world-examples)

---

## Function Basics

### Regular Function Declaration

```typescript
// ✅ Basic function
function greet(name: string): string {
  return `Hello, ${name}`;
}

// ✅ With multiple parameters
function add(a: number, b: number): number {
  return a + b;
}

// ✅ Void return type (no return value)
function logMessage(message: string): void {
  console.log(message);
  // No return statement
}

// ✅ Never return type (function never returns)
function throwError(message: string): never {
  throw new Error(message);
  // Never reaches end
}

function infiniteLoop(): never {
  while (true) {
    // Infinite loop
  }
}
```

**When to use each return type:**
- **`string`, `number`, etc.** - Function returns a value
- **`void`** - Function doesn't return anything (side effects only)
- **`never`** - Function never completes (throws error or infinite loop)

---

### Function Expression

```typescript
// ✅ Function expression
const greet = function(name: string): string {
  return `Hello, ${name}`;
};

// ✅ With type annotation
const add: (a: number, b: number) => number = function(a, b) {
  return a + b;
};
```

---

## Function Types & Signatures

### Defining Function Types

```typescript
// ✅ Function type
type GreetFunction = (name: string) => string;

const greet: GreetFunction = (name) => `Hello, ${name}`;

// ✅ Multiple parameters
type AddFunction = (a: number, b: number) => number;

const add: AddFunction = (a, b) => a + b;

// ✅ With optional parameters
type FormatFunction = (value: string, uppercase?: boolean) => string;

const format: FormatFunction = (value, uppercase = false) => {
  return uppercase ? value.toUpperCase() : value;
};
```

---

### Interface for Functions

```typescript
// ✅ Using interface (can add properties)
interface SearchFunction {
  (query: string, limit: number): string[];
  description: string;  // Additional property
}

const search: SearchFunction = (query, limit) => {
  return [`Result 1 for ${query}`, `Result 2 for ${query}`];
};

search.description = "Search function with limit";

console.log(search("typescript", 10));
console.log(search.description);
```

**When to use interface vs type for functions:**
- **Type:** Simple function signatures
- **Interface:** When you need to add properties to the function

---

### Callback Functions

```typescript
// ✅ Callback type definition
type Callback = (error: Error | null, result?: string) => void;

function fetchData(url: string, callback: Callback): void {
  // Simulate async operation
  setTimeout(() => {
    if (url === "") {
      callback(new Error("URL is empty"));
    } else {
      callback(null, "Data from " + url);
    }
  }, 1000);
}

// Usage
fetchData("https://api.example.com", (error, result) => {
  if (error) {
    console.error(error.message);
  } else {
    console.log(result);
  }
});
```

---

## Arrow Functions

### Basic Arrow Functions

```typescript
// ✅ Basic arrow function
const greet = (name: string): string => {
  return `Hello, ${name}`;
};

// ✅ Implicit return (one-liner)
const greet2 = (name: string): string => `Hello, ${name}`;

// ✅ No parameters
const sayHello = (): string => "Hello!";

// ✅ Single parameter (parentheses optional)
const double = (n: number): number => n * 2;
const double2 = n => n * 2;  // Type inferred

// ✅ Multiple parameters
const add = (a: number, b: number): number => a + b;

// ✅ Returning object (wrap in parentheses)
const createUser = (name: string, age: number) => ({
  name,
  age,
  createdAt: new Date()
});
```

---

### Arrow Functions vs Regular Functions

```typescript
// ❌ Regular function - 'this' is dynamic
function Counter() {
  this.count = 0;
  
  setInterval(function() {
    this.count++;  // ❌ 'this' is undefined or window
    console.log(this.count);
  }, 1000);
}

// ✅ Arrow function - 'this' is lexical (inherits from parent)
function Counter() {
  this.count = 0;
  
  setInterval(() => {
    this.count++;  // ✅ 'this' refers to Counter instance
    console.log(this.count);
  }, 1000);
}

// Real-world example: Event handlers in classes
class Button {
  label: string = "Click me";
  
  // ❌ Regular method - loses 'this' when passed as callback
  handleClick() {
    console.log(this.label);
  }
  
  // ✅ Arrow function - preserves 'this'
  handleClickArrow = () => {
    console.log(this.label);
  }
}

const button = new Button();

// With regular function
document.addEventListener('click', button.handleClick);  
// ❌ 'this' is undefined

// With arrow function
document.addEventListener('click', button.handleClickArrow);  
// ✅ 'this' is Button instance
```

**When to use arrow functions:**
- ✅ Callbacks (preserves `this`)
- ✅ Array methods (map, filter, etc.)
- ✅ Short inline functions
- ✅ Event handlers in classes
- ❌ Methods that need `arguments` object
- ❌ Constructors (arrow functions can't be constructors)

---

## Optional & Default Parameters

### Optional Parameters

```typescript
// ✅ Optional parameter (use ?)
function greet(name: string, greeting?: string): string {
  if (greeting) {
    return `${greeting}, ${name}`;
  }
  return `Hello, ${name}`;
}

greet("John");              // "Hello, John"
greet("John", "Hi");        // "Hi, John"

// ❌ Optional parameters must come after required ones
function invalid(name?: string, age: number) { }  // ❌ Error
function valid(age: number, name?: string) { }    // ✅ OK
```

---

### Default Parameters

```typescript
// ✅ Default parameters
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}`;
}

greet("John");              // "Hello, John"
greet("John", "Hi");        // "Hi, John"

// ✅ Default with complex values
function createUser(
  name: string,
  role: string = "user",
  isActive: boolean = true
) {
  return { name, role, isActive };
}

// ✅ Default with object
function fetchData(
  url: string,
  options: { method: string; headers: Record<string, string> } = {
    method: "GET",
    headers: {}
  }
) {
  console.log(`Fetching ${url} with ${options.method}`);
}
```

**Default vs Optional:**
```typescript
// Optional - can be undefined
function greet(name?: string): string {
  return `Hello, ${name || 'Guest'}`;  // Need to handle undefined
}

// Default - always has a value
function greet2(name: string = 'Guest'): string {
  return `Hello, ${name}`;  // name is always string
}
```

---

## Rest Parameters

### Basic Rest Parameters

```typescript
// ✅ Rest parameters (array of values)
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3);           // 6
sum(1, 2, 3, 4, 5);     // 15

// ✅ With other parameters (rest must be last)
function introduce(greeting: string, ...names: string[]): string {
  return `${greeting} ${names.join(", ")}`;
}

introduce("Hello", "John", "Jane", "Bob");
// "Hello John, Jane, Bob"

// ❌ Rest parameter must be last
function invalid(...numbers: number[], name: string) { }  // ❌ Error
```

---

### Rest with Tuples

```typescript
// ✅ Rest with specific types (tuple)
function logValues(first: string, ...rest: [number, boolean]): void {
  console.log(first, rest[0], rest[1]);
}

logValues("hello", 42, true);  // ✅ OK
logValues("hello", 42);        // ❌ Error: Expected 3 arguments
```

---

## Function Overloading

### Basic Overloading

```typescript
// ✅ Function overload signatures
function format(value: string): string;
function format(value: number): string;
function format(value: boolean): string;

// Implementation signature (must be compatible with all overloads)
function format(value: string | number | boolean): string {
  if (typeof value === 'string') {
    return value.toUpperCase();
  } else if (typeof value === 'number') {
    return value.toFixed(2);
  } else {
    return value ? 'Yes' : 'No';
  }
}

format("hello");    // "HELLO"
format(42.678);     // "42.68"
format(true);       // "Yes"
```

---

### Overloading with Different Return Types

```typescript
// ✅ Different return types based on input
function getValue(key: 'name'): string;
function getValue(key: 'age'): number;
function getValue(key: 'active'): boolean;
function getValue(key: string): string | number | boolean {
  const data: Record<string, string | number | boolean> = {
    name: 'John',
    age: 30,
    active: true
  };
  return data[key];
}

const name = getValue('name');      // Type: string
const age = getValue('age');        // Type: number
const active = getValue('active');  // Type: boolean
```

---

### Real-World Overloading Example

```typescript
// ✅ Database query overloading
interface User {
  id: string;
  name: string;
  email: string;
}

// Overload signatures
function findUser(id: string): Promise<User | null>;
function findUser(email: string, password: string): Promise<User | null>;
function findUser(criteria: { name: string }): Promise<User[]>;

// Implementation
function findUser(
  idOrEmailOrCriteria: string | { name: string },
  password?: string
): Promise<User | User[] | null> {
  if (typeof idOrEmailOrCriteria === 'string' && !password) {
    // Find by ID
    return db.users.findById(idOrEmailOrCriteria);
  } else if (typeof idOrEmailOrCriteria === 'string' && password) {
    // Find by email and password
    return db.users.findByCredentials(idOrEmailOrCriteria, password);
  } else {
    // Find by criteria
    return db.users.findMany(idOrEmailOrCriteria);
  }
}

// Usage - TypeScript knows the return type based on arguments
const user1 = await findUser('user_123');                    // User | null
const user2 = await findUser('john@example.com', 'pass123'); // User | null
const users = await findUser({ name: 'John' });              // User[]
```

---

## Generics - Deep Dive

### Why Generics?

```typescript
// ❌ Without generics - need separate functions
function wrapInArrayString(value: string): string[] {
  return [value];
}

function wrapInArrayNumber(value: number): number[] {
  return [value];
}

// ✅ With generics - one function for all types
function wrapInArray<T>(value: T): T[] {
  return [value];
}

const strings = wrapInArray<string>("hello");   // string[]
const numbers = wrapInArray<number>(42);        // number[]
const users = wrapInArray<User>({ id: "1" });   // User[]

// ✅ Type inference (TypeScript figures out T automatically)
const strings2 = wrapInArray("hello");  // string[] (inferred)
const numbers2 = wrapInArray(42);       // number[] (inferred)
```

---

### Multiple Type Parameters

```typescript
// ✅ Multiple generics
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result1 = pair<string, number>("age", 30);        // [string, number]
const result2 = pair<User, Product>(user, product);     // [User, Product]

// ✅ With inference
const result3 = pair("hello", 42);                      // [string, number]
```

---

### Generic Functions with Constraints

```typescript
// ✅ Constrain T to have specific properties
function getProperty<T extends { id: string }>(obj: T, key: keyof T): T[keyof T] {
  return obj[key];
}

const user = { id: "1", name: "John", age: 30 };
const id = getProperty(user, "id");      // string | number (union of all property types)
const name = getProperty(user, "name");

// ❌ Error - object doesn't have 'id' property
const invalid = getProperty({ name: "John" }, "name");
```

---

### Better Generic Property Access

```typescript
// ✅ More specific - get exact property type
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: "1", name: "John", age: 30 };

const id = getProperty(user, "id");      // Type: string
const name = getProperty(user, "name");  // Type: string
const age = getProperty(user, "age");    // Type: number

// ❌ Error - 'email' is not a key of user
const email = getProperty(user, "email");
```

---

### Generic Array Functions

```typescript
// ✅ Generic find function
function findInArray<T>(
  array: T[],
  predicate: (item: T) => boolean
): T | undefined {
  for (const item of array) {
    if (predicate(item)) {
      return item;
    }
  }
  return undefined;
}

interface User {
  id: string;
  name: string;
  age: number;
}

const users: User[] = [
  { id: "1", name: "John", age: 30 },
  { id: "2", name: "Jane", age: 25 }
];

const user = findInArray(users, u => u.name === "John");
// Type: User | undefined
```

---

## Generic Constraints

### Extending Types

```typescript
// ✅ Constrain to objects with specific shape
interface HasId {
  id: string;
}

function findById<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

const users = [
  { id: "1", name: "John" },
  { id: "2", name: "Jane" }
];

const user = findById(users, "1");  // ✅ Works

// ❌ Error - number[] doesn't extend HasId
const number = findById([1, 2, 3], "1");
```

---

### Constraining to Constructor

```typescript
// ✅ Constrain to classes/constructors
function create<T>(Constructor: new () => T): T {
  return new Constructor();
}

class User {
  name: string = "John";
}

class Product {
  price: number = 100;
}

const user = create(User);        // Type: User
const product = create(Product);  // Type: Product

// ❌ Error - string is not a constructor
const str = create(String);
```

---

### Multiple Constraints

```typescript
// ✅ Multiple constraints using &
interface HasId {
  id: string;
}

interface HasTimestamps {
  createdAt: Date;
  updatedAt: Date;
}

function logEntity<T extends HasId & HasTimestamps>(entity: T): void {
  console.log(`Entity ${entity.id} created at ${entity.createdAt}`);
}

const user = {
  id: "1",
  name: "John",
  createdAt: new Date(),
  updatedAt: new Date()
};

logEntity(user);  // ✅ Works

// ❌ Error - missing createdAt and updatedAt
logEntity({ id: "1", name: "John" });
```

---

## Advanced Generic Patterns

### Generic Type Inference

```typescript
// ✅ Inferring return type from callback
function map<T, U>(
  array: T[],
  transform: (item: T) => U
): U[] {
  return array.map(transform);
}

const numbers = [1, 2, 3];
const strings = map(numbers, n => n.toString());  // Type: string[]
const doubled = map(numbers, n => n * 2);         // Type: number[]

// TypeScript infers:
// - T = number (from numbers array)
// - U = string (from transform return type)
```

---

### Conditional Types in Functions

```typescript
// ✅ Return different types based on input
type ArrayOrSingle<T, Multiple extends boolean> = 
  Multiple extends true ? T[] : T;

function fetch<T, M extends boolean = false>(
  url: string,
  multiple?: M
): Promise<ArrayOrSingle<T, M>> {
  if (multiple) {
    return fetch(url).then(r => r.json()) as Promise<ArrayOrSingle<T, M>>;
  }
  return fetch(url).then(r => r.json()) as Promise<ArrayOrSingle<T, M>>;
}

// Usage
const user = await fetch<User>('/user/1');         // Promise<User>
const users = await fetch<User>('/users', true);   // Promise<User[]>
```

---

### Generic Factory Pattern

```typescript
// ✅ Generic factory
interface Entity {
  id: string;
  createdAt: Date;
}

function createEntity<T extends Omit<Entity, 'id' | 'createdAt'>>(
  data: T
): T & Entity {
  return {
    ...data,
    id: generateId(),
    createdAt: new Date()
  };
}

interface UserData {
  name: string;
  email: string;
}

const user = createEntity<UserData>({
  name: "John",
  email: "john@example.com"
});

// Type: UserData & Entity
// { name: string; email: string; id: string; createdAt: Date }
```

---

### Generic Promise Wrapper

```typescript
// ✅ Generic async wrapper with error handling
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

async function tryCatch<T>(
  promise: Promise<T>
): Promise<Result<T>> {
  try {
    const data = await promise;
    return { success: true, data };
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error : new Error(String(error))
    };
  }
}

// Usage
const result = await tryCatch(fetchUser("1"));

if (result.success) {
  console.log(result.data);  // Type: User
} else {
  console.error(result.error);  // Type: Error
}
```

---

## Real-World Examples

### Example 1: API Client with Generics

```typescript
// ✅ Generic API client
class ApiClient {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async get<T>(endpoint: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json() as Promise<T>;
  }

  async post<T, D = unknown>(endpoint: string, data: D): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json() as Promise<T>;
  }
}

// Usage
interface User {
  id: string;
  name: string;
  email: string;
}

interface CreateUserDto {
  name: string;
  email: string;
}

const api = new ApiClient('https://api.example.com');

// Type-safe API calls
const user = await api.get<User>('/users/1');           // Type: User
const users = await api.get<User[]>('/users');          // Type: User[]

const newUser = await api.post<User, CreateUserDto>('/users', {
  name: 'John',
  email: 'john@example.com'
});  // Type: User
```

---

### Example 2: Repository Pattern with Generics

```typescript
// ✅ Generic repository
interface BaseEntity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

class Repository<T extends BaseEntity> {
  constructor(private collectionName: string) {}

  async findById(id: string): Promise<T | null> {
    const doc = await db.collection(this.collectionName).findOne({ id });
    return doc as T | null;
  }

  async findMany(filter: Partial<T>): Promise<T[]> {
    const docs = await db.collection(this.collectionName).find(filter).toArray();
    return docs as T[];
  }

  async create(data: Omit<T, 'id' | 'createdAt' | 'updatedAt'>): Promise<T> {
    const entity: T = {
      ...data,
      id: generateId(),
      createdAt: new Date(),
      updatedAt: new Date()
    } as T;
    
    await db.collection(this.collectionName).insertOne(entity);
    return entity;
  }

  async update(id: string, data: Partial<Omit<T, 'id' | 'createdAt'>>): Promise<T | null> {
    const updated = {
      ...data,
      updatedAt: new Date()
    };
    
    await db.collection(this.collectionName).updateOne({ id }, { $set: updated });
    return this.findById(id);
  }

  async delete(id: string): Promise<boolean> {
    const result = await db.collection(this.collectionName).deleteOne({ id });
    return result.deletedCount === 1;
  }
}

// Usage
interface User extends BaseEntity {
  name: string;
  email: string;
  role: 'admin' | 'user';
}

interface Product extends BaseEntity {
  name: string;
  price: number;
  stock: number;
}

const userRepo = new Repository<User>('users');
const productRepo = new Repository<Product>('products');

// Type-safe operations
const user = await userRepo.findById('user_1');      // Type: User | null
const users = await userRepo.findMany({ role: 'admin' });  // Type: User[]

const newUser = await userRepo.create({
  name: 'John',
  email: 'john@example.com',
  role: 'user'
});  // Type: User

const product = await productRepo.create({
  name: 'Laptop',
  price: 1000,
  stock: 10
});  // Type: Product
```

---

### Example 3: Event Emitter with Generics

```typescript
// ✅ Type-safe event emitter
type EventMap = Record<string, any>;

class TypedEventEmitter<Events extends EventMap> {
  private listeners: {
    [K in keyof Events]?: Array<(data: Events[K]) => void>;
  } = {};

  on<K extends keyof Events>(
    event: K,
    listener: (data: Events[K]) => void
  ): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }

  emit<K extends keyof Events>(event: K, data: Events[K]): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach(listener => listener(data));
    }
  }

  off<K extends keyof Events>(
    event: K,
    listener: (data: Events[K]) => void
  ): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      this.listeners[event] = eventListeners.filter(l => l !== listener) as any;
    }
  }
}

// Usage
interface AppEvents {
  'user:login': { userId: string; timestamp: Date };
  'user:logout': { userId: string };
  'order:created': { orderId: string; total: number };
  'order:cancelled': { orderId: string; reason: string };
}

const emitter = new TypedEventEmitter<AppEvents>();

// ✅ Type-safe event handling
emitter.on('user:login', (data) => {
  console.log(`User ${data.userId} logged in at ${data.timestamp}`);
  // data is typed as { userId: string; timestamp: Date }
});

emitter.on('order:created', (data) => {
  console.log(`Order ${data.orderId} created with total ${data.total}`);
  // data is typed as { orderId: string; total: number }
});

// ✅ Type-safe emitting
emitter.emit('user:login', {
  userId: 'user_123',
  timestamp: new Date()
});

// ❌ Error - wrong data type
emitter.emit('user:login', { userId: 123 });  // userId should be string

// ❌ Error - wrong event name
emitter.emit('user:signin', { userId: 'user_123' });  // Event doesn't exist
```

---

### Example 4: Validator with Generics

```typescript
// ✅ Generic validator
type ValidationRule<T> = (value: T) => string | null;

class Validator<T> {
  private rules: Array<ValidationRule<T>> = [];

  addRule(rule: ValidationRule<T>): this {
    this.rules.push(rule);
    return this;
  }

  validate(value: T): string[] {
    const errors: string[] = [];
    for (const rule of this.rules) {
      const error = rule(value);
      if (error) {
        errors.push(error);
      }
    }
    return errors;
  }

  isValid(value: T): boolean {
    return this.validate(value).length === 0;
  }
}

// Usage
const emailValidator = new Validator<string>()
  .addRule(value => value.length === 0 ? 'Email is required' : null)
  .addRule(value => !value.includes('@') ? 'Invalid email format' : null)
  .addRule(value => value.length > 100 ? 'Email is too long' : null);

const errors = emailValidator.validate('invalid-email');
// ["Invalid email format"]

const isValid = emailValidator.isValid('john@example.com');
// true

// Number validator
const ageValidator = new Validator<number>()
  .addRule(value => value < 0 ? 'Age cannot be negative' : null)
  .addRule(value => value > 150 ? 'Age is too high' : null);

const ageErrors = ageValidator.validate(200);
// ["Age is too high"]
```

---

### Example 5: Query Builder with Generics

```typescript
// ✅ Type-safe query builder
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
  role: 'admin' | 'user';
}

class QueryBuilder<T> {
  private filters: Partial<T> = {};
  private selectFields?: (keyof T)[];
  private limitValue?: number;

  where(filter: Partial<T>): this {
    this.filters = { ...this.filters, ...filter };
    return this;
  }

  select(...fields: (keyof T)[]): this {
    this.selectFields = fields;
    return this;
  }

  limit(count: number): this {
    this.limitValue = count;
    return this;
  }

  async execute(): Promise<T[]> {
    // Simulate database query
    console.log('Filters:', this.filters);
    console.log('Select:', this.selectFields);
    console.log('Limit:', this.limitValue);
    
    // In real app, execute actual database query
    return [] as T[];
  }
}

// Usage
const query = new QueryBuilder<User>();

const users = await query
  .where({ role: 'admin' })
  .where({ age: 30 })
  .select('id', 'name', 'email')
  .limit(10)
  .execute();

// ✅ Type-safe - only User properties allowed
const invalidQuery = query
  .where({ invalidField: 'value' });  // ❌ Error

const invalidSelect = query
  .select('id', 'invalidField');  // ❌ Error
```

---

## Best Practices Summary

### Function Best Practices

1. **Always type parameters and return values explicitly**
   ```typescript
   // ✅ Good
   function add(a: number, b: number): number {
     return a + b;
   }
   
   // ❌ Bad - relying on inference
   function add(a, b) {
     return a + b;
   }
   ```

2. **Use arrow functions for callbacks and preserving `this`**
   ```typescript
   // ✅ Good
   array.map(item => item.name);
   
   class MyClass {
     handleClick = () => {
       // 'this' is preserved
     }
   }
   ```

3. **Use function overloading for multiple signatures**
   ```typescript
   // ✅ Good - clear different behaviors
   function format(value: string): string;
   function format(value: number): string;
   function format(value: string | number): string { }
   ```

4. **Prefer optional parameters over undefined unions**
   ```typescript
   // ✅ Good
   function greet(name?: string) { }
   
   // ❌ Bad - more verbose
   function greet(name: string | undefined) { }
   ```

---

### Generic Best Practices

1. **Use meaningful generic names for clarity**
   ```typescript
   // ✅ Good - descriptive names
   function map<TInput, TOutput>(items: TInput[], transform: (item: TInput) => TOutput): TOutput[]
   
   // ❌ Bad - single letters for complex functions
   function map<T, U>(items: T[], fn: (item: T) => U): U[]
   ```

2. **Constrain generics when possible**
   ```typescript
   // ✅ Good - constraints prevent misuse
   function getId<T extends { id: string }>(obj: T): string {
     return obj.id;
   }
   
   // ❌ Bad - too permissive
   function getId<T>(obj: T): any {
     return (obj as any).id;
   }
   ```

3. **Let TypeScript infer when possible**
   ```typescript
   // ✅ Good - inference works
   const result = wrapInArray("hello");  // string[]
   
   // ❌ Bad - unnecessary explicit type
   const result = wrapInArray<string>("hello");
   ```

4. **Use default generic types for common cases**
   ```typescript
   // ✅ Good - default makes common case easier
   function fetch<T, E = Error>(url: string): Promise<T> { }
   
   // Usage
   const data = await fetch<User>('/api/user');  // E defaults to Error
   ```

---

