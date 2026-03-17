# TypeScript: Zero to Ship

> JavaScript with a spell-checker that catches your mistakes before your users do.

## Why Should You Care?

You already write JavaScript. It works. But JavaScript has a fundamental problem: it doesn't tell you when you're wrong until the code actually runs. You misspell a property name, you call a function with the wrong arguments, you try to access `.length` on something that's `undefined`. JavaScript says nothing. It waits until a user clicks a button, the code runs, and the app crashes. That's the feedback loop: write code, deploy, wait for the bug report.

TypeScript changes when you find out about mistakes. Instead of learning about errors from your users, you learn about them from your editor, in real time, as you type. That red squiggly line under `user.nmae` (you meant `user.name`) shows up instantly, not after a deploy. The feedback loop shrinks from hours to seconds.

If you're building with AI tools, TypeScript matters even more. When Cursor or Claude Code generates code, TypeScript validates it immediately. If the AI writes a function that expects a number but you're passing a string, TypeScript flags it before you even run the code. Without TypeScript, you're trusting the AI blindly. With TypeScript, the compiler is a second reviewer that catches the AI's mistakes (and yours) in real time. Every Next.js project, every modern React app, every serious Node.js backend uses TypeScript. It's the default now, not the exception.

## The 30-Second Version

TypeScript is JavaScript with type annotations. You tell the compiler what type each variable, parameter, and return value should be, and it checks that everything matches. `let name: string = 42` is an error because 42 isn't a string. But you don't have to annotate everything, because TypeScript infers most types automatically. It's JavaScript that knows about your data shapes, catches type mismatches before runtime, and provides autocompletion in your editor. It compiles down to plain JavaScript, so it runs anywhere JavaScript runs.

## The Real Explanation (ELI5)

Imagine you're writing a recipe. In plain English (JavaScript), you might write:

"Add the thing to the other thing and cook for some time."

This is valid English. A human might figure it out from context. But it's also how you end up adding sugar instead of salt, because "the thing" was ambiguous.

In a precise, typed recipe (TypeScript), you'd write:

"Add 2 teaspoons of salt (ingredient: Salt, amount: number, unit: Teaspoon) to the pot (container: Pot) and cook for 20 minutes (duration: number, unit: Minutes)."

More verbose? Yes. But nobody adds sugar by accident. And if someone writes "add 2 teaspoons of salt to the fork," the system immediately says "a Fork is not a valid container. Did you mean Pot or Pan?"

Now replace "recipe" with "code." Replace "salt" with `string`, "2 teaspoons" with `number`, and "pot" with a specific object shape. TypeScript is the system that checks your recipe for impossible instructions before you start cooking.

The analogy is accurate in another way too: you don't type every ingredient on every line. Once you say "let broth = makeChickenBroth()", TypeScript knows `broth` is of type `ChickenBroth` without you labeling it. It infers the type from the function's return type. You only need explicit annotations when TypeScript can't figure it out on its own.

## How It Actually Works

### What TypeScript Adds to JavaScript

TypeScript is a superset of JavaScript. Every valid JavaScript file is also valid TypeScript. TypeScript adds one thing: types. These types exist only during development. When you build your project, TypeScript compiles to plain JavaScript with all the type annotations stripped out. The browser or Node.js never sees your types.

```
TypeScript (what you write):
  const greet = (name: string): string => {
    return `Hello, ${name}`;
  };

JavaScript (what runs):
  const greet = (name) => {
    return `Hello, ${name}`;
  };
```

This means types have zero runtime cost. They don't make your app slower or bigger. They're a development tool, like a linter that understands your entire codebase.

### Basic Types

```typescript
// Primitives
const name: string = "Harshit";
const age: number = 28;
const isAdmin: boolean = true;

// Arrays
const scores: number[] = [98, 87, 92];
const names: string[] = ["Priya", "Rahul", "Sara"];

// Objects (inline)
const user: { name: string; email: string; age: number } = {
  name: "Harshit",
  email: "harshit@example.com",
  age: 28,
};

// null and undefined
const middleName: string | null = null;     // explicitly nullable
const nickname: string | undefined = undefined;

// The "any" escape hatch
const mystery: any = "could be anything";
// "any" turns off type checking. It's a surrender flag. Avoid it.
```

**The types you'll use 90% of the time:** `string`, `number`, `boolean`, `string[]`, `number[]`, and object types defined with interfaces or type aliases (next section). That's it. Don't let the advanced type system intimidate you. Most real-world TypeScript uses a small set of types composed together.

### Interfaces and Type Aliases

Both let you name an object shape so you can reuse it. They're almost interchangeable, with subtle differences.

**Interface (use for objects):**

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  role: "admin" | "editor" | "viewer";
  avatarUrl?: string;  // ? means optional
}

const user: User = {
  id: 42,
  name: "Priya",
  email: "priya@example.com",
  role: "admin",
  // avatarUrl is optional, can be omitted
};

