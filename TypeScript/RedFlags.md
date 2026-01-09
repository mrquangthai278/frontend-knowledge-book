# TypeScript Red Flags

## Definition / Concept

**Red flags** in TypeScript are coding patterns, practices, and anti-patterns that indicate potential problems, maintainability issues, or type safety vulnerabilities. These warning signs don't necessarily cause immediate errors but can lead to bugs, performance issues, and difficult refactoring down the line. Recognizing red flags helps developers write more robust, maintainable code and avoid common pitfalls. Understanding what constitutes a red flag is crucial for code reviews, architectural decisions, and catching issues before they cause problems in production.

- **Anti-patterns**: Practices that look convenient but cause problems
- **Type safety bypasses**: Using `any`, unsafe casts, or skipping type checks
- **Code smells**: Patterns that suggest deeper architectural issues
- **Maintainability risks**: Code that's hard to refactor or understand
- **Performance concerns**: Patterns that can cause unexpected slowdowns
- **Type inference failures**: Situations where types become too broad or too narrow

## Visual Representation

```
RED FLAG CATEGORIES

┌──────────────────────────────────────────────────────┐
│              TypeScript Red Flags                     │
└──────────────────────────────────────────────────────┘
        │
    ┌───┴───────────────────────────────────┐
    │                                       │
┌───▼──────────┐  ┌─────────────┐  ┌──────▼──────────┐
│ TYPE SAFETY  │  │   PATTERNS  │  │ MAINTAINABILITY│
│   ISSUES     │  │   & SMELLS  │  │    RISKS       │
├──────────────┤  ├─────────────┤  ├────────────────┤
│ - any usage  │  │ - Magic     │  │ - Implicit any │
│ - unsafe as  │  │   numbers   │  │ - Dead code    │
│ - ! operator │  │ - Hardcoded │  │ - Duplication  │
│ - @ts-ignore │  │   values    │  │ - Deep nesting │
└──────────────┘  │ - Loose     │  │ - God objects  │
                  │   coupling  │  └────────────────┘
                  └─────────────┘

SEVERITY LEVELS:
🔴 Critical: Will cause bugs or security issues
🟠 High: Causes maintainability/scalability problems
🟡 Medium: Could be problematic in larger codebases
🟢 Low: Style preference, minor refactor
```

## Example

