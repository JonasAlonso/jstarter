# 🧪 Universal DB Bootstrapper

Welcome to the **Universal DB Bootstrapper**, a Makefile-based setup for spinning up local database environments for your backend apps — MySQL, PostgreSQL, and beyond.

No GUI. No clicking around. Just pure config-driven database bootstrapping with style, clarity, and repeatability.

---

## ✨ Features

- 🔧 **Pluggable Driver Support** — currently supports PostgreSQL and MySQL
- 🧬 **App-Specific Schema/DB Isolation** — each app gets its own DB or schema and user
- 💥 **Reproducible SQL Scripts** — init files are templated and versioned
- 🧠 **Generated Configs** — get ready-to-use `application.yml`, `docker-compose.yml`, and more
- 💾 **Volume & Network Persistence** — all containers share a stable dev environment

---

## 🗂️ Structure Overview

```
.
├── Makefile               # Top-level dispatcher (optional)
├── postgres/              # PostgreSQL driver
│   ├── Makefile
│   ├── initdb.sql
│   ├── .env
│   └── out/
├── mysql/                 # MySQL driver
│   ├── Makefile
│   ├── initdb.sql
│   ├── .env
│   └── out/
└── README.md              # You're reading this
```

---

## 🚀 Quickstart

1. **Choose a Driver Folder**  
   e.g., `cd postgres/` or `cd mysql/`

2. **Configure Your `.env`**  
   Each driver folder has its own `.env` with root DB credentials

3. **Bootstrap the Root DB Container**

```bash
make db-bootstrap
```

4. **Bootstrap a New App**

```bash
make APP=cauth APP_NAME=central-authentication <driver>-app-bootstrap
```

> Replace `<driver>` with `postgres` or `mysql`.

---

## 🛠️ Template Tokens

Each `initdb.sql` contains placeholders like:

```sql
CREATE DATABASE IF NOT EXISTS __APP___db;
CREATE USER IF NOT EXISTS '__APP__'@'%' IDENTIFIED BY '__APP___pass';
```

These will be expanded per app into unique, reproducible SQL.

---

## 🧬 Output Examples

Every app bootstrap generates:

- `out/cauth-init.sql`
- `out/cauth-verify.log`
- `out/cauth/docker-compose.yml`
- `out/cauth/application.yml`
- Optionally: `out/cauth-docker-run.sh`

---

## 🧼 Cleanup

To nuke everything the clean way:

```bash
rm -rf out/
docker container rm -f <container-name>
docker volume rm <volume-name>
docker network rm <network-name>
```

---

## 📦 Supported Drivers

| Driver     | Init Style | Container       | Status       |
|------------|------------|------------------|--------------|
| PostgreSQL | Schema     | `postgres`       | ✅ Stable     |
| MySQL      | Full DB    | `mysql`          | ✅ Stable     |
| Mongo      | TBD        | `MongoDB`        | 🚧 Planned    |
| MariaDB    | TBD        | `mariadb`        | 💤 Optional   |

> Want to add support for your favorite DB? PRs welcome.

---

## 📚 Per-Driver Docs

these are not the real names fml
- [PostgreSQL README](./postgres/README.md)
- [MySQL README](./mysql/README.md)

---

## ⚖️ License

MIT — use, abuse, remix, automate.

---

## 🧙 Author

Conjured by a backend sorcerer who writes Makefiles like love letters and treats `init.sql` like poetry.