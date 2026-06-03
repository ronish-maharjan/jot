# Daily Learnings

---

## Date: `18/12/2025`

### Insight
Everything passed into domain functions must be a Value Object.

### Notes
- All data entering the Application layer is in raw form.
- It is the responsibility of the Application layer to convert raw data into Value Objects.
- Data flowing inside the Domain layer should always be in Value Object form.
- Use the `Vo` suffix only at boundary layers.
- Do not use naming conventions like `accountVo` inside the domain.

### Gotcha
- Passing raw primitives directly into domain logic breaks domain purity.

---

## Date: `19/12/2025`

### Insight
CQRS is an advanced way of structuring use cases in Hexagonal Architecture.

### Notes
- CQRS separates command (write) and query (read) responsibilities.
- Useful when application complexity increases.

---

## Date: `23/12/2025`

### Insight
Named parameters make domain APIs safer and more expressive.

### Notes
- Prefer named parameters for domain `create` and behavior methods.
- Avoid positional parameters to reduce argument-order bugs.

### Gotcha
- Positional parameters become error-prone as the number of arguments grows.

---

## Date: `24/12/2025`

### Insight
JavaScript edge cases and mutation can silently affect domain correctness.

### Notes
- `-0 === 0` evaluates to true in JavaScript.
- Use `Object.is(value, -0)` to explicitly detect `-0`.
- Objects passed into constructors can be mutated unintentionally.

### Gotcha
- Internal mutation inside constructors.
- Mutation caused by passing objects by reference.

---

## Date: `25/12/2025`

### Insight
Javascript difference between splice and slice.

### Notes
- `splice` will change the original array where as `slice` will not change.
- `splice` take parameter (index,count) how many to delete form the given index).
- `slice` take parameter (index, index) exclude the last index.

### Example

```js
const arr = [1, 2, 3];

arr.slice(1); // default value for end is length of array here its 3  
// arr → [1, 2, 3]

arr.splice(1);  //default value for count is 1.
// arr → [1]

```

---

### Insight
Why use private specifier than public readonly in most of the class and valueobject.

### Notes
- `Public` are generally dumb attribute it return exact value showing whole object structure as well. Also we cant add extra logic.
- `Private` helps to hide what data are present in object and just provide an interface like getter to access data where later if we neeed to add any logic we can add easily.

### Example: `Comparision Between private and public attributes`

```typescript
// using public type
class Thermometer {
  public readonly celsius: number; // Structure is EXPOSED

  constructor(value: number) {
    this.celsius = value;
  }
}

// USAGE in 100 different files:
const t = new Thermometer(25);
console.log(t.celsius + "°C"); 

// THE DISASTER:
// If the boss says "We now store everything in Kelvin internally,"
// you have to find and fix all 100 files because '.celsius' 
// doesn't exist or has changed.

// Using Private type (The Senior Way)
class Thermometer {
  #_internalValue: number; // Hidden from the world

  constructor(celsius: number) {
    this.#_internalValue = celsius; // Initial internal state
  }

  // The Interface (The "Window" people look through)
  public get celsius(): number {
    return this.#_internalValue;
  }
}

// THE FUTURE REFACTOR (The "Boss" change):
class Thermometer {
  #_kelvin: number; // We changed the internal "Truth" to Kelvin

  constructor(celsius: number) {
    this.#_kelvin = celsius + 273.15; // Convert on the way IN
  }

  public get celsius(): number {
    // We convert on the way OUT. 
    // Result: 100 files using .celsius NEVER BREAK.
    return this.#_kelvin - 273.15; 
  }
}

// so in future even if we make the celcius to kelvin we jsut have to add some logic to change it to celcius agian

get celcius(){
    // we just have to write a logic to convert the kelvin to celcisu and return it
    const celcius = kelvin -273.15 //we just have to add this logic
    reutrn this.celcius
}

```
---


## Date: `26/12/2025`

### Insight
Learning indepth on application layer of hexagonal architecture.

### Notes
- The **usecase** cannot use another usecases inside it like we use repository through ports.
- We use **Event Driven architecture** here for handling multiple usecases.

---

## Date: `27/12/2025`

### Overview
Senior developers mostly make the class attributes private so start using # for private for attribute mostly.Also `getter` and `method` to provide private data have some differences.