```typescript
// ===== 🔴 RED FLAG: EXCESSIVE 'any' TYPE =====

// ❌ BAD - Uses any, defeats type safety
function processData(data: any): any {
  return data.value * 2;
}

// ✅ GOOD - Explicit type
function processData(data: { value: number }): number {
  return data.value * 2;
}

// ❌ BAD - any in array
const results: any[] = fetchResults();
results.forEach(item => console.log(item.property));

// ✅ GOOD - Specific type
interface Result {
  property: string;
}
const results: Result[] = fetchResults();
results.forEach(item => console.log(item.property));

// ===== 🔴 RED FLAG: UNSAFE TYPE CASTING =====

// ❌ BAD - Unsafe 'as' cast
const user = JSON.parse(data) as User;
// What if JSON doesn't match User shape?

// ✅ GOOD - Validate before casting
function parseUser(data: unknown): User {
  if (!isValidUser(data)) {
    throw new Error('Invalid user data');
  }
  return data as User;
}

// ✅ EVEN BETTER - Use type guard
function isValidUser(data: unknown): data is User {
  return (
    typeof data === 'object' &&
    data !== null &&
    'name' in data &&
    'email' in data
  );
}

// ===== 🔴 RED FLAG: USING NON-NULL ASSERTION (!) =====

// ❌ BAD - ! suppresses null check without reason
function getUser(id: number): User | null {
  return users.find(u => u.id === id) || null;
}

const user = getUser(1)!;  // Assumes non-null
console.log(user.name);    // Could crash if user is null

// ✅ GOOD - Check before access
function getUser(id: number): User | null {
  return users.find(u => u.id === id) || null;
}

const user = getUser(1);
if (user) {
  console.log(user.name);
}

// ===== 🟠 RED FLAG: @ts-ignore DIRECTIVE =====

// ❌ BAD - Ignoring type errors
// @ts-ignore
const value: string = 123;  // Type error, but ignored

// ✅ GOOD - Fix the actual error
const value: number = 123;

// ⚠️ Only acceptable in rare cases with explanation
// @ts-ignore - Third-party library has incomplete types
const result = externalLibrary.process(data);

// ===== 🟠 RED FLAG: IMPLICIT ANY =====

// ❌ BAD - Function parameter lacks type
function calculate(a, b) {  // a and b are implicitly any
  return a + b;
}

// ✅ GOOD - Explicit types
function calculate(a: number, b: number): number {
  return a + b;
}

// ❌ BAD - No return type annotation
function fetchData() {
  return api.get('/data');  // Return type unclear
}

// ✅ GOOD - Explicit return type
function fetchData(): Promise<Data[]> {
  return api.get('/data');
}

// ===== 🟠 RED FLAG: MAGIC NUMBERS AND STRINGS =====

// ❌ BAD - Magic values scattered in code
function processOrder(status: number): string {
  if (status === 1) return "Pending";
  if (status === 2) return "Shipped";
  if (status === 3) return "Delivered";
}

// ✅ GOOD - Use enums or constants
enum OrderStatus {
  Pending = 1,
  Shipped = 2,
  Delivered = 3
}

function processOrder(status: OrderStatus): string {
  switch (status) {
    case OrderStatus.Pending: return "Pending";
    case OrderStatus.Shipped: return "Shipped";
    case OrderStatus.Delivered: return "Delivered";
  }
}

// ===== 🟠 RED FLAG: HARDCODED VALUES =====

// ❌ BAD - Hardcoded config values
async function fetchData() {
  const response = await fetch('https://api.example.com/data');
  return response.json();
}

// ✅ GOOD - Use config/environment
const API_URL = process.env.API_URL || 'https://api.example.com';

async function fetchData(): Promise<Data> {
  const response = await fetch(`${API_URL}/data`);
  return response.json();
}

// ===== 🟠 RED FLAG: OVERLY BROAD TYPES =====

// ❌ BAD - Object type too broad
function displayUser(user: object): void {
  console.log((user as any).name);
}

// ✅ GOOD - Specific interface
interface User {
  id: number;
  name: string;
  email: string;
}

function displayUser(user: User): void {
  console.log(user.name);
}

// ===== 🟠 RED FLAG: DEEPLY NESTED TYPES =====

// ❌ BAD - Hard to read and maintain
type ApiResponse = {
  status: number;
  data: {
    users: {
      items: Array<{
        id: number;
        profile: {
          name: string;
          contact: {
            email: string;
            phone?: string;
          };
        };
      }>;
      total: number;
    };
  };
};

// ✅ GOOD - Break into smaller, named types
interface Contact {
  email: string;
  phone?: string;
}

interface UserProfile {
  name: string;
  contact: Contact;
}

interface User {
  id: number;
  profile: UserProfile;
}

interface UserList {
  items: User[];
  total: number;
}

interface ApiResponse {
  status: number;
  data: {
    users: UserList;
  };
}

// ===== 🟠 RED FLAG: LOOSE COUPLING / TOO FLEXIBLE =====

// ❌ BAD - Accepts anything, too flexible
function saveData(data: any, format: any): any {
  // Could do anything
}

// ✅ GOOD - Clear, strict types
type DataFormat = 'json' | 'csv' | 'xml';

interface Data {
  id: number;
  content: string;
}

function saveData(data: Data, format: DataFormat): Promise<void> {
  // Clear expectations
}

// ===== 🟡 RED FLAG: UNNECESSARY TYPE CASTING =====

// ❌ BAD - Redundant casting
const value: string = "hello" as string;
const num: number = 42 as number;

// ✅ GOOD - Let inference work
const value = "hello";
const num = 42;

// ===== 🟡 RED FLAG: MISSING NULL CHECKS =====

// ❌ BAD - Assumes value exists
function getName(user: User | null): string {
  return user.name;  // Crashes if user is null
}

// ✅ GOOD - Check before access
function getName(user: User | null): string {
  return user?.name || "Unknown";
}

// ===== 🟡 RED FLAG: DUPLICATE CODE =====

// ❌ BAD - Similar validation repeated
function validateUser(data: any) {
  if (!data.name || typeof data.name !== 'string') throw new Error();
  if (!data.email || typeof data.email !== 'string') throw new Error();
}

function validateAdmin(data: any) {
  if (!data.name || typeof data.name !== 'string') throw new Error();
  if (!data.email || typeof data.email !== 'string') throw new Error();
  if (!data.permissions || !Array.isArray(data.permissions)) throw new Error();
}

// ✅ GOOD - DRY principle with inheritance
interface BaseUser {
  name: string;
  email: string;
}

interface Admin extends BaseUser {
  permissions: string[];
}

function validateBaseUser(data: unknown): data is BaseUser {
  return (
    typeof data === 'object' &&
    data !== null &&
    'name' in data &&
    typeof data.name === 'string' &&
    'email' in data &&
    typeof data.email === 'string'
  );
}

// ===== 🟢 RED FLAG: GOD OBJECTS =====

// ❌ BAD - Single massive interface
interface AppState {
  users: User[];
  posts: Post[];
  comments: Comment[];
  notifications: Notification[];
  cache: Map<string, any>;
  settings: Settings;
  // ... 50 more properties
}

// ✅ GOOD - Separated concerns
interface UserState {
  users: User[];
}

interface ContentState {
  posts: Post[];
  comments: Comment[];
}

interface NotificationState {
  notifications: Notification[];
}

interface AppState {
  user: UserState;
  content: ContentState;
  notifications: NotificationState;
  // ... etc
}

// ===== 🟢 RED FLAG: INCONSISTENT NAMING =====

// ❌ BAD - Inconsistent conventions
interface user_data {  // snake_case
  firstName: string;   // camelCase
  last_name: string;   // snake_case
  Email: string;       // PascalCase
}

// ✅ GOOD - Consistent conventions
interface UserData {
  firstName: string;
  lastName: string;
  email: string;
}

// ===== 🔴 RED FLAG: MISSING ERROR HANDLING =====

// ❌ BAD - No error handling
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// ✅ GOOD - Proper error handling
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error(`Failed to fetch user: ${response.statusText}`);
  }
  return response.json();
}
```

