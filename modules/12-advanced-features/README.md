# Module 12: Advanced Features

## Learning Objectives

- Implement file uploads (resumes)
- Build search and filtering
- Add pagination
- Send email notifications

## 12.1 File Uploads

### Backend: File Upload Endpoint

```bash
pip install python-multipart aiofiles
```

```python
# backend/app/routers/uploads.py
import os
import uuid
from fastapi import APIRouter, UploadFile, File, Depends, HTTPException
from app.auth.dependencies import get_current_user
from app.models.user import User
from app.config import get_settings

settings = get_settings()
router = APIRouter(prefix="/uploads", tags=["uploads"])

ALLOWED_EXTENSIONS = {".pdf", ".doc", ".docx"}
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB

@router.post("/resume")
async def upload_resume(
    file: UploadFile = File(...),
    user: User = Depends(get_current_user),
):
    # Validate file extension
    ext = os.path.splitext(file.filename)[1].lower()
    if ext not in ALLOWED_EXTENSIONS:
        raise HTTPException(400, f"File type not allowed. Allowed: {ALLOWED_EXTENSIONS}")

    # Validate file size
    contents = await file.read()
    if len(contents) > MAX_FILE_SIZE:
        raise HTTPException(400, f"File too large. Max size: {MAX_FILE_SIZE // 1024 // 1024}MB")

    # Generate unique filename
    filename = f"{user.id}_{uuid.uuid4().hex}{ext}"
    filepath = os.path.join(settings.upload_dir, "resumes", filename)

    # Ensure directory exists
    os.makedirs(os.path.dirname(filepath), exist_ok=True)

    # Save file
    with open(filepath, "wb") as f:
        f.write(contents)

    # Return URL
    return {
        "filename": filename,
        "url": f"/uploads/resumes/{filename}",
        "size": len(contents)
    }
```

### Frontend: File Upload Component

```vue
<!-- src/components/ResumeUpload.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import FileUpload from 'primevue/fileupload'
import { useToast } from 'primevue/usetoast'
import apiClient from '@/api/client'

const emit = defineEmits<{
  (e: 'uploaded', url: string): void
}>()

const toast = useToast()
const isUploading = ref(false)

async function onUpload(event: any) {
  isUploading.value = true
  const file = event.files[0]

  const formData = new FormData()
  formData.append('file', file)

  try {
    const response = await apiClient.post('/uploads/resume', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    })
    emit('uploaded', response.data.url)
    toast.add({
      severity: 'success',
      summary: 'Success',
      detail: 'Resume uploaded successfully'
    })
  } catch (error) {
    toast.add({
      severity: 'error',
      summary: 'Error',
      detail: 'Failed to upload resume'
    })
  } finally {
    isUploading.value = false
  }
}
</script>

<template>
  <FileUpload
    mode="basic"
    :auto="true"
    accept=".pdf,.doc,.docx"
    :maxFileSize="5000000"
    chooseLabel="Upload Resume"
    :customUpload="true"
    @uploader="onUpload"
    :disabled="isUploading"
  />
</template>
```

---

## 12.2 Advanced Search

### Backend: Full-Text Search

```python
# backend/app/repositories/job.py
from sqlalchemy import select, func, or_, and_

class JobRepository:
    async def search(
        self,
        *,
        query: str | None = None,
        job_type: str | None = None,
        experience_level: str | None = None,
        is_remote: bool | None = None,
        salary_min: int | None = None,
        salary_max: int | None = None,
        location: str | None = None,
        skills: list[str] | None = None,
        company_id: int | None = None,
        offset: int = 0,
        limit: int = 20,
        sort_by: str = "created_at",
        sort_order: str = "desc",
    ) -> tuple[list[Job], int]:
        stmt = select(Job).options(selectinload(Job.company))

        # Text search
        if query:
            search_term = f"%{query}%"
            stmt = stmt.where(
                or_(
                    Job.title.ilike(search_term),
                    Job.description.ilike(search_term),
                    Job.company.has(Company.name.ilike(search_term))
                )
            )

        # Filters
        if job_type:
            stmt = stmt.where(Job.job_type == job_type)

        if experience_level:
            stmt = stmt.where(Job.experience_level == experience_level)

        if is_remote is not None:
            stmt = stmt.where(Job.is_remote == is_remote)

        if salary_min:
            stmt = stmt.where(Job.salary_max >= salary_min)

        if salary_max:
            stmt = stmt.where(Job.salary_min <= salary_max)

        if location:
            stmt = stmt.where(Job.location.ilike(f"%{location}%"))

        if skills:
            # Job must have all specified skills
            stmt = stmt.where(Job.skills.contains(skills))

        if company_id:
            stmt = stmt.where(Job.company_id == company_id)

        # Only active jobs
        stmt = stmt.where(Job.is_active == True)

        # Count total before pagination
        count_stmt = select(func.count()).select_from(stmt.subquery())
        total = await self.db.scalar(count_stmt) or 0

        # Sorting
        sort_column = getattr(Job, sort_by, Job.created_at)
        if sort_order == "desc":
            stmt = stmt.order_by(sort_column.desc())
        else:
            stmt = stmt.order_by(sort_column.asc())

        # Pagination
        stmt = stmt.offset(offset).limit(limit)

        result = await self.db.execute(stmt)
        return list(result.scalars().all()), total
```

### Frontend: Search Filters

