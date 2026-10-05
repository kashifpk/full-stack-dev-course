# Module 9: Styling & PrimeVue

## Learning Objectives

- Master modern CSS (Flexbox, Grid, CSS Variables)
- Set up and customize PrimeVue 4
- Implement dark/light theme switching
- Build responsive layouts

## 9.1 Modern CSS Essentials

### CSS Variables (Custom Properties)

```css
/* Define variables */
:root {
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --color-success: #22c55e;
  --color-danger: #ef4444;

  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;

  --border-radius: 0.5rem;
  --shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

/* Use variables */
.button {
  background-color: var(--color-primary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--border-radius);
  box-shadow: var(--shadow);
}

/* Dark theme override */
[data-theme="dark"] {
  --color-primary: #60a5fa;
  --background: #1e293b;
  --text: #f1f5f9;
}
```

### Flexbox

```css
/* Container */
.flex-container {
  display: flex;
  flex-direction: row;           /* row | column */
  justify-content: space-between; /* main axis alignment */
  align-items: center;           /* cross axis alignment */
  gap: 1rem;                     /* spacing between items */
  flex-wrap: wrap;               /* allow wrapping */
}

/* Items */
.flex-item {
  flex: 1;                /* grow to fill space */
  flex: 0 0 200px;        /* fixed width, no grow/shrink */
  align-self: flex-start; /* individual alignment */
}

/* Common patterns */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
}

.card-actions {
  display: flex;
  gap: 0.5rem;
  justify-content: flex-end;
}
```

### CSS Grid

```css
/* Basic grid */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);  /* 3 equal columns */
  gap: 1.5rem;
}

/* Responsive grid */
.job-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
}

/* Complex layout */
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "sidebar header"
    "sidebar main"
    "sidebar footer";
  min-height: 100vh;
}

.sidebar { grid-area: sidebar; }
.header { grid-area: header; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

---

## 9.2 PrimeVue 4 Setup

### Installation

```bash
bun install primevue @primevue/themes
```

### Configuration

```typescript
// src/main.ts
import { createApp } from 'vue'
import PrimeVue from 'primevue/config'
import Aura from '@primevue/themes/aura'
import ToastService from 'primevue/toastservice'
import ConfirmationService from 'primevue/confirmationservice'
import App from './App.vue'

// PrimeVue styles
import 'primeicons/primeicons.css'

const app = createApp(App)

app.use(PrimeVue, {
  theme: {
    preset: Aura,
    options: {
      darkModeSelector: '.dark-mode',
      cssLayer: {
        name: 'primevue',
        order: 'tailwind-base, primevue, tailwind-utilities'
      }
    }
  }
})

app.use(ToastService)
app.use(ConfirmationService)

app.mount('#app')
```

### Common Components

```vue
<script setup lang="ts">
import Button from 'primevue/button'
import InputText from 'primevue/inputtext'
import Select from 'primevue/select'
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import Card from 'primevue/card'
import Dialog from 'primevue/dialog'
import Toast from 'primevue/toast'
import { useToast } from 'primevue/usetoast'
import { ref } from 'vue'

const toast = useToast()
const searchQuery = ref('')
const selectedType = ref(null)

const jobTypes = [
  { label: 'Full Time', value: 'full_time' },
  { label: 'Part Time', value: 'part_time' },
  { label: 'Contract', value: 'contract' }
]

function showSuccess() {
  toast.add({
    severity: 'success',
    summary: 'Success',
    detail: 'Job created successfully',
    life: 3000
  })
}
</script>

