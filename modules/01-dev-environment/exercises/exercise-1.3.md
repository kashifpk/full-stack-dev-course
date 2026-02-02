# Exercise 1.3: VS Code Debugging

## Objective
Set up and use VS Code debugging for Python/FastAPI.

## Tasks

### Task 1: Create Debug Configuration

Create `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "FastAPI Debug",
            "type": "debugpy",
            "request": "launch",
            "module": "uvicorn",
            "args": [
                "app.main:app",
                "--reload",
                "--host", "0.0.0.0",
                "--port", "8000"
            ],
            "cwd": "${workspaceFolder}/backend",
            "env": {
                "PYTHONPATH": "${workspaceFolder}/backend"
            },
            "jinja": true
        },
        {
            "name": "Python: Current File",
            "type": "debugpy",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "cwd": "${workspaceFolder}/backend",
            "env": {
                "PYTHONPATH": "${workspaceFolder}/backend"
            }
        },
        {
            "name": "Pytest Debug",
            "type": "debugpy",
            "request": "launch",
            "module": "pytest",
            "args": [
                "-v",
                "-s",
                "${workspaceFolder}/backend/tests"
            ],
            "cwd": "${workspaceFolder}/backend"
        }
    ]
}
```

### Task 2: Set Breakpoints

1. Open `backend/app/main.py`
2. Click in the gutter (left of line numbers) on the `return` line in the `root()` function
3. A red dot appears - this is your breakpoint

### Task 3: Start Debugging

1. Press `F5` or click "Run and Debug" in sidebar
2. Select "FastAPI Debug"
3. Open http://localhost:8000 in browser
4. VS Code should pause at your breakpoint

### Task 4: Debug Controls

While paused at a breakpoint:

| Key | Action |
|-----|--------|
| F5 | Continue |
| F10 | Step Over |
| F11 | Step Into |
| Shift+F11 | Step Out |
| Shift+F5 | Stop |

### Task 5: Inspect Variables

While paused:
1. Hover over variables to see their values
2. Use the "Variables" panel on the left
3. Use the "Debug Console" at the bottom to evaluate expressions

Try typing in Debug Console:
```python
settings.debug
settings.database_url
```

### Task 6: Conditional Breakpoints

1. Right-click on a breakpoint
2. Select "Edit Breakpoint"
3. Add a condition like `settings.debug == True`
4. The breakpoint only triggers when the condition is true

## Verification

- [ ] Debug configuration is set up
- [ ] Can set and hit breakpoints
- [ ] Can step through code
- [ ] Can inspect variables
- [ ] Can use Debug Console

## Tips

- **Logpoints:** Right-click gutter → Add Logpoint. Logs without pausing.
- **Watch expressions:** Add expressions to watch panel for continuous monitoring
- **Call stack:** See the function call hierarchy when paused
