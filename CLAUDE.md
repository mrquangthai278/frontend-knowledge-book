# CLAUDE.md - Markdown Generation Guide

This document provides instructions for automatically generating markdown files for the Frontend Handbook.

---

## ⚡ Quick Command

Use this simple format to create new topic markdown files:

**Format**:
```
Create md for [TopicName]
```
or
```
Create md for [TopicName] in [Category]
```

**Examples**:
```
Create md for Hoisting
Create md for BEM in CSS
Create md for Closure in JavaScript
Create md for Hooks in React
```

---

## 📝 Standard Markdown Structure

Every markdown file created follows this exact structure:

```markdown
# [Topic Name]

## Definition / Concept
[2-4 sentences explaining the core concept]
- Key point 1
- Key point 2
- Key point 3

## Visual Representation
[Optional: ASCII diagram or simple flowchart if helpful]

## Example
\`\`\`javascript
// Practical code example
// Shows how the concept works
// Use realistic scenarios
\`\`\`

## Usage
- **When to use**: [Specific use cases]
- **Real-world example**: [Practical application]
- **Best practices**: [How to apply effectively]

## FAQ / Interview Questions

**Q: [Common interview question]**
A: [Clear, concise answer with key points]

**Q: [Follow-up question]**
A: [Answer]

**Q: [Another common question]**
A: [Answer]

## References
- [MDN Web Docs - Topic](https://developer.mozilla.org/...)
- [Official Documentation](link)
- [Relevant Article](link)

---
*See also: [Related Topic](../Category/RelatedTopic.md)*
```

---

## ✅ Auto-Generation Checklist

When creating a file, these will be automatically included:

- ✅ File name in **PascalCase** (Closure.md, not closure.md)
- ✅ Placed in **correct category folder** (JavaScript, CSS, React, etc.)
- ✅ **Definition/Concept** section (2-4 sentences, with bold keywords)
- ✅ **Visual Representation** (if concept benefits from diagram/flowchart)
- ✅ **Example** section (1-2 practical code blocks with comments)
- ✅ **Usage** section (when, where, why, best practices)
- ✅ **FAQ / Interview Questions** (4-5 relevant questions with answers)
- ✅ **References** section (MDN, official docs, quality sources)
- ✅ **Related topics link** (See also section)
- ✅ All code is **syntactically correct**
- ✅ Keywords highlighted with **bold**
- ✅ Modern **ES6+ syntax** (unless discussing legacy)

---

## 📂 Category Auto-Detection

Topics are automatically assigned to categories:

### JavaScript/
Closure, Hoisting, Scope, Prototype, EventLoop, Callbacks, Promises, AsyncAwait, This, Destructuring, SpreadOperator, Typeof, Equality, etc.

### CSS/
BEM, BoxModel, Flexbox, Grid, ResponsiveDesign, Positioning, Specificity, Cascade, Inheritance, CSSVariables, Selectors, Pseudo-classes, etc.

### HTML/
SemanticHTML, Accessibility, FormValidation, HTMLStructure, Attributes, Metadata, etc.

### React/
Hooks, VirtualDOM, StateManagement, ComponentLifecycle, Props, Keys, Performance, Context, Refs, etc.

### Performance/
LazyLoading, CodeSplitting, BundleOptimization, CachingStrategies, WebVitals, ImageOptimization, etc.

and auto-detection.

---

## 📝 Writing Standards

### Definition/Concept
- Simple, clear explanation (avoid jargon)
- Why it matters in frontend development
- 2-4 sentences maximum
- **Bold** important keywords

**Good Example**:
"**Hoisting** is JavaScript's behavior of moving declarations to the top of their scope before code execution. Variables declared with `var` are hoisted with `undefined`, while `let` and `const` are hoisted but not initialized, causing a **Temporal Dead Zone**."

### Code Examples
- Modern ES6+ syntax
- Include explanatory comments
- Show both good and bad patterns when relevant
- Keep focused and concise
- Use realistic variable names

**Good Example**:
```javascript
// Bad: Using var causes unexpected behavior
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 3, 3, 3
}

// Good: Using let scopes i to each iteration
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 0, 1, 2
}
```

### Interview Questions (4-5 questions)
- Start with basic definition
- Include "why is it important"
- Add practical example questions
- Include technical follow-ups
- Focus on real interview questions

**Good Example**:
```
Q: What is a closure and why is it important?
A: A closure is a function that has access to variables from its outer scope
even after the outer function returns. This is important because:
- It enables data privacy and encapsulation
- It's the foundation for callbacks and higher-order functions
- Understanding closures is crucial for async code

Q: Can you give a practical example?
A: Event handlers use closures...
```

### References
- Always include MDN links when available
- Cite official documentation
- Include 2-4 quality sources
- Verify links are current

---

## 🚫 What's NOT Included

CLAUDE.md focuses **only** on markdown generation. For other information, see:
- **Project overview** → README.md
- **Learning paths** → README.md
- **Contribution guidelines** → README.md
- **Project structure** → README.md

---

## 🔄 What Claude Does Automatically

When you use the quick command:

✅ **Analyzes** the topic name to infer category
✅ **Creates** the file in the correct folder
✅ **Generates** complete Definition/Concept section
✅ **Includes** practical code examples
✅ **Writes** 4-5 interview questions with answers
✅ **Adds** proper references and links
✅ **Links** to related topics in "See also" section
✅ **Verifies** all code is syntactically correct
✅ **Formats** everything consistently

**No need to provide additional details** — just the topic name!

---

## 📋 Quality Standards

Every generated file will meet these standards:

- [ ] Follows the standard structure exactly
- [ ] Definition is clear and concise
- [ ] Examples are practical and working
- [ ] Code is syntactically correct
- [ ] Interview questions are relevant
- [ ] References are valid
- [ ] Keywords are bolded
- [ ] Related topics are linked
- [ ] Language is simple and clear
- [ ] Modern syntax is used

---

**Version**: 1.0
**Last Updated**: January 2026