<template>
  <Toast />

  <!-- Buttons -->
  <Button label="Primary" />
  <Button label="Secondary" severity="secondary" />
  <Button label="Success" severity="success" />
  <Button label="Danger" severity="danger" />
  <Button label="Loading" :loading="true" />
  <Button icon="pi pi-search" rounded />

  <!-- Form inputs -->
  <InputText v-model="searchQuery" placeholder="Search jobs..." />

  <Select
    v-model="selectedType"
    :options="jobTypes"
    optionLabel="label"
    optionValue="value"
    placeholder="Select job type"
  />

  <!-- Card -->
  <Card>
    <template #title>Job Title</template>
    <template #subtitle>Company Name</template>
    <template #content>
      <p>Job description here...</p>
    </template>
    <template #footer>
      <Button label="Apply" />
    </template>
  </Card>

  <!-- Data Table -->
  <DataTable :value="jobs" paginator :rows="10">
    <Column field="title" header="Title" sortable />
    <Column field="company" header="Company" sortable />
    <Column field="salary_min" header="Salary">
      <template #body="{ data }">
        ${{ data.salary_min?.toLocaleString() }}
      </template>
    </Column>
    <Column header="Actions">
      <template #body="{ data }">
        <Button icon="pi pi-eye" text @click="viewJob(data)" />
        <Button icon="pi pi-pencil" text @click="editJob(data)" />
      </template>
    </Column>
  </DataTable>
</template>
```

---

## 9.3 Theme Customization

### Custom Theme Colors

```typescript
// src/main.ts
import { definePreset } from '@primevue/themes'
import Aura from '@primevue/themes/aura'

const JobBoardTheme = definePreset(Aura, {
  semantic: {
    primary: {
      50: '{blue.50}',
      100: '{blue.100}',
      200: '{blue.200}',
      300: '{blue.300}',
      400: '{blue.400}',
      500: '{blue.500}',
      600: '{blue.600}',
      700: '{blue.700}',
      800: '{blue.800}',
      900: '{blue.900}',
      950: '{blue.950}'
    }
  }
})

app.use(PrimeVue, {
  theme: {
    preset: JobBoardTheme
  }
})
```

### Dark Mode Toggle

```vue
<!-- src/components/ThemeToggle.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import ToggleSwitch from 'primevue/toggleswitch'

const isDark = ref(false)

onMounted(() => {
  isDark.value = document.documentElement.classList.contains('dark-mode')
})

function toggleTheme() {
  document.documentElement.classList.toggle('dark-mode')
  isDark.value = !isDark.value
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
}
</script>

<template>
  <div class="theme-toggle">
    <i class="pi pi-sun" />
    <ToggleSwitch v-model="isDark" @change="toggleTheme" />
    <i class="pi pi-moon" />
  </div>
</template>