```vue
<!-- src/components/JobSearchFilters.vue -->
<script setup lang="ts">
import { ref, watch } from 'vue'
import InputText from 'primevue/inputtext'
import Select from 'primevue/select'
import MultiSelect from 'primevue/multiselect'
import Slider from 'primevue/slider'
import ToggleSwitch from 'primevue/toggleswitch'
import Button from 'primevue/button'
import { useDebounce } from '@/composables/useDebounce'

interface Filters {
  query?: string
  job_type?: string
  experience_level?: string
  is_remote?: boolean
  salary_range?: [number, number]
  skills?: string[]
}

const props = defineProps<{ modelValue: Filters }>()
const emit = defineEmits<{ (e: 'update:modelValue', filters: Filters): void }>()

const localFilters = ref({ ...props.modelValue })
const debouncedQuery = useDebounce(ref(''), 300)

const jobTypes = [
  { label: 'Full Time', value: 'full_time' },
  { label: 'Part Time', value: 'part_time' },
  { label: 'Contract', value: 'contract' },
  { label: 'Internship', value: 'internship' }
]

const experienceLevels = [
  { label: 'Entry', value: 'entry' },
  { label: 'Mid', value: 'mid' },
  { label: 'Senior', value: 'senior' },
  { label: 'Lead', value: 'lead' }
]

const commonSkills = [
  'JavaScript', 'TypeScript', 'Python', 'Java', 'Go',
  'React', 'Vue', 'Angular', 'Node.js', 'FastAPI',
  'PostgreSQL', 'MongoDB', 'Redis', 'Docker', 'AWS'
]

watch(localFilters, (newVal) => {
  emit('update:modelValue', newVal)
}, { deep: true })

function clearFilters() {
  localFilters.value = {}
}
</script>

<template>
  <div class="search-filters">
    <div class="filter-row">
      <InputText
        v-model="localFilters.query"
        placeholder="Search jobs..."
        class="search-input"
      />
    </div>

    <div class="filter-row">
      <Select
        v-model="localFilters.job_type"
        :options="jobTypes"
        optionLabel="label"
        optionValue="value"
        placeholder="Job Type"
        showClear
      />

      <Select
        v-model="localFilters.experience_level"
        :options="experienceLevels"
        optionLabel="label"
        optionValue="value"
        placeholder="Experience"
        showClear
      />

      <div class="remote-toggle">
        <label>Remote Only</label>
        <ToggleSwitch v-model="localFilters.is_remote" />
      </div>
    </div>

    <div class="filter-row">
      <div class="salary-filter">
        <label>Salary Range: ${{ localFilters.salary_range?.[0]?.toLocaleString() }} - ${{ localFilters.salary_range?.[1]?.toLocaleString() }}</label>
        <Slider
          v-model="localFilters.salary_range"
          :range="true"
          :min="0"
          :max="300000"
          :step="5000"
        />
      </div>
    </div>

    <div class="filter-row">
      <MultiSelect
        v-model="localFilters.skills"
        :options="commonSkills"
        placeholder="Skills"
        class="skills-select"
      />
    </div>

    <div class="filter-actions">
      <Button label="Clear Filters" severity="secondary" @click="clearFilters" />
    </div>
  </div>
</template>
```

---

## 12.3 Job Applications

### Backend: Application Flow

```python
# backend/app/routers/applications.py
from fastapi import APIRouter, Depends, HTTPException
from app.auth.dependencies import get_current_user
from app.models.user import User

router = APIRouter(prefix="/applications", tags=["applications"])

@router.post("", response_model=ApplicationResponse, status_code=201)
async def create_application(
    application_data: ApplicationCreate,
    db: AsyncSession = Depends(get_db),
    user: User = Depends(get_current_user),
):
    # Check if already applied
    existing = await repo.get_by_user_and_job(user.id, application_data.job_id)
    if existing:
        raise HTTPException(400, "You have already applied to this job")

    # Check if job exists and is active
    job = await job_repo.get_by_id(application_data.job_id)
    if not job or not job.is_active:
        raise HTTPException(404, "Job not found or inactive")

    # Create application
    application = await repo.create(
        user_id=user.id,
        job_id=application_data.job_id,
        cover_letter=application_data.cover_letter,
        resume_url=application_data.resume_url,
    )

    # Send notification email (async)
    await send_application_notification(job.company.owner, application)

    return application

@router.get("/my", response_model=list[ApplicationResponse])
async def get_my_applications(
    db: AsyncSession = Depends(get_db),
    user: User = Depends(get_current_user),
):
    return await repo.get_by_user(user.id)
```

---

## 12.4 Email Notifications

```python
# backend/app/services/email.py
import aiosmtplib
from email.message import EmailMessage
from app.config import get_settings

settings = get_settings()

async def send_email(to: str, subject: str, body: str):
    message = EmailMessage()
    message["From"] = settings.smtp_from
    message["To"] = to
    message["Subject"] = subject
    message.set_content(body)

    await aiosmtplib.send(
        message,
        hostname=settings.smtp_host,
        port=settings.smtp_port,
        username=settings.smtp_user,
        password=settings.smtp_password,
        use_tls=True,
    )

async def send_application_notification(employer: User, application: Application):
    await send_email(
        to=employer.email,
        subject=f"New Application for {application.job.title}",
        body=f"""
        You have received a new application for {application.job.title}.

        Applicant: {application.user.full_name}
        Email: {application.user.email}

        View the application in your dashboard.
        """
    )
```

---

## Exercises

1. [Exercise 12.1: File Uploads](exercises/exercise-12.1.md)
2. [Exercise 12.2: Advanced Search](exercises/exercise-12.2.md)
3. [Exercise 12.3: Applications](exercises/exercise-12.3.md)

## Assignment

[Assignment 12: Complete Job Application Flow](assignments/assignment-12.md)

## Quiz

[Module 12 Quiz](quiz/quiz-12.md)

---

## Summary

- ✅ File upload handling
- ✅ Advanced search with multiple filters
- ✅ Job application workflow
- ✅ Email notifications

**Next Module:** [Frontend Testing](../13-frontend-testing/README.md)
