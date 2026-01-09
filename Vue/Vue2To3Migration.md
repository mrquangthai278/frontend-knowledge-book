# Vue 2 to Vue 3 Migration

## Definition / Concept
**Vue 3 Migration** is the process of upgrading applications from Vue 2 to Vue 3, involving significant architectural changes including the **Composition API**, improved **TypeScript** support, better **performance**, and a modernized **reactivity system**. The migration includes updating component syntax, replacing the Options API with Composition API patterns, and adapting to new tooling and build systems.

- Vue 3 introduces a new **Composition API** alongside the Options API
- Complete rewrite of the **reactivity system** for better performance
- **TypeScript** support is first-class rather than optional
- Better **tree-shaking** and smaller bundle sizes
- Improved **developer experience** and debugging

## Visual Representation

```
Vue 2 Application
    ↓
Migration Process
    ├─ Update Dependencies
    ├─ Adapt Component Syntax
    ├─ Refactor to Composition API (Optional)
    ├─ Update Global Configuration
    └─ Update Tests & Tooling
    ↓
Vue 3 Application
```

## Example

### Vue 2 Component (Options API)
```javascript
// Vue 2 - Options API
export default {
  data() {
    return {
      count: 0,
      name: 'Vue 2'
    };
  },
  methods: {
    increment() {
      this.count++;
    },
    updateName(newName) {
      this.name = newName;
    }
  },
  computed: {
    doubleCount() {
      return this.count * 2;
    }
  },
  watch: {
    count(newVal, oldVal) {
      console.log(`Count changed from ${oldVal} to ${newVal}`);
    }
  },
  mounted() {
    console.log('Component mounted');
  }
};
```

### Vue 3 Component (Composition API)
```javascript
// Vue 3 - Composition API
import { ref, computed, watch, onMounted } from 'vue';

export default {
  setup() {
    const count = ref(0);
    const name = ref('Vue 3');

    const increment = () => {
      count.value++;
    };

    const updateName = (newName) => {
      name.value = newName;
    };

    const doubleCount = computed(() => count.value * 2);

    watch(count, (newVal, oldVal) => {
      console.log(`Count changed from ${oldVal} to ${newVal}`);
    });

    onMounted(() => {
      console.log('Component mounted');
    });

    return {
      count,
      name,
      increment,
      updateName,
      doubleCount
    };
  }
};
```

### Vue 3 with `<script setup>` (Recommended)
```javascript
// Vue 3 - Composition API with <script setup>
<script setup>
import { ref, computed, watch, onMounted } from 'vue';

const count = ref(0);
const name = ref('Vue 3');

const increment = () => {
  count.value++;
};

const updateName = (newName) => {
  name.value = newName;
};

const doubleCount = computed(() => count.value * 2);

watch(count, (newVal, oldVal) => {
  console.log(`Count changed from ${oldVal} to ${newVal}`);
});

onMounted(() => {
  console.log('Component mounted');
});
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Double: {{ doubleCount }}</p>
    <button @click="increment">Increment</button>
    <input :value="name" @input="updateName($event.target.value)" />
  </div>
</template>
```

## Usage

### When to Migrate
- **Upgrading legacy Vue 2 projects** to leverage modern features
- **Taking advantage of performance improvements** and better TypeScript support
- **Using new libraries and frameworks** that only support Vue 3
- **Reducing bundle size** for better application performance
- **Implementing modern component patterns** like Composition API

### Real-world Example
```javascript
// Before: Vue 2 Component
export default {
  data() {
    return {
      todos: [],
      loading: false
    };
  },
  methods: {
    async fetchTodos() {
      this.loading = true;
      const response = await fetch('/api/todos');
      this.todos = await response.json();
      this.loading = false;
    }
  },
  mounted() {
    this.fetchTodos();
  }
};

// After: Vue 3 Component
<script setup>
import { ref, onMounted } from 'vue';

const todos = ref([]);
const loading = ref(false);

const fetchTodos = async () => {
  loading.value = true;
  const response = await fetch('/api/todos');
  todos.value = await response.json();
  loading.value = false;
};

onMounted(fetchTodos);
</script>

<template>
  <div>
    <div v-if="loading">Loading...</div>
    <ul v-else>
      <li v-for="todo in todos" :key="todo.id">{{ todo.title }}</li>
    </ul>
  </div>
</template>
```

### Best Practices
- **Migrate incrementally**: Update one component at a time rather than all at once
- **Use the migration build**: Vue provides a migration build with warnings for deprecated features
- **Adopt Composition API gradually**: Keep using Options API initially, transition to Composition API when comfortable
- **Update tests early**: Test during migration to catch issues quickly
- **Leverage TypeScript**: Take advantage of better TypeScript support in Vue 3
- **Use `<script setup>`**: Prefer the new `<script setup>` syntax for cleaner, more concise code
- **Update tooling**: Migrate to **Vite** (modern build tool) instead of **webpack** when possible

## Key Migration Areas

