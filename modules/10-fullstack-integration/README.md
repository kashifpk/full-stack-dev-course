# Module 10: Full-Stack Integration

## Learning Objectives

- Connect Vue frontend to FastAPI backend
- Handle API communication patterns
- Implement proper error handling
- Configure CORS and environment variables

## 10.1 API Client Setup

### Axios Configuration

```bash
bun install axios
```

```typescript
// src/api/client.ts
import axios, { type AxiosError, type AxiosResponse } from 'axios'
import { useAuthStore } from '@/stores/auth'
import router from '@/router'

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8000',
  headers: {
    'Content-Type': 'application/json'
  }
})

// Request interceptor - add auth token
apiClient.interceptors.request.use((config) => {
  const authStore = useAuthStore()
  if (authStore.token) {
    config.headers.Authorization = `Bearer ${authStore.token}`
  }
  return config
})

// Response interceptor - handle errors
apiClient.interceptors.response.use(
  (response: AxiosResponse) => response,
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      const authStore = useAuthStore()
      await authStore.logout()
      router.push({ name: 'login', query: { redirect: router.currentRoute.value.fullPath } })
    }
    return Promise.reject(error)
  }
)

export default apiClient
```

### Type Definitions

```typescript
// src/types/index.ts
export interface User {
  id: number
  email: string
  full_name: string
  role: 'job_seeker' | 'employer' | 'admin'
  is_active: boolean
  created_at: string
}

export interface Company {
  id: number
  name: string
  description: string | null
  website: string | null
  location: string | null
  size: 'startup' | 'small' | 'medium' | 'large' | 'enterprise'
  owner_id: number
  created_at: string
}

export interface Job {
  id: number
  title: string
  description: string
  company_id: number
  company: Company
  job_type: 'full_time' | 'part_time' | 'contract' | 'internship'
  experience_level: 'entry' | 'mid' | 'senior' | 'lead' | 'executive'
  salary_min: number | null
  salary_max: number | null
  is_remote: boolean
  location: string | null
  skills: string[]
  is_active: boolean
  created_at: string
}

export interface PaginatedResponse<T> {
  items: T[]
  total: number
  page: number
  per_page: number
  pages: number
}

export interface ApiError {
  error_code: string
  message: string
  details: Array<{ field: string; message: string }>
}
```

### API Services

```typescript
// src/api/jobs.ts
import apiClient from './client'
import type { Job, PaginatedResponse } from '@/types'

export interface JobFilters {
  search?: string
  job_type?: string
  experience_level?: string
  is_remote?: boolean
  salary_min?: number
  company_id?: number
  page?: number
  per_page?: number
}

export interface JobCreate {
  title: string
  description: string
  company_id: number
  job_type: string
  experience_level: string
  salary_min?: number
  salary_max?: number
  is_remote: boolean
  location?: string
  skills: string[]
}

export const jobsApi = {
  async list(filters: JobFilters = {}): Promise<PaginatedResponse<Job>> {
    const params = new URLSearchParams()
    Object.entries(filters).forEach(([key, value]) => {
      if (value !== undefined && value !== null && value !== '') {
        params.append(key, String(value))
      }
    })
    const response = await apiClient.get(`/jobs?${params}`)
    return response.data
  },

  async get(id: number): Promise<Job> {
    const response = await apiClient.get(`/jobs/${id}`)
    return response.data
  },

  async create(data: JobCreate): Promise<Job> {
    const response = await apiClient.post('/jobs', data)
    return response.data
  },

  async update(id: number, data: Partial<JobCreate>): Promise<Job> {
    const response = await apiClient.put(`/jobs/${id}`, data)
    return response.data
  },

  async delete(id: number): Promise<void> {
    await apiClient.delete(`/jobs/${id}`)
  }
}
```

```typescript
// src/api/auth.ts
import apiClient from './client'
import type { User } from '@/types'

export interface LoginCredentials {
  email: string
  password: string
}

export interface RegisterData {
  email: string
  password: string
  full_name: string
  role?: 'job_seeker' | 'employer'
}

export interface LoginResponse {
  access_token: string
  token_type: string
  user: User
}

export const authApi = {
  async login(credentials: LoginCredentials): Promise<LoginResponse> {
    const response = await apiClient.post('/auth/login', credentials)
    return response.data
  },

  async register(data: RegisterData): Promise<User> {
    const response = await apiClient.post('/auth/register', data)
    return response.data
  },

  async me(): Promise<User> {
    const response = await apiClient.get('/auth/me')
    return response.data
  },

  async logout(): Promise<void> {
    await apiClient.post('/auth/logout')
  }
}
```

---

## 10.2 Error Handling

### Error Handler Composable