### Notes
- Always make **classes** **attributes** private.
- Use **Getter** for giving values rather than using the method to return the private values.
- Use **Methods** for performing logics or task not for throwing **single** value

---

### Overview
**`Events`** are basically a **class** that stores the value of a certain task performed.The main benifit of this class is we can use that event class to pass in other functions which are called **Handlers** and perform specific task.

`More on`: [Featured Based Structure](../topics/featured-based-structure.md)

---

## Date: `29/12/2025`

### Overview
Value objects are used for like reducing same code repetation as well as to reduce bugs of passing value as references.

### Notes
- when we dont use valueobjects and use normal way like when we pass object data or array and want to change it then if we change something which is passed as references then everywhere the object is used will have that change effect.
- Value object remove that references bug cause the value are immutable in vos if we want to change the value of vos we have to create new valueobject making it safe.

### New Learnings
- use **Factory functions** also in **Value Objects**.
- return **value objects** from the getters rather than actual raw values.

---

## Date: `09/05/2026`

### Overview
- Naming the variables inside the repositroy code should be in this fromat row, rows, db<column_name>

### Notes

#### Backend Variable Naming Convention 

**Golden Rule**

> **DB = snake_case | Code = camelCase | Mapper = bridge between them**

---

#### 1. Database layer (RAW Drizzle results)

- **Multiple rows**

```ts
const rows = await db.select().from(jobs);
```

- **Single row**

```ts
const row = await db.select().from(jobs).where(...);
```

> Use when data is **directly from DB**

---

#### 2. Data going INTO database (INSERT / UPDATE)

```ts
const dbJob = JobMapper.toDB(job);
```

> Meaning: ready for DB insert/update
> Prefix rule: `db`

---

#### 3. Domain layer (business logic objects)

```ts
const job = JobMapper.toDomain(row);
```

> Meaning: clean application object
> camelCase fields
> No DB format inside

---

#### 4. Lists (VERY IMPORTANT)

- **DB list**

```ts
const rows = await db.select().from(jobs);
```

- **Domain list**

```ts
const jobs = rows.map(JobMapper.toDomain);
```

> Always plural for arrays

---

#### Quick Recaps

**Single items**

| Type            | Name    |
| --------------- | ------- |
| DB row          | `row`   |
| Domain object   | `job`   |
| DB write object | `dbJob` |

---

**Collections**

| Type        | Name   |
| ----------- | ------ |
| DB rows     | `rows` |
| Domain list | `jobs` |

---

#### Example (perfect clean flow)

```ts
const dbJob = JobMapper.toDB(job);

await db.insert(jobs).values(dbJob);

const rows = await db.select().from(jobs);

const jobs = rows.map(JobMapper.toDomain);

const row = rows[0];

const job = JobMapper.toDomain(row);
```

---

## Date: `12/05/2026`

### Overview
So generally we should not use try catch inside the domain , use only one outer try catch on usecase and in each external call in adapter should use the try catch block and also in controller.

### Notes
- **Domain entity** will only contain logic and even if error its catched by usecase cause usecase already have **one outermost try catch block**.
- **Usecase** will have only one outer most trycatch block thats wrap whole execute function if any unexpected error happen we return unexpected error.
- **Adapter** should use try catch block each external call like calling apis.
- **Repository** should not use try catch bubble up the error and it will be catched by try catch block in usecase if any unexpected error like connection failed occured.
- **Controller** should always use try catch and use catch to pass all error on global error handler. 


## Date: `13/05/2026`

### Overview
what exactly is **stacktrace** and what is that **cause** property in the errro.

### Notes
- From my understanding the stack trace is created on where the error is created or occured and by created i mean using **new or automatically created which is Error** and include every bubble up it passed up to where the error is thrown.

### 1. What is a Stack Trace?

A **stack trace** is a snapshot of the **function call stack** at the moment an `Error` object is created.

It shows:

* The sequence of function calls that led to the error
* Where the error originated in the code
* The execution path (call chain)

### Example:

```js
function a() {
  b();
}

function b() {
  c();
}

function c() {
  throw new Error("Something went wrong");
}

a();
```

### Stack trace output:

```
Error: Something went wrong
    at c (app.js:10)
    at b (app.js:6)
    at a (app.js:2)
```

