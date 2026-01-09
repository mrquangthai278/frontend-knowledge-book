# Event Delegation

## Definition / Concept
**Event delegation** is a JavaScript programming **technique/pattern** where instead of attaching event listeners to multiple child elements, you attach a single event listener to a parent element and use **event bubbling** (a browser/DOM feature) to handle events from its descendants. This leverages the fact that most events bubble up through the DOM tree, allowing you to handle events efficiently with fewer event listeners.

- **Event delegation** = JavaScript technique (how you write code)
- **Event bubbling** = Browser/DOM feature (how events propagate)
- Uses event bubbling to catch events from child elements at a parent level
- Reduces memory usage by minimizing the number of event listeners
- Handles dynamically added elements automatically
- Uses `event.target` to identify which child element triggered the event

## Visual Representation
```
Without Event Delegation:
<ul>
  <li> [listener] </li>    ← Event Listener
  <li> [listener] </li>    ← Event Listener
  <li> [listener] </li>    ← Event Listener
  <li> [listener] </li>    ← Event Listener
</ul>
Multiple listeners = High memory usage

With Event Delegation:
<ul> [listener]            ← Single Event Listener
  <li></li>                ↑ Event bubbles up
  <li></li>                ↑ Event bubbles up
  <li></li>                ↑ Event bubbles up
  <li></li>                ↑ Event bubbles up
</ul>
One listener = Efficient
```

## Example
```javascript
// ❌ Without delegation - inefficient
const items = document.querySelectorAll('.item');
items.forEach(item => {
  item.addEventListener('click', (e) => {
    console.log('Item clicked:', e.target.textContent);
  });
});
// Problem: 100 items = 100 event listeners

// ✅ With delegation - efficient
const list = document.querySelector('.item-list');
list.addEventListener('click', (e) => {
  // Check if clicked element is an item
  if (e.target.classList.contains('item')) {
    console.log('Item clicked:', e.target.textContent);
  }
});
// Solution: 1 event listener handles all items

// Real-world example: Todo list
const todoList = document.getElementById('todo-list');

todoList.addEventListener('click', (e) => {
  // Delete button clicked
  if (e.target.classList.contains('delete-btn')) {
    const todoItem = e.target.closest('.todo-item');
    todoItem.remove();
  }
  
  // Checkbox clicked
  if (e.target.classList.contains('todo-checkbox')) {
    const todoItem = e.target.closest('.todo-item');
    todoItem.classList.toggle('completed');
  }
  
  // Edit button clicked
  if (e.target.classList.contains('edit-btn')) {
    const todoItem = e.target.closest('.todo-item');
    editTodo(todoItem);
  }
});

// Works automatically with dynamically added items
function addTodo(text) {
  const li = document.createElement('li');
  li.className = 'todo-item';
  li.innerHTML = `
    <input type="checkbox" class="todo-checkbox">
    <span>${text}</span>
    <button class="edit-btn">Edit</button>
    <button class="delete-btn">Delete</button>
  `;
  todoList.appendChild(li);
  // No need to attach listeners - delegation handles it!
}

// Using matches() for complex selectors
document.addEventListener('click', (e) => {
  if (e.target.matches('.card .delete-icon')) {
    // Handle delete for elements matching selector
  }
  
  if (e.target.matches('button[data-action="submit"]')) {
    // Handle submit buttons
  }
});

// Handling different event types
const container = document.querySelector('.container');

container.addEventListener('click', (e) => {
  if (e.target.matches('.button')) {
    console.log('Button clicked');
  }
});

container.addEventListener('change', (e) => {
  if (e.target.matches('input[type="checkbox"]')) {
    console.log('Checkbox changed:', e.target.checked);
  }
});
```

## Usage
- **When to use**: 
  - Lists with many items (tables, menus, galleries)
  - Dynamically added/removed elements
  - Repeating elements with similar behavior
  - Improving performance with many event listeners
  - Single-page applications with frequent DOM updates
  