### 1. Global API Changes
```javascript
// Vue 2
Vue.component('MyComponent', MyComponent);
Vue.use(MyPlugin);
Vue.mixin(MyMixin);

// Vue 3
const app = createApp({});
app.component('MyComponent', MyComponent);
app.use(MyPlugin);
app.mixin(MyMixin);
```

### 2. Template Changes
```html
<!-- Vue 2 -->
<my-component v-on:custom-event="handleEvent" />
<div v-bind:key="id">...</div>

<!-- Vue 3 -->
<MyComponent @custom-event="handleEvent" />
<div :key="id">...</div>
```

### 3. Lifecycle Hooks
```javascript
// Vue 2                    // Vue 3
beforeCreate            →   setup()
created                 →   setup()
beforeMount             →   onBeforeMount()
mounted                 →   onMounted()
beforeUpdate            →   onBeforeUpdate()
updated                 →   onUpdated()
beforeDestroy           →   onBeforeUnmount()
destroyed               →   onUnmounted()
activated               →   onActivated()
deactivated             →   onDeactivated()
errorCaptured           →   onErrorCaptured()
```

### 4. Props and Emit
```javascript
// Vue 2
export default {
  props: ['title', 'count'],
  methods: {
    updateTitle(newTitle) {
      this.$emit('update-title', newTitle);
    }
  }
};

// Vue 3 with <script setup>
<script setup>
defineProps(['title', 'count']);
const emit = defineEmits(['update-title']);

const updateTitle = (newTitle) => {
  emit('update-title', newTitle);
};
</script>
```

## FAQ / Interview Questions

**Q: What are the main differences between Vue 2 and Vue 3?**
A: The key differences include:
- **Composition API** alongside Options API for better code organization
- **Improved reactivity system** using Proxy instead of Object.defineProperty
- **Better TypeScript support** with full type inference
- **Smaller bundle size** with better tree-shaking
- **Performance improvements** in rendering and updates
- **Fragment support** allowing multiple root elements
- **Teleport component** for rendering components outside their parent

**Q: Do I have to use the Composition API in Vue 3?**
A: No, Vue 3 fully supports the Options API. You can continue using it if you prefer. However, the Composition API is recommended for better code reusability and organization, especially in larger applications.

**Q: What is `ref()` and when should I use it?**
A: `ref()` creates a **reactive variable** that wraps a primitive value. Use it for:
- Primitive values (strings, numbers, booleans)
- When you need `.value` to access the actual value in JavaScript code
- Creating reactive state in the Composition API
In templates, Vue automatically unwraps refs, so you don't need `.value`.

**Q: How do I migrate a large Vue 2 application to Vue 3?**
A: Follow this strategy:
1. Set up Vue 3 and Vite in a new branch
2. Migrate global plugins and configuration
3. Create a migration build with Vue 2 compatibility warnings
4. Gradually migrate components starting with the most independent ones
5. Update tests as you migrate components
6. Test extensively throughout the process
7. Deploy incrementally if possible using feature flags

**Q: What tools help with Vue 2 to Vue 3 migration?**
A: Key tools include:
- **Migration Build**: Official Vue distribution with warnings for deprecated features
- **Vue DevTools**: Updated for Vue 3 with better debugging
- **vue-codemod**: Automated migration scripts for common patterns
- **Vite**: Modern build tool with better performance
- **TypeScript**: Provides type safety during migration

**Q: How do performance improvements happen in Vue 3?**
A: Performance gains come from:
- **Proxy-based reactivity** instead of Object.defineProperty
- **Optimized compiler** that analyzes templates at build time
- **Better tree-shaking** removing unused code from bundles
- **Fragment support** reducing wrapper elements
- **Improved diff algorithm** for faster rendering

## Migration Checklist

- [ ] Review Vue 3 migration guide and breaking changes
- [ ] Update `package.json` dependencies to Vue 3
- [ ] Install and configure build tool (Vite recommended)
- [ ] Update global configuration (plugins, mixins, components)
- [ ] Migrate custom directives to Vue 3 syntax
- [ ] Update component templates (kebab-case to PascalCase)
- [ ] Refactor components to Composition API or keep Options API
- [ ] Update lifecycle hooks to new names
- [ ] Fix TypeScript types and configurations
- [ ] Update unit tests for new Vue 3 API
- [ ] Test all features thoroughly
- [ ] Update documentation and team training

## References
- [Official Vue 3 Migration Guide](https://v3-migration.vuejs.org/)
- [Vue 3 Composition API Documentation](https://vuejs.org/guide/extras/composition-api-faq.html)
- [Vite Documentation](https://vitejs.dev/)
- [Vue 3 Official Guide](https://vuejs.org/guide/introduction.html)
- [TypeScript Support in Vue 3](https://vuejs.org/guide/typescript/overview.html)

---
*See also: [Vue Composition API](./CompositionAPI.md), [Lifecycle Hooks](./LifecycleHooks.md), [Reactivity System](./Reactivity.md)*
