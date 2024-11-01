# Stalo

Stalo is a robust, state-management library built with a focus on simplicity, efficiency, and extensibility for modern React applications. It leverages concepts like Immer.js to enable immutable updates and provides middleware mechanisms for state manipulation and debugging.

---

## Installation

You can install Stalo using npm or yarn:

```bash
npm install stalo
# or
yarn add stalo
```

---

## Basic Concepts

Stalo provides a set of utilities for managing state and comes with middleware features such as DevTools and Immer.js integration for seamless patch generation and debugging.

### Core API

1. **create** - Creates a store with an initial state.
2. **Middleware** - A mechanism for extending the store's behavior.
3. **Persistent Storage** - Utilities to persist state in `localStorage` or via URL hashing.
4. **DevTools** - Debugging tools that enable state inspection and patches.
5. **Utilities** - Functions like `compose`, `uid`, and `deepEqual` to facilitate advanced state management.

---

## Usage

### 1. Creating a Store

The simplest way to create a store is using the `create` function:

```typescript
import create from "stalo";

const [useValue, setValue] = create(false);

// Example component
function ExampleComponent() {
  const value = useValue();
  
  return (
    <button onClick={() => setValue(true)}>
      {value ? "Active" : "Inactive"}
    </button>
  );
}
```

### 2. Using Middleware

Stalo supports middleware to extend the functionality of state updates. You can use the built-in `compose` function to apply middleware:

#### Example: Adding and Composing Middleware

```typescript
import { create } from "stalo";
import { compose, Middleware } from "stalo/utils";

// Define custom middleware
const logger: Middleware<boolean> = (set) => (newState, ctx) => {
  console.log("Previous State:", ctx.previousState);
  console.log("New State:", newState);
  set(newState, ctx);
};

// Create a store with middleware
const [useValue, setValue] = create(false);
const setWithLogger = compose(setValue, logger);

function ExampleWithLogger() {
  return (
    <button onClick={() => setWithLogger(true)}>
      Click Me
    </button>
  );
}
```

### 3. Using Immer.js for Immutable Updates

Immer.js makes it easy to apply immutable state changes. Use `immerWithPatch` to automatically generate patches.

```typescript
import { create } from "stalo";
import { immerWithPatch } from "stalo/devtools";

// Create a store with Immer
const [useValue, setValue] = create({ counter: 0 });
const setWithImmer = immerWithPatch(setValue);

// Update state immutably
function IncrementCounter() {
  return (
    <button onClick={() => setWithImmer((draft) => { draft.counter += 1; })}>
      Increment
    </button>
  );
}
```

### 4. DevTools Integration

Stalo provides DevTools for state inspection and debugging:

```typescript
import { create } from "stalo";
import devtools from "stalo/devtools";

const [useValue, setValue] = create(0);
const setWithDevTools = devtools(setValue, { name: "CounterStore" });

function CounterComponent() {
  const counter = useValue();
  
  return (
    <div>
      <p>Count: {counter}</p>
      <button onClick={() => setWithDevTools(counter + 1)}>Increment</button>
    </div>
  );
}
```

### 5. Persistent State

Stalo can persist state using URL hashes or `localStorage`.

#### Using URL Hash for Persistence

```typescript
import { createPersistentStorage } from "stalo/persistent";

const [useValue, setValue] = createPersistentStorage("initialValue");

function PersistentComponent() {
  const value = useValue();
  
  return (
    <button onClick={() => setValue("newValue")}>
      {value}
    </button>
  );
}
```

#### Using Local Storage for Persistence

```typescript
import { createLocalStorage } from "stalo/persistent";

const [useValue, setValue] = createLocalStorage("defaultValue", "myKey");

function LocalStorageComponent() {
  const value = useValue();
  
  return (
    <div>
      <p>{value}</p>
      <button onClick={() => setValue("updatedValue")}>Update</button>
    </div>
  );
}
```

---

## Advanced Features

### Composing Multiple Middlewares

You can combine multiple middlewares using the `compose` utility:

```typescript
import { create } from "stalo";
import { compose, Middleware } from "stalo/utils";

// Define middlewares
const addOne: Middleware<number> = (set) => (newState) => set(newState + 1);
const double: Middleware<number> = (set) => (newState) => set(newState * 2);

// Create store and compose middlewares
const [useValue, setValue] = create(1);
const setWithMiddleware = compose(setValue, addOne, double);

function ComplexComponent() {
  return (
    <button onClick={() => setWithMiddleware(3)}>
      Update
    </button>
  );
}
```

### Utilities

1. **deepEqual**: Deeply compares two objects.

   ```typescript
   import { deepEqual } from "stalo/utils";
   
   const isEqual = deepEqual({ a: 1, b: [2, 3] }, { a: 1, b: [2, 3] }); // true
   ```

2. **uid**: Generates a unique identifier.

   ```typescript
   import { uid } from "stalo/utils";
   
   const uniqueId = uid(); // Example: "1a2b3c4d"
   ```

---

## API Reference

### `create(initialState)`

- **Description**: Initializes a new store.
- **Arguments**:
  - `initialState`: The initial state value.
- **Returns**: `[useState, setState]`

### `compose(...middlewares)`

- **Description**: Composes multiple middlewares to enhance state management.
- **Arguments**:
  - `middlewares`: A list of middleware functions.

### `devtools(setFunction, options)`

- **Description**: Integrates state management with DevTools.
- **Arguments**:
  - `setFunction`: The set function of the store.
  - `options`: `{ name: string }` for naming the store in DevTools.

### `immerWithPatch(setFunction)`

- **Description**: Applies Immer.js and generates patches for state updates.
- **Arguments**:
  - `setFunction`: The set function of the store.

### Persistent State API

- **createPersistentStorage(initialValue)**
- **createLocalStorage(initialValue, key)**