// Interfaces can be extended
interface AdminUser extends User {
  permissions: string[];
  lastLogin: Date;
}
```

**Type alias (use for unions, intersections, and everything else):**

```typescript
// Union type (this OR that)
type Status = "pending" | "active" | "cancelled";

// Intersection type (this AND that)
type UserWithPosts = User & { posts: Post[] };

// Function type
type Formatter = (value: number) => string;

// Utility types
type PartialUser = Partial<User>;     // all fields optional
type UserName = Pick<User, "name" | "email">;  // only name and email
type UserWithoutId = Omit<User, "id">;  // everything except id
```

**When to use which:**

| Use | When |
|-----|------|
| `interface` | Defining object shapes (props, API responses, database records) |
| `type` | Union types (`"a" | "b"`), intersections, function signatures, utility types |
| Either | They're interchangeable for most object definitions. Pick one convention and stick with it. |

The practical difference: interfaces can be "declaration merged" (if you declare the same interface twice, they combine). Types can't. This matters for library authors. For app code, it doesn't matter. Use `interface` for objects, `type` for everything else, and don't overthink it.

### Union Types and Narrowing

Union types describe a value that could be one of several types. Narrowing is how you tell TypeScript which specific type it is.

```typescript
// Union type: could be a string or a number
type Id = string | number;

const printId = (id: Id) => {
  // TypeScript doesn't know if id is string or number here
  // console.log(id.toUpperCase()); // Error: toUpperCase doesn't exist on number

  // Narrowing: check the type, then TypeScript knows
  if (typeof id === "string") {
    console.log(id.toUpperCase());  // OK: TypeScript knows id is string here
  } else {
    console.log(id.toFixed(2));     // OK: TypeScript knows id is number here
  }
};
```

**Common narrowing techniques:**

```typescript
// typeof (for primitives)
if (typeof value === "string") { /* value is string */ }

// Truthiness (for null/undefined)
if (user) { /* user is not null/undefined */ }
if (user?.name) { /* user exists and has a name */ }

// "in" operator (for objects)
if ("email" in contact) { /* contact has an email property */ }

// instanceof (for classes)
if (error instanceof TypeError) { /* error is a TypeError */ }

// Discriminated unions (the most powerful pattern)
type Result =
  | { status: "success"; data: User }
  | { status: "error"; message: string };

const handleResult = (result: Result) => {
  if (result.status === "success") {
    console.log(result.data.name);    // TypeScript knows data exists here
  } else {
    console.log(result.message);      // TypeScript knows message exists here
  }
};
```

Discriminated unions are everywhere in well-typed codebases. They're how you model states (loading/success/error), actions (create/update/delete), and any situation where "what fields exist depends on some flag."

### Generics: Placeholder Types

Generics are types that take parameters. Think of them as functions for types.

```typescript
// Without generics: you'd need a separate function for each type
const firstString = (arr: string[]): string | undefined => arr[0];
const firstNumber = (arr: number[]): number | undefined => arr[0];

// With generics: one function works for any type
const first = <T>(arr: T[]): T | undefined => arr[0];

first(["a", "b", "c"]);  // TypeScript infers T = string, returns string | undefined
first([1, 2, 3]);         // TypeScript infers T = number, returns number | undefined
```

**Why `Array<string>` works:** `Array` is a generic type built into TypeScript. `Array<string>` means "an array where every element is a string." `Array<number>` means "an array where every element is a number." The `<string>` part is the type parameter. `string[]` is just shorthand for `Array<string>`.

**Practical generics you'll use:**

```typescript
// Generic function for API calls
const fetchData = async <T>(url: string): Promise<T> => {
  const response = await fetch(url);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json() as T;
};

// TypeScript knows `users` is User[]
const users = await fetchData<User[]>("/api/users");

// Generic with constraints
const getProperty = <T, K extends keyof T>(obj: T, key: K): T[K] => {
  return obj[key];
};

const user = { name: "Priya", age: 28 };
getProperty(user, "name");  // OK, returns string
getProperty(user, "age");   // OK, returns number
// getProperty(user, "foo"); // Error: "foo" is not a key of user
```

You don't need to write generics often. But you need to read them, because every library uses them. When you see `<T>`, just read it as "some type that gets filled in later."

### Type Inference

TypeScript figures out most types automatically. You don't need to annotate everything.

```typescript
// TypeScript infers these types automatically:
const name = "Harshit";           // type: string
const age = 28;                    // type: number
const scores = [98, 87, 92];      // type: number[]
const user = { name: "Priya" };   // type: { name: string }

// Function return types are inferred
const double = (n: number) => n * 2;  // return type inferred as number

// Array methods preserve types
const names = ["Priya", "Rahul", "Sara"];
const upper = names.map(n => n.toUpperCase());  // type: string[]
const lengths = names.map(n => n.length);        // type: number[]
```

**When you DO need explicit types:**

```typescript
// 1. Function parameters (TypeScript can't infer these)
const greet = (name: string) => `Hello, ${name}`;

