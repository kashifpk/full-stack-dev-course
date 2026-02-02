# Module 1 Quiz: Development Environment

Test your understanding of the concepts covered in this module.

---

## Questions

### Q1: Container Volumes

What happens to data in PostgreSQL when you run `podman compose down` (or `docker compose down`)?

A) All data is deleted immediately
B) Data persists because we defined a named volume
C) Data is backed up to the cloud
D) PostgreSQL crashes

---

### Q2: Environment Variables

Given this configuration:
```python
class Settings(BaseSettings):
    debug: bool = False
    class Config:
        env_file = ".env"
```

And `.env` contains `DEBUG=true`, and you run: `DEBUG=false uvicorn app.main:app`

What will `settings.debug` be?

A) `True` (from .env file)
B) `False` (from shell environment)
C) `False` (from default value)
D) An error occurs

---

### Q3: FastAPI Documentation

FastAPI automatically generates API documentation. Which statement is correct?

A) You must manually write OpenAPI specs
B) Documentation is only available at `/docs`
C) Both `/docs` (Swagger) and `/redoc` are available by default
D) Documentation requires a paid license

---

### Q4: Virtual Environments

Why do we use Python virtual environments?

A) They make Python run faster
B) They isolate project dependencies from system Python
C) They are required by FastAPI
D) They provide better security

---

### Q5: Container Commands

Which command shows logs for a container and follows new output?

A) `podman logs jobboard-db`
B) `podman logs -f jobboard-db`
C) `podman tail jobboard-db`
D) `podman show logs jobboard-db`

---

### Q6: pydantic-settings

What does `@lru_cache` on `get_settings()` accomplish?

A) Makes settings read-only
B) Caches the settings object to avoid re-reading .env on every call
C) Encrypts the settings
D) Allows settings to be modified at runtime

---

### Q7: Docker Compose

In `docker-compose.yml`, what does `ports: "5432:5432"` mean?

A) Container port 5432 is mapped to host port 5432
B) Two containers share port 5432
C) The database has two connections
D) Port 5432 is blocked

---

### Q8: Health Checks

Why should a `/health` endpoint check the database connection?

A) To satisfy PostgreSQL license requirements
B) To allow monitoring systems to detect when the app cannot serve requests
C) FastAPI requires it
D) It makes the API faster

---

### Q9: .gitignore

Which of these should be in `.gitignore`?

A) `requirements.txt`
B) `.env`
C) `docker-compose.yml`
D) `main.py`

---

### Q10: Debugging

In VS Code debugging, what does F10 do?

A) Continue execution
B) Step into function
C) Step over (execute current line, don't enter functions)
D) Stop debugging

---

*Answers will be provided separately after you complete the quiz.*
