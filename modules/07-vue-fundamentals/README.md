# Module 7: Vue 3 Fundamentals

## Learning Objectives

- Understand Vue 3 Composition API
- Master reactivity with ref and reactive
- Build components with props and events
- Use computed properties and watchers
- Handle lifecycle hooks

## 7.1 Why Vue 3 Composition API?

Coming from PHP/traditional web development:

| Traditional (PHP/jQuery) | Vue 3 Composition API |
|-------------------------|----------------------|
| Server renders HTML | Client renders components |
| Manual DOM manipulation | Reactive data binding |
| Page reloads | Single Page Application |
| Scattered logic | Organized by feature |

---

## 7.2 Project Setup

### Create Vue Project

```bash
bun create vue@latest frontend
cd frontend

# Select options:
# ✔ TypeScript: Yes
# ✔ Vue Router: Yes
# ✔ Pinia: Yes
# ✔ ESLint: Yes
# ✔ Prettier: Yes

bun install
bun run dev
```

### Project Structure

```
frontend/
├── src/
│   ├── main.ts           # App entry point
│   ├── App.vue           # Root component
│   ├── components/       # Reusable components
│   ├── views/            # Page components
│   ├── composables/      # Reusable logic (hooks)
│   ├── stores/           # Pinia stores
│   ├── router/           # Vue Router config
│   ├── api/              # API client
│   ├── types/            # TypeScript types
│   └── assets/           # Static assets
├── index.html
├── vite.config.ts
└── tsconfig.json
```

---

## 7.3 Component Basics

### Single File Component (SFC)

```vue
<!-- src/components/JobCard.vue -->
<script setup lang="ts">
// Logic goes here
import { ref, computed } from 'vue'

const count = ref(0)
const doubled = computed(() => count.value * 2)

function increment() {
  count.value++
}
</script>

<template>
  <!-- Template goes here -->
  <div class="job-card">
    <h2>{{ count }}</h2>
    <p>Doubled: {{ doubled }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>

<style scoped>
/* Scoped styles - only affect this component */
.job-card {
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 8px;
}
</style>
```

---

## 7.4 Reactivity

### ref - For Primitives

```vue
<script setup lang="ts">
import { ref } from 'vue'

// ref wraps primitives in a reactive object
const count = ref(0)
const name = ref('Alice')
const isLoading = ref(false)

// Access/modify with .value in script
console.log(count.value) // 0
count.value++

// In template, .value is automatic
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Name: {{ name }}</p>
    <p v-if="isLoading">Loading...</p>
  </div>
</template>
```

### reactive - For Objects

```vue
<script setup lang="ts">
import { reactive } from 'vue'

// reactive makes entire object reactive
const user = reactive({
  name: 'Alice',
  age: 30,
  settings: {
    theme: 'dark',
    notifications: true
  }
})

// No .value needed
user.name = 'Bob'
user.settings.theme = 'light'
</script>

<template>
  <div>
    <p>{{ user.name }} ({{ user.age }})</p>
    <p>Theme: {{ user.settings.theme }}</p>
  </div>
</template>
```

### Computed Properties

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const jobs = ref([
  { id: 1, title: 'Developer', is_remote: true },
  { id: 2, title: 'Designer', is_remote: false },
  { id: 3, title: 'Manager', is_remote: true },
])

const showRemoteOnly = ref(false)

// Computed - automatically updates when dependencies change
const filteredJobs = computed(() => {
  if (showRemoteOnly.value) {
    return jobs.value.filter(job => job.is_remote)
  }
  return jobs.value
})

const jobCount = computed(() => filteredJobs.value.length)
</script>

<template>
  <label>
    <input type="checkbox" v-model="showRemoteOnly" />
    Remote only
  </label>
  <p>Showing {{ jobCount }} jobs</p>
  <ul>
    <li v-for="job in filteredJobs" :key="job.id">
      {{ job.title }}
    </li>
  </ul>
</template>
```

### Watchers

```vue
<script setup lang="ts">
import { ref, watch, watchEffect } from 'vue'

const searchQuery = ref('')
const page = ref(1)

// Watch specific value
watch(searchQuery, (newValue, oldValue) => {
  console.log(`Search changed from "${oldValue}" to "${newValue}"`)
  page.value = 1 // Reset to first page on search
})

// Watch multiple values
watch([searchQuery, page], ([newSearch, newPage]) => {
  fetchJobs(newSearch, newPage)
})

// watchEffect - runs immediately and tracks all reactive dependencies
watchEffect(() => {
  console.log(`Fetching page ${page.value} with search "${searchQuery.value}"`)
})
</script>
```

---

## 7.5 Props and Events

### Defining Props

```vue
<!-- src/components/JobCard.vue -->
<script setup lang="ts">
interface Job {
  id: number
  title: string
  company: string
  salary_min: number | null
  salary_max: number | null
  is_remote: boolean
}

// Define props with TypeScript
const props = defineProps<{
  job: Job
  highlighted?: boolean  // Optional prop
}>()