// 2. Empty arrays (TypeScript doesn't know what will go in them)
const items: string[] = [];

// 3. API responses (TypeScript doesn't know what the server returns)
const data = await response.json() as User;

// 4. When inference gets it wrong
const config = { timeout: 5000 } as const;  // makes it readonly { timeout: 5000 } not { timeout: number }
```

**The rule:** Let TypeScript infer when it can. Only annotate when TypeScript can't figure it out, gets it wrong, or when the type serves as documentation (function signatures, API boundaries).

### Common Patterns with React

**Typing component props:**

```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary" | "danger";
  disabled?: boolean;
  children?: React.ReactNode;
}

const Button = ({ label, onClick, variant = "primary", disabled = false }: ButtonProps) => {
  return (
    <button onClick={onClick} disabled={disabled} className={variant}>
      {label}
    </button>
  );
};
```

**Typing useState:**

```typescript
// TypeScript infers the type from the initial value
const [count, setCount] = useState(0);                   // type: number
const [name, setName] = useState("Harshit");              // type: string

// Explicit type needed when initial value doesn't reveal the full type
const [user, setUser] = useState<User | null>(null);      // starts null, will be User later
const [items, setItems] = useState<string[]>([]);          // starts empty, will have strings
const [status, setStatus] = useState<"idle" | "loading" | "error">("idle");
```

**Typing event handlers:**

```typescript
// Form events
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
  // ...
};

// Input change
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setName(e.target.value);
};

// Click events
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  // ...
};

// Keyboard events
const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
  if (e.key === "Enter") submit();
};
```

**Typing API responses:**

```typescript
interface ApiResponse<T> {
  data: T;
  pagination: {
    page: number;
    total: number;
    hasMore: boolean;
  };
}

interface Contact {
  id: number;
  name: string;
  email: string;
  company: string | null;
}

