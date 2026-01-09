# TypeScript Basics

## Definition / Concept

**TypeScript** is a **superset of JavaScript** that adds **static typing** to the language. It allows you to specify what types variables, parameters, and return values should have, enabling TypeScript to catch type-related errors at **compile-time** before your code runs. TypeScript code is compiled to plain JavaScript, which means it runs in any JavaScript environment (browsers, Node.js, etc.). Understanding TypeScript basics is essential for writing type-safe code, leveraging IDE support for better autocomplete and refactoring, and building more maintainable applications.

- **Static typing**: Types checked before code execution
- **Compilation**: TypeScript compiles to JavaScript
- **Type annotations**: Explicit or inferred type declarations
- **JavaScript compatible**: All valid JavaScript is valid TypeScript
- **IDE support**: Better autocomplete, refactoring, and error detection

## Visual Representation

```
TYPESCRIPT WORKFLOW

┌─────────────────────────────────────────────────┐
│           TypeScript Source Code                 │
│         (with type annotations)                  │
└─────────────────────────────────────────────────┘
                      │
                      ▼
         ┌──────────────────────────┐
         │  TypeScript Compiler     │
         │  - Type checking         │
         │  - Error detection       │
         │  - Compilation           │
         └──────────────────────────┘
                      │
        ┌─────────────┴─────────────┐
        │                           │
    ✅ Success                   ❌ Errors
        │                           │
        ▼                           ▼
  ┌──────────────┐         ┌──────────────┐
  │ JavaScript   │         │ Fix & Retry  │
  │ Output       │         │              │
  └──────────────┘         └──────────────┘
        │
        ▼
  ┌──────────────────────────┐
  │ Execute in Browser/Node  │
  │ (plain JavaScript)       │
  └──────────────────────────┘

TYPE SYSTEM BASICS

Primitives: string, number, boolean
Special: any, unknown, null, undefined, never, void
Collections: Array, Tuple
Custom: Interface, Type, Class, Enum

ANY vs UNKNOWN:
─────────────────
any    → Opt out (no type checks)
unknown → Type-safe (must narrow type)
```

## Example

```typescript
// ===== BASIC TYPE ANNOTATIONS =====

// Variables with types
const name: string = "Alice";
const age: number = 30;
const isActive: boolean = true;

// Type inference (no annotation needed)
const message = "Hello";  // TypeScript infers string
const count = 42;         // TypeScript infers number

// ===== FUNCTION TYPES =====

// Function with parameters and return type
function greet(name: string, age: number): string {
  return `Hello ${name}, you are ${age} years old`;
}

// Arrow function
const add = (a: number, b: number): number => a + b;

// Optional parameters (?)
function sendEmail(to: string, cc?: string): void {
  console.log(`Sending to ${to}`);
  if (cc) console.log(`CC: ${cc}`);
}

// Default parameters
function createUser(name: string, role: string = "user"): void {
  console.log(`User ${name} with role ${role}`);
}

// Rest parameters (...)
function sum(...numbers: number[]): number {
  return numbers.reduce((a, b) => a + b, 0);
}

// ===== INTERFACE =====

interface User {
  id: number;
  name: string;
  email: string;
  age?: number;  // optional property
}

const user: User = {
  id: 1,
  name: "Bob",
  email: "bob@example.com"
  // age is optional
};

// ===== ARRAY TYPES =====

// Array of strings
const fruits: string[] = ["apple", "banana"];
const numbers: Array<number> = [1, 2, 3];

// Array of objects
const users: User[] = [
  { id: 1, name: "Alice", email: "alice@example.com" },
  { id: 2, name: "Bob", email: "bob@example.com" }
];

// ===== UNION TYPES =====

type Status = 'pending' | 'success' | 'error';
type Value = string | number;

function processStatus(status: Status): void {
  if (status === 'success') {
    console.log("Operation succeeded");
  }
}

// ===== LITERAL TYPES =====

type Direction = 'up' | 'down' | 'left' | 'right';
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';

function navigate(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

// ===== TUPLE TYPES =====

type Tuple = [string, number];
const tuple: Tuple = ["hello", 42];

type Response = [status: number, message: string];
const response: Response = [200, "OK"];

// ===== ENUM TYPES =====

enum Color {
  Red = 0,
  Green = 1,
  Blue = 2
}

enum Status {
  Pending = "pending",
  Active = "active",
  Inactive = "inactive"
}

let currentColor: Color = Color.Red;
let currentStatus: Status = Status.Active;

// ===== TYPE ALIASES =====

type Point = {
  x: number;
  y: number;
};

type ID = string | number;

const point: Point = { x: 10, y: 20 };
const userId: ID = "user-123";

// ===== CLASSES =====

class Animal {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  makeSound(): void {
    console.log("Generic animal sound");
  }
}

class Dog extends Animal {
  breed: string;

  constructor(name: string, age: number, breed: string) {
    super(name, age);
    this.breed = breed;
  }

  makeSound(): void {
    console.log("Woof!");
  }
}

// ===== ACCESS MODIFIERS =====

class BankAccount {
  public accountNumber: string;
  private balance: number;  // only accessible inside class
  protected accountType: string;  // accessible in subclasses

  constructor(accountNumber: string, balance: number) {
    this.accountNumber = accountNumber;
    this.balance = balance;
    this.accountType = "savings";
  }

  getBalance(): number {
    return this.balance;
  }
}

// ===== ANY AND UNKNOWN =====

// ❌ any - opts out of type checking (avoid)
let value: any = "string";
value.toUpperCase();  // no error even if wrong method

// ✅ unknown - type-safe (preferred)
let unknownValue: unknown = "string";
// unknownValue.toUpperCase();  // ❌ Error - must narrow type first

if (typeof unknownValue === 'string') {
  unknownValue.toUpperCase();  // ✅ Now safe
}

// ===== NULL AND UNDEFINED =====

// With strictNullChecks enabled
let nullable: string | null = null;
let optional: string | undefined = undefined;

// Without null/undefined in type
let required: string = "value";
// required = null;  // ❌ Error with strict mode

// ===== CONST ASSERTIONS =====

const config = {
  api: "https://api.example.com",
  timeout: 5000
} as const;  // ✅ immutable, types are literals

// Without as const - type is wider
const config2 = {
  api: "https://api.example.com",
  timeout: 5000
};  // api is string, timeout is number

// ===== GENERIC BASICS =====

function identity<T>(value: T): T {
  return value;
}

const str = identity<string>("hello");
const num = identity<number>(42);
const inferred = identity("auto");  // T inferred as string

// Generic with interface
interface Box<T> {
  value: T;
  getValue(): T;
}

const stringBox: Box<string> = {
  value: "hello",
  getValue() { return this.value; }
};

// ===== COMPILATION CONFIGURATION =====

// tsconfig.json example
{
  "compilerOptions": {
    "target": "ES2020",              // JavaScript version
    "module": "commonjs",             // Module system
    "strict": true,                   // Enable all strict checks
    "esModuleInterop": true,          // CommonJS/ES interop
    "skipLibCheck": true,             // Skip type checking .d.ts files
    "forceConsistentCasingInFileNames": true  // Case-sensitive imports
  }
}
```

