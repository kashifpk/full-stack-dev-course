# Exercise 2.2: Async Programming

## Objective
Understand async/await patterns and concurrent execution.

## Tasks

### Task 1: Convert Sync to Async

Convert this synchronous code to async:

```python
import time

def fetch_job(job_id: int) -> dict:
    time.sleep(0.5)  # Simulate DB call
    return {"id": job_id, "title": f"Job {job_id}"}

def fetch_company(company_id: int) -> dict:
    time.sleep(0.3)  # Simulate DB call
    return {"id": company_id, "name": f"Company {company_id}"}

def get_job_with_company(job_id: int, company_id: int) -> dict:
    job = fetch_job(job_id)
    company = fetch_company(company_id)
    return {"job": job, "company": company}

# This takes 0.8 seconds (0.5 + 0.3)
result = get_job_with_company(1, 1)
```

Your async version should run both fetches concurrently, taking ~0.5 seconds total.

### Task 2: Async Generator

Create an async generator that yields job postings with a delay:

```python
import asyncio

async def job_stream(job_ids: list[int]):
    """
    Yield jobs one by one with a small delay between each.
    Use 'async for' to consume this generator.
    """
    # Your code here
    pass

# Usage:
async def main():
    async for job in job_stream([1, 2, 3, 4, 5]):
        print(f"Received: {job}")

asyncio.run(main())
```

### Task 3: Concurrent Batch Processing

Process a batch of job IDs concurrently, but limit concurrency to 3 at a time:

```python
import asyncio

async def process_job(job_id: int) -> dict:
    """Simulate processing a job (e.g., enriching data from external API)."""
    await asyncio.sleep(0.5)
    return {"id": job_id, "processed": True}

async def process_batch(job_ids: list[int], max_concurrent: int = 3) -> list[dict]:
    """
    Process all job_ids but only run max_concurrent at a time.
    Hint: Use asyncio.Semaphore
    """
    # Your code here
    pass

# Test: Processing 10 jobs with max 3 concurrent should take ~2 seconds
# (10 jobs / 3 concurrent = 4 batches * 0.5s = 2s)
async def main():
    results = await process_batch(list(range(10)), max_concurrent=3)
    print(f"Processed {len(results)} jobs")

asyncio.run(main())
```

### Task 4: Async Context Manager

Create an async context manager for timing operations:

```python
import asyncio
import time
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_timer(name: str):
    """
    Async context manager that prints how long the block took.

    Usage:
        async with async_timer("fetch"):
            await some_async_operation()
        # Prints: "fetch took 0.50 seconds"
    """
    # Your code here
    pass
```

## Verification

```bash
python your_file.py
```

Each task should complete without errors and with expected timing.

## Need Help?

Solutions will be provided separately. Try to complete the tasks on your own first!