- **Real-world example**:
  ```javascript
  // Data table with actions
  const table = document.querySelector('.data-table');
  
  table.addEventListener('click', (e) => {
    const row = e.target.closest('tr');
    if (!row) return;
    
    const id = row.dataset.id;
    
    // Handle different actions
    if (e.target.matches('.btn-edit')) {
      editRecord(id);
    } else if (e.target.matches('.btn-delete')) {
      deleteRecord(id);
    } else if (e.target.matches('.btn-view')) {
      viewRecord(id);
    }
  });
  
  // Image gallery with lightbox
  const gallery = document.querySelector('.gallery');
  
  gallery.addEventListener('click', (e) => {
    if (e.target.matches('.gallery-image')) {
      openLightbox(e.target.src);
    }
  });
  
  // Form validation
  const form = document.querySelector('form');
  
  form.addEventListener('input', (e) => {
    if (e.target.matches('input[required]')) {
      validateField(e.target);
    }
  });
  
  form.addEventListener('blur', (e) => {
    if (e.target.matches('input, textarea')) {
      showValidationMessage(e.target);
    }
  }, true); // Use capture phase for blur
  
  // Navigation menu
  const nav = document.querySelector('.nav-menu');
  
  nav.addEventListener('click', (e) => {
    if (e.target.matches('.nav-link')) {
      e.preventDefault();
      
      // Remove active from all
      nav.querySelectorAll('.nav-link').forEach(link => {
        link.classList.remove('active');
      });
      
      // Add active to clicked
      e.target.classList.add('active');
      
      // Load content
      loadPage(e.target.dataset.page);
    }
  });
  ```

- **Best practices**:
  - Use `e.target.closest()` to find parent elements
  - Check `e.target.matches()` for flexible selector matching
  - Use data attributes for storing element-specific data
  - Consider event bubbling behavior (some events don't bubble)
  - Guard against null with early returns
  - Use capture phase for events that don't bubble (focus, blur)

## FAQ / Interview Questions

**Q: What is event delegation and why is it useful?**
A: **Event delegation** is attaching a single event listener to a parent element to handle events from its children, leveraging event bubbling. Benefits include:
- **Performance**: Fewer event listeners = less memory usage
- **Dynamic elements**: Automatically handles newly added elements
- **Simpler code**: One listener instead of many
- **Better performance**: Less DOM manipulation when adding/removing elements
Example: One listener on `<ul>` handles clicks on all `<li>` children.

**Q: How does event delegation work with event bubbling?**
A: When an event occurs on an element, it first runs handlers on that element, then on its parent, then grandparent, and so on up to the document root. This is **event bubbling**. Event delegation uses this by:
1. Placing a listener on a parent element
2. When child is clicked, event bubbles up to parent
3. Parent's listener checks `event.target` to identify which child triggered the event
4. Handler executes appropriate logic based on the target

**Q: What's the difference between event.target and event.currentTarget?**
A: 
- **event.target**: The element that triggered the event (the actual clicked element)
- **event.currentTarget**: The element that has the event listener attached

Example:
```javascript
ul.addEventListener('click', (e) => {
  console.log(e.target);        // <li> (clicked element)
  console.log(e.currentTarget); // <ul> (element with listener)
});
```
In delegation, `currentTarget` is the parent, `target` is the child that was clicked.

**Q: Which events don't bubble and how do you handle them with delegation?**
A: Events that don't bubble include:
- `focus` / `blur`
- `mouseenter` / `mouseleave`
- `load` / `unload`
- `scroll` (in some cases)

Solutions:
```javascript
// Use focusin/focusout instead (they bubble)
form.addEventListener('focusin', (e) => {
  if (e.target.matches('input')) {
    // Handle focus
  }
});

// Use capture phase
form.addEventListener('focus', (e) => {
  if (e.target.matches('input')) {
    // Handle focus
  }
}, true); // true = capture phase

// Use mouseover/mouseout instead of mouseenter/mouseleave
container.addEventListener('mouseover', (e) => {
  if (e.target.matches('.item')) {
    // Handle hover
  }
});
```

**Q: What are potential pitfalls of event delegation?**
A: Common issues include:
- **Event.target changes**: If child elements contain other elements, `e.target` might be the inner element. Use `closest()` to find the intended target.
- **Performance**: If the parent has many children and events fire frequently (like mousemove), it can impact performance
- **Event stopping**: If `e.stopPropagation()` is called on a child, events won't bubble to your delegated listener
- **Specificity**: Need careful selector matching to avoid handling wrong elements
```javascript
// ❌ Problem: e.target might be <span> inside <button>
// <button><span>Click</span></button>

// ✅ Solution: Use closest()
if (e.target.closest('.button')) {
  // Handles button click regardless of inner elements
}
```

## References
- [MDN Web Docs - Event delegation](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#event_delegation)
- [MDN Web Docs - Event bubbling](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#event_bubbling)
- [JavaScript.info - Event delegation](https://javascript.info/event-delegation)
- [David Walsh - Event Delegation](https://davidwalsh.name/event-delegate)

---
*See also: [Event Loop](EventLoop.md), [DOM Manipulation](../JavaScript/)*