const fetchContacts = async (page: number): Promise<ApiResponse<Contact[]>> => {
  const res = await fetch(`/api/contacts?page=${page}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
};

// Usage: TypeScript knows contacts is Contact[]
const { data: contacts, pagination } = await fetchContacts(1);
```

### The tsconfig Basics

`tsconfig.json` configures the TypeScript compiler. Most of it is set once and forgotten. Here are the flags that matter:

```jsonc
{
  "compilerOptions": {
    // The important ones:
    "strict": true,              // Enables ALL strict checks. Always turn this on.
    "target": "ES2022",          // What JS version to compile to
    "module": "ESNext",          // Module system (ESNext for modern projects)
    "moduleResolution": "bundler", // How to find imported files (use "bundler" for Vite/Next.js)
    "esModuleInterop": true,     // Fixes import compatibility issues
    "skipLibCheck": true,        // Skip type-checking node_modules (faster builds)

    // Path aliases (optional but nice):
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]         // import { Button } from "@/components/ui/button"
    },

    // Output:
    "outDir": "./dist",          // Where compiled JS goes
    "declaration": true,         // Generate .d.ts type declaration files

    // What files to include:
    "jsx": "react-jsx"           // For React projects
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**What "strict: true" actually enables:**

| Flag | What it does | Why it matters |
|------|-------------|----------------|
| `strictNullChecks` | `null` and `undefined` are their own types, not assignable to everything | Catches the most common runtime errors |
| `noImplicitAny` | Can't have untyped variables or parameters | Forces you to actually use the type system |
| `strictFunctionTypes` | Function parameter types are checked correctly | Prevents subtle callback bugs |
| `strictPropertyInitialization` | Class properties must be initialized | Prevents undefined property access |
| `noImplicitThis` | `this` must have a type | Prevents `this` confusion in callbacks |

`strict: true` is one flag that enables all of these. Always use it. If you're adding TypeScript to an existing project, you might need to start without it and enable strict checks one by one, but for new projects, start strict.

### Reading TypeScript Errors

TypeScript errors look intimidating but follow a pattern. Here's how to decode them.

**Error: "Type X is not assignable to type Y"**

This is the most common error. It means you're putting a value of one type where a different type is expected.

```typescript
const user: User = { name: "Priya", email: "priya@example.com" };
// Error: Property 'id' is missing in type '{ name: string; email: string; }'
//        but required in type 'User'.

// Translation: User requires an 'id' field and you didn't provide it.
// Fix: add the missing field.
```

**Error: "Property X does not exist on type Y"**

You're accessing a property that doesn't exist on that type.

```typescript
const user = { name: "Priya", email: "priya@example.com" };
console.log(user.age);
// Error: Property 'age' does not exist on type '{ name: string; email: string; }'.

// Translation: the object doesn't have an 'age' field.
// Fix: either add 'age' to the object, or you meant a different property.
```

**Error: "Object is possibly undefined/null"**

You're accessing something that might be null or undefined without checking first.

```typescript
const user: User | null = getUser();
console.log(user.name);
// Error: 'user' is possibly 'null'.

// Fix: check for null first
if (user) {
  console.log(user.name);  // OK
}
// Or use optional chaining
console.log(user?.name);    // OK, returns undefined if user is null
```

**Error: "Argument of type X is not assignable to parameter of type Y"**

You're passing the wrong type to a function.

```typescript
const greet = (name: string) => `Hello, ${name}`;
greet(42);
// Error: Argument of type 'number' is not assignable to parameter of type 'string'.

// Translation: the function expects a string but you gave it a number.
// Fix: pass a string, or change the function to accept numbers.
```

**The general approach to any TypeScript error:**
1. Read the last line first. It usually has the most specific information.
2. Identify the two types it's comparing (what you have vs what's expected).
3. Figure out which side to fix: change your value, or change the type definition.

## The Mental Model

**Mental model 1: "Types are contracts, not descriptions."** When you write `interface User { name: string; email: string }`, you're not describing what a user might look like. You're declaring a contract: every value of type User MUST have a name that's a string and an email that's a string. TypeScript enforces this contract at compile time, the same way a database enforces constraints at write time. If the data doesn't match the contract, the code doesn't compile.

**Mental model 2: "TypeScript is a spell-checker, not a grammar teacher."** It catches when you misspell a property, pass the wrong argument, or forget a null check. It doesn't tell you whether your code is well-designed, efficient, or correct in business logic. `user.age > 200` is valid TypeScript (age is a number, 200 is a number, comparison is valid). TypeScript doesn't know that 200 is unreasonable for a human age. Types catch mechanical errors, not logical ones.

**Mental model 3: "Every type is a set of possible values."** `string` is the set of all possible strings. `number` is the set of all possible numbers. `"admin" | "viewer"` is a set of exactly two strings. `User` is the set of all objects that have the required User properties. Assignability means "is this value a member of that set?" `42` is a member of `number` but not `string`. Understanding types as sets makes union types, intersection types, and narrowing intuitive.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Type annotation | Explicitly writing the type of something (`: string`, `: number`) | Variable and parameter declarations |
| Type inference | TypeScript figuring out the type without you writing it | Most variable assignments |
| Interface | A named description of an object's shape | Defining props, API responses, data models |
| Type alias | A name for any type (including unions, functions, primitives) | Union types, utility types |
| Union type | A value that could be one of several types (`string \| number`) | Nullable values, status fields |
| Narrowing | Checking which specific type a union value is (`if (typeof x === "string")`) | Handling nullable values, discriminated unions |
| Generic | A type that takes a type parameter (`Array<T>`, `Promise<T>`) | Utility functions, React hooks, API wrappers |
| `any` | A type that turns off type checking entirely | Escape hatch (avoid in production code) |
| `unknown` | Like `any` but safer: you must narrow before using it | API boundaries, JSON parsing |
| `never` | A type with no possible values (function that always throws, exhaustive switches) | Error handling, exhaustiveness checks |
| `as` | Type assertion: telling TypeScript "trust me, I know the type" | API responses, DOM elements |
| `as const` | Makes a value readonly with literal types | Configuration objects, constant arrays |
| `satisfies` | Checks a value matches a type without changing the inferred type | Configuration objects (preserves literal types) |
| Discriminated union | A union where each member has a literal type field that identifies it | State machines, API responses, event handling |
| `keyof` | A union of all property names of a type | Generic utility functions |
| Utility types | Built-in types like `Partial<T>`, `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>` | Transforming existing types |
| `.d.ts` file | A file that contains only type declarations (no implementation) | Library type definitions, ambient types |
| `strict` mode | A tsconfig flag that enables all strict type checks | Every new project (always enable) |

## Common Patterns

### 1. Typing API Responses

**When to use it:** Every API call in your app. Never trust that the server returns what you expect without typing it.

**How it works:** Define an interface for the expected response shape, use it when parsing the response, and optionally validate with Zod at runtime.

**Code sketch:**

```typescript
// Define the shape
interface Contact {
  id: number;
  name: string;
  email: string;
  company: string | null;
  createdAt: string;  // ISO date string from the API
}

interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
  };
}

// Type-safe fetch wrapper
const apiFetch = async <T>(path: string): Promise<T> => {
  const res = await fetch(`${API_BASE}${path}`, {
    headers: { Authorization: `Bearer ${getToken()}` },
  });
  if (!res.ok) {
    throw new Error(`API error: ${res.status}`);
  }
  return res.json() as Promise<T>;
};

// Usage: TypeScript knows the shape of the response
const { data: contacts, pagination } = await apiFetch<PaginatedResponse<Contact>>(
  "/contacts?page=1&limit=20"
);

// For runtime validation (catch API changes):
import { z } from "zod";

const ContactSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  company: z.string().nullable(),
  createdAt: z.string(),
});

