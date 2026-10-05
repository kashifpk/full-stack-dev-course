# Assignment 10: Connected Application

## Objective

Connect your Vue frontend to the FastAPI backend, creating a fully functional job listing page.

## Requirements

### 1. API Client

Set up complete API client with:
- Base Axios configuration
- Auth token interceptor
- Error response interceptor
- Type-safe API services (jobs, companies, auth)

### 2. Jobs Store Integration

Update Pinia store to use real API:
- `fetchJobs()` calls `/jobs` endpoint
- `fetchJob(id)` calls `/jobs/{id}` endpoint
- Handle loading and error states
- Implement filtering through query params

### 3. Jobs View

Connect `JobsView.vue` to store:
- Fetch jobs on mount
- Display loading spinner
- Show error messages
- Working filters that update URL
- Pagination that persists in URL

### 4. Job Detail View

Create `JobDetailView.vue`:
- Fetch job by ID from route params
- Display full job information
- Handle 404 errors
- Back to list navigation

### 5. Error Handling

Implement error handling:
- Network error messages
- 401 redirect to login
- 404 "not found" page
- Validation error display

## Deliverables

- [ ] API client with interceptors
- [ ] Jobs store using real API
- [ ] Jobs list page functional
- [ ] Job detail page functional
- [ ] Filter changes update URL
- [ ] Error states handled gracefully

## Testing Checklist

```bash
# Start backend
cd backend && uvicorn app.main:app --reload

# Start frontend
cd frontend && bun run dev

# Test these flows:
# 1. Jobs page loads and displays jobs
# 2. Filters work and update URL
# 3. Pagination works
# 4. Clicking job shows detail
# 5. Invalid job ID shows 404
# 6. Network disconnect shows error
```