```typescript
// src/composables/useApiError.ts
import { ref } from 'vue'
import { useToast } from 'primevue/usetoast'
import type { AxiosError } from 'axios'
import type { ApiError } from '@/types'

export function useApiError() {
  const toast = useToast()
  const error = ref<string | null>(null)
  const fieldErrors = ref<Record<string, string>>({})

  function handleError(err: unknown) {
    const axiosError = err as AxiosError<ApiError>

    if (axiosError.response?.data) {
      const apiError = axiosError.response.data
      error.value = apiError.message

      // Map field errors
      fieldErrors.value = {}
      apiError.details?.forEach((detail) => {
        if (detail.field) {
          fieldErrors.value[detail.field] = detail.message
        }
      })

      // Show toast for non-validation errors
      if (apiError.error_code !== 'validation_error') {
        toast.add({
          severity: 'error',
          summary: 'Error',
          detail: apiError.message,
          life: 5000
        })
      }
    } else if (axiosError.request) {
      error.value = 'Network error. Please check your connection.'
      toast.add({
        severity: 'error',
        summary: 'Network Error',
        detail: 'Could not connect to server',
        life: 5000
      })
    } else {
      error.value = 'An unexpected error occurred'
      toast.add({
        severity: 'error',
        summary: 'Error',
        detail: 'Something went wrong',
        life: 5000
      })
    }
  }

  function clearErrors() {
    error.value = null
    fieldErrors.value = {}
  }

  function getFieldError(field: string): string | undefined {
    return fieldErrors.value[field]
  }

  return {
    error,
    fieldErrors,
    handleError,
    clearErrors,
    getFieldError
  }
}
```

### Using in Components

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useApiError } from '@/composables/useApiError'
import { jobsApi, type JobCreate } from '@/api/jobs'
import InputText from 'primevue/inputtext'
import Button from 'primevue/button'

const { error, handleError, clearErrors, getFieldError } = useApiError()
const isLoading = ref(false)

const form = ref<JobCreate>({
  title: '',
  description: '',
  company_id: 1,
  job_type: 'full_time',
  experience_level: 'mid',
  is_remote: false,
  skills: []
})

async function submit() {
  clearErrors()
  isLoading.value = true

  try {
    const job = await jobsApi.create(form.value)
    // Success - redirect or show message
  } catch (err) {
    handleError(err)
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <form @submit.prevent="submit">
    <div class="field">
      <label>Title</label>
      <InputText
        v-model="form.title"
        :class="{ 'p-invalid': getFieldError('title') }"
      />
      <small class="p-error">{{ getFieldError('title') }}</small>
    </div>

    <div v-if="error" class="error-message">
      {{ error }}
    </div>

    <Button type="submit" label="Create Job" :loading="isLoading" />
  </form>
</template>
```

---

## 10.3 Loading States

### Loading Composable

```typescript
// src/composables/useLoading.ts
import { ref, computed } from 'vue'

export function useLoading() {
  const loadingStates = ref<Record<string, boolean>>({})

  function setLoading(key: string, value: boolean) {
    loadingStates.value[key] = value
  }

  function isLoading(key: string): boolean {
    return loadingStates.value[key] ?? false
  }

  const anyLoading = computed(() =>
    Object.values(loadingStates.value).some(Boolean)
  )

  async function withLoading<T>(key: string, fn: () => Promise<T>): Promise<T> {
    setLoading(key, true)
    try {
      return await fn()
    } finally {
      setLoading(key, false)
    }
  }

  return {
    loadingStates,
    setLoading,
    isLoading,
    anyLoading,
    withLoading
  }
}
```

---

## 10.4 Backend CORS Configuration

```python
# backend/app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.config import get_settings

settings = get_settings()

app = FastAPI()

# Configure CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",  # Vue dev server
        "http://localhost:3000",
        settings.frontend_url,    # Production frontend
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 10.5 Environment Variables

```bash
# frontend/.env.development
VITE_API_URL=http://localhost:8000

# frontend/.env.production
VITE_API_URL=https://api.jobboard.com
```

```typescript
// Usage in code
const apiUrl = import.meta.env.VITE_API_URL
```

---

## Exercises

1. [Exercise 10.1: API Client](exercises/exercise-10.1.md)
2. [Exercise 10.2: Error Handling](exercises/exercise-10.2.md)
3. [Exercise 10.3: Loading States](exercises/exercise-10.3.md)

## Assignment

[Assignment 10: Connected Application](assignments/assignment-10.md)

## Quiz

[Module 10 Quiz](quiz/quiz-10.md)

---

## Summary

- ✅ Axios API client with interceptors
- ✅ Type-safe API services
- ✅ Centralized error handling
- ✅ Loading state management
- ✅ CORS and environment configuration

**Next Module:** [Authentication & Authorization](../11-auth/README.md)