const validated = ContactSchema.parse(rawData);
// Now `validated` is both type-safe AND runtime-validated
```

### 2. Typing React Component Props

**When to use it:** Every React component. Props are the contract between a parent and child component.

**How it works:** Define an interface for props. TypeScript checks that every usage of the component passes valid props.

**Code sketch:**

```typescript
interface CardProps {
  title: string;
  description?: string;
  variant?: "default" | "highlighted" | "muted";
  onClick?: () => void;
  children: React.ReactNode;
}

const Card = ({
  title,
  description,
  variant = "default",
  onClick,
  children,
}: CardProps) => {
  return (
    <div className={`card card-${variant}`} onClick={onClick}>
      <h3>{title}</h3>
      {description && <p>{description}</p>}
      {children}
    </div>
  );
};

// Usage: TypeScript catches mistakes immediately
<Card title="Hello" variant="highlighted">Content</Card>  // OK
<Card title={42}>Content</Card>                             // Error: number is not string
<Card variant="big">Content</Card>                          // Error: "big" is not a valid variant
<Card>Content</Card>                                        // Error: 'title' is required

// Extending HTML element props
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

const Input = ({ label, error, ...rest }: InputProps) => (
  <div>
    <label>{label}</label>
    <input {...rest} />
    {error && <span className="error">{error}</span>}
  </div>
);

// Gets all native input props (type, placeholder, onChange, etc.) for free
<Input label="Email" type="email" placeholder="you@example.com" />
```

### 3. Typing Form State

**When to use it:** Any form with multiple fields that submits data to an API.

**How it works:** Define the form data shape, use it for state and the submit handler. TypeScript ensures the form state matches what the API expects.

**Code sketch:**

```typescript
interface ContactForm {
  name: string;
  email: string;
  company: string;
  phone: string;
}

// Initial state matches the interface
const emptyForm: ContactForm = {
  name: "",
  email: "",
  company: "",
  phone: "",
};

const ContactFormPage = () => {
  const [form, setForm] = useState<ContactForm>(emptyForm);
  const [errors, setErrors] = useState<Partial<Record<keyof ContactForm, string>>>({});

  // Type-safe field updater
  const updateField = <K extends keyof ContactForm>(field: K, value: ContactForm[K]) => {
    setForm(prev => ({ ...prev, [field]: value }));
    setErrors(prev => ({ ...prev, [field]: undefined }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    // form is guaranteed to match ContactForm shape
    const response = await createContact(form);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={form.name}
        onChange={e => updateField("name", e.target.value)}
      />
      {errors.name && <span>{errors.name}</span>}
      {/* ... */}
    </form>
  );
};
```

### 4. Generic Utility Functions

**When to use it:** Reusable functions that work with different data types (grouping, sorting, filtering, transforming).

**How it works:** Use generics so the function works with any type while preserving type safety.

**Code sketch:**

```typescript
// Group an array by a key
const groupBy = <T, K extends string | number>(
  items: T[],
  getKey: (item: T) => K,
): Record<K, T[]> => {
  const groups = {} as Record<K, T[]>;
  for (const item of items) {
    const key = getKey(item);
    if (!groups[key]) groups[key] = [];
    groups[key].push(item);
  }
  return groups;
};

// TypeScript infers the return type correctly
const contactsByCompany = groupBy(contacts, c => c.company ?? "No company");
// type: Record<string, Contact[]>

// Sort by any property
const sortBy = <T>(items: T[], key: keyof T, order: "asc" | "desc" = "asc"): T[] => {
  return [...items].sort((a, b) => {
    if (a[key] < b[key]) return order === "asc" ? -1 : 1;
    if (a[key] > b[key]) return order === "asc" ? 1 : -1;
    return 0;
  });
};

const sorted = sortBy(contacts, "name", "asc");
// TypeScript ensures "name" is actually a key of Contact
```

### 5. Discriminated Unions for State Machines

**When to use it:** Any component or function that has distinct states with different data available in each state (loading/success/error, form steps, API request lifecycle).

**How it works:** A union type where each member has a literal type discriminator field. TypeScript narrows to the correct member when you check the discriminator.

**Code sketch:**

```typescript
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string; retryCount: number };

const ContactList = () => {
  const [state, setState] = useState<RequestState<Contact[]>>({ status: "idle" });

  const fetchContacts = async () => {
    setState({ status: "loading" });
    try {
      const data = await apiFetch<Contact[]>("/contacts");
      setState({ status: "success", data });
    } catch (err) {
      setState({
        status: "error",
        error: err instanceof Error ? err.message : "Unknown error",
        retryCount: state.status === "error" ? state.retryCount + 1 : 0,
      });
    }
  };

  // TypeScript narrows the type in each branch
  switch (state.status) {
    case "idle":
      return <button onClick={fetchContacts}>Load contacts</button>;
    case "loading":
      return <Spinner />;
    case "success":
      // TypeScript knows state.data exists here
      return <ul>{state.data.map(c => <li key={c.id}>{c.name}</li>)}</ul>;
    case "error":
      // TypeScript knows state.error and state.retryCount exist here
      return (
        <div>
          <p>Error: {state.error}</p>
          <button onClick={fetchContacts}>Retry ({state.retryCount})</button>
        </div>
      );
  }
};
```

This is one of the most powerful TypeScript patterns. It makes impossible states impossible. You can't access `state.data` when `status` is `"loading"`, because TypeScript knows `data` doesn't exist in that branch.

## Mistakes Everyone Makes

### 1. Using "any" everywhere to make errors go away

**What people do:** Encounter a TypeScript error, add `: any` or `as any` to silence it, and move on.

**Why it seems right:** "The code works. TypeScript is just being annoying."

**What actually happens:** Every `any` is a hole in your type safety. If a function parameter is `any`, TypeScript can't catch wrong arguments. If an API response is `any`, TypeScript can't catch missing fields. You end up with a codebase that has TypeScript syntax but no TypeScript benefits. The errors you silenced will return as runtime bugs.

**What to do instead:** Fix the actual type error. If you don't know the type, use `unknown` instead of `any`. `unknown` forces you to narrow before using the value, which is exactly the check you should be making.

```typescript
// BAD: any turns off all checking
const data: any = await response.json();
console.log(data.user.name); // No error, even if data has no user

// GOOD: unknown forces you to validate
const data: unknown = await response.json();
if (isUser(data)) {
  console.log(data.name); // Safe, you've validated
}

// GOOD: type assertion with a known shape
const data = (await response.json()) as User;
```

### 2. Over-typing things TypeScript can already infer

**What people do:** Add type annotations to everything, even when TypeScript already knows the type.

**Why it seems right:** "Being explicit is better than being implicit."

**What actually happens:** The code becomes cluttered with redundant annotations. When you refactor, you have to update both the value and the annotation. And sometimes the annotation is wrong (specifying a broader type than what's actually there), which hides useful information from TypeScript.

**What to do instead:** Let TypeScript infer. Only annotate function parameters, return types at API boundaries, and cases where inference gets it wrong.

```typescript
// OVER-TYPED: TypeScript already knows all of these
const name: string = "Harshit";
const numbers: number[] = [1, 2, 3];
const doubled: number[] = numbers.map((n: number): number => n * 2);

// JUST RIGHT: only annotate what TypeScript can't infer
const name = "Harshit";
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);