<style scoped>
.theme-toggle {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
</style>
```

---

## 9.4 Building the Job Board UI

### Job Card Component

```vue
<!-- src/components/JobCard.vue -->
<script setup lang="ts">
import Card from 'primevue/card'
import Button from 'primevue/button'
import Tag from 'primevue/tag'
import type { Job } from '@/types'

const props = defineProps<{ job: Job }>()
const emit = defineEmits<{ (e: 'view', job: Job): void }>()

function formatSalary(min?: number, max?: number): string {
  if (min && max) return `$${min.toLocaleString()} - $${max.toLocaleString()}`
  if (min) return `From $${min.toLocaleString()}`
  if (max) return `Up to $${max.toLocaleString()}`
  return 'Not specified'
}
</script>

<template>
  <Card class="job-card">
    <template #header>
      <div class="job-header">
        <Tag v-if="job.is_remote" value="Remote" severity="success" />
        <Tag :value="job.job_type.replace('_', ' ')" />
      </div>
    </template>

    <template #title>{{ job.title }}</template>
    <template #subtitle>{{ job.company?.name }}</template>

    <template #content>
      <div class="job-meta">
        <span v-if="job.location">
          <i class="pi pi-map-marker" /> {{ job.location }}
        </span>
        <span>
          <i class="pi pi-dollar" /> {{ formatSalary(job.salary_min, job.salary_max) }}
        </span>
      </div>

      <div class="job-skills">
        <Tag
          v-for="skill in job.skills.slice(0, 4)"
          :key="skill"
          :value="skill"
          severity="secondary"
        />
        <span v-if="job.skills.length > 4">+{{ job.skills.length - 4 }}</span>
      </div>
    </template>

    <template #footer>
      <Button label="View Details" @click="emit('view', job)" />
    </template>
  </Card>
</template>

<style scoped>
.job-card {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.job-header {
  display: flex;
  gap: 0.5rem;
  padding: 1rem;
}

.job-meta {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  color: var(--text-color-secondary);
  margin-bottom: 1rem;
}

.job-meta span {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.job-skills {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
}
</style>
```

### Job List View

```vue
<!-- src/views/JobsView.vue -->
<script setup lang="ts">
import { onMounted, watch } from 'vue'
import { storeToRefs } from 'pinia'
import { useJobsStore } from '@/stores/jobs'
import InputText from 'primevue/inputtext'
import Select from 'primevue/select'
import ToggleSwitch from 'primevue/toggleswitch'
import Paginator from 'primevue/paginator'
import ProgressSpinner from 'primevue/progressspinner'
import JobCard from '@/components/JobCard.vue'

const jobsStore = useJobsStore()
const { jobs, isLoading, error, filters, total } = storeToRefs(jobsStore)

const jobTypes = [
  { label: 'All Types', value: null },
  { label: 'Full Time', value: 'full_time' },
  { label: 'Part Time', value: 'part_time' },
  { label: 'Contract', value: 'contract' }
]

onMounted(() => jobsStore.fetchJobs())

watch(filters, () => jobsStore.fetchJobs(), { deep: true })

function onPageChange(event: { page: number }) {
  jobsStore.setPage(event.page + 1)
}
</script>

<template>
  <div class="jobs-page">
    <h1>Find Your Next Job</h1>

    <!-- Filters -->
    <div class="filters">
      <InputText
        v-model="filters.search"
        placeholder="Search jobs..."
        class="search-input"
      />
      <Select
        v-model="filters.type"
        :options="jobTypes"
        optionLabel="label"
        optionValue="value"
        placeholder="Job Type"
      />
      <div class="remote-filter">
        <label>Remote only</label>
        <ToggleSwitch v-model="filters.remote" />
      </div>
    </div>

    <!-- Loading State -->
    <div v-if="isLoading" class="loading">
      <ProgressSpinner />
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="error">
      <i class="pi pi-exclamation-triangle" />
      <p>{{ error }}</p>
    </div>

    <!-- Job Grid -->
    <template v-else>
      <p class="results-count">{{ total }} jobs found</p>

      <div class="job-grid">
        <JobCard
          v-for="job in jobs"
          :key="job.id"
          :job="job"
          @view="$router.push({ name: 'job-detail', params: { id: job.id } })"
        />
      </div>

      <Paginator
        :rows="filters.perPage"
        :totalRecords="total"
        :first="(filters.page - 1) * filters.perPage"
        @page="onPageChange"
      />
    </template>
  </div>
</template>

<style scoped>
.jobs-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.filters {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  margin-bottom: 2rem;
}

.search-input {
  flex: 1;
  min-width: 200px;
}

.remote-filter {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.job-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.loading, .error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 4rem;
}
</style>
```

---

## Exercises

1. [Exercise 9.1: CSS Layouts](exercises/exercise-9.1.md)
2. [Exercise 9.2: PrimeVue Components](exercises/exercise-9.2.md)
3. [Exercise 9.3: Theming](exercises/exercise-9.3.md)

## Assignment

[Assignment 9: Complete Job Board UI](assignments/assignment-9.md)

## Quiz

[Module 9 Quiz](quiz/quiz-9.md)

---

## Summary

- ✅ Modern CSS with Flexbox, Grid, and Variables
- ✅ PrimeVue 4 setup and components
- ✅ Theme customization and dark mode
- ✅ Responsive job board UI

**Next Module:** [Full-Stack Integration](../10-fullstack-integration/README.md)
