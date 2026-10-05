# Module 8: Vue Ecosystem

## Learning Objectives

- Implement client-side routing with Vue Router
- Manage state with Pinia
- Create reusable composables
- Handle async data fetching patterns

## 8.1 Vue Router

### Installation and Setup

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'home',
      component: () => import('@/views/HomeView.vue')
    },
    {
      path: '/jobs',
      name: 'jobs',
      component: () => import('@/views/JobsView.vue')
    },
    {
      path: '/jobs/:id',
      name: 'job-detail',
      component: () => import('@/views/JobDetailView.vue'),
      props: true  // Pass route params as props
    },
    {
      path: '/companies/:companyId/jobs',
      name: 'company-jobs',
      component: () => import('@/views/CompanyJobsView.vue')
    },
    {
      path: '/dashboard',
      name: 'dashboard',
      component: () => import('@/views/DashboardView.vue'),
      meta: { requiresAuth: true }
    },
    {
      path: '/:pathMatch(.*)*',
      name: 'not-found',
      component: () => import('@/views/NotFoundView.vue')
    }
  ]
})

export default router
```

### Using Router in Components

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// Access route params
const jobId = route.params.id

// Access query params
const page = route.query.page

// Navigate programmatically
function goToJob(id: number) {
  router.push({ name: 'job-detail', params: { id } })
}

function goToJobsWithFilter() {
  router.push({
    name: 'jobs',
    query: { remote: 'true', type: 'full_time' }
  })
}

function goBack() {
  router.back()
}
</script>

<template>
  <nav>
    <!-- RouterLink for navigation -->
    <RouterLink to="/">Home</RouterLink>
    <RouterLink :to="{ name: 'jobs' }">Jobs</RouterLink>
    <RouterLink :to="{ name: 'job-detail', params: { id: 1 } }">
      Job #1
    </RouterLink>
  </nav>

  <!-- RouterView renders matched component -->
  <RouterView />
</template>
```

### Route Guards

```typescript
// src/router/index.ts
import { useAuthStore } from '@/stores/auth'

router.beforeEach((to, from) => {
  const authStore = useAuthStore()

  // Check if route requires auth
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    return { name: 'login', query: { redirect: to.fullPath } }
  }

  // Check role-based access
  if (to.meta.requiresAdmin && authStore.user?.role !== 'admin') {
    return { name: 'forbidden' }
  }
})
```

---

## 8.2 Pinia State Management

### Creating a Store

```typescript
// src/stores/jobs.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { jobsApi } from '@/api/jobs'
import type { Job, JobFilters } from '@/types'

export const useJobsStore = defineStore('jobs', () => {
  // State
  const jobs = ref<Job[]>([])
  const currentJob = ref<Job | null>(null)
  const isLoading = ref(false)
  const error = ref<string | null>(null)
  const filters = ref<JobFilters>({
    search: '',
    type: null,
    remote: null,
    page: 1,
    perPage: 20
  })
  const total = ref(0)

  // Getters (computed)
  const remoteJobs = computed(() =>
    jobs.value.filter(job => job.is_remote)
  )

  const totalPages = computed(() =>
    Math.ceil(total.value / filters.value.perPage)
  )

  const hasNextPage = computed(() =>
    filters.value.page < totalPages.value
  )

  // Actions
  async function fetchJobs() {
    isLoading.value = true
    error.value = null

    try {
      const response = await jobsApi.list(filters.value)
      jobs.value = response.items
      total.value = response.total
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Failed to fetch jobs'
    } finally {
      isLoading.value = false
    }
  }

  async function fetchJob(id: number) {
    isLoading.value = true
    error.value = null

    try {
      currentJob.value = await jobsApi.get(id)
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Failed to fetch job'
      currentJob.value = null
    } finally {
      isLoading.value = false
    }
  }

  function setFilters(newFilters: Partial<JobFilters>) {
    filters.value = { ...filters.value, ...newFilters, page: 1 }
  }

  function setPage(page: number) {
    filters.value.page = page
  }

  function reset() {
    jobs.value = []
    currentJob.value = null
    filters.value = { search: '', type: null, remote: null, page: 1, perPage: 20 }
  }

  return {
    // State
    jobs,
    currentJob,
    isLoading,
    error,
    filters,
    total,
    // Getters
    remoteJobs,
    totalPages,
    hasNextPage,
    // Actions
    fetchJobs,
    fetchJob,
    setFilters,
    setPage,
    reset
  }
})
```

### Using the Store

```vue
<!-- src/views/JobsView.vue -->
<script setup lang="ts">
import { onMounted, watch } from 'vue'
import { useJobsStore } from '@/stores/jobs'
import { storeToRefs } from 'pinia'
import JobCard from '@/components/JobCard.vue'
import JobFilters from '@/components/JobFilters.vue'
import Pagination from '@/components/Pagination.vue'

const jobsStore = useJobsStore()

// Use storeToRefs for reactive destructuring
const { jobs, isLoading, error, filters, total, totalPages } = storeToRefs(jobsStore)

// Actions don't need storeToRefs
const { fetchJobs, setFilters, setPage } = jobsStore

// Fetch on mount
onMounted(() => {
  fetchJobs()
})

// Re-fetch when filters change
watch(filters, () => {
  fetchJobs()
}, { deep: true })
</script>

<template>
  <div class="jobs-page">
    <h1>Job Listings</h1>

    <JobFilters
      :filters="filters"
      @update="setFilters"
    />

    <div v-if="isLoading" class="loading">Loading...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <p>{{ total }} jobs found</p>

      <div class="job-grid">
        <JobCard
          v-for="job in jobs"
          :key="job.id"
          :job="job"
        />
      </div>

      <Pagination
        :current-page="filters.page"
        :total-pages="totalPages"
        @change="setPage"
      />
    </div>
  </div>
</template>
```

