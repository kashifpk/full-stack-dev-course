# Module 11: Authentication & Authorization

## Learning Objectives

- Implement JWT authentication in FastAPI
- Secure frontend routes
- Handle role-based access control
- Manage authentication state

## 11.1 Backend: JWT Authentication

### Install Dependencies

```bash
pip install python-jose[cryptography] passlib[bcrypt]
```

### Password Hashing

```python
# backend/app/auth/password.py
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)
```

### JWT Token Handling

```python
# backend/app/auth/jwt.py
from datetime import datetime, timedelta
from jose import JWTError, jwt
from pydantic import BaseModel
from app.config import get_settings

settings = get_settings()

class TokenData(BaseModel):
    user_id: int
    email: str
    role: str

def create_access_token(data: TokenData, expires_delta: timedelta | None = None) -> str:
    to_encode = data.model_dump()
    expire = datetime.utcnow() + (expires_delta or timedelta(hours=24))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.secret_key, algorithm="HS256")

def decode_token(token: str) -> TokenData | None:
    try:
        payload = jwt.decode(token, settings.secret_key, algorithms=["HS256"])
        return TokenData(
            user_id=payload["user_id"],
            email=payload["email"],
            role=payload["role"]
        )
    except JWTError:
        return None
```

### Auth Dependencies

```python
# backend/app/auth/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession
from app.database.session import get_db
from app.auth.jwt import decode_token, TokenData
from app.repositories.user import UserRepository
from app.models.user import User

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: AsyncSession = Depends(get_db),
) -> User:
    token_data = decode_token(credentials.credentials)
    if not token_data:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication token",
        )

    repo = UserRepository(db)
    user = await repo.get_by_id(token_data.user_id)

    if not user or not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found or inactive",
        )

    return user

async def get_current_active_user(
    user: User = Depends(get_current_user),
) -> User:
    if not user.is_active:
        raise HTTPException(status_code=400, detail="Inactive user")
    return user

def require_role(*roles: str):
    async def role_checker(user: User = Depends(get_current_user)) -> User:
        if user.role not in roles:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Insufficient permissions",
            )
        return user
    return role_checker

# Specific role dependencies
require_employer = require_role("employer", "admin")
require_admin = require_role("admin")
```

### Auth Router

```python
# backend/app/routers/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from app.database.session import get_db
from app.schemas.user import UserCreate, UserResponse, UserLogin
from app.schemas.auth import Token
from app.repositories.user import UserRepository
from app.auth.password import hash_password, verify_password
from app.auth.jwt import create_access_token, TokenData
from app.auth.dependencies import get_current_user
from app.models.user import User

router = APIRouter(prefix="/auth", tags=["auth"])

@router.post("/register", response_model=UserResponse, status_code=201)
async def register(
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db),
):
    repo = UserRepository(db)

    # Check if email exists
    existing = await repo.get_by_email(user_data.email)
    if existing:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Email already registered",
        )

    # Create user
    user = await repo.create(
        email=user_data.email,
        hashed_password=hash_password(user_data.password),
        full_name=user_data.full_name,
        role=user_data.role,
    )

    return user

@router.post("/login", response_model=Token)
async def login(
    credentials: UserLogin,
    db: AsyncSession = Depends(get_db),
):
    repo = UserRepository(db)
    user = await repo.get_by_email(credentials.email)

    if not user or not verify_password(credentials.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
        )

    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Account is inactive",
        )

    token = create_access_token(TokenData(
        user_id=user.id,
        email=user.email,
        role=user.role,
    ))

    return {"access_token": token, "token_type": "bearer", "user": user}

@router.get("/me", response_model=UserResponse)
async def get_me(user: User = Depends(get_current_user)):
    return user
```

### Protected Routes

```python
# backend/app/routers/jobs.py
from app.auth.dependencies import get_current_user, require_employer

@router.post("", response_model=JobResponse, status_code=201)
async def create_job(
    job_data: JobCreate,
    db: AsyncSession = Depends(get_db),
    user: User = Depends(require_employer),  # Only employers can create
):
    # Verify user owns the company
    company = await company_repo.get_by_id(job_data.company_id)
    if not company or company.owner_id != user.id:
        raise HTTPException(403, "Not authorized to post for this company")

    # Create job...
```

---

## 11.2 Frontend: Auth Store

