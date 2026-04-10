```md
# TypeScript Advanced Types - Quick Reference

## 1️⃣ Arrays

```typescript
let numbers: number[] = [1, 2, 3];
let fruits: Array<string> = ["apple", "banana"];
```

**Key Points:**
- Two syntaxes: `T[]` or `Array<T>` (generic)
- Arrays hold only one type (unless using union types)

---

## 2️⃣ Tuples

```typescript
let person: [string, number] = ["Alice", 25];
```

**Key Points:**
- Fixed length array
- Each element can have a different type
- Optional elements: `let user: [string, number?] = ["Bob"];`

---

## 3️⃣ Enums

```typescript
// Numeric enum
enum Direction { Up, Down, Left, Right }

// String enum
enum Status { Active = "ACTIVE", Inactive = "INACTIVE" }

// ✅ Better: Use const objects instead
const Direction = { Up: "UP", Down: "DOWN" } as const;
type Direction = typeof Direction[keyof typeof Direction];
```

**Key Points:**
- Use string literal objects with `as const` for type-safety and smaller JS bundle
- Safer alternative to traditional enums

---

## 4️⃣ as const

```typescript
const colors = ["red", "green", "blue"] as const;
type Color = typeof colors[number]; // "red" | "green" | "blue"
```

**Key Points:**
- Makes arrays/objects **readonly**
- Turns values into **literal types**
- Safer alternative to enums for constants

---

## 5️⃣ unknown vs any

```typescript
let a: any = 42;
let u: unknown = 42;

// ❌ any allows unsafe operations
a.toUpperCase(); // Compiles, may crash at runtime

// ✅ unknown requires type check
if (typeof u === "number") {
  console.log(u + 1); // Safe
}
```

**Key Points:**
- `any`: Bypasses TypeScript type checks (unsafe)
- `unknown`: Safe alternative, must check/narrow before use

---

## 6️⃣ null and undefined

```typescript
let a: string | null | undefined;

a = null;        // ✅
a = undefined;   // ✅
a = "Alice";     // ✅
```

**Key Points:**
- `undefined`: Variable declared but not assigned
- `null`: Explicit "no value"
- Use union types: `string | null | undefined`

---

## 7️⃣ Object Types

```typescript
let person: { name: string; age: number };
person = { name: "Alice", age: 25 };

// Optional properties
let user: { name: string; age?: number };

// Readonly
let config: { readonly apiKey: string };

// Index signatures
let scores: { [key: string]: number } = { Alice: 90 };
```

**Prefer interfaces or type aliases:**
```typescript
interface Person { 
  name: string; 
  age: number; 
}

type Point = { 
  x: number; 
  y: number; 
}
```

### object vs {} vs Object

| Type     | Meaning                                      |
|----------|----------------------------------------------|
| `object` | Any non-primitive (object, array, function)  |
| `{}`     | Any value except `null` or `undefined`       |
| `Object` | All JS values except `null`/`undefined`      |

---

## 8️⃣ Type Assertion

```typescript
let value: unknown = "Hello";
let strLength = (value as string).length; // ✅ Tell TS this is a string
```

**Analogy: Subtype vs Supertype**
```typescript
class Animal {}
class Cat extends Animal {}

let animal: Animal = new Cat();
let cat = animal as Cat; // ✅ Trust me, it's a Cat
```

**Key Points:**
- Use when **you know more than TypeScript**
- ⚠️ Does **NOT** check runtime type
- Syntax: `value as Type` or `<Type>value`

---

## 💡 Quick Tips

✅ Use `as const` or string unions instead of enums  
✅ Use `unknown` instead of `any` for type safety  
✅ Always narrow `null | undefined` before use  
✅ Prefer `interface`/`type` for object shapes  
✅ Type assertion = "trust me, TS, I know the type"

---

## Summary Table

| Feature          | Syntax                              | Use Case                          |
|------------------|-------------------------------------|-----------------------------------|
| Array            | `number[]` or `Array<number>`       | List of same type                 |
| Tuple            | `[string, number]`                  | Fixed-length, mixed types         |
| Enum             | `enum Status { Active }`            | Named constants (prefer `as const`)|
| `as const`       | `const x = [1, 2] as const`         | Readonly + literal types          |
| `unknown`        | `let x: unknown`                    | Safe alternative to `any`         |
| `null/undefined` | `string \| null`                    | Allow missing values              |
| Object           | `{ name: string }`                  | Object shape                      |
| Type Assertion   | `value as string`                   | Override TypeScript's inference   |
```


