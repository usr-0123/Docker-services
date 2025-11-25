# Microsoft SQL Server + CloudBeaver

**What it's for:** SQL Server is Microsoft's enterprise **relational (SQL)** database.
It's the natural choice for .NET / Windows ecosystems and applications that rely on
T-SQL, stored procedures, and Microsoft tooling. This stack runs the **Developer**
edition (full features, free for non-production use).

**CloudBeaver** is a browser-based universal SQL client. SQL Server has no free
official web GUI, so CloudBeaver fills that role — it also connects to PostgreSQL,
MongoDB, and others, so you can use one UI for every database in this project.

| Service      | Image                                   | Port  | Access                  |
|--------------|-----------------------------------------|-------|-------------------------|
| mssql        | mcr.microsoft.com/mssql/server:2022     | 1433  | `localhost:1433`        |
| cloudbeaver  | dbeaver/cloudbeaver                     | 8978  | http://localhost:8978   |

Credentials come from the root `.env` (`MSSQL_*`).
Default admin login: user **`sa`** / password **`MsSql_Pass123!`**.

> **Note:** SQL Server enforces password complexity — at least 8 chars with 3 of:
> uppercase, lowercase, digits, symbols. Keep this in mind if you change the password.

---

## Run (from the project root)

```bash
docker compose up -d mssql cloudbeaver
docker compose logs -f mssql
docker compose down            # stop; add -v to delete data
```

> To run from inside this folder instead: `docker compose --env-file ../.env up -d`

## Connect via CloudBeaver

Open http://localhost:8978, complete the first-run admin setup, then create a
**SQL Server** connection:

- **Host:** `mssql`  (the service name — same Docker network)
- **Port:** `1433`
- **Username:** `sa`  **Password:** `MsSql_Pass123!`

## Connect via CLI (sqlcmd)

```bash
# Enter an interactive sqlcmd session inside the container
docker exec -it mssql /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'MsSql_Pass123!' -C

# Run a single query without an interactive session
docker exec -it mssql /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P 'MsSql_Pass123!' -C \
  -Q "SELECT name FROM sys.databases;"
```

> `-C` trusts the server's self-signed certificate. In `sqlcmd`, statements are
> executed by typing `GO` on its own line.

---

## Common T-SQL commands

```sql
-- Databases
SELECT name FROM sys.databases;          -- list databases
CREATE DATABASE appdb;
DROP DATABASE appdb;
USE appdb;
GO

-- Logins & users
CREATE LOGIN myuser WITH PASSWORD = 'MyUser_Pass123!';
CREATE USER myuser FOR LOGIN myuser;
ALTER ROLE db_owner ADD MEMBER myuser;
GO

-- Tables
CREATE TABLE users (
  id         INT IDENTITY(1,1) PRIMARY KEY,
  name       NVARCHAR(100) NOT NULL,
  email      NVARCHAR(255) UNIQUE,
  created_at DATETIME2 DEFAULT SYSUTCDATETIME()
);
GO
DROP TABLE users;
GO

-- CRUD
INSERT INTO users (name, email) VALUES ('Alice', 'alice@mail.com');
SELECT * FROM users WHERE name = 'Alice';
UPDATE users SET email = 'new@mail.com' WHERE name = 'Alice';
DELETE FROM users WHERE name = 'Alice';
GO

-- Indexes
CREATE INDEX idx_users_name ON users (name);
CREATE UNIQUE INDEX idx_users_email ON users (email);
DROP INDEX idx_users_name ON users;
GO
```

## Backup & restore

```bash
# Back up a database to a .bak inside the container
docker exec -it mssql /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'MsSql_Pass123!' -C \
  -Q "BACKUP DATABASE appdb TO DISK = '/var/opt/mssql/backup/appdb.bak' WITH FORMAT;"

# Restore a .bak
docker exec -it mssql /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'MsSql_Pass123!' -C \
  -Q "RESTORE DATABASE appdb FROM DISK = '/var/opt/mssql/backup/appdb.bak' WITH REPLACE;"
```
