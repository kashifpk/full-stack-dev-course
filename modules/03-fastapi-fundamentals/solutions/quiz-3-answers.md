# Module 3 Quiz Answers

1. **D** - Both are valid; B uses path param, C uses query param
2. **A** - Pydantic model in function signature triggers body parsing
3. **B** - 201 Created is correct for resource creation
4. **B** - response_model filters/validates output data
5. **B** - Depends calls the function and injects the result
6. **C** - HTTPException is converted to HTTP error response
7. **D** - Both provide defaults; Optional[int] needs explicit None default
8. **B** - APIRouter groups endpoints for modular organization
9. **B** - CORS allows cross-origin requests from browsers
10. **B** - FastAPI uses 422 for Pydantic validation errors
11. **C** - Path(...) means required, ge=1 means >= 1
12. **C** - Query(default=[]) handles multiple values as list
13. **B** - async for I/O (DB, HTTP), sync for CPU work
14. **D** - All work, but 204 No Content is most RESTful for DELETE
15. **B** - Code after yield runs as cleanup after response

## Scoring

- 13-15: FastAPI expert!
- 10-12: Good understanding
- 7-9: Review dependency injection and validation
- Below 7: Re-study the module