// DO annotate function parameters (TypeScript can't infer these)
const greet = (name: string) => `Hello, ${name}`;

// DO annotate when the initial value doesn't reveal the type
const items: string[] = [];
const user: User | null = null;
```

### 3. Not typing API responses (trusting the backend blindly)

**What people do:** `const data = await response.json()` and start accessing properties without defining any types.

**Why it seems right:** "The backend team said it returns a user object."

**What actually happens:** The backend changes a field name from `createdAt` to `created_at`. Or a field that was always present becomes optional. Or a new field is added that your code doesn't handle. Without types, you find out when the page crashes. With types, you find out when the contract changes and your IDE shows errors at every usage site.

**What to do instead:** Define an interface for every API response. Use it when parsing the response. For maximum safety, validate at runtime with Zod.

```typescript
// Type definition
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: string;
}

// Typed fetch
const user = await apiFetch<User>("/users/42");

// Even better: runtime validation
const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string(),
  createdAt: z.string(),
});

type User = z.infer<typeof UserSchema>;  // derive the type from the schema
const user = UserSchema.parse(await response.json());
// Now it's both compile-time AND runtime safe
```

### 4. Fighting the type system instead of redesigning the data shape

**What people do:** Write increasingly complex type assertions, casts, and workarounds to force their data through types that don't fit.

**Why it seems right:** "The types should work with my data, not the other way around."

**What actually happens:** You end up with `(data as any as User).name` or five lines of type gymnastics for what should be a simple access. The complexity is a signal: your data shape doesn't match what your code expects. The fix isn't more type magic. The fix is to reshape the data.

**What to do instead:** If types are fighting you, the problem is usually your data structure, not TypeScript. Common fixes:

```typescript
// FIGHTING: API returns { first_name, last_name } but your component wants { name }
// BAD: casting and concatenating everywhere
const name = (user as any).first_name + " " + (user as any).last_name;

// GOOD: transform at the boundary
interface ApiUser {
  first_name: string;
  last_name: string;
}

interface User {
  name: string;
  email: string;
}

const toUser = (apiUser: ApiUser): User => ({
  name: `${apiUser.first_name} ${apiUser.last_name}`,
  email: apiUser.email,
});

