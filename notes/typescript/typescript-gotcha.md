# TypeScript Gotchas - Complete Reference

## Table of Contents
1. [Core TypeScript Gotchas](#core-typescript-gotchas)
2. [Interface & Type Gotchas](#interface--type-gotchas)
3. [Generic Gotchas](#generic-gotchas)
4. [Readonly Gotchas](#readonly-gotchas)
5. [Branded Types Gotchas](#branded-types-gotchas)
6. [Utility Types Gotchas](#utility-types-gotchas)
7. [Runtime vs Compile-time Gotchas](#runtime-vs-compile-time-gotchas)
8. [Common Mistakes](#common-mistakes)

---

## Core TypeScript Gotchas

### 1. TypeScript Sees Types, Not Values
```typescript
// ❌ GOTCHA - TypeScript doesn't care about actual values
type UserId = string;
type ProductId = string;

const userId: UserId = "user_123";
const productId: ProductId = "prod_456";

function getUser(id: UserId) { }

// ❌ NO ERROR - Both are strings, TypeScript allows this!
getUser(productId);  // BUG! Wrong ID type but compiles fine
```

**Why:** TypeScript uses **structural typing** (duck typing), not nominal typing.  
**Fix:** Use branded types for distinct IDs.

```typescript
// ✅ FIX - Use branding
type UserId = string & { readonly __brand: "UserId" };
type ProductId = string & { readonly __brand: "ProductId" };

const userId = "user_123" as UserId;
const productId = "prod_456" as ProductId;

getUser(productId);  // ✅ Now gives error!
```

---

### 2. Type Inference Can Be Too Smart
```typescript
// ❌ GOTCHA - TypeScript infers literal types
const user = {
  id: "123",
  role: "admin"  // Type: "admin" (literal), not string
};

user.role = "user";  // ❌ Error: Type '"user"' is not assignable to type '"admin"'
```

**Why:** TypeScript infers the most specific type possible.

**Fix:**
```typescript
// ✅ Option 1: Explicit type
const user: { id: string; role: string } = {
  id: "123",
  role: "admin"
};

// ✅ Option 2: Type assertion
const user = {
  id: "123",
  role: "admin" as string
};

// ✅ Option 3: Interface
interface User {
  id: string;
  role: string;
}

const user: User = {
  id: "123",
  role: "admin"
};
```

---

### 3. `any` Breaks Everything
```typescript
// ❌ GOTCHA - any disables type checking
function processUser(user: any) {
  console.log(user.name.toUpperCase());  // No error!
}

processUser({ age: 30 });  // ❌ Runtime error: Cannot read property 'name' of undefined
processUser(null);         // ❌ Runtime error
processUser(123);          // ❌ Runtime error
```

**Why:** `any` opts out of type checking entirely.

**Fix:**
```typescript
// ✅ Use proper types
interface User {
  name: string;
}

function processUser(user: User) {
  console.log(user.name.toUpperCase());
}

processUser({ age: 30 });  // ✅ Compile error caught!

// ✅ If type is truly unknown, use 'unknown'
function processData(data: unknown) {
  if (typeof data === 'object' && data !== null && 'name' in data) {
    console.log((data as { name: string }).name);
  }
}
```

---

### 4. Null and Undefined Are Different
```typescript
// ❌ GOTCHA - null !== undefined in TypeScript
function greet(name: string | null) {
  console.log(name.toUpperCase());
}

greet(undefined);  // ❌ Error: Argument of type 'undefined' is not assignable to parameter of type 'string | null'
```

**Why:** TypeScript treats `null` and `undefined` as distinct types.

**Fix:**
```typescript
// ✅ Include both if needed
function greet(name: string | null | undefined) {
  if (name) {
    console.log(name.toUpperCase());
  }
}

// ✅ Or use optional parameter
function greet(name?: string) {  // Same as string | undefined
  if (name) {
    console.log(name.toUpperCase());
  }
}
```

---

### 5. Array Methods Don't Narrow Types
```typescript
// ❌ GOTCHA - filter doesn't narrow type
const items: (string | number)[] = ["a", 1, "b", 2];

const strings = items.filter(item => typeof item === "string");
// Type is still (string | number)[], not string[]!

strings[0].toUpperCase();  // ❌ Error: Property 'toUpperCase' does not exist on type 'string | number'
```

**Why:** TypeScript can't track what array methods do to element types.

**Fix:**
```typescript
// ✅ Use type predicate
function isString(value: string | number): value is string {
  return typeof value === "string";
}

const strings = items.filter(isString);  // Now type is string[]
strings[0].toUpperCase();  // ✅ Works!

// ✅ Or type assertion
const strings = items.filter(item => typeof item === "string") as string[];
```

---

## Interface & Type Gotchas

### 6. Interface vs Type - Declaration Merging
```typescript
// ❌ GOTCHA - Interfaces can be merged, types cannot
interface User {
  name: string;
}

interface User {
  age: number;
}

// Result: User has both name and age (merged!)
const user: User = {
  name: "John",
  age: 30
};

// ❌ Types cannot be merged
type Product = {
  name: string;
};

type Product = {  // ❌ Error: Duplicate identifier 'Product'
  price: number;
};
```

**Why:** Interfaces support declaration merging (useful for extending third-party types).

**When to use:**
- **Interface:** When you might need to extend/merge later
- **Type:** When you want to prevent accidental merging

---

### 7. Interface Extends Multiple, Type Uses Intersection
```typescript
// ✅ Interface - extends keyword
interface A { a: string; }
interface B { b: number; }
interface C extends A, B { c: boolean; }

// ✅ Type - intersection (&)
type A = { a: string; };
type B = { b: number; };
type C = A & B & { c: boolean; };

// ❌ GOTCHA - Can't use extends with type
type C = A extends B { }  // ❌ Syntax error
```

**Remember:**
- **Interface:** Use `extends`
- **Type:** Use `&`

---

### 8. Optional Properties vs Undefined
```typescript
// ❌ GOTCHA - These are NOT the same
interface User1 {
  age?: number;  // Can be number or undefined, or omitted entirely
}

interface User2 {
  age: number | undefined;  // MUST be present, but can be undefined
}

const user1: User1 = {};  // ✅ OK (age is optional)
const user2: User2 = {};  // ❌ Error: Property 'age' is missing

const user3: User1 = { age: undefined };  // ✅ OK
const user4: User2 = { age: undefined };  // ✅ OK
```

**Key difference:**
- `age?: number` - Property can be omitted
- `age: number | undefined` - Property must exist (can be undefined)

---

## Generic Gotchas

### 9. Generic Types Are Not Functions
```typescript
// ❌ GOTCHA - Can't use generic types like functions
type Box<T> = { value: T };

const box: Box<string> = { value: "hello" };  // ❌ Verbose, not recommended

// ✅ Create type alias first
type StringBox = Box<string>;
const box: StringBox = { value: "hello" };
```

**Remember:**
- Generic **functions** can be called: `func<T>(value)`
- Generic **types** need aliases: `type NewType = GenericType<T>`

---

### 10. Generic Type Parameters Can't Be Inferred in Types
```typescript
// ❌ GOTCHA - Can't infer T from type alone
type Container<T> = {
  value: T;
};

const container: Container = { value: "hello" };  // ❌ Error: Generic type 'Container' requires 1 type argument(s)

// ✅ Must specify T
const container: Container<string> = { value: "hello" };

// ✅ Generic FUNCTIONS can infer
function createContainer<T>(value: T): Container<T> {
  return { value };
}

const container = createContainer("hello");  // ✅ T is inferred as string
```

---

### 11. Constraints Don't Narrow Return Type
```typescript
// ❌ GOTCHA - T is still generic in return type
function getProperty<T extends { id: string }>(obj: T) {
  return obj.id;  // Type: string (good!)
}

function clone<T extends { id: string }>(obj: T): T {
  return { ...obj };
}

const user = { id: "123", name: "John" };
const cloned = clone(user);
// cloned type is { id: string; name: string }, not just { id: string }
// Even though T extends { id: string }, T preserves original type
```

**Why:** Generic type `T` preserves the specific type passed in.

---

## Readonly Gotchas

### 12. Readonly Is Only Shallow
```typescript
// ❌ GOTCHA - readonly only affects first level
interface User {
  readonly id: string;
  readonly profile: {
    name: string;
    age: number;
  };
}

const user: User = {
  id: "123",
  profile: { name: "John", age: 30 }
};

user.id = "456";  // ❌ Error: Cannot assign to 'id' because it is a read-only property

// ⚠️ BUT THIS WORKS (profile object is readonly, but its properties are NOT)
user.profile.name = "Jane";  // ✅ No error!
user.profile.age = 31;       // ✅ No error!
```

**Fix:**
```typescript
// ✅ Use DeepReadonly for nested immutability
type DeepReadonly<T> = {
  readonly [P in keyof T]: DeepReadonly<T[P]>;
};

type ImmutableUser = DeepReadonly<User>;

const user: ImmutableUser = {
  id: "123",
  profile: { name: "John", age: 30 }
};

user.profile.name = "Jane";  // ✅ Now gives error!
```

---

### 13. Readonly Arrays vs Regular Arrays
```typescript
// ❌ GOTCHA - readonly array is not assignable to mutable array
const mutableArray: string[] = ["a", "b"];
const readonlyArray: readonly string[] = ["c", "d"];

function addItem(arr: string[]) {
  arr.push("e");
}

addItem(mutableArray);    // ✅ OK
addItem(readonlyArray);   // ❌ Error: readonly string[] is not assignable to string[]
```

**Why:** Readonly arrays can't guarantee they won't be mutated if passed to functions expecting mutable arrays.

**Fix:**
```typescript
// ✅ Accept readonly in function
function printItems(arr: readonly string[]) {
  console.log(arr.join(", "));
}

printItems(mutableArray);   // ✅ OK
printItems(readonlyArray);  // ✅ OK
```

---

### 14. Readonly Doesn't Prevent Reassignment
```typescript
// ❌ GOTCHA - readonly prevents property mutation, not variable reassignment
interface Config {
  readonly apiUrl: string;
}

let config: Config = { apiUrl: "https://api.example.com" };

config.apiUrl = "https://new.com";  // ❌ Error: Cannot assign to 'apiUrl'

config = { apiUrl: "https://new.com" };  // ✅ OK! Variable can be reassigned
```

**Fix:**
```typescript
// ✅ Use const to prevent reassignment
const config: Config = { apiUrl: "https://api.example.com" };

config = { apiUrl: "https://new.com" };  // ❌ Error: Cannot assign to 'config'
```

---

## Branded Types Gotchas

### 15. Brands Are Erased at Runtime
```typescript
// ❌ GOTCHA - Brands only exist at compile-time
type UserId = string & { readonly __brand: "UserId" };

const userId = "user_123" as UserId;

console.log(typeof userId);  // "string" (not "UserId"!)
console.log(userId.__brand); // undefined (property doesn't exist at runtime!)

// JSON serialization loses brands
const json = JSON.stringify({ id: userId });
const parsed = JSON.parse(json);
// parsed.id is now 'string', not UserId
```

**Fix:**
```typescript
// ✅ Re-brand after parsing
function userId(id: string): UserId {
  return id as UserId;
}

const json = JSON.stringify({ id: userId("user_123") });
const parsed = JSON.parse(json);
const typedId = userId(parsed.id);  // Re-brand
```

---

### 16. Can't Use Branded Types in Type Guards
```typescript
// ❌ GOTCHA - Can't check brands at runtime
type UserId = string & { readonly __brand: "UserId" };

function isUserId(value: unknown): value is UserId {
  return typeof value === "string" && value.startsWith("user_");  
  // Can't check __brand because it doesn't exist at runtime
}
```

**Remember:** Brands are compile-time only. Runtime checks must validate the actual value structure.

---

### 17. Array Methods Lose Brands
```typescript
// ❌ GOTCHA - Array methods don't preserve brands
type UserId = string & { readonly __brand: "UserId" };

const userIds: UserId[] = ["user_1", "user_2"].map(id => id as UserId);

const mapped = userIds.map(id => id.toUpperCase());
// Type is string[], not UserId[]! Brand is lost

const filtered = userIds.filter(id => id !== "user_1");
// Type is UserId[] ✅ (filter preserves type)
```

**Fix:**
```typescript
// ✅ Re-brand after transformation
const userId = (id: string): UserId => id as UserId;

const mapped = userIds.map(id => userId(id.toUpperCase()));
// Now type is UserId[]
```

---

## Utility Types Gotchas

### 18. Partial Doesn't Work on Nested Objects
```typescript
// ❌ GOTCHA - Partial only affects first level
interface User {
  id: string;
  profile: {
    name: string;
    age: number;
  };
}

type PartialUser = Partial<User>;
// Result: { id?: string; profile?: { name: string; age: number } }
// profile is optional, but name and age inside are still required!

const user: PartialUser = {
  profile: { name: "John" }  // ❌ Error: Property 'age' is missing
};
```

**Fix:**
```typescript
// ✅ Create DeepPartial
type DeepPartial<T> = {
  [P in keyof T]?: DeepPartial<T[P]>;
};

type DeepPartialUser = DeepPartial<User>;

const user: DeepPartialUser = {
  profile: { name: "John" }  // ✅ OK now!
};
```

---

### 19. Omit Doesn't Check Key Existence
```typescript
// ❌ GOTCHA - Omit accepts non-existent keys without error
interface User {
  id: string;
  name: string;
}

type WithoutPassword = Omit<User, 'password'>;
// No error! Even though 'password' doesn't exist in User
// Result is just User (nothing omitted)
```

**Why:** TypeScript allows this for flexibility (useful when extending types).

**Watch out for typos:**
```typescript
type UserResponse = Omit<User, 'passwrd'>;  // Typo! No error, nothing omitted
```

---

### 20. Pick/Omit With Union Types
```typescript
// ❌ GOTCHA - Pick/Omit work differently with unions
type A = { a: string; b: number };
type B = { b: number; c: boolean };

type Union = A | B;

type PickedB = Pick<Union, 'b'>;
// Result: { b: number } (intersection of 'b' from both)

type OmittedB = Omit<Union, 'b'>;
// Result: { a: string } | { c: boolean } (removes 'b' from both)
```

**Remember:** Pick intersects, Omit unions.

---

## Runtime vs Compile-time Gotchas

### 21. Types Are Erased at Runtime
```typescript
// ❌ GOTCHA - Can't check types at runtime
interface User {
  name: string;
}

interface Admin {
  name: string;
  role: string;
}

function isAdmin(user: User | Admin): user is Admin {
  return 'role' in user;  // ✅ Check structure, not type
  // return user instanceof Admin;  // ❌ Error: 'Admin' only refers to a type
}
```

**Why:** Interfaces and types don't exist at runtime.

**Fix:** Check actual properties/structure.

---

### 22. Enums Are NOT Erased
```typescript
// ⚠️ GOTCHA - Enums exist at runtime (unlike interfaces)
enum Role {
  Admin = "ADMIN",
  User = "USER"
}

console.log(Role.Admin);  // "ADMIN" ✅ Works at runtime!

// Compiled JS:
var Role;
(function (Role) {
    Role["Admin"] = "ADMIN";
    Role["User"] = "USER";
})(Role || (Role = {}));
```

**Remember:** Enums are actual objects at runtime. Use const enums or union types if you want them erased.

```typescript
// ✅ This is erased at runtime
const enum Role {
  Admin = "ADMIN",
  User = "USER"
}

// ✅ Or use union type (also erased)
type Role = "ADMIN" | "USER";
```

---

### 23. Type Assertions Don't Run at Runtime
```typescript
// ❌ GOTCHA - 'as' doesn't validate or convert
const data = '{"name": "John"}';
const user = JSON.parse(data) as User;
// user is typed as User, but might not be! No runtime validation

user.email.toUpperCase();  // ❌ Runtime error if email doesn't exist
```

**Fix:**
```typescript
// ✅ Validate at runtime
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'name' in obj &&
    'email' in obj &&
    typeof (obj as any).email === 'string'
  );
}

const parsed = JSON.parse(data);
if (isUser(parsed)) {
  parsed.email.toUpperCase();  // ✅ Safe
}

// ✅ Or use validation libraries (Zod, Yup, io-ts)
import { z } from 'zod';

const UserSchema = z.object({
  name: z.string(),
  email: z.string().email()
});

const user = UserSchema.parse(JSON.parse(data));  // Validates at runtime
```

---

### 24. typeof Returns Runtime Types, Not TypeScript Types
```typescript
// ❌ GOTCHA - typeof in type position vs value position
interface User {
  name: string;
}

const user: User = { name: "John" };

// Type position (TypeScript)
type UserType = typeof user;  // { name: string }

// Value position (JavaScript)
console.log(typeof user);  // "object" (not "User"!)

if (typeof user === "User") { }  // ❌ This doesn't work
if (typeof user === "object") { }  // ✅ This is what you get
```

---

## Common Mistakes

### 25. Forgetting await with Promises
```typescript
// ❌ GOTCHA - TypeScript doesn't warn about missing await
async function getUser(): Promise<User> {
  return { name: "John" };
}

function processUser() {
  const user = getUser();  // Type: Promise<User>, not User!
  console.log(user.name);  // ❌ Error: Property 'name' does not exist on type 'Promise<User>'
}

// ✅ Fix
async function processUser() {
  const user = await getUser();
  console.log(user.name);  // ✅ Works
}
```

---

### 26. Mutating Function Parameters
```typescript
// ❌ GOTCHA - TypeScript allows parameter mutation
function updateUser(user: User) {
  user.name = "Jane";  // ✅ No error, even though it mutates input
}

const user = { name: "John" };
updateUser(user);
console.log(user.name);  // "Jane" (original object mutated!)
```

**Fix:**
```typescript
// ✅ Use readonly parameters
function updateUser(user: Readonly<User>): User {
  return { ...user, name: "Jane" };  // Return new object
}

// ✅ Or make it explicit
function updateUserMutating(user: User): void {
  user.name = "Jane";
}
```

---

### 27. Index Signatures Allow Any Key
```typescript
// ❌ GOTCHA - Index signature allows non-existent keys
interface StringMap {
  [key: string]: string;
}

const map: StringMap = { name: "John" };

console.log(map.age);  // undefined (no error!)
map.anything.toUpperCase();  // ❌ Runtime error, but TypeScript thinks it's string
```

**Fix:**
```typescript
// ✅ Use Record for known keys
type StringMap = Record<'name' | 'email', string>;

const map: StringMap = { name: "John", email: "john@example.com" };
console.log(map.age);  // ❌ Error: Property 'age' does not exist

// ✅ Or use Map
const map = new Map<string, string>();
map.set('name', 'John');
console.log(map.get('age'));  // undefined (explicit)
```

---

### 28. Non-Null Assertion (!) Bypasses Checks
```typescript
// ❌ GOTCHA - ! suppresses null/undefined checks
function getUser(): User | null {
  return null;
}

const user = getUser()!;  // "I promise this is not null!"
console.log(user.name);   // ❌ Runtime error: Cannot read property 'name' of null
```

**Fix:**
```typescript
// ✅ Handle null properly
const user = getUser();
if (user) {
  console.log(user.name);  // ✅ Safe
}

// ✅ Or use optional chaining
console.log(user?.name);  // undefined if user is null
```

---

### 29. Strict Null Checks Must Be Enabled
```typescript
// ❌ GOTCHA - Without strict null checks, null is everywhere
// tsconfig.json: { "strictNullChecks": false }

function greet(name: string) {
  console.log(name.toUpperCase());
}

greet(null);  // ✅ No error with strictNullChecks: false
              // ❌ Runtime error!
```

**Fix:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true  // Included in "strict"
  }
}
```

---

### 30. Object.keys Returns string[]
```typescript
// ❌ GOTCHA - Object.keys loses type information
interface User {
  name: string;
  age: number;
}

const user: User = { name: "John", age: 30 };

Object.keys(user).forEach(key => {
  console.log(user[key]);  // ❌ Error: Element implicitly has an 'any' type
                           // because expression of type 'string' can't be used to index type 'User'
});

// typeof key is 'string', not 'keyof User'
```

**Why:** JavaScript allows extra properties at runtime, so TypeScript can't guarantee keys match the type.

**Fix:**
```typescript
// ✅ Type assertion
(Object.keys(user) as (keyof User)[]).forEach(key => {
  console.log(user[key]);  // ✅ Works
});

// ✅ Or use a typed helper
function typedKeys<T>(obj: T): (keyof T)[] {
  return Object.keys(obj) as (keyof T)[];
}

typedKeys(user).forEach(key => {
  console.log(user[key]);  // ✅ Works
});
```

---

## Summary Checklist

### Type System
- [ ] TypeScript uses structural typing (shape matters, not name)
- [ ] Types are erased at runtime (can't check types with `instanceof`)
- [ ] `any` disables all type checking (avoid it)
- [ ] `unknown` is safer than `any` (requires type narrowing)
- [ ] `null` and `undefined` are different types
- [ ] Type assertions (`as`) don't validate at runtime

### Interfaces & Types
- [ ] Interfaces can be merged, types cannot
- [ ] `readonly` is shallow (doesn't affect nested properties)
- [ ] Optional (`?`) ≠ `| undefined` (optional can be omitted)
- [ ] Index signatures allow any key (use with caution)

### Generics
- [ ] Generic types need type parameters filled in
- [ ] Generic type parameters can't be inferred in type aliases
- [ ] Constraints don't narrow return types

### Branded Types
- [ ] Brands exist only at compile-time (erased at runtime)
- [ ] Must re-brand after JSON parsing
- [ ] Array methods can lose brands

### Utility Types
- [ ] `Partial`, `Readonly`, etc. are shallow (first level only)
- [ ] `Omit` doesn't validate key existence
- [ ] `Object.keys` returns `string[]`, not `keyof T`

### Runtime
- [ ] Enums exist at runtime (unlike interfaces/types)
- [ ] `typeof` in type position ≠ `typeof` at runtime
- [ ] Must await Promises (TypeScript won't warn)
- [ ] Non-null assertion (`!`) bypasses checks (dangerous)

### Best Practices
- [ ] Enable `strict: true` in tsconfig.json
- [ ] Validate external data at runtime (use Zod, etc.)
- [ ] Use type guards for runtime checks
- [ ] Prefer `unknown` over `any`
- [ ] Use factory functions for branded types
- [ ] Don't mutate readonly parameters

---

**Last Updated:** 2024  
**TypeScript Version:** 5.x  
**Remember:** TypeScript helps at compile-time, but runtime safety requires validation!

---

