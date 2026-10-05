# Assignment 7: Job Listing Components

## Objective

Build the core Vue components for the Job Board frontend.

## Requirements

### 1. JobCard Component

Create `src/components/JobCard.vue`:

- Display job title, company name, location
- Show remote badge if applicable
- Display salary range (formatted)
- Show up to 4 skills as tags
- Emit `view` event when clicked
- Emit `save` event when bookmark icon clicked

### 2. JobList Component

Create `src/components/JobList.vue`:

- Accept `jobs` array as prop
- Display jobs in a responsive grid
- Show "No jobs found" when empty
- Include loading skeleton state

### 3. JobFilters Component

Create `src/components/JobFilters.vue`:

- Search input (text)
- Job type select
- Experience level select
- Remote only toggle
- Clear filters button
- Emit `filter-change` with filter object

### 4. Pagination Component

Create `src/components/Pagination.vue`:

- Accept `currentPage`, `totalPages` props
- Show page numbers with ellipsis for large ranges
- Previous/Next buttons
- Emit `page-change` event

### 5. JobDetail Component

Create `src/components/JobDetail.vue`:

- Display full job information
- Format description with line breaks
- Show all skills
- Display company info
- Apply button

## Deliverables

- [ ] All components created with TypeScript
- [ ] Props properly typed with defineProps
- [ ] Events properly typed with defineEmits
- [ ] Components use scoped styles
- [ ] Loading and empty states handled

## Testing

Create a test page that displays mock data:

```vue
<!-- src/views/TestComponentsView.vue -->
<script setup>
import JobCard from '@/components/JobCard.vue'
// ... import others

const mockJobs = [/* ... */]
</script>

<template>
  <div class="test-page">
    <h2>JobCard</h2>
    <JobCard :job="mockJobs[0]" />

    <h2>JobList</h2>
    <JobList :jobs="mockJobs" />

    <!-- Test other components -->
  </div>
</template>
```
