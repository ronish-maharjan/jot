# TypeScript Interfaces & Branded Types - Professional Guide

## Table of Contents
1. [Interface Best Practices](#interface-best-practices)
2. [Composition Over Duplication](#composition-over-duplication)
3. [Utility Types](#utility-types)
4. [Readonly Properties](#readonly-properties)
5. [Branded Types (Nominal Typing)](#branded-types-nominal-typing)
6. [Real-World Patterns](#real-world-patterns)

---

## Interface Best Practices

### Naming Convention
```typescript
// ✅ GOOD - PascalCase, descriptive (NO 'I' prefix in modern TS)
interface User {
  id: string;
  name: string;
}

interface UserProfile {
  userId: string;
  bio: string;
}

// ❌ BAD - camelCase, vague, 'I' prefix (outdated)
interface IUser { }
interface user { }
```

**Reason:** Modern TypeScript doesn't use 'I' prefix. It's a C# convention that fell out of favor.

---

### Structure & Organization
```typescript
// ✅ GOOD - Clear, explicit types, required fields first
interface User {
  id: string;                    // Required fields
  email: string;
  name: string;
  age?: number;                  // Optional fields last
  createdAt?: Date;
  role: 'admin' | 'user';        // Union types for specific values
}

// ❌ BAD - Using 'any', unclear structure
interface User {
  data: any;
  info?: any;
}
```

**Reason:** Explicit types catch bugs at compile time. Optional fields at the end improve readability.

---

### File Organization (Enterprise Pattern)
```typescript
// types/user.types.ts
export interface User {
  id: string;
  email: string;
  name: string;
}

export interface CreateUserDto {
  email: string;
  name: string;
  password: string;
}

export interface UpdateUserDto {
  name?: string;
  email?: string;
}

export interface UserResponse {
  user: User;
  token: string;
}
```

**Reason:** Separation of concerns. Related types grouped together, exported for reuse.

---

### Interface vs Type - When to Use What

```typescript
// ✅ Use INTERFACE for object shapes (preferred in enterprise)
interface User {
  id: string;
  name: string;
}

// Can be extended
interface Admin extends User {
  permissions: string[];
}

// ✅ Use TYPE for unions, intersections, primitives
type Status = 'pending' | 'approved' | 'rejected';
type ID = string | number;
type UserWithStatus = User & { status: Status };
```

**Reason:**
- **Interface:** Better error messages, can be extended/merged, works with classes
- **Type:** More flexible for unions, intersections, computed properties

**Gotcha:** Interfaces can be declaration merged (useful for extending third-party types), types cannot.

---

## Composition Over Duplication

### The Problem (Duplication)
```typescript
// ❌ BAD - Repeating same fields everywhere
interface User {
  id: string;
  createdAt: Date;
  updatedAt: Date;
  email: string;
  name: string;
}

interface Product {
  id: string;
  createdAt: Date;
  updatedAt: Date;
  name: string;
  price: number;
}

interface Order {
  id: string;
  createdAt: Date;
  updatedAt: Date;
  userId: string;
  total: number;
}
```

**Problem:** If you change timestamp type, you change 3+ places. Hard to maintain.

---

### Solution 1: Extends (MOST COMMON - 70% usage)
```typescript
// ✅ GOOD - Extract common fields to base
interface BaseEntity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

interface User extends BaseEntity {
  email: string;
  name: string;
}

interface Product extends BaseEntity {
  name: string;
  price: number;
}

interface Order extends BaseEntity {
  userId: string;
  total: number;
}
```

**Reason:**
- ✅ Clean & readable
- ✅ Clear inheritance chain
- ✅ IDE autocomplete works perfectly
- ✅ Easy for junior devs to understand

**When to use:** Shared fields across multiple entities (id, timestamps, audit fields)

---

### Solution 2: Intersection Types (10% usage)
```typescript
// ✅ GOOD - For mixing multiple concerns
interface Timestamps {
  createdAt: Date;
  updatedAt: Date;
}

interface Identifiable {
  id: string;
}

interface SoftDeletable {
  deletedAt?: Date;
}

// Mix and match
type User = Identifiable & Timestamps & {
  email: string;
  name: string;
}

type Product = Identifiable & Timestamps & SoftDeletable & {
  name: string;
  price: number;
}
```

**Reason:** Flexible composition when entities need different combinations of traits.

**When to use:** Complex compositions, mixing unrelated concerns

**Gotcha:** Error messages can be confusing. Harder for teams to read.

---

### Decision Tree

```
Do multiple interfaces share EXACT same fields?
│
├─ YES → Use `extends BaseEntity`
│   └─ Example: id, createdAt, updatedAt
│
└─ NO → Do you need variations of the same interface?
    │
    ├─ YES → Use Utility Types (Omit, Pick, Partial)
    │   └─ Example: CreateUserDto, UpdateUserDto
    │
    └─ NO → Do you need to mix unrelated interfaces?
        │
        └─ YES → Use intersection (&)
            └─ Example: type Admin = User & Permissions
```

---

## Utility Types

### Omit - Remove Properties (20% usage)
```typescript
interface User {
  id: string;
  email: string;
  password: string;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

// ✅ Return to frontend - remove password
type UserResponse = Omit<User, 'password'>;
// Result: { id, email, name, createdAt, updatedAt }

// ✅ Create user - remove auto-generated fields
type CreateUserDto = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
// Result: { email, password, name }
```

**Reason:** Single source of truth. If User changes, DTOs update automatically.

**When to use:** API responses, DTOs, derived types

---

### Pick - Select Properties
```typescript
interface User {
  id: string;
  email: string;
  password: string;
  name: string;
  role: string;
}

// ✅ Only need specific fields
type UserPreview = Pick<User, 'id' | 'name'>;
// Result: { id, name }

type LoginDto = Pick<User, 'email' | 'password'>;
// Result: { email, password }
```

**When to use:** Need only a subset of properties

---

### Partial - Make All Optional
```typescript
interface User {
  id: string;
  email: string;
  name: string;
}

// ✅ Update - all fields optional except id
type UpdateUserDto = Partial<Omit<User, 'id'>> & { id: string };
// Result: { id: string; email?: string; name?: string }
```

**When to use:** Update operations, optional configurations

---

### Required - Make All Required
```typescript
interface Config {
  apiUrl?: string;
  timeout?: number;
  retries?: number;
}

// ✅ Validated config - all required
type ValidatedConfig = Required<Config>;
// Result: { apiUrl: string; timeout: number; retries: number }
```

**When to use:** Ensuring all fields are set after validation

---

### Readonly - Make All Readonly
```typescript
interface MutableUser {
  id: string;
  name: string;
}

// ✅ Immutable version
type ImmutableUser = Readonly<MutableUser>;
// Result: { readonly id: string; readonly name: string }
```

**When to use:** Immutable data structures, preventing accidental mutations

---

### Combining Utility Types (Real-World Pattern)
```typescript
interface User {
  id: string;
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

// Create - no auto-generated fields, no isActive
export type CreateUserDto = Omit<User, 'id' | 'createdAt' | 'updatedAt' | 'isActive'>;
// { email, password, firstName, lastName }

// Update - id required, other fields optional
export type UpdateUserDto = { id: string } & Partial<Pick<User, 'firstName' | 'lastName'>>;
// { id: string; firstName?: string; lastName?: string }

// Response - no password
export type UserResponse = Omit<User, 'password'>;
// Everything except password

// List item - only preview fields
export type UserListItem = Pick<User, 'id' | 'email' | 'firstName' | 'lastName'>;
// { id, email, firstName, lastName }
```

**Gotcha:** Overly complex utility type chains are hard to debug. Keep them simple.

---

## Readonly Properties

### Basic Usage
```typescript
interface User {
  readonly id: string;           // Cannot change after creation
  readonly email: string;        // Cannot change
  name: string;                  // Can change
}

const user: User = {
  id: '123',
  email: 'john@example.com',
  name: 'John'
};

// ✅ ALLOWED
user.name = 'John Doe';

// ❌ ERROR - Cannot assign to 'id' because it is a read-only property
user.id = '456';
user.email = 'new@example.com';
```

**Reason:** Prevents accidental mutations of immutable data.

---

### When to Use Readonly

```typescript
interface BaseEntity {
  readonly id: string;           // IDs NEVER change
  readonly createdAt: Date;      // Creation time NEVER changes
  updatedAt: Date;               // Update time CAN change
}

interface Config {
  readonly apiUrl: string;       // Config values NEVER change at runtime
  readonly apiKey: string;
  readonly timeout: number;
}

interface Transaction {
  readonly transactionId: string;  // Financial records are immutable
  readonly amount: number;
  readonly timestamp: Date;
  status: 'pending' | 'completed';  // Only status can change
}
```

**Always readonly:**
- ✅ IDs (database primary keys)
- ✅ `createdAt` timestamps
- ✅ Configuration values
- ✅ Financial/audit data
- ✅ Unique identifiers (SKU, order numbers)

**Never readonly:**
- ❌ User-editable fields (name, email, address)
- ❌ Status fields (status, isActive)
- ❌ Counters (stock, views, likes)
- ❌ `updatedAt` timestamps

---

### Readonly Arrays
```typescript
interface Product {
  readonly tags: readonly string[];  // Array AND elements are readonly
}

const product: Product = {
  tags: ['electronics', 'laptop']
};

// ❌ Cannot modify
product.tags.push('gaming');
product.tags[0] = 'computers';
product.tags = ['new'];

// ✅ Can only read
console.log(product.tags[0]); // 'electronics'
```

**Gotcha:** Regular `readonly` only applies to first level. Nested objects are mutable unless using `DeepReadonly<T>`.

---

### Deep Readonly (Advanced)
```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: DeepReadonly<T[P]>;
};

interface User {
  id: string;
  profile: {
    name: string;
    age: number;
  };
}

type ImmutableUser = DeepReadonly<User>;

const user: ImmutableUser = {
  id: '123',
  profile: { name: 'John', age: 30 }
};

// ❌ Everything is readonly
user.id = '456';
user.profile.name = 'Jane';
user.profile.age = 31;
```

---

### Real-World Example: Order System
```typescript
interface Order {
  // Immutable - set once, never change
  readonly orderId: string;
  readonly userId: string;
  readonly createdAt: Date;
  readonly items: readonly OrderItem[];
  readonly totalAmount: number;
  
  // Mutable - can change
  status: 'pending' | 'processing' | 'shipped' | 'delivered';
  trackingNumber?: string;
  updatedAt: Date;
}

interface OrderItem {
  readonly productId: string;
  readonly productName: string;    // Snapshot at order time
  readonly price: number;          // Price at order time
  readonly quantity: number;
}

const order: Order = {
  orderId: 'ORD_12345',
  userId: 'user_789',
  createdAt: new Date(),
  items: [
    { productId: 'p1', productName: 'Laptop', price: 1500, quantity: 1 }
  ],
  totalAmount: 1500,
  status: 'pending',
  updatedAt: new Date()
};

// ✅ Status can update
order.status = 'shipped';
order.trackingNumber = 'TRACK123';

// ❌ Financial data cannot change (prevents fraud)
order.totalAmount = 2000;           // Error
order.items[0].price = 1000;        // Error
```

**Reason:** Historical records (orders, invoices, transactions) must be immutable for legal/audit purposes.

---

## Branded Types (Nominal Typing)

### The Problem - Structural Typing
```typescript
type UserId = string;
type ProductId = string;

function getUser(id: UserId) { }
function getProduct(id: ProductId) { }

const userId: UserId = "user_123";
const productId: ProductId = "prod_456";

// ❌ BUG - TypeScript allows this! Both are strings
getUser(productId);  // No error, but logically WRONG
```

**Problem:** TypeScript uses structural typing (duck typing). All strings are compatible.

---

### Solution - Branding
```typescript
// ✅ Brand types to make them incompatible
type Brand<T, Name> = T & { readonly __brand: Name };

type UserId = Brand<string, "UserId">;
type ProductId = Brand<string, "ProductId">;

function getUser(id: UserId) { }
function getProduct(id: ProductId) { }

const userId = "user_123" as UserId;
const productId = "prod_456" as ProductId;

// ✅ Works
getUser(userId);

// ❌ ERROR - Type 'ProductId' is not assignable to type 'UserId'
getUser(productId);  // TypeScript catches the bug!
```

**Reason:** Creates nominal typing (type by name, not structure). Prevents wrong ID type bugs.

---

### Generic vs Type - Understanding Brand

```typescript
// Brand is a GENERIC TYPE (not a function)
type Brand<T, Name> = T & { readonly __brand: Name };
//        ↑   ↑
//    parameters (like function parameters)

// Fill in the parameters to create a NEW TYPE
type UserId = Brand<string, "UserId">;
//   ↑         ↑      ↑       ↑
//  new      template  T     Name
//  type

// What TypeScript does internally:
// type UserId = string & { readonly __brand: "UserId" };
```

**Key concept:** Generic types are templates. You fill them in to create specific types.

**Gotcha:** You cannot use `Brand<string, "UserId">` directly as a variable type. Create a type alias first.

---

### Factory Functions (RECOMMENDED - Industry Standard)
```typescript
type Brand<T, Name> = T & { readonly __brand: Name };

type UserId = Brand<string, "UserId">;
type ProductId = Brand<string, "ProductId">;
type Email = Brand<string, "Email">;

// ✅ Create factory functions
export const userId = (id: string): UserId => id as UserId;
export const productId = (id: string): ProductId => id as ProductId;

// ✅ With validation
export const email = (value: string): Email => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(value)) {
    throw new Error('Invalid email format');
  }
  return value as Email;
};

// Usage
const uid = userId("user_123");
const pid = productId("prod_456");
const mail = email("john@example.com");

function deleteUser(id: UserId) { }

deleteUser(uid);   // ✅ Works
deleteUser(pid);   // ❌ Error - ProductId not UserId
deleteUser("user_123");  // ❌ Error - string not UserId
```

**Reason:**
- ✅ Clean API
- ✅ Centralized validation
- ✅ Easy to refactor
- ✅ Used by Stripe, Shopify, Airbnb

---

### Safe Factory (No Exceptions)
```typescript
type Result<T, E = Error> = 
  | { success: true; value: T }
  | { success: false; error: E };

function createEmail(value: string): Result<Email> {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(value)) {
    return {
      success: false,
      error: new Error('Invalid email')
    };
  }
  return {
    success: true,
    value: value as Email
  };
}

// Usage
const result = createEmail("john@example.com");

if (result.success) {
  const email = result.value; // Type: Email
  sendEmail(email);
} else {
  console.error(result.error.message);
}
```

**When to use:** Functions that shouldn't throw (API handlers, user input processing)

---

### Real-World Example - Banking System
```typescript
// ===== types/branded.ts =====
type Brand<T, Name> = T & { readonly __brand: Name };

export type AccountId = Brand<string, "AccountId">;
export type TransactionId = Brand<string, "TransactionId">;
export type USD = Brand<number, "USD">;
export type EUR = Brand<number, "EUR">;

// ===== utils/factories.ts =====
export const accountId = (id: string): AccountId => id as AccountId;
export const transactionId = (id: string): TransactionId => id as TransactionId;
export const usd = (amount: number): USD => {
  if (amount < 0) throw new Error('Amount cannot be negative');
  return amount as USD;
};
export const eur = (amount: number): EUR => {
  if (amount < 0) throw new Error('Amount cannot be negative');
  return amount as EUR;
};

// ===== services/transaction.service.ts =====
function transferMoney(
  from: AccountId,
  to: AccountId,
  amount: USD
) {
  console.log(`Transfer ${amount} from ${from} to ${to}`);
}

function convertCurrency(amount: USD): EUR {
  return eur(amount * 0.92);
}

// ===== Usage =====
const account1 = accountId("ACC_001");
const account2 = accountId("ACC_002");
const txn = transactionId("TXN_123");

const dollars = usd(100);
const euros = eur(92);

// ✅ Works
transferMoney(account1, account2, dollars);

// ❌ Errors - caught at compile time
transferMoney(txn, account2, dollars);        // TransactionId not AccountId
transferMoney(account1, account2, euros);     // EUR not USD
transferMoney(account1, account2, 100);       // number not USD
```

**Reason:** Prevents catastrophic bugs in financial systems (wrong currency, wrong account type).

---

### When to Use Branded Types

**✅ Always use:**
- IDs (`UserId`, `ProductId`, `OrderId`)
- Currencies (`USD`, `EUR`, `BTC`)
- Sensitive data (`SSN`, `CreditCard`, `Email`)
- Units (`Meters`, `Feet`, `Kilograms`)
- Financial systems (any money/account operations)

**❌ Don't use:**
- Simple CRUD apps
- Prototypes/MVPs
- When team is unfamiliar with pattern (training cost)

**Companies using this:**
- Stripe (Payment IDs, Customer IDs)
- Airbnb (Listing IDs, Booking IDs)
- Uber (Driver IDs, Trip IDs)
- All major banks and fintech

---

### Gotchas

1. **Runtime vs Compile-time:**
   ```typescript
   type UserId = Brand<string, "UserId">;
   const id = "user_123" as UserId;
   
   console.log(typeof id); // "string" (brands are erased at runtime)
   ```
   **Gotcha:** Brands only exist at compile-time. At runtime, it's just a string.

2. **Deep nesting:**
   ```typescript
   type UserId = Brand<string, "UserId">;
   
   interface User {
     id: UserId;
     friends: UserId[];  // Array of branded types
   }
   
   const user: User = {
     id: userId("user_1"),
     friends: ["user_2", "user_3"]  // ❌ Error - need to brand each
   };
   
   // ✅ Correct
   const user: User = {
     id: userId("user_1"),
     friends: ["user_2", "user_3"].map(userId)
   };
   ```

3. **JSON serialization:**
   ```typescript
   const id = userId("user_123");
   const json = JSON.stringify({ id });
   // json = '{"id":"user_123"}' (brand is lost)
   
   const parsed = JSON.parse(json);
   // parsed.id is 'string', not UserId
   // Need to re-brand after parsing
   const typedId = userId(parsed.id);
   ```

---

## Real-World Patterns

### Complete Enterprise Example
```typescript
// ===== shared/types/base.ts =====
export interface BaseEntity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

// ===== shared/types/branded.ts =====
type Brand<T, Name> = T & { readonly __brand: Name };

export type UserId = Brand<string, "UserId">;
export type ProductId = Brand<string, "ProductId">;
export type OrderId = Brand<string, "OrderId">;
export type Email = Brand<string, "Email">;

// ===== shared/utils/factories.ts =====
export const userId = (id: string): UserId => id as UserId;
export const productId = (id: string): ProductId => id as ProductId;
export const orderId = (id: string): OrderId => id as OrderId;

export const email = (value: string): Email => {
  if (!value.includes('@')) throw new Error('Invalid email');
  return value as Email;
};

// ===== modules/user/interfaces/user.interface.ts =====
import { BaseEntity } from '@/shared/types/base';
import { UserId, Email } from '@/shared/types/branded';

export interface User extends BaseEntity {
  id: UserId;  // Override with branded type
  email: Email;
  password: string;
  firstName: string;
  lastName: string;
  role: 'admin' | 'user' | 'guest';
  isActive: boolean;
}

export type CreateUserDto = Omit<User, 'id' | 'createdAt' | 'updatedAt' | 'isActive'>;
export type UpdateUserDto = { id: UserId } & Partial<Pick<User, 'firstName' | 'lastName'>>;
export type UserResponse = Omit<User, 'password'>;
export type UserListItem = Pick<User, 'id' | 'email' | 'firstName' | 'lastName' | 'role'>;

// ===== modules/user/user.service.ts =====
import { UserId, Email, userId, email } from '@/shared';
import { User, CreateUserDto, UpdateUserDto, UserResponse } from './interfaces';

export class UserService {
  async create(dto: CreateUserDto): Promise<UserResponse> {
    const id = userId(`user_${Date.now()}`);
    const validatedEmail = email(dto.email);
    
    const user: User = {
      id,
      email: validatedEmail,
      password: await hash(dto.password),
      firstName: dto.firstName,
      lastName: dto.lastName,
      role: dto.role,
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date()
    };
    
    await db.users.insert(user);
    
    return this.toResponse(user);
  }

  async findById(id: UserId): Promise<UserResponse | null> {
    const user = await db.users.findOne({ id });
    return user ? this.toResponse(user) : null;
  }

  async update(dto: UpdateUserDto): Promise<UserResponse> {
    const user = await db.users.findOne({ id: dto.id });
    if (!user) throw new Error('User not found');
    
    const updated = {
      ...user,
      ...dto,
      updatedAt: new Date()
    };
    
    await db.users.update(updated);
    return this.toResponse(updated);
  }

  private toResponse(user: User): UserResponse {
    const { password, ...response } = user;
    return response;
  }
}

// ===== Usage in controller =====
const userService = new UserService();

// ✅ Type-safe creation
const newUser = await userService.create({
  email: 'john@example.com',
  password: 'secret',
  firstName: 'John',
  lastName: 'Doe',
  role: 'user'
});

// ✅ Type-safe retrieval
const foundUser = await userService.findById(userId('user_123'));

// ❌ Compile-time errors
await userService.findById('user_123');           // string not UserId
await userService.findById(productId('prod_1'));  // ProductId not UserId
```

---

### Pattern Summary

| Pattern | Usage | When to Use |
|---------|-------|-------------|
| **`extends`** | 70% | Base entities, shared fields |
| **Utility types** | 20% | DTOs, API responses |
| **`&` intersection** | 10% | Complex compositions |
| **`readonly`** | Always | IDs, timestamps, immutable data |
| **Branded types** | Critical systems | IDs, currencies, sensitive data |

---

### Key Principles

1. **Single Source of Truth:** Define once, derive variations with utility types
2. **Type Safety:** Use brands to prevent wrong type bugs
3. **Immutability:** Use `readonly` for data that shouldn't change
4. **Composition:** Extend/compose instead of duplicate
5. **Validation:** Factory functions for branded types should validate
6. **Clarity:** Code is read more than written - prioritize readability

---

### Common Gotchas Checklist

- [ ] Don't use 'I' prefix for interfaces (outdated)
- [ ] Don't use `any` (defeats type safety)
- [ ] Don't duplicate fields across interfaces (use `extends` or utilities)
- [ ] Don't make everything `readonly` (only truly immutable data)
- [ ] Don't use branded types everywhere (adds complexity)
- [ ] Don't forget to re-brand after JSON.parse()
- [ ] Don't use complex utility type chains (hard to debug)
- [ ] Regular `readonly` only affects first level (use DeepReadonly for nested)
- [ ] Brands are compile-time only (erased at runtime)
- [ ] Generic types need type aliases (can't use `Brand<string, "X">` directly)

---

### Quick Reference Commands

```typescript
// Create base entity
interface BaseEntity { id: string; createdAt: Date; updatedAt: Date; }

// Extend
interface User extends BaseEntity { name: string; }

// Omit fields
type UserResponse = Omit<User, 'password'>;

// Pick fields
type UserPreview = Pick<User, 'id' | 'name'>;

// Make optional
type UpdateDto = Partial<User>;

// Make readonly
type ImmutableUser = Readonly<User>;

// Brand type
type UserId = Brand<string, "UserId">;
const userId = (id: string): UserId => id as UserId;
```

---

**Last Updated:** 2024  
**TypeScript Version:** 5.x  
**Pattern Source:** FAANG companies, enterprise production codebases

---
```