### How to read it:

* Top line → error message
* Below lines → call stack (most recent first)
* Bottom → earliest function in the chain

---

### 2. When is Stack Trace Created?

### Stack trace is created when:

```js
new Error("message")
```

or when the runtime automatically creates an error.

### NOT when:

```js
throw error;
```

Throwing only propagates the error — it does NOT create or modify the stack trace.

---

### 3. Important Rule

> The stack trace is captured at the moment the `Error` object is created.

NOT:

* when it is thrown
* when it is caught
* when it bubbles up

---

### 4. Error as Return Value vs Throw

### Case 1: Returning an Error (no stack impact at throw site)

```js
function c() {
  return new Error("fail");
}

function a() {
  const err = c();
  throw err;
}
```

### Result:

* Stack trace reflects where `new Error()` was created (inside `c`)
* NOT where it was thrown (`a`)

---

### Case 2: Throwing Immediately (correct stack)

```js
function c() {
  throw new Error("fail");
}
```

### Result:

* Stack trace shows full call chain:

```
at c
at b
at a
```

---

### 5. What is the `cause` Property?

The `cause` property allows **error chaining**.

It helps preserve the original error when wrapping it.

### Example:

```js
function fetchData() {
  throw new Error("Database connection failed");
}

function getUser() {
  try {
    fetchData();
  } catch (err) {
    throw new Error("Failed to get user", { cause: err });
  }
}
```

### Result:

* Main error = "Failed to get user"
* Cause = original DB error

---

### 6. Stack Trace vs Cause

| Concept     | Meaning                                                  |
| ----------- | -------------------------------------------------------- |
| Stack Trace | Where the error object was created                       |
| Cause       | The underlying error that triggered a higher-level error |

---

### 7. Mental Model

### Stack Trace:

> “How did execution reach the point where the error was created?”

### Cause:

> “What original error caused this higher-level failure?”

---

## 8. Key Takeaways

* Stack trace is captured only at `new Error()`
* Throwing an error does NOT change stack trace
* Returning errors removes automatic stack propagation
* Use `cause` to preserve original errors in layered systems
* Stack trace = execution path snapshot at creation time

---

## 9. Quick Summary

> Stack trace = snapshot of function calls at error creation time
> Cause = link to original underlying error
> Throwing = just propagates the existing error object

---

## Date: `16/05/2026`

### overview
Variable naming conventions 

### notes 

- use actual name as **params** what is **passing** inside the function like **function(user:User)** instead of **function(entity:User)**
- the variable name holding the entity in repository name it as entitty name like **user = mapper.toDomain(row)**  
- **important to remmenber** for the **attributes** while creating and restoring entity the type name are differnet 
    - while **creating** the type name have suffix **input** eg **createUserInput** as it need to validate
    - But for **restoring** the type name have suffix **props** eg ** restoreUserProps** as it is aready validated.

---

## Date: `27/05/2026`

### overview 
How data should be manipulated in different boundaries.

### notes 

- use **mapper** to convert the db to domain and domain to db
- use **ApiResponse** class like the result pattern to return response from controller
- the **Vos** are used by the entity not by usecase cause if we do then the domain is leaked to application so usecase just create entity

### conclusion
- Use mapper to convert data ,create an apiclass that will convert/format the response data for api
---

## Date: `28/05/2026`

## overview
Learning Pattern 

## Notes
- Instead of learning randomly follow this 
    - SOLID Principle
    - Error Handling
    - Env Configs
    - Zod Validation 
    - Hexagonal Architecture
    - Testing
    - Db Migration 
    - Logging / open telementary
    - Graceful Shutdown 
    - Circuit Breaker
    - Idempotency
    - Event Driven Architecture
    - CQRS 

- Different patterns to learn 
    - Result pattern  also known as railway pattern
    - Observer pattern 
    - Specification Pattern 
    - strategy pattern

    - Better to know have some idea 
        - builder pattern 
        - factory pattern


- After knowing how to write better code lets understan on how system works like databases 
    - Basic DB stuff
        - CRUD operations
        - Joins
    -- more to be added in future 


## Date `29/05/2026`

## Overview

## Notes 
    - **replace** function will only replace **first match** not all
    - use **replaceAll** function to replace all 

