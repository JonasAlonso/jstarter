# PostgreSQL Bootstrapper for Dockerized Development

This subproject helps you bootstrap and manage multiple app-specific schemas and roles on a shared PostgreSQL container using Docker and Make.

It is designed to:

- Spin up an original **PostgreSQL root container** (`pg-<MACHINE>-<VERSION>`)
- Dynamically generate and apply **SQL initialization scripts** per app
- **Verify** the creation of new schemas and users
- Output reproducible `docker-compose.yml` and `application.yml` configs for client apps

---

## 🗂️ Directory Structure

```
postgres/
├── Makefile               # The command center
├── initdb.sql             # SQL template with __APP__ placeholder
├── .env                   # Root credentials, don't look here!
├── out/
│   ├── cauth-init.sql                 # SQL file for app
│   ├── cauth-verify.log               # Output of database verification
│   ├── cauth-docker-run.sh (optional) # Ready-to-run docker command (optional)
│   └── cauth/
│       ├── application.yml            # Config for Spring Boot apps
│       └── docker-compose.yml         # App-specific service definition
```

---

## 🚀 Getting Started

### 1. Bootstrap the PostgreSQL Root Container

```bash
make db-bootstrap
```

This will create:

- Docker container: `pg-<MACHINE>-<VERSION>`
- Volume: `pg-<MACHINE>-<VERSION>-volume`
- Network: `<MACHINE>-net`

Uses credentials from `.env` like:

```dotenv
DB=postgres
DB_PASSWORD=your-root-password
```

> You **only need to run this once** to create the main Postgres server. This note is  mainly for myself because I generated wrongly the Postgres container about a zillion times

---

### 2. Bootstrap a Schema + Role for an App

```bash
make APP=cauth APP_NAME=central-authentication postgres-app-bootstrap
```

This will:

- Expand `initdb.sql` with `__APP__` → `cauth`
- Save the result in `postgres/out/cauth-init.sql`
- Execute the SQL in the root container
- Verify the schema + role were created
- Generate:
  - `postgres/out/cauth/application.yml` (Spring Boot config)
  - `postgres/out/cauth/docker-compose.yml` (for local development)

---

### 3. Use the Output

Run your app with the generated config:

- Mount the application.yml into your Spring Boot app
- Or use the generated `docker-compose.yml` as a starting point

---

## 🧾 SQL Template (initdb.sql)

The script is dynamic and gets customized per app.

```sql
DROP SCHEMA IF EXISTS __APP__ CASCADE;
DROP ROLE IF EXISTS __APP__;
CREATE ROLE __APP__ LOGIN PASSWORD '__APP___pass';
CREATE SCHEMA __APP__ AUTHORIZATION __APP__;
REVOKE ALL ON SCHEMA __APP__ FROM PUBLIC;
GRANT USAGE, CREATE ON SCHEMA __APP__ TO __APP__;
GRANT USAGE, CREATE ON SCHEMA __APP__ TO postgres;
ALTER DEFAULT PRIVILEGES IN SCHEMA __APP__ GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO __APP__;
ALTER DEFAULT PRIVILEGES IN SCHEMA __APP__ GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO postgres;
```

---


## 🧹 Cleanup

Remove generated artifacts:

```bash
rm -rf postgres/out/
```


## 🧠 Tips

- Use short values for `APP` (used in DB identifiers)
- Use readable values for `APP_NAME` (used in app configs)
- You can run this bootstrapper for as many apps as you want — the root container is shared

---


## 🤘 Author

Crafted by a developer who loves schemas, like for real, hates boilerplate, and will absolutely judge your indentation.
and sometimes wants to template the templates