// Transform once, use the clean type everywhere
const user = toUser(await apiFetch<ApiUser>("/user/42"));
```

### 5. Ignoring strict mode

**What people do:** Start a project with `strict: false` (or don't set it, which defaults to false) because strict mode has too many errors.

**Why it seems right:** "I'll enable it later when the codebase is more mature."

**What actually happens:** "Later" never comes. Without strict mode, TypeScript doesn't check for null/undefined (the most common source of runtime errors), doesn't require explicit types on parameters (so many things silently become `any`), and doesn't catch several classes of function signature errors. You're writing TypeScript but getting 30% of the benefit. It's like wearing a seatbelt but not clicking it in.

**What to do instead:** Start every new project with `strict: true`. For existing projects migrating from JavaScript, enable strict checks incrementally:

```jsonc
{
  "compilerOptions": {
    // Start with these (least disruptive):
    "noImplicitAny": true,
    "strictNullChecks": true,

    // Then add these:
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true,

    // Eventually:
    "strict": true  // enables everything
  }
}
```

### 6. Using type assertions (`as`) when narrowing would be safer

**What people do:** `const input = document.getElementById("email") as HTMLInputElement` everywhere.

**Why it seems right:** "I know it's an input element."

**What actually happens:** If the element doesn't exist or is a different type, the assertion silently lies. `null as HTMLInputElement` doesn't crash at the assertion. It crashes later when you access `.value` on null. Assertions bypass type checking. They're "trust me" statements that can be wrong.

**What to do instead:** Narrow with runtime checks when possible. Use assertions only when you genuinely can't narrow (and add a comment explaining why).

```typescript
// BAD: assertion can be wrong
const input = document.getElementById("email") as HTMLInputElement;
input.value; // crashes if element doesn't exist

// GOOD: narrow with a check
const element = document.getElementById("email");
if (element instanceof HTMLInputElement) {
  element.value; // safe, TypeScript knows it's HTMLInputElement
}

// ACCEPTABLE: when you control the DOM and know it exists
const input = document.getElementById("email") as HTMLInputElement | null;
if (!input) throw new Error("Email input not found");
input.value; // safe after the null check
```

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Enable strict mode on every new project.** `"strict": true` in tsconfig.json. Don't start without it. The errors it catches are worth the extra annotations.

2. **Let TypeScript infer. Only annotate what it can't figure out.** Function parameters: annotate. Variable assignments: don't. Function return types: annotate at module boundaries, skip for internal functions. Over-annotation is noise.

3. **Type every API response at the boundary.** Define an interface for every endpoint's response. Ideally validate at runtime with Zod. Never let `any` flow from an API call into your components.

4. **Use `unknown` instead of `any` for truly unknown values.** `unknown` forces you to check the type before using it. `any` lets you do anything, which means TypeScript can't help you.

5. **Prefer discriminated unions for state that has different shapes in different conditions.** Instead of `{ data?: T; error?: string; loading: boolean }` (where invalid states are possible), use `{ status: "loading" } | { status: "success"; data: T } | { status: "error"; error: string }`.

6. **Use `interface` for objects, `type` for unions and intersections.** Pick this convention and be consistent. Both work for objects, but using them differently by purpose makes code easier to scan.

7. **Read the last line of TypeScript errors first.** Long error messages have the most specific information at the end. "Type A is not assignable to type B" tells you exactly what's wrong. The lines above are the context leading to the conflict.

8. **Transform data at the boundary, not throughout your code.** If the API returns snake_case and your components use camelCase, convert once in your API layer. Don't scatter type assertions across your codebase.

## The "Just Tell Me What to Do" Quickstart

Add TypeScript to a simple Node.js project in under 5 minutes.

**Step 1: Create a project**

```bash
mkdir ts-demo && cd ts-demo
pnpm init
```

**Step 2: Install TypeScript**

```bash
pnpm add -D typescript @types/node tsx
```

`tsx` is a tool that runs TypeScript files directly without a separate compile step. Great for development.

**Step 3: Create tsconfig.json**

```bash
npx tsc --init
```

Edit it to be minimal:

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"]
}
```

**Step 4: Write some TypeScript**

Create `src/index.ts`:

```typescript
interface Todo {
  id: number;
  title: string;
  completed: boolean;
  createdAt: Date;
}

const todos: Todo[] = [];
let nextId = 1;

const addTodo = (title: string): Todo => {
  const todo: Todo = {
    id: nextId++,
    title,
    completed: false,
    createdAt: new Date(),
  };
  todos.push(todo);
  return todo;
};

const completeTodo = (id: number): Todo | undefined => {
  const todo = todos.find(t => t.id === id);
  if (todo) {
    todo.completed = true;
  }
  return todo;
};

const listTodos = (filter?: "all" | "active" | "completed"): Todo[] => {
  switch (filter) {
    case "active":
      return todos.filter(t => !t.completed);
    case "completed":
      return todos.filter(t => t.completed);
    default:
      return [...todos];
  }
};

// Try it
addTodo("Learn TypeScript");
addTodo("Build something");
completeTodo(1);

console.log("All:", listTodos());
console.log("Active:", listTodos("active"));
console.log("Completed:", listTodos("completed"));
```

**Step 5: Run it**

```bash
npx tsx src/index.ts
```

**Step 6: Try breaking things**

Add these lines and watch TypeScript complain:

