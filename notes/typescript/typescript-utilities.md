# TypeScript Utility Types: Partial & Awaited

> Purpose  
> This section exists to remind me how `Partial`, `Awaited`, `Required`, `Readonly` work in TypeScript, especially for backend engineering.  
> Focus is on understanding and recalling their **practical uses**.

---

## Partial<T>

**Definition**  
`Partial<T>` is a utility type that **makes all properties of a type optional**.

**Remember**
- Converts `T` like `{ id: string; name: string }` → `{ id?: string; name?: string }`
- Extremely useful for **update DTOs**, where only some fields may be sent
- Avoids manually rewriting optional fields

**Example**
```ts
type User = { id: string; name: string; email: string };

// Partial<User> makes all fields optional
type UpdateUserDto = Partial<User>;

const update: UpdateUserDto = { name: "Bob" }; 

```

--- 

## Awaited<T>

**Definition**
`Awaited<T>` is a utility type that unwraps a Promise type to get the resolved value type.

**Remember**
- Use for async functions to get the type you actually receive after await
- Handles nested Promises automatically

**Example**
```ts
async function getUser() {
  return { id: "1", name: "Alice" };
}

// ReturnType gives Promise<{ id: string; name: string }>
type UserPromise = ReturnType<typeof getUser>;

// Awaited unwraps it to the actual value
type User = Awaited<ReturnType<typeof getUser>>;
// User = { id: string; name: string }

//Nested Promises

const nested = Promise.resolve(Promise.resolve("hello"));
type Value = Awaited<typeof nested>; // Value = string

```

> `Awaited` is crucial in backend to derive types from async functions automatically, avoiding hardcoded types and keeping your DTOs in sync.

**Key Takeaways**
- `Partial<T>` → optional properties, perfect for update payloads
- `Awaited<T>` → resolved type from async functions, prevents Promise type mismatches
- Senior backend devs use these to avoid hardcoding types and maintain type safety automatically
- Combining these with `ReturnType`, `Pick`, `Omit`, and intersections gives flexible, maintainable backend DTOs


--- 

## Required<T>

**Definition**

`Required<T>` is a utility type that **makes all optional properties of a type required**.

**Remember**
- Converts `?` properties into required ones
- Useful when you want to enforce **all fields are present** in a payload
- Opposite of `Partial<T>`

## Example

```ts
type User = {
  id?: string;
  name?: string;
  email?: string;
};

// Make all properties required
type CompleteUser = Required<User>;

const user: CompleteUser = {
  id: "1",
  name: "Alice",
  email: "alice@example.com" 
};
```

> `Required<T>` is great when validating payloads before saving to database or sending full API responses.

**Key Takeaways**
- Required<T> → converts all optional fields (?) to required
- Useful for enforcing complete objects
- Works well with Partial<T>: you can toggle optional ↔ required depending on context
- Helps avoid runtime errors by making TypeScript enforce all necessary fields

--- 

# Readonly<T> in TypeScript

## Definition
`Readonly<T>` is a utility type that **makes all properties of a type read-only**.

**Remember**
- Converts every property into `readonly`  
- Once assigned, properties **cannot be modified**  
- Useful for **immutability**, caching, and safe API responses
- Focus is on **preventing modification of properties**.

## Example

```ts
type User = {
  id: string;
  name: string;
  email: string;
};

// Make all properties readonly
type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = {
  id: "1",
  name: "Alice",
  email: "alice@example.com"
};

user.name = "Bob"; // ❌ Error: Cannot assign to 'name' because it is a read-only property
```
> `Readonly` is often combined with Partial, Required, or other utility types to control mutability precisely.

**Key Takeaways**
- Readonly<T> → makes all properties immutable (`readonly`)
- Useful for immutable DTOs, caching, and preventing accidental mutations
- Works with arrays (`ReadonlyArray<T>`) and objects
- Helps ensure type safety and immutability in backend code

---

## Pick<T, K> in TypeScript

**Definition**
`Pick<T, K>` is a utility type that **constructs a new type by picking a subset of properties** `K` from type `T`.

**Remember**
- `T` = original type  
- `K` = keys of `T` you want to keep  
- Useful for **selecting only required fields** for APIs, responses, or partial updates
- This note reminds me how `Pick<T, K>` works in TypeScript, especially for backend DTOs.  
- Focus is on **selecting only specific properties from a type**.

## Example

```ts
type User = {
  id: string;
  name: string;
  email: string;
  role: string;
};

// Pick only id and name
type UserSummary = Pick<User, "id" | "name">;

const user: UserSummary = {
  id: "1",
  name: "Alice"
};

//Only the picked fields exist in the new type
//Other fields (email, role) are excluded
//TypeScript ensures type safety for the selected keys
```

> Pick is often combined with `Partial`, `Omit` or intersections to create flexible DTOs for backend APIs.

**Key Takeaways**
- Pick<T, K> → select only the properties you need from a type
- Useful for API responses, DTOs, and creating subsets of types
- Works great with Partial<T> for update payloads
- Helps avoid exposing unnecessary fields in backend code

---

## Omit<T, K> in TypeScript

**Definition**
`Omit<T, K>` is a utility type that **constructs a new type by removing the properties** `K` from type `T`.

**Remember**
- `T` = original type  
- `K` = keys of `T` you want to remove  
- Useful for **excluding fields** like sensitive data or internal IDs in responses
- This note reminds me how `Omit<T, K>` works in TypeScript, especially for backend DTOs.  
- Focus is on **excluding specific properties from a type**.

## Example

```ts
type User = {
  id: string;
  name: string;
  email: string;
  password: string;
};

// Omit the password
type PublicUser = Omit<User, "password">;

const user: PublicUser = {
  id: "1",
  name: "Alice",
  email: "alice@example.com"
  // password is not allowed here
};

//The new type excludes the omitted keys
//TypeScript ensures you cannot include them by mistake
```
> Omit is often used with Pick, Partial, or intersections to create flexible, safe DTOs for backend APIs.

**Key Takeaways**
- Omit<T, K> → remove the properties you don’t want from a type
- Useful for hiding sensitive fields or simplifying DTOs
- Works great with Partial<T> for update or patch operations
- Helps avoid accidental exposure of internal or sensitive data
