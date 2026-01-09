# DRY (Don't Repeat Yourself)

## Definition / Concept
**DRY** (Don't Repeat Yourself) is a fundamental architectural principle stating that every piece of knowledge should have a single, unambiguous representation within a system. The principle emphasizes reducing duplication of logic, data, and functionality across codebases to minimize maintenance burden and improve system reliability. By centralizing shared logic into reusable components or utilities, developers can ensure consistency and reduce the risk of bugs introduced through multiple implementations of the same functionality. This principle is applicable across all programming contexts—from frontend frameworks to backend systems to infrastructure as code.

- **Single source of truth**: Shared logic exists in one place, reducing inconsistencies
- **Reduced maintenance burden**: Changes to shared functionality need to be made only once
- **Improved code quality**: Less duplication means fewer bugs and easier debugging
- **Better scalability**: Systems are easier to extend and modify as they grow

## Visual Representation

```
┌─────────────────────────────────────────────────────────────────┐
│                    DRY PRINCIPLE VISUALIZATION                  │
└─────────────────────────────────────────────────────────────────┘

❌ VIOLATION (Multiple Implementations)
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  Component A     │    │  Component B     │    │  Service C       │
│  validateEmail() │    │  validateEmail() │    │  validateEmail() │
│  (3 places)      │    │  (different)     │    │  (different)     │
└──────────────────┘    └──────────────────┘    └──────────────────┘
        │                        │                        │
        └────────────┬───────────┴────────────┬───────────┘
                     │                        │
            Inconsistent behavior   Difficult to maintain
            Bug fixes needed in 3    Takes 3x longer to update
            different places


✅ DRY PRINCIPLE (Single Source of Truth)
┌────────────────────────────────────────┐
│        Shared Utility Module           │
│        validateEmail()                 │
│   (Single, canonical implementation)   │
└────────────────────────────────────────┘
        │              │              │
        ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ Component A  │ │ Component B  │ │  Service C   │
│ imports from │ │ imports from │ │ imports from │
│   shared     │ │   shared     │ │   shared     │
└──────────────┘ └──────────────┘ └──────────────┘

Benefits:
✓ One place to fix bugs
✓ Consistent behavior across application
✓ Easy to enhance or modify
✓ Reduced code size
```

## Example

### ❌ DRY Violation
```javascript
// Bad: validateEmail logic duplicated across multiple files

// src/components/SignupForm.js
function SignupForm() {
  const handleSubmit = (email) => {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!regex.test(email)) {
      console.error('Invalid email format');
      return false;
    }
    // Process signup
  };
  return <form onSubmit={handleSubmit}>...</form>;
}

// src/components/ProfileSettings.js
function ProfileSettings() {
  const updateEmail = (email) => {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!regex.test(email)) {
      alert('Invalid email');
      return false;
    }
    // Update profile
  };
  return <div>{/* form fields */}</div>;
}

// src/services/api.js
export function submitContactForm(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!regex.test(email)) {
    throw new Error('Email validation failed');
  }
  // Send to API
}

// Problems: 3 different regex patterns, inconsistent error handling,
// maintenance nightmare if validation rules change
```

### ✅ DRY Solution
```javascript
// src/utils/validators.js (Single source of truth)
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
const PHONE_REGEX = /^\d{3}-\d{3}-\d{4}$/;

export const validators = {
  email: (value) => {
    if (!value) return { valid: false, error: 'Email is required' };
    if (!EMAIL_REGEX.test(value)) {
      return { valid: false, error: 'Invalid email format' };
    }
    return { valid: true };
  },

  phone: (value) => {
    if (!value) return { valid: false, error: 'Phone is required' };
    if (!PHONE_REGEX.test(value)) {
      return { valid: false, error: 'Phone must be XXX-XXX-XXXX format' };
    }
    return { valid: true };
  },

  password: (value) => {
    if (value.length < 8) {
      return { valid: false, error: 'Password must be at least 8 characters' };
    }
    if (!/[A-Z]/.test(value)) {
      return { valid: false, error: 'Password must contain uppercase letter' };
    }
    return { valid: true };
  }
};

// src/components/SignupForm.js (Reuses validator)
import { validators } from '../utils/validators';

function SignupForm() {
  const handleSubmit = (email) => {
    const validation = validators.email(email);
    if (!validation.valid) {
      setError(validation.error);
      return false;
    }
    // Process signup
  };
  return <form onSubmit={handleSubmit}>...</form>;
}

// src/components/ProfileSettings.js (Reuses same validator)
import { validators } from '../utils/validators';

function ProfileSettings() {
  const updateEmail = (email) => {
    const validation = validators.email(email);
    if (!validation.valid) {
      alert(validation.error);
      return false;
    }
    // Update profile
  };
  return <div>{/* form fields */}</div>;
}

// src/services/api.js (Reuses same validator)
import { validators } from '../utils/validators';

export function submitContactForm(email) {
  const validation = validators.email(email);
  if (!validation.valid) {
    throw new Error(validation.error);
  }
  // Send to API
}

// Benefits: Single regex pattern, consistent error messages,
// one place to update validation rules
```

### Advanced: Custom Hooks (React DRY Pattern)
```javascript
// src/hooks/useFormValidation.js (Reusable form logic)
import { useState, useCallback } from 'react';
import { validators } from '../utils/validators';

export function useFormValidation(initialValues) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const validateField = useCallback((name, value) => {
    const validator = validators[name];
    if (!validator) return { valid: true };

    const result = validator(value);
    return result;
  }, []);

  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));

    const validation = validateField(name, value);
    setErrors(prev => ({
      ...prev,
      [name]: validation.valid ? '' : validation.error
    }));
  }, [validateField]);

  return {
    values,
    errors,
    handleChange,
    setValues,
    setErrors
  };
}

// Usage in multiple components - no duplication!
function LoginForm() {
  const form = useFormValidation({ email: '', password: '' });

  return (
    <form>
      <input name="email" value={form.values.email} onChange={form.handleChange} />
      {form.errors.email && <span>{form.errors.email}</span>}
      {/* more fields */}
    </form>
  );
}

function RegisterForm() {
  const form = useFormValidation({ email: '', password: '', phone: '' });

  return (
    <form>
      <input name="email" value={form.values.email} onChange={form.handleChange} />
      {form.errors.email && <span>{form.errors.email}</span>}
      {/* more fields */}
    </form>
  );
}
```

## Usage

- **When to use**:
  - When the same logic, pattern, or data structure appears in multiple places
  - When building reusable components or utilities that serve multiple features
  - When establishing shared constants, configurations, or validation rules
  - When creating utility functions or helper methods used across the application

- **Real-world example**:
  - A large e-commerce platform has user authentication logic that needs to be used in signup, login, password reset, and profile components. Instead of implementing authentication checks in each component, DRY principle suggests creating a centralized authentication service that all components can import and use.
  - An enterprise dashboard has consistent API error handling logic needed across 20+ different data fetching operations. Using a custom hook or error handler utility prevents duplicating error handling code in each request.

- **Best practices**:
  - Extract shared logic into utility functions, custom hooks, or shared services early in development
  - Use configuration files for constants (API endpoints, validation rules, feature flags)
  - Create composable, single-responsibility components that can be combined rather than duplicating markup
  - Implement common patterns using higher-order components, hooks, or mixins
  - Keep DRY in balance with KISS (Keep It Simple, Stupid)—don't over-engineer abstractions
  - Use clear naming conventions for shared utilities to make dependencies obvious
  - Document shared utilities and keep them maintainable as they're critical infrastructure

## FAQ / Interview Questions

**Q: What is the DRY principle and why is it important in software architecture?**
A: DRY (Don't Repeat Yourself) is the principle that every piece of knowledge should exist in exactly one place in the system. It's important because:
- Reduces maintenance burden—changes only need to be made in one location
- Improves consistency—shared logic behaves identically everywhere it's used
- Decreases bugs—less duplication means fewer places where bugs can hide
- Improves scalability—systems are easier to extend when logic is centralized
- Makes code reviews easier—reviewers can focus on the logic once rather than multiple implementations

**Q: How do you identify DRY violations in existing code?**
A: Look for these patterns:
- Copy-paste code blocks appearing multiple times with minimal changes
- Similar logic in different components with slightly different variable names
- Multiple implementations of the same business rule (e.g., three different email validators)
- Constants defined in multiple places that should be centralized
- Duplicated API call patterns across different services
- Tools like ESLint with DRY-focused plugins can help identify some violations automatically

**Q: Can you give an example of applying DRY to form validation?**
A: Instead of writing validation logic in each form component, create a centralized validators utility that defines all validation rules once. Each component imports and uses these validators, ensuring consistent behavior and making it easy to update validation rules in a single place. You can further apply DRY by creating a custom hook like `useFormValidation` that encapsulates the entire form handling logic (state management, validation, error display).

**Q: What's the difference between DRY and related principles like WET (Write Everything Twice)?**
A: DRY advocates eliminating repetition from the start, while WET is an anti-pattern that describes codebases with unnecessary duplication. The distinction matters because premature abstraction can violate the YAGNI principle (You Aren't Gonna Need It). However, DRY should generally guide you to extract logic once you identify a pattern repeating at least twice or when it's clear a pattern will repeat.

**Q: How do you balance DRY with maintainability and avoiding over-engineering?**
A: The key is to apply DRY pragmatically:
- Don't abstract after seeing code only once—wait until a pattern repeats
- Keep abstractions simple and focused on a single responsibility
- Ensure the abstraction doesn't make the code harder to understand
- Consider the cost of maintaining the abstraction versus the duplication
- Use the "Rule of Three": if you see the same code pattern three times, create an abstraction
- Prioritize clarity over cleverness—a slightly duplicated but clear solution beats an over-engineered one
- Document why shared utilities exist and how to use them

## References
- [MDN - DRY Principle](https://developer.mozilla.org/en-US/)
- [The Pragmatic Programmer - DRY](https://pragprog.com/)
- [Clean Code by Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Refactoring: Improving the Design of Existing Code by Martin Fowler](https://refactoring.com/)
- [SOLID Principles and Code Reusability](https://en.wikipedia.org/wiki/SOLID)

---
*See also: [SOLID Principles](./SOLID.md), [Code Reusability Patterns](./CodeReusability.md), [Custom Hooks](../React/CustomHooks.md), [Composition vs Inheritance](../JavaScript/CompositionVsInheritance.md)*