## Usage

- **When to watch for red flags**:
  - Code reviews and pull requests
  - Refactoring legacy code
  - Onboarding to new codebases
  - Architecture discussions
  - Performance optimization reviews
  - Scaling applications

- **Real-world scenario**:
  ```typescript
  // Red flag audit checklist

  // ❌ Before - Multiple red flags
  class UserService {
    private users: any[] = [];

    getUser(id: any): any {
      return this.users.find(u => u.id == id);  // loose equality
    }

    addUser(data: any): any {
      // @ts-ignore - no validation
      this.users.push(data);
      return data!;
    }

    deleteUser(id: number) {
      const index = this.users.findIndex(u => u.id === id);
      if (index !== -1) {
        this.users.splice(index, 1);
      }
    }
  }

  // ✅ After - Red flags resolved
  interface User {
    id: number;
    name: string;
    email: string;
  }

  interface CreateUserInput {
    name: string;
    email: string;
  }

  function isValidUser(data: unknown): data is User {
    return (
      typeof data === 'object' &&
      data !== null &&
      'id' in data && typeof (data as any).id === 'number' &&
      'name' in data && typeof (data as any).name === 'string' &&
      'email' in data && typeof (data as any).email === 'string'
    );
  }

  class UserService {
    private users: User[] = [];
    private nextId = 1;

    getUser(id: number): User | null {
      return this.users.find(u => u.id === id) || null;
    }

    addUser(input: CreateUserInput): User {
      const user: User = {
        id: this.nextId++,
        ...input
      };
      this.users.push(user);
      return user;
    }

    deleteUser(id: number): boolean {
      const index = this.users.findIndex(u => u.id === id);
      if (index === -1) return false;
      this.users.splice(index, 1);
      return true;
    }
  }
  ```

- **Best practices for avoiding red flags**:
  - Enable **strict mode** in tsconfig.json
  - Use a **linter** (ESLint with TypeScript plugin) to catch issues
  - Perform **code reviews** focusing on type safety
  - Use **type checking tools** like TypeScript strict flag
  - **Refactor incrementally**—break large types into smaller ones
  - **Document decisions**—why certain patterns were chosen
  - **Test edge cases**—especially null/undefined scenarios
  - **Use type guards**—avoid trusting external data

## FAQ / Interview Questions

**Q: Why is using `any` considered a red flag?**
A: `any` disables type checking for that value, defeating TypeScript's purpose of catching errors at compile-time. It hides potential bugs, makes refactoring risky, and reduces IDE support. Use `unknown` instead and narrow types with guards. `any` should only be used as a last resort.

**Q: When is it acceptable to use the non-null assertion operator (!)?**
A: The `!` operator tells TypeScript to treat a value as non-null without runtime checks. It's acceptable only when you're **certain** the value isn't null and the check is expensive or impossible to express. Even then, prefer type guards. Most `!` usage indicates a design problem—reconsider the types or logic.

**Q: What does "implicit any" mean and why is it problematic?**
A: Implicit any occurs when TypeScript infers a type as `any` because no annotation is provided and inference fails. This happens with function parameters or return types without annotations. It's problematic because TypeScript can't catch errors. Solve it by enabling `noImplicitAny` in tsconfig.json and adding explicit type annotations.

**Q: How do "god objects" become a red flag?**
A: A god object is an interface or class with too many responsibilities and properties. It's hard to understand, difficult to test, and violates single responsibility principle. It indicates the need to break the object into smaller, focused types. Large types are a sign the code needs refactoring.

**Q: Why are magic numbers/strings red flags?**
A: Magic numbers (like `1`, `2`, `3`) and strings are unclear and scattered throughout code. They're hard to maintain—changing the meaning requires finding all occurrences. Use named constants, enums, or config objects instead. This makes code self-documenting and easier to refactor.

**Q: What's the difference between a code smell and a red flag?**
A: A **code smell** is a surface-level indication something might be wrong (like duplicate code). A **red flag** is a stronger warning that something is likely problematic (like excessive `any` usage). Red flags demand action; code smells might be refactored when convenient. Both indicate the code needs review.

## References

- [TypeScript Handbook - Best Practices](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)
- [TypeScript Deep Dive - Type Safety](https://basarat.gitbook.io/typescript/type-system)
- [ESLint TypeScript Plugin](https://typescript-eslint.io/)
- [TypeScript Strict Mode Guide](https://www.typescriptlang.org/tsconfig#strict)
- [Code Smells and Anti-patterns](https://refactoring.guru/refactoring/smells)

---

*See also: [Basics](./Basics.md), [Types](./Types.md), [Guards](./Guards.md), [StrictMode](./StrictMode.md), [AdvancedTypes](./AdvancedTypes.md)*
