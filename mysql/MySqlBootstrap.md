# MySQL Bootstrapper for Dockerized Development

This subproject helps you bootstrap and manage multiple MySQL app-specific databases on a shared base container using Docker and Make.

It is designed to:

- Spin up an original **MySQL root container** (`mysql-4kgbbad-1`)
- Dynamically generate and apply **SQL initialization scripts** per app
- **Verify** the creation of new databases and users
- Output reproducible `` scripts to create clone containers (if needed)

## Directory Structure

```
mysql/
├── Makefile              # The command center
├── initdb.sql            # SQL template
├── .env                  # Root credentials
├── out/
│   ├── cauth3-initdb.sql     # SQL file for app
│   ├── cauth3-verify.log     # Output of database verification
│   └── cauth3-docker-run.sh  # Ready-to-run docker command
```

## Getting Started

### 1. Create the Root Container

This is the original database server. It uses root credentials from `.env`.

```bash
make mysql-bootstrap
```

This will create:

- Docker container: `mysql-4kgbbad-1`
- Volume: `mysql-4kgbbad-8.3-volume`
- Network: `4kgbbad-net`

The root user, default database, and user will be taken from:

```dotenv
DB_ROOT_PASSWORD=...
DB_DATABASE=...
DB_USER=...
DB_PASSWORD=...
```

> You **only run this once** to initialize the infrastructure.

---

### 2. Bootstrap an App-Specific Database

```bash
make APP=cauth3 mysql-app-bootstrap
```

This will:

- Generate `out/cauth3-initdb.sql` from `initdb.sql` template
- Execute the SQL inside the original container
- Verify the database and user creation (`out/cauth3-verify.log`)
- Output a `docker run` script for a potential clone container at `out/cauth3-docker-run.sh`

---

### 3. Run a Clone (Optional)

You can run a separate container (e.g., for testing, migrations, or read-only queries) with:

```bash
bash out/cauth3-docker-run.sh
```

> This container uses the same volume and network as the root instance, but only one MySQL process can safely access the volume at a time.

---

## SQL Template (`initdb.sql`)

The script uses the `__APP__` placeholder and gets expanded dynamically. Example content:

```sql
CREATE DATABASE IF NOT EXISTS __APP___db;
CREATE USER IF NOT EXISTS '__APP__'@'%' IDENTIFIED BY '__APP___pass';
GRANT ALL PRIVILEGES ON __APP___db.* TO '__APP__'@'%';
FLUSH PRIVILEGES;
```

---

## Contributing

Pull requests welcome. Add features like:

- Secure password generation
- Support for multiple database engines
- Export credentials to `.properties` for integration into Java apps

---

## License

use it, tweak it, break it, rebuild it.

---


