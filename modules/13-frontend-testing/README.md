# Module 13: Frontend Testing

## Learning Objectives

- Test Vue components with Vitest
- Use Vue Test Utils
- Mock API calls
- Test Pinia stores

## 13.1 Setup

### Install Dependencies

```bash
bun install -D vitest @vue/test-utils @testing-library/vue jsdom
```

### Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
  }
})
```

```typescript
// src/test/setup.ts
import { config } from '@vue/test-utils'
import { createTestingPinia } from '@pinia/testing'

// Global plugins for all tests
config.global.plugins = [createTestingPinia()]
```

---

## 13.2 Component Testing

### Basic Component Test

```typescript
// src/components/__tests__/JobCard.spec.ts
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import JobCard from '../JobCard.vue'

const mockJob = {
  id: 1,
  title: 'Senior Developer',
  company: { id: 1, name: 'TechCorp' },
  salary_min: 100000,
  salary_max: 150000,
  is_remote: true,
  job_type: 'full_time',
  skills: ['Python', 'FastAPI']
}

describe('JobCard', () => {
  it('renders job title', () => {
    const wrapper = mount(JobCard, {
      props: { job: mockJob }
    })

    expect(wrapper.text()).toContain('Senior Developer')
  })

  it('renders company name', () => {
    const wrapper = mount(JobCard, {
      props: { job: mockJob }
    })

    expect(wrapper.text()).toContain('TechCorp')
  })

  it('shows remote badge when job is remote', () => {
    const wrapper = mount(JobCard, {
      props: { job: mockJob }
    })

    expect(wrapper.find('.remote-badge').exists()).toBe(true)
  })

  it('emits view event when clicked', async () => {
    const wrapper = mount(JobCard, {
      props: { job: mockJob }
    })

    await wrapper.find('.job-card').trigger('click')

    expect(wrapper.emitted('view')).toBeTruthy()
    expect(wrapper.emitted('view')[0]).toEqual([mockJob])
  })

  it('formats salary range correctly', () => {
    const wrapper = mount(JobCard, {
      props: { job: mockJob }
    })

    expect(wrapper.text()).toContain('$100,000 - $150,000')
  })
})
```

### Testing with User Interaction

```typescript
// src/components/__tests__/LoginForm.spec.ts
import { describe, it, expect, vi } from 'vitest'
import { mount, flushPromises } from '@vue/test-utils'
import { createTestingPinia } from '@pinia/testing'
import LoginForm from '../LoginForm.vue'
import { useAuthStore } from '@/stores/auth'

describe('LoginForm', () => {
  it('submits form with credentials', async () => {
    const wrapper = mount(LoginForm, {
      global: {
        plugins: [createTestingPinia({ createSpy: vi.fn })]
      }
    })

    const authStore = useAuthStore()

    // Fill form
    await wrapper.find('input[type="email"]').setValue('test@example.com')
    await wrapper.find('input[type="password"]').setValue('password123')

    // Submit
    await wrapper.find('form').trigger('submit')
    await flushPromises()

    // Assert store action was called
    expect(authStore.login).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    })
  })

  it('shows validation errors', async () => {
    const wrapper = mount(LoginForm, {
      global: {
        plugins: [createTestingPinia()]
      }
    })

    // Submit empty form
    await wrapper.find('form').trigger('submit')
    await flushPromises()

    expect(wrapper.find('.error-message').exists()).toBe(true)
  })

  it('disables submit button while loading', async () => {
    const wrapper = mount(LoginForm, {
      global: {
        plugins: [createTestingPinia({
          initialState: {
            auth: { isLoading: true }
          }
        })]
      }
    })

    expect(wrapper.find('button[type="submit"]').attributes('disabled')).toBeDefined()
  })
})
```

---

## 13.3 Store Testing

```typescript
// src/stores/__tests__/jobs.spec.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useJobsStore } from '../jobs'
import { jobsApi } from '@/api/jobs'

// Mock the API
vi.mock('@/api/jobs', () => ({
  jobsApi: {
    list: vi.fn(),
    get: vi.fn(),
    create: vi.fn()
  }
}))

describe('Jobs Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
    vi.clearAllMocks()
  })

  it('fetches jobs successfully', async () => {
    const mockResponse = {
      items: [{ id: 1, title: 'Developer' }],
      total: 1,
      page: 1,
      pages: 1
    }

    vi.mocked(jobsApi.list).mockResolvedValue(mockResponse)

    const store = useJobsStore()
    await store.fetchJobs()

    expect(store.jobs).toEqual(mockResponse.items)
    expect(store.total).toBe(1)
    expect(store.isLoading).toBe(false)
  })

  it('handles fetch error', async () => {
    vi.mocked(jobsApi.list).mockRejectedValue(new Error('Network error'))

    const store = useJobsStore()
    await store.fetchJobs()

    expect(store.error).toBe('Network error')
    expect(store.jobs).toEqual([])
  })

  it('updates filters and resets page', () => {
    const store = useJobsStore()
    store.filters.page = 3

    store.setFilters({ search: 'python' })

    expect(store.filters.search).toBe('python')
    expect(store.filters.page).toBe(1)
  })

  it('computes remote jobs correctly', async () => {
    const store = useJobsStore()
    store.jobs = [
      { id: 1, is_remote: true },
      { id: 2, is_remote: false },
      { id: 3, is_remote: true }
    ]

    expect(store.remoteJobs).toHaveLength(2)
  })
})
```

---

## 13.4 Testing Composables

```typescript
// src/composables/__tests__/usePagination.spec.ts
import { describe, it, expect } from 'vitest'
import { usePagination } from '../usePagination'

describe('usePagination', () => {
  it('calculates offset correctly', () => {
    const { page, perPage, offset } = usePagination(20)

    page.value = 3
    expect(offset.value).toBe(40) // (3 - 1) * 20
  })

  it('calculates total pages', () => {
    const { total, totalPages, perPage } = usePagination(10)

    total.value = 25
    expect(totalPages.value).toBe(3) // ceil(25 / 10)
  })

  it('handles next and prev page', () => {
    const { page, total, perPage, nextPage, prevPage, hasNextPage, hasPrevPage } = usePagination(10)

    total.value = 30
    page.value = 2

    expect(hasPrevPage.value).toBe(true)
    expect(hasNextPage.value).toBe(true)

    nextPage()
    expect(page.value).toBe(3)

    nextPage() // Should not go past last page
    expect(page.value).toBe(3)

    prevPage()
    expect(page.value).toBe(2)
  })
})
```

---

## 13.5 Mocking API Calls

```typescript
// src/test/mocks/handlers.ts
import { http, HttpResponse } from 'msw'

export const handlers = [
  http.get('/api/jobs', () => {
    return HttpResponse.json({
      items: [
        { id: 1, title: 'Developer', company: { name: 'TechCorp' } }
      ],
      total: 1,
      page: 1,
      pages: 1
    })
  }),

  http.post('/api/jobs', async ({ request }) => {
    const body = await request.json()
    return HttpResponse.json(
      { id: 1, ...body },
      { status: 201 }
    )
  }),

  http.post('/api/auth/login', async ({ request }) => {
    const { email, password } = await request.json()
    if (email === 'test@example.com' && password === 'password') {
      return HttpResponse.json({
        access_token: 'mock-token',
        user: { id: 1, email, name: 'Test User' }
      })
    }
    return HttpResponse.json(
      { message: 'Invalid credentials' },
      { status: 401 }
    )
  })
]
```

---

## 13.6 Running Tests

```bash
# Run all tests
bun run test

# Run with coverage
bun run test -- --coverage

# Run in watch mode
bun run test -- --watch

# Run specific file
bun run test -- src/components/__tests__/JobCard.spec.ts
```

---

## Exercises

1. [Exercise 13.1: Component Tests](exercises/exercise-13.1.md)
2. [Exercise 13.2: Store Tests](exercises/exercise-13.2.md)
3. [Exercise 13.3: Integration Tests](exercises/exercise-13.3.md)

## Assignment

[Assignment 13: Test Suite](assignments/assignment-13.md)

## Quiz

[Module 13 Quiz](quiz/quiz-13.md)

---

## Summary

- ✅ Vitest setup for Vue
- ✅ Component testing with Vue Test Utils
- ✅ Pinia store testing
- ✅ Composable testing
- ✅ API mocking

**Next Module:** [DevOps & Deployment](../14-devops/README.md)