## Usage

- **When to use TypeScript**:
  - Large codebases where type safety prevents bugs
  - Team projects where types serve as documentation
  - Building libraries or frameworks
  - Projects where IDE support and refactoring are important
  - Any JavaScript project where you want better error detection

- **Real-world example**:
  ```typescript
  // User management service
  interface User {
    id: number;
    name: string;
    email: string;
    createdAt: Date;
  }

  class UserService {
    private users: User[] = [];
    private nextId: number = 1;

    addUser(name: string, email: string): User {
      const user: User = {
        id: this.nextId++,
        name,
        email,
        createdAt: new Date()
      };
      this.users.push(user);
      return user;
    }

    getUserById(id: number): User | null {
      return this.users.find(user => user.id === id) || null;
    }

    getAllUsers(): User[] {
      return [...this.users];
    }

    deleteUser(id: number): boolean {
      const index = this.users.findIndex(user => user.id === id);
      if (index === -1) return false;
      this.users.splice(index, 1);
      return true;
    }
  }

  // Usage
  const userService = new UserService();
  const newUser = userService.addUser("Alice", "alice@example.com");
  const found = userService.getUserById(newUser.id);
  const allUsers = userService.getAllUsers();
  ```

- **Best practices**:
  - Enable **strict mode** (`"strict": true` in tsconfig.json)
  - Use **type annotations** for function parameters and return types
  - Prefer **interfaces** for object shapes, **types** for complex definitions
  - Use **const** instead of **let** for immutable data
  - Avoid **any** type; use **unknown** and narrow types instead
  - Use **optional chaining** (`?.`) and **nullish coalescing** (`??`)
  - Keep types **simple and readable**—complexity should be justified
  - Test your types with different inputs to catch edge cases

## FAQ / Interview Questions

**Q: What is TypeScript and how does it differ from JavaScript?**
A: TypeScript is a superset of JavaScript that adds static typing. JavaScript allows any type to be assigned to any variable (dynamic typing), while TypeScript requires you to specify types (static typing). TypeScript is compiled to JavaScript before running, catching type errors during compilation rather than at runtime. All JavaScript is valid TypeScript, so you can gradually adopt TypeScript in existing projects.

**Q: Why should I use TypeScript?**
A: TypeScript provides several benefits: (1) **Early error detection**—catch type errors before runtime, (2) **Better IDE support**—autocomplete, refactoring, and navigation, (3) **Self-documenting code**—types serve as documentation, (4) **Easier refactoring**—types ensure changes don't break dependencies, (5) **Scalability**—types help maintain large codebases.

**Q: What's the difference between type annotations and type inference?**
A: Type annotations explicitly declare a type: `const name: string = "Alice"`. Type inference automatically determines a type from the value: `const name = "Alice"`. TypeScript infers `name` is a string. Inference is preferred when obvious; annotations are useful for clarity or when inference can't determine the type accurately.

**Q: How do I set up TypeScript in a project?**
A: Install TypeScript (`npm install --save-dev typescript`), create a `tsconfig.json` file with compiler options, write `.ts` files, and compile with `tsc`. Most modern frameworks (React, Node.js, Next.js) have built-in TypeScript support. Configuration is usually automatic in frameworks.

**Q: What does the `any` type do and why should I avoid it?**
A: `any` tells TypeScript to skip type checking for that value—it disables type safety. Use `any` only as a last resort when you can't express a type properly. Instead, use `unknown` (which requires type narrowing) or invest time in expressing the correct type. `any` defeats the purpose of TypeScript.

**Q: How do strict null checks work?**
A: With `strictNullChecks: true` in tsconfig.json, `null` and `undefined` are not assignable to regular types. To allow them, use a union: `string | null`. This prevents null reference errors at compile-time. It's highly recommended to enable strict null checks for safer code.

## References

- [TypeScript Official Website](https://www.typescriptlang.org/)
- [TypeScript Handbook - The Basics](https://www.typescriptlang.org/docs/handbook/2/basic-types.html)
- [TypeScript Handbook - Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [TypeScript Handbook - tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

---

*See also: [Types](./Types.md), [Generics](./Generics.md), [Guards](./Guards.md), [Interface](./Interface.md), [AdvancedTypes](./AdvancedTypes.md)*
