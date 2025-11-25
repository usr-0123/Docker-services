# PostgreSQL + pgAdmin

**What it's for:** PostgreSQL is a powerful open-source **relational (SQL)** database —
the default choice for transactional applications, structured data with strong
consistency, complex joins, and advanced features (JSON columns, full-text search,
extensions like PostGIS and pgvector).

**pgAdmin** is the official web-based administration and query GUI for PostgreSQL.

| Service   | Image             | Port  | Access                  |
|-----------|-------------------|-------|-------------------------|
| postgres  | postgres:16       | 5432  | `localhost:5432`        |
| pgadmin   | dpage/pgadmin4    | 5050  | http://localhost:5050   |

Credentials come from the root `.env` (`POSTGRES_*`, `PGADMIN_*`).

---

## Run (from the project root)

```bash
docker compose up -d postgres pgadmin
docker compose logs -f postgres
docker compose down            # stop; add -v to delete data
```

> To run from inside this folder instead: `docker compose --env-file ../.env up -d`

## Connect via pgAdmin

Open http://localhost:5050 (login is disabled). Register a server:

- **Host:** `postgres`  (the service name — pgAdmin runs on the same Docker network)
- **Port:** `5432`
- **Username:** `admin`  **Password:** `Postgres_Pass123!`

## Connect via CLI (psql)

```bash
# Enter psql inside the container
docker exec -it postgres psql -U admin -d appdb

# From your host (requires psql installed)
psql -h localhost -p 5432 -U admin -d appdb
```

---

## Common psql commands

```sql
-- Meta commands
\l                 -- list databases
\c appdb           -- connect to a database
\dt                -- list tables
\d tablename       -- describe a table
\du                -- list roles/users
\q                 -- quit

-- Databases & roles
CREATE DATABASE mydb;
DROP DATABASE mydb;
CREATE USER myuser WITH PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE appdb TO myuser;
ALTER USER myuser WITH PASSWORD 'newpassword';

-- Tables
CREATE TABLE users (
  id    SERIAL PRIMARY KEY,
  name  TEXT NOT NULL,
  email TEXT UNIQUE,
  created_at TIMESTAMPTZ DEFAULT now()
);
DROP TABLE users;

-- CRUD
INSERT INTO users (name, email) VALUES ('Alice', 'alice@mail.com');
SELECT * FROM users WHERE name = 'Alice';
UPDATE users SET email = 'new@mail.com' WHERE name = 'Alice';
DELETE FROM users WHERE name = 'Alice';

-- Indexes
CREATE INDEX idx_users_name ON users (name);
CREATE UNIQUE INDEX idx_users_email ON users (email);
DROP INDEX idx_users_name;
```

## Backup & restore

```bash
# Dump a database to a file (on the host)
docker exec -t postgres pg_dump -U admin appdb > appdb_backup.sql

# Restore a dump into the database
cat appdb_backup.sql | docker exec -i postgres psql -U admin -d appdb
```