```typescript
// src/stores/auth.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { authApi, type LoginCredentials, type RegisterData } from '@/api/auth'
import type { User } from '@/types'

export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)
  const token = ref<string | null>(localStorage.getItem('token'))
  const returnUrl = ref<string | null>(null)

  const isAuthenticated = computed(() => !!token.value && !!user.value)
  const isEmployer = computed(() => user.value?.role === 'employer')
  const isAdmin = computed(() => user.value?.role === 'admin')

  async function login(credentials: LoginCredentials) {
    const response = await authApi.login(credentials)
    token.value = response.access_token
    user.value = response.user
    localStorage.setItem('token', response.access_token)
  }

  async function register(data: RegisterData) {
    const newUser = await authApi.register(data)
    // Auto-login after registration
    await login({ email: data.email, password: data.password })
    return newUser
  }

  async function logout() {
    try {
      await authApi.logout()
    } catch {
      // Ignore errors on logout
    }
    token.value = null
    user.value = null
    localStorage.removeItem('token')
  }

  async function fetchUser() {
    if (!token.value) return

    try {
      user.value = await authApi.me()
    } catch {
      // Token invalid, logout
      await logout()
    }
  }

  function setReturnUrl(url: string) {
    returnUrl.value = url
  }

  return {
    user,
    token,
    returnUrl,
    isAuthenticated,
    isEmployer,
    isAdmin,
    login,
    register,
    logout,
    fetchUser,
    setReturnUrl
  }
})
```

### Route Guards

```typescript
// src/router/index.ts
import { useAuthStore } from '@/stores/auth'

const router = createRouter({
  // ...routes
})

router.beforeEach(async (to, from) => {
  const authStore = useAuthStore()

  // Fetch user if we have token but no user
  if (authStore.token && !authStore.user) {
    await authStore.fetchUser()
  }

  // Check auth requirement
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    authStore.setReturnUrl(to.fullPath)
    return { name: 'login' }
  }

  // Check role requirement
  if (to.meta.requiresEmployer && !authStore.isEmployer) {
    return { name: 'forbidden' }
  }

  if (to.meta.requiresAdmin && !authStore.isAdmin) {
    return { name: 'forbidden' }
  }

  // Redirect authenticated users away from auth pages
  if (to.meta.guestOnly && authStore.isAuthenticated) {
    return { name: 'home' }
  }
})
```

### Login Component

```vue
<!-- src/views/LoginView.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { useApiError } from '@/composables/useApiError'
import InputText from 'primevue/inputtext'
import Password from 'primevue/password'
import Button from 'primevue/button'

const router = useRouter()
const authStore = useAuthStore()
const { handleError, getFieldError, clearErrors } = useApiError()

const form = ref({
  email: '',
  password: ''
})
const isLoading = ref(false)

async function submit() {
  clearErrors()
  isLoading.value = true

  try {
    await authStore.login(form.value)
    const returnUrl = authStore.returnUrl || '/'
    authStore.setReturnUrl(null)
    router.push(returnUrl)
  } catch (err) {
    handleError(err)
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="login-page">
    <Card class="login-card">
      <template #title>Login</template>
      <template #content>
        <form @submit.prevent="submit">
          <div class="field">
            <label for="email">Email</label>
            <InputText
              id="email"
              v-model="form.email"
              type="email"
              :class="{ 'p-invalid': getFieldError('email') }"
            />
            <small class="p-error">{{ getFieldError('email') }}</small>
          </div>

          <div class="field">
            <label for="password">Password</label>
            <Password
              id="password"
              v-model="form.password"
              :feedback="false"
              toggleMask
            />
          </div>

          <Button
            type="submit"
            label="Login"
            :loading="isLoading"
            class="w-full"
          />
        </form>
      </template>
    </Card>
  </div>
</template>
```

---

## Exercises

1. [Exercise 11.1: Backend Auth](exercises/exercise-11.1.md)
2. [Exercise 11.2: Frontend Auth](exercises/exercise-11.2.md)
3. [Exercise 11.3: Role-Based Access](exercises/exercise-11.3.md)

## Assignment

[Assignment 11: Complete Auth System](assignments/assignment-11.md)

## Quiz

[Module 11 Quiz](quiz/quiz-11.md)

---

## Summary

- ✅ JWT token authentication
- ✅ Password hashing with bcrypt
- ✅ Role-based authorization
- ✅ Protected routes on frontend and backend
- ✅ Auth state management with Pinia

**Next Module:** [Advanced Features](../12-advanced-features/README.md)