```typescript
addTodo(42);                    // Error: number is not string
completeTodo("one");            // Error: string is not number
listTodos("cancelled");          // Error: "cancelled" is not a valid filter
const t: Todo = { title: "x" }; // Error: missing id, completed, createdAt
```

Each error is TypeScript catching a bug before it runs. That's the value.

## How to Prompt AI Tools About This

### What context to give AI tools

When asking Claude Code, Cursor, or Copilot to write TypeScript, always include:
- The framework (React, Next.js, Node.js, Express)
- Whether strict mode is enabled (it should be)
- The data shapes involved (paste your interfaces or describe the structure)
- Whether you're using Zod for runtime validation
- If it's React: whether it's a Server Component or Client Component

### Example prompts that produce good results

**For typing an API layer:**
```
Write a type-safe API client for these endpoints:
- GET /contacts returns { data: Contact[], pagination: { page, total, hasMore } }
- GET /contacts/:id returns Contact
- POST /contacts takes { name: string, email: string, company?: string }
- PUT /contacts/:id takes Partial<Contact>

Define all interfaces. Use a generic fetch wrapper. Include error handling.
TypeScript strict mode is on.
```

**For typing a React component:**
```
Create a TypeScript React component for a data table.
Props: data (array of objects), columns (config for which fields to show
and how to format them), onRowClick (optional callback), sortable (boolean).
The data type should be generic so the table works with any data shape.
Include proper typing for the column config (each column references a key
of the data type).
```

**For fixing type errors:**
```
I'm getting this TypeScript error:
"Type '{ name: string; }' is not assignable to type 'User'.
Property 'email' is missing in type '{ name: string; }'
but required in type 'User'."

Here's my User interface: [paste it]
Here's the code that causes the error: [paste it]
What's wrong and how do I fix it?
```

### What to watch out for in AI-generated TypeScript

- **Overuse of `any`.** AI reaches for `any` when the correct type is complex. Ask it to use the proper type or `unknown`.
- **Missing strict null checks.** AI generates `user.name` without checking if `user` might be null. Ensure null handling.
- **Type assertions instead of narrowing.** AI writes `as User` instead of runtime checks. Ask for narrowing.
- **Overly complex generic types.** AI sometimes generates generics with 4 type parameters when a simpler approach works. If a type is hard to read, ask for a simpler version.
- **Missing React event types.** AI sometimes uses `(e: any)` for event handlers instead of `React.ChangeEvent<HTMLInputElement>`.

### Key terms that improve output quality

- "Use strict TypeScript" (gets proper null checks and no implicit any)
- "Use discriminated unions for state" (gets proper state modeling)
- "Define interfaces for all data shapes" (gets typed API boundaries)
- "Don't use any" (forces the AI to find the real type)
- "Use Zod for runtime validation" (gets both compile-time and runtime safety)

## Ship It: Build This

### Convert a JavaScript Project to Strict TypeScript

Take an existing small JavaScript project (a todo app, a CLI tool, a utility library) and convert it to TypeScript with strict mode enabled. Fix every type error along the way.

**Why this project:** Converting JS to TS is the single best way to understand what TypeScript actually does. You'll see the errors TypeScript catches, understand why each annotation exists, and develop the skill of reading type errors. Unlike starting fresh, migration forces you to confront every type decision the original code made implicitly.

**Rough architecture:**
- Start with a working JavaScript project (~200-400 lines). If you don't have one, use a simple Express/Fastify API with 3-4 endpoints and in-memory data.
- Rename `.js` files to `.ts` one by one
- Enable strict mode from the start
- Fix every error, one at a time, without using `any`
- Add interfaces for all data shapes
- Type all function signatures
- Add Zod validation at API boundaries

**Key steps to follow:**
1. Install TypeScript and create tsconfig.json with `strict: true`
2. Rename the entry file from `.js` to `.ts`
3. Fix errors top to bottom: untyped parameters first, then null checks, then complex types
4. Define interfaces for your core data (User, Todo, Order, whatever the domain is)
5. Type all function parameters and return types
6. Add `| null` or `| undefined` where values are genuinely optional
7. Use discriminated unions for any state that has different shapes
8. Validate external input (API requests, file reads, env vars) with Zod
9. Run `npx tsc --noEmit` and confirm zero errors
10. Add a build script to package.json

**Estimated time:** 2-3 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn conditional types** when you need types that change based on other types (`T extends string ? A : B`). Used heavily in library code and advanced utility types, rarely needed in app code.
- **Learn template literal types** when you need to type string patterns like route paths (`/users/${string}/posts`), CSS values, or event names. Powerful for type-safe routing.
- **Learn `satisfies`** when you want TypeScript to check a value matches a type without widening the inferred type. It's the "best of both worlds" between `as const` and type annotation.
- **Learn module augmentation** when you need to add types to third-party libraries (adding custom properties to Express's Request object, extending Next.js's Session type).
- **Learn the `infer` keyword** when you need to extract types from other types (getting the return type of a function, the element type of an array). This is TypeScript's most powerful and most confusing feature.
