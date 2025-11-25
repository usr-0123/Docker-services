# Databases

A collection of self-contained database stacks (PostgreSQL, MongoDB, SQL Server),
each in its own folder with a matching admin GUI. All configuration is driven from
a **single `.env` file at the project root**.

```
databases/
├── .env                 # the ONLY env file — all real values live here
├── .env.example         # template with placeholders (safe to commit)
├── .gitignore           # ignores .env, keeps .env.example
├── docker-compose.yml   # root orchestrator — run everything from HERE
├── postgres/            # PostgreSQL + pgAdmin
├── mongoDb/             # MongoDB + mongo-express
└── mssql/               # SQL Server + CloudBeaver
```

Each folder has its own `README.md` with provider-specific commands and notes.

---

## What's inside

| Stack     | Database          | Admin GUI      | DB port | GUI URL                |
|-----------|-------------------|----------------|---------|------------------------|
| postgres  | PostgreSQL 16     | pgAdmin 4      | 5432    | http://localhost:5050  |
| mongoDb   | MongoDB 7         | mongo-express  | 27017   | http://localhost:8081  |
| mssql     | SQL Server 2022   | CloudBeaver    | 1433    | http://localhost:8978  |

---

## How to run — from the project root

The root `docker-compose.yml` `include`s all three stacks, and Docker Compose
auto-loads the root `.env`. **Run all `docker compose` commands from this folder.**

```bash
# Start everything
docker compose up -d

# Start a single stack (its GUI pulls the DB up via depends_on)
docker compose up -d postgres pgadmin        # PostgreSQL + pgAdmin
docker compose up -d mongodb mongo-express    # MongoDB + mongo-express
docker compose up -d mssql cloudbeaver        # SQL Server + CloudBeaver

# Status / logs
docker compose ps
docker compose logs -f postgres

# Stop (keep data)
docker compose down

# Stop and DELETE all data volumes
docker compose down -v
```

---

## ⚠️ Running a stack from inside its own folder

Because there is only **one `.env` at the root**, Docker Compose will **not** find
it if you `cd` into a subfolder — it only auto-loads a `.env` from the current
directory. Running a stack directly from its folder will produce
`variable is not set` warnings and blank values.

**Preferred:** always run from the project root (see above).

**If you must run from inside a folder,** point at the root env file explicitly:

```bash
cd postgres
docker compose --env-file ../.env up -d
```

---

## Configuration

- Edit the single root **`.env`** to change credentials, database names, or host ports.
  Variable names are namespaced per provider (`POSTGRES_*`, `MONGO_*`, `MSSQL_*`,
  `PGADMIN_*`, `ME_CONFIG_*`) so there are no collisions.
- `.env` is git-ignored. Copy `.env.example` → `.env` when setting up a fresh clone:

  ```bash
  cp .env.example .env
  ```

- Passwords containing a literal `$` must be escaped as `$$` in `.env`, otherwise
  Compose treats `$` as the start of a variable reference.
