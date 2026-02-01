# Database (PostgreSQL in Docker)

This repo uses a local PostgreSQL database running in a Docker container.  
Goal: reproducible setup, no local DB installation, and an easy path to later hosting.

---

## Requirements
- Docker Desktop installed and running

---

## Configuration (local only)
Create a local file **`docker/db/.env`** (do **not** commit it).  
Use `docker/db/.env.example` as a template.

Example `.env` values:
- `POSTGRES_DB=tickets`
- `POSTGRES_USER=tickets_user`
- `POSTGRES_PASSWORD=...`
- `CONTAINER_NAME=ticket_db`
- `VOLUME_NAME=ticket_db_data`
- `PORT=5432`

> Never commit real passwords to Git.

---

## Start the database (first time)
1) Create the persistent volume (stores the actual DB data):
```powershell

docker volume create ticket_db_data

Start PostgreSQL container (replace password!):

docker run -d --name ticket_db `
  -e POSTGRES_DB=ticket_system `
  -e POSTGRES_USER=admin `
  -e POSTGRES_PASSWORD=YOUR_PASSWORD `
  -p 5432:5432 `
  -v ticket_db_data:/var/lib/postgresql/data `
  postgres:16

Verify it is running

docker ps
docker logs ticket_db --tail 50
Look for something like: database system is ready to accept connections

Open a SQL shell (psql)

docker exec -it ticket_db psql -U tickets_user -d tickets

Useful psql commands:

\dt        -- list tables
\d <table> -- describe table
\q         -- quit
Stop / start / remove container

Stop:

docker stop ticket_db

Start again:

docker start ticket_db

Remove container (data stays, because volume stays):

docker rm ticket_db

⚠️ Delete all DB data (removes volume):

docker volume rm ticket_db_data

Export schema (structure only, no data)

Exports tables, constraints, indexes, etc. into docker/db/schema.sql:

docker exec -t ticket_db pg_dump -U tickets_user -d tickets --schema-only > docker/dbschema.sql

Import schema (structure only)

Imports docker/db/schema.sql into the DB:

Get-Content docker/db/schema/schema.sql -Raw | docker exec -i ticket_db psql -U tickets_user -d tickets
Note: If objects already exist, import may fail. In professional setups you usually use migrations (Flyway/Liquibase).

Connection details (default)
Host: localhost

Port: 5432

Database: tickets

User: tickets_user

Password: from your local .env