// With defaults
const props = withDefaults(defineProps<{
  job: Job
  highlighted?: boolean
}>(), {
  highlighted: false
})
</script>

<template>
  <div class="job-card" :class="{ highlighted }">
    <h3>{{ job.title }}</h3>
    <p>{{ job.company }}</p>
    <span v-if="job.is_remote" class="badge">Remote</span>
  </div>
</template>
```

### Emitting Events

```vue
<!-- JobCard.vue -->
<script setup lang="ts">
interface Job {
  id: number
  title: string
}

const props = defineProps<{ job: Job }>()

// Define events
const emit = defineEmits<{
  (e: 'select', job: Job): void
  (e: 'delete', jobId: number): void
}>()

function handleClick() {
  emit('select', props.job)
}

function handleDelete() {
  emit('delete', props.job.id)
}
</script>

<template>
  <div class="job-card" @click="handleClick">
    <h3>{{ job.title }}</h3>
    <button @click.stop="handleDelete">Delete</button>
  </div>
</template>
```

### Using Components

```vue
<!-- Parent component -->
<script setup lang="ts">
import { ref } from 'vue'
import JobCard from './components/JobCard.vue'

const jobs = ref([...])
const selectedJob = ref(null)

function onJobSelect(job) {
  selectedJob.value = job
}

function onJobDelete(jobId) {
  jobs.value = jobs.value.filter(j => j.id !== jobId)
}
</script>

<template>
  <div class="job-list">
    <JobCard
      v-for="job in jobs"
      :key="job.id"
      :job="job"
      :highlighted="selectedJob?.id === job.id"
      @select="onJobSelect"
      @delete="onJobDelete"
    />
  </div>
</template>
```

---

## 7.6 Template Syntax

### Directives

```vue
<template>
  <!-- v-if/v-else-if/v-else - Conditional rendering -->
  <div v-if="loading">Loading...</div>
  <div v-else-if="error">Error: {{ error }}</div>
  <div v-else>Content loaded</div>

  <!-- v-show - Toggle visibility (stays in DOM) -->
  <div v-show="isVisible">I'm toggled with CSS</div>

  <!-- v-for - Loop -->
  <ul>
    <li v-for="(job, index) in jobs" :key="job.id">
      {{ index + 1 }}. {{ job.title }}
    </li>
  </ul>

  <!-- v-model - Two-way binding -->
  <input v-model="searchQuery" placeholder="Search..." />
  <select v-model="selectedType">
    <option value="full_time">Full Time</option>
    <option value="part_time">Part Time</option>
  </select>

  <!-- v-bind - Dynamic attributes (shorthand :) -->
  <img :src="user.avatar" :alt="user.name" />
  <button :disabled="isLoading">Submit</button>

  <!-- v-on - Event handlers (shorthand @) -->
  <button @click="handleClick">Click me</button>
  <form @submit.prevent="handleSubmit">...</form>
  <input @keyup.enter="search" />

  <!-- Class and style bindings -->
  <div :class="{ active: isActive, 'has-error': hasError }">...</div>
  <div :class="[baseClass, isActive ? 'active' : '']">...</div>
  <div :style="{ color: textColor, fontSize: fontSize + 'px' }">...</div>
</template>
```

---

## 7.7 Lifecycle Hooks

```vue
<script setup lang="ts">
import { onMounted, onUnmounted, onUpdated } from 'vue'

// Called when component is mounted to DOM
onMounted(() => {
  console.log('Component mounted')
  fetchData()
  window.addEventListener('scroll', handleScroll)
})

// Called when component is about to be unmounted
onUnmounted(() => {
  console.log('Component unmounting')
  window.removeEventListener('scroll', handleScroll)
})

// Called after reactive data changes cause re-render
onUpdated(() => {
  console.log('Component updated')
})
</script>
```

---

## 7.8 Slots

```vue
<!-- Card.vue - Component with slots -->
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header">Default Header</slot>
    </div>
    <div class="card-body">
      <slot>Default content</slot>
    </div>
    <div class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<!-- Using Card.vue -->
<template>
  <Card>
    <template #header>
      <h2>Job Details</h2>
    </template>

    <p>This is the main content</p>

    <template #footer>
      <button>Apply</button>
    </template>
  </Card>
</template>
```

---

## Exercises

1. [Exercise 7.1: Reactivity Practice](exercises/exercise-7.1.md)
2. [Exercise 7.2: Component Communication](exercises/exercise-7.2.md)
3. [Exercise 7.3: Forms and Validation](exercises/exercise-7.3.md)

## Assignment

[Assignment 7: Job Listing Components](assignments/assignment-7.md)

## Quiz

[Module 7 Quiz](quiz/quiz-7.md)

---

## Summary

- ✅ Vue 3 Composition API with `<script setup>`
- ✅ Reactivity with ref, reactive, computed
- ✅ Props and events for component communication
- ✅ Template syntax and directives
- ✅ Lifecycle hooks

**Next Module:** [Vue Ecosystem](../08-vue-ecosystem/README.md)
