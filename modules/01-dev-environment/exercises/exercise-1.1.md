# Exercise 1.1: Container Basics

## Objective
Get comfortable with basic container commands.

> **Note:** The commands below use `podman`. If you're using Docker, simply replace `podman` with `docker` and `podman compose` with `docker compose`.

## Tasks

### Task 1: Explore Running Containers

```bash
# List running containers
podman ps

# List all containers (including stopped)
podman ps -a

# View container logs
podman logs jobboard-db

# Follow logs in real-time
podman logs -f jobboard-db
```

**Question:** What information does `podman ps` show you? (List at least 5 columns)

### Task 2: Execute Commands in Container

```bash
# Open a shell in the running postgres container
podman exec -it jobboard-db /bin/bash

# Inside the container, explore:
ls /var/lib/postgresql/data
cat /etc/passwd | grep postgres
exit
```

### Task 3: Database Interaction

```bash
# Connect to PostgreSQL directly
podman exec -it jobboard-db psql -U jobboard -d jobboard

# Inside psql, run these commands:
\l                    -- List all databases
\dt                   -- List tables (empty for now)
\du                   -- List users
SELECT version();     -- Check PostgreSQL version
\q                    -- Quit
```

### Task 4: Container Lifecycle

```bash
# Stop the database
podman compose stop db

# Verify it's stopped
podman ps

# Start it again
podman compose start db

# Restart
podman compose restart db
```

### Task 5: Volume Persistence

```bash
# Stop and remove the container
podman compose down

# Start again
podman compose up -d db

# Connect and verify data persists
podman exec -it jobboard-db psql -U jobboard -d jobboard -c "SELECT 1;"
```

## Verification

You've completed this exercise when you can:
- [ ] List running and all containers
- [ ] View container logs
- [ ] Execute commands inside a container
- [ ] Connect to PostgreSQL in the container
- [ ] Stop and start containers
- [ ] Understand that volume data persists across container restarts
