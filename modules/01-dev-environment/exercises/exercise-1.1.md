# Exercise 1.1: Container Basics

## Objective
Get comfortable with basic container commands.

> **Note:** The commands below use `docker`. If you're using Podman, simply replace `docker` with `podman` and `docker compose` with `podman-compose`.

## Tasks

### Task 1: Explore Running Containers

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs
docker logs jobboard-db

# Follow logs in real-time
docker logs -f jobboard-db
```

**Question:** What information does `docker ps` show you? (List at least 5 columns)

### Task 2: Execute Commands in Container

```bash
# Open a shell in the running postgres container
docker exec -it jobboard-db /bin/bash

# Inside the container, explore:
ls /var/lib/postgresql/data
cat /etc/passwd | grep postgres
exit
```

### Task 3: Database Interaction

```bash
# Connect to PostgreSQL directly
docker exec -it jobboard-db psql -U jobboard -d jobboard

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
docker compose stop db

# Verify it's stopped
docker ps

# Start it again
docker compose start db

# Restart
docker compose restart db
```

### Task 5: Volume Persistence

```bash
# Stop and remove the container
docker compose down

# Start again
docker compose up -d db

# Connect and verify data persists
docker exec -it jobboard-db psql -U jobboard -d jobboard -c "SELECT 1;"
```

## Verification

You've completed this exercise when you can:
- [ ] List running and all containers
- [ ] View container logs
- [ ] Execute commands inside a container
- [ ] Connect to PostgreSQL in the container
- [ ] Stop and start containers
- [ ] Understand that volume data persists across container restarts
