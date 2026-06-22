# Exercise 2.2 Solutions

## Task 1 Solution

```python
import asyncio

async def fetch_job(job_id: int) -> dict:
    await asyncio.sleep(0.5)
    return {"id": job_id, "title": f"Job {job_id}"}

async def fetch_company(company_id: int) -> dict:
    await asyncio.sleep(0.3)
    return {"id": company_id, "name": f"Company {company_id}"}

async def get_job_with_company(job_id: int, company_id: int) -> dict:
    # Run both concurrently
    job, company = await asyncio.gather(
        fetch_job(job_id),
        fetch_company(company_id)
    )
    return {"job": job, "company": company}

async def main():
    import time
    start = time.time()
    result = await get_job_with_company(1, 1)
    elapsed = time.time() - start
    print(f"Result: {result}")
    print(f"Time: {elapsed:.2f}s")  # Should be ~0.5s, not 0.8s

asyncio.run(main())
```

## Task 2 Solution

```python
import asyncio
from collections.abc import AsyncGenerator

async def job_stream(job_ids: list[int]) -> AsyncGenerator[dict, None]:
    for job_id in job_ids:
        await asyncio.sleep(0.1)  # Small delay between yields
        yield {"id": job_id, "title": f"Job {job_id}"}

async def main():
    async for job in job_stream([1, 2, 3, 4, 5]):
        print(f"Received: {job}")

asyncio.run(main())
```

## Task 3 Solution

```python
import asyncio

async def process_job(job_id: int) -> dict:
    await asyncio.sleep(0.5)
    return {"id": job_id, "processed": True}

async def process_batch(job_ids: list[int], max_concurrent: int = 3) -> list[dict]:
    semaphore = asyncio.Semaphore(max_concurrent)

    async def process_with_semaphore(job_id: int) -> dict:
        async with semaphore:
            return await process_job(job_id)

    tasks = [process_with_semaphore(job_id) for job_id in job_ids]
    return await asyncio.gather(*tasks)

async def main():
    import time
    start = time.time()
    results = await process_batch(list(range(10)), max_concurrent=3)
    elapsed = time.time() - start
    print(f"Processed {len(results)} jobs in {elapsed:.2f}s")

asyncio.run(main())
```

## Task 4 Solution

```python
import asyncio
import time
from contextlib import asynccontextmanager
from collections.abc import AsyncGenerator

@asynccontextmanager
async def async_timer(name: str) -> AsyncGenerator[None, None]:
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{name} took {elapsed:.2f} seconds")

async def main():
    async with async_timer("fetch"):
        await asyncio.sleep(0.5)

asyncio.run(main())
```
