# MVVM

## Definition / Concept

**MVVM** (Model-View-ViewModel) is an architectural pattern that separates an application into three interconnected layers: the **Model** (data and logic), the **View** (UI), and the **ViewModel** (intermediate layer that manages state and commands). The key innovation of MVVM is **two-way data binding**, which automatically synchronizes the View and ViewModel, reducing the need for imperative view updates and improving code maintainability.

- Separates concerns between data logic and presentation
- Enables automatic synchronization between UI and data through data binding
- Facilitates unit testing by decoupling View from business logic

## Visual Representation

```
┌─────────────────────────────────────────┐
│              USER (Input/Events)        │
└────────────────┬────────────────────────┘
                 │
                 ▼
        ┌────────────────┐
        │      VIEW      │ (HTML, UI Components)
        │   (Markup)     │
        └────────────────┘
               ▲  │
    Two-Way   │  │ Data Binding
    Binding   │  │ (Computed Properties)
               │  ▼
        ┌────────────────┐
        │   VIEWMODEL    │ (State, Commands, Logic)
        │   (Logic)      │
        └────────────────┘
               │
               ▼
        ┌────────────────┐
        │     MODEL      │ (Data, Business Logic)
        │    (Data)      │
        └────────────────┘
```

## Example

```javascript
// MODEL: Business logic and data
class UserModel {
  constructor() {
    this.users = [];
  }

  addUser(name, email) {
    const user = { id: Date.now(), name, email };
    this.users.push(user);
    return user;
  }

  removeUser(id) {
    this.users = this.users.filter(u => u.id !== id);
  }
}

// VIEWMODEL: Manages state and commands
class UserViewModel {
  constructor(model) {
    this.model = model;
    this.name = '';
    this.email = '';
    this.userList = [];
    this.updateUserList();
  }

  addUser() {
    if (this.name && this.email) {
      this.model.addUser(this.name, this.email);
      this.name = '';
      this.email = '';
      this.updateUserList();
    }
  }

  removeUser(id) {
    this.model.removeUser(id);
    this.updateUserList();
  }

  updateUserList() {
    this.userList = [...this.model.users];
  }
}

// VIEW: HTML with data binding (Vue.js example)
const app = {
  data() {
    const model = new UserModel();
    return {
      viewModel: new UserViewModel(model)
    };
  },
  template: `
    <div>
      <input v-model="viewModel.name" placeholder="Name">
      <input v-model="viewModel.email" placeholder="Email">
      <button @click="viewModel.addUser()">Add User</button>

      <ul>
        <li v-for="user in viewModel.userList" :key="user.id">
          {{ user.name }} ({{ user.email }})
          <button @click="viewModel.removeUser(user.id)">Delete</button>
        </li>
      </ul>
    </div>
  `
};
```

## Usage

- **When to use**: Applications requiring clear separation between UI and business logic, especially with frameworks that support data binding (Vue, Angular, WPF)
- **Real-world example**: E-commerce product filters where user input automatically updates displayed products; complex forms with dynamic validation feedback
- **Best practices**: Keep ViewModels focused on presentation logic, avoid putting business logic in Views, ensure Models are framework-agnostic for reusability

## FAQ / Interview Questions

**Q: What is MVVM and how does it differ from MVC?**
A: MVVM separates the application into Model, View, and ViewModel. Unlike MVC where the Controller directly updates the View, MVVM uses **two-way data binding** to automatically synchronize the ViewModel with the View. This reduces boilerplate and makes the ViewModel more testable since it doesn't reference the View directly.

**Q: How does two-way data binding work in MVVM?**
A: Two-way data binding means changes in the View automatically update the ViewModel, and changes in the ViewModel automatically update the View. Frameworks like Vue and Angular detect property changes and propagate updates in both directions through binding mechanisms, eliminating manual DOM manipulation.

**Q: What's the main advantage of using MVVM over MVC?**
A: The main advantage is **reduced coupling** between the UI and business logic. The ViewModel acts as an intermediary that exposes data and commands, allowing Views to be completely replaced without changing the ViewModel. This makes applications more maintainable, testable, and easier to refactor.

**Q: Is the ViewModel allowed to reference the View?**
A: No. The ViewModel should never directly reference the View. It only exposes properties and commands. The View binds to these properties, but the ViewModel remains framework-agnostic. This is crucial for testability—you can test the ViewModel without a UI framework.

**Q: Name some frameworks that use MVVM.**
A: Vue.js (uses MVVM with v-model and computed properties), Angular (with components as Views and services as ViewModels), WPF (Microsoft's desktop framework), and Xamarin (for cross-platform mobile development). Each implements two-way data binding differently, but the pattern remains consistent.

## References
- [MDN Web Docs - MVC Architecture](https://developer.mozilla.org/en-US/docs/Glossary/MVC)
- [Vue.js Official Documentation - Declarative Rendering](https://vuejs.org/guide/essentials/declarations.html)
- [Angular Guide - Architecture Overview](https://angular.io/guide/architecture)
- [MVVM Pattern - Microsoft Docs](https://docs.microsoft.com/en-us/archive/msdn-magazine/2009/february/patterns-wpf-apps-with-the-model-view-viewmodel-design-pattern)

---

*See also: [SOLID](./SOLID.md), [OOP](./OOP.md)*