### Auth Store

```typescript
// src/stores/auth.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { authApi } from '@/api/auth'
import type { User, LoginCredentials } from '@/types'

export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)
  const token = ref<string | null>(localStorage.getItem('token'))

  const isAuthenticated = computed(() => !!token.value)
  const isEmployer = computed(() => user.value?.role === 'employer')
  const isAdmin = computed(() => user.value?.role === 'admin')

  async function login(credentials: LoginCredentials) {
    const response = await authApi.login(credentials)
    token.value = response.access_token
    user.value = response.user
    localStorage.setItem('token', response.access_token)
  }

  async function logout() {
    token.value = null
    user.value = null
    localStorage.removeItem('token')
  }

  async function fetchUser() {
    if (!token.value) return
    try {
      user.value = await authApi.me()
    } catch {
      await logout()
    }
  }

  return {
    user,
    token,
    isAuthenticated,
    isEmployer,
    isAdmin,
    login,
    logout,
    fetchUser
  }
}, {
  // Persist to localStorage
  persist: true
})
```

---

## 8.3 Composables

### What are Composables?

Composables are functions that encapsulate reusable stateful logic - Vue's answer to React hooks.

### useFetch Composable

```typescript
// src/composables/useFetch.ts
import { ref, type Ref } from 'vue'

interface UseFetchReturn<T> {
  data: Ref<T | null>
  error: Ref<string | null>
  isLoading: Ref<boolean>
  execute: () => Promise<void>
}

export function useFetch<T>(
  fetchFn: () => Promise<T>
): UseFetchReturn<T> {
  const data = ref<T | null>(null) as Ref<T | null>
  const error = ref<string | null>(null)
  const isLoading = ref(false)

  async function execute() {
    isLoading.value = true
    error.value = null

    try {
      data.value = await fetchFn()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'An error occurred'
    } finally {
      isLoading.value = false
    }
  }

  return { data, error, isLoading, execute }
}
```

### usePagination Composable

```typescript
// src/composables/usePagination.ts
import { ref, computed } from 'vue'

export function usePagination(initialPerPage = 20) {
  const page = ref(1)
  const perPage = ref(initialPerPage)
  const total = ref(0)

  const totalPages = computed(() =>
    Math.ceil(total.value / perPage.value)
  )

  const offset = computed(() =>
    (page.value - 1) * perPage.value
  )

  const hasNextPage = computed(() => page.value < totalPages.value)
  const hasPrevPage = computed(() => page.value > 1)

  function nextPage() {
    if (hasNextPage.value) page.value++
  }

  function prevPage() {
    if (hasPrevPage.value) page.value--
  }

  function goToPage(p: number) {
    if (p >= 1 && p <= totalPages.value) {
      page.value = p
    }
  }

  function setTotal(t: number) {
    total.value = t
  }

  function reset() {
    page.value = 1
  }

  return {
    page,
    perPage,
    total,
    totalPages,
    offset,
    hasNextPage,
    hasPrevPage,
    nextPage,
    prevPage,
    goToPage,
    setTotal,
    reset
  }
}
```

### useDebounce Composable

```typescript
// src/composables/useDebounce.ts
import { ref, watch, type Ref } from 'vue'

export function useDebounce<T>(value: Ref<T>, delay = 300): Ref<T> {
  const debouncedValue = ref(value.value) as Ref<T>

  let timeout: ReturnType<typeof setTimeout>

  watch(value, (newValue) => {
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      debouncedValue.value = newValue
    }, delay)
  })

  return debouncedValue
}
```

### Using Composables

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
import { usePagination } from '@/composables/usePagination'
import { useDebounce } from '@/composables/useDebounce'
import { jobsApi } from '@/api/jobs'

const search = ref('')
const debouncedSearch = useDebounce(search, 300)

const { page, totalPages, setTotal, goToPage, reset } = usePagination(20)

const jobs = ref([])

async function fetchJobs() {
  const response = await jobsApi.list({
    search: debouncedSearch.value,
    page: page.value
  })
  jobs.value = response.items
  setTotal(response.total)
}

// Re-fetch when search or page changes
watch([debouncedSearch, page], fetchJobs, { immediate: true })

// Reset pagination when search changes
watch(debouncedSearch, reset)
</script>
```

---

## Exercises

1. [Exercise 8.1: Routing Setup](exercises/exercise-8.1.md)
2. [Exercise 8.2: Pinia Store](exercises/exercise-8.2.md)
3. [Exercise 8.3: Custom Composables](exercises/exercise-8.3.md)

## Assignment

[Assignment 8: Full Vue Application Structure](assignments/assignment-8.md)

## Quiz

[Module 8 Quiz](quiz/quiz-8.md)

---

## Summary

- ✅ Vue Router for SPA navigation
- ✅ Route guards for authentication
- ✅ Pinia for state management
- ✅ Composables for reusable logic

**Next Module:** [Styling & PrimeVue](../09-styling-primevue/README.md)
