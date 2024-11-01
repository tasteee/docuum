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

---

# `create()` API Reference

The `create()` function is the core of Stalo's state management solution. It initializes a new store with an initial state and provides two functions: `useState` and `setState`.

---

## Function Signature

```typescript
const [useState, setState] = create(initialState);
```

- **`initialState`**: The initial state for your store. This can be of any type: primitive, object, array, etc.
- **Returns**: An array containing:
  - `useState`: A hook for accessing the state.
  - `setState`: A function for updating the state.

---

## `useState` Hook

The `useState` function returned by `create()` is a custom React hook that allows components to subscribe to the store's state. This hook can be used in two primary ways: without arguments or with a selector function.

### Usage

1. **Accessing the Entire State**

   ```typescript
   const [useState] = create(initialState);

   function MyComponent() {
     const state = useState(); // Access the entire state

     return <div>{JSON.stringify(state)}</div>;
   }
   ```

2. **Accessing Part of the State with a Selector**

   You can pass a selector function to `useState` to subscribe to only a specific part of the state. This is useful for optimizing re-renders by only subscribing to the necessary portion of the state.

   ```typescript
   const [useState] = create({ count: 0, user: { name: "Alice" } });

   function CountComponent() {
     const count = useState((state) => state.count); // Access only the 'count' property

     return <div>Count: {count}</div>;
   }

   function UserComponent() {
     const userName = useState((state) => state.user.name); // Access only the 'user.name' property

     return <div>User: {userName}</div>;
   }
   ```

### Arguments

- **`selector` (optional)**: A function that receives the state and returns a selected portion of it.
- **Returns**: The current state or the selected part of the state.

### Behavior

- When `useState()` is called without a selector, the component will subscribe to the entire state, meaning it will re-render whenever any part of the state changes.
- When `useState(selector)` is called with a selector, the component will only re-render if the selected part of the state changes.

---

## `setState` Function

The `setState` function returned by `create()` is used to update the state. It supports both direct updates with a new value and updates using a function that receives the current state.

### Usage

1. **Setting State Directly**

   ```typescript
   const [useState, setState] = create(0);

   function IncrementButton() {
     return <button onClick={() => setState(1)}>Set to 1</button>;
   }
   ```

2. **Using a Function to Update State**

   When you need to update the state based on the current state, pass a function to `setState`. This function receives the current state as an argument.

   ```typescript
   const [useState, setState] = create(0);

   function Counter() {
     const count = useState();
     
     return (
       <button onClick={() => setState((prev) => prev + 1)}>
         Count: {count}
       </button>
     );
   }
   ```

3. **Setting State with Context (optional)**

   `setState` also supports an optional context argument, which can be useful for passing additional metadata when updating the state.

   ```typescript
   const [useState, setState] = create(0);

   function UpdateWithMeta() {
     // Set state and pass an optional context object
     setState(2, { reason: "manual update" });
   }
   ```

### Arguments

- **`newState`**: The new state or a function that receives the current state and returns the updated state.
- **`context` (optional)**: Additional context information for the state update.

### Behavior

- If `newState` is a value, it replaces the current state.
- If `newState` is a function, it receives the current state and should return the updated state. This is useful for deriving the next state based on the current state.

---

## Examples

### Basic Counter Example

```typescript
import { create } from "stalo";

const [useCounter, setCounter] = create(0);

function Counter() {
  const count = useCounter();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCounter(count + 1)}>Increment</button>
    </div>
  );
}
```

### Updating State Based on Current State

```typescript
import { create } from "stalo";

const [useCounter, setCounter] = create(0);

function Counter() {
  const count = useCounter();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCounter((current) => current + 1)}>Increment</button>
      <button onClick={() => setCounter((current) => current - 1)}>Decrement</button>
    </div>
  );
}
```

### Using a Selector to Optimize Re-renders

```typescript
import { create } from "stalo";

const [useAppState, setAppState] = create({ count: 0, name: "Alice" });

function CountComponent() {
  // Only subscribe to changes in 'count'
  const count = useAppState((state) => state.count);

  return <div>Count: {count}</div>;
}

function NameComponent() {
  // Only subscribe to changes in 'name'
  const name = useAppState((state) => state.name);

  return <div>Name: {name}</div>;
}
```

---

## Summary

- **`useState()`**: Hook to access the store's state. It can be used without arguments to get the entire state or with a selector to get a specific part of the state.
- **`setState()`**: Function to update the store's state. You can pass a new state value directly or a function that derives the new state from the current state. It also supports an optional context argument.

------

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
