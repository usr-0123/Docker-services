# Docker services

# NOTE: **Comment out the services**

# Getting started
- Create `.env` file that is similar to the `.env.template` in the same location.
- Copy and supply values to the variables transferred from the template into the environment file.
- Make sure `Docker Desktop` is up and running.
- In the same folder, run the command
```bash
docker compose up
```

## RabbitMQ

## PostgreSQL and pgAdmin
You log to `pgAdmin` in with:
- Login email: `POSTGRE_DB_EMAIL`
- Login password: `POSTGRE_DB_PASSWORD`

After login, you’ll add a new server inside pgAdmin:
- Host: `postgres` - the service name in docker compose, resolves via Docker network DNS
- Port: 5432
- Username: `user`
- Password: `POSTGRE_DB_PASSWORD`
- Database: `POSTGRE_DB`