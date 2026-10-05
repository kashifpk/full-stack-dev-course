# Module 3 Quiz: FastAPI Fundamentals

---

## Questions

### Q1: Path vs Query Parameters

Which is correct for getting job ID 5?

A) `GET /jobs?id=5`
B) `GET /jobs/5`
C) `GET /jobs?job_id=5`
D) Both B and C are valid REST patterns

---

### Q2: Request Body

What decorator creates an endpoint that accepts JSON body?

A) `@app.post("/jobs")` with a Pydantic model parameter
B) `@app.body("/jobs")`
C) `@app.json("/jobs")`
D) `@app.post("/jobs", body=True)`

---

### Q3: Status Codes

What status code should `POST /jobs` return on success?

A) 200 OK
B) 201 Created
C) 204 No Content
D) 202 Accepted

---

### Q4: Response Models

What does `response_model=JobResponse` do?

A) Validates input data
B) Filters output to only include fields in JobResponse
C) Creates database schema
D) Generates TypeScript types

---

### Q5: Dependency Injection

What does `Depends(get_db)` do?

```python
@app.get("/jobs")
async def list_jobs(db: Session = Depends(get_db)):
    ...
```

A) Imports the get_db function
B) Calls get_db and passes its return value as the db parameter
C) Creates a new database
D) Decorates the function

---

### Q6: HTTPException

What happens when you raise HTTPException?

A) Program crashes
B) Request is cancelled
C) FastAPI returns an HTTP error response to the client
D) Exception is logged but ignored

---

### Q7: Query Parameters

How do you make a query parameter optional with a default?

A) `page: int = None`
B) `page: int = 1`
C) `page: Optional[int]`
D) Both B and C are correct

---

### Q8: Routers

What is the purpose of `APIRouter`?

A) Handles HTTP routing at the OS level
B) Groups related endpoints and allows modular code organization
C) Increases API performance
D) Provides authentication

---

### Q9: CORS

Why do we need CORS middleware?

A) To encrypt data
B) To allow frontend apps on different origins to access the API
C) To compress responses
D) To validate JSON

---

### Q10: Validation Error

What status code does FastAPI return for invalid request data?

A) 400 Bad Request
B) 422 Unprocessable Entity
C) 500 Internal Server Error
D) 404 Not Found

---

### Q11: Path Parameter Validation

How do you validate that a path parameter is >= 1?

A) `job_id: int = Path(ge=1)`
B) `job_id: int >= 1`
C) `job_id: int = Path(..., ge=1)`
D) Both A and C work

---

### Q12: Multiple Query Parameters

How do you accept multiple values for the same parameter (`?skill=python&skill=fastapi`)?

A) `skill: str`
B) `skill: list[str]`
C) `skill: list[str] = Query(default=[])`
D) `skill: str = Query(multi=True)`

---

### Q13: Async Endpoints

When should you use `async def` vs `def` for endpoints?

A) Always use `async def`
B) Use `async def` for I/O operations, `def` for CPU-bound
C) They're identical
D) Only use `def`

---

### Q14: DELETE Response

What should `DELETE /jobs/{id}` return on success?

A) The deleted job
B) `{"message": "Deleted"}`
C) 204 No Content (empty body)
D) All are acceptable, but C is most RESTful

---

### Q15: Dependency with Yield

What is special about a dependency function that uses `yield`?

A) It's invalid syntax
B) Code after yield runs after the response is sent (cleanup)
C) It returns multiple values
D) It creates a generator endpoint

---

*Answers will be provided separately after you complete the quiz.*
