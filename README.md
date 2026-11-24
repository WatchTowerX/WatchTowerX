<div align="center">

# 🔔 WatchTowerX

**Self-hosted uptime monitoring, alerting, and public status pages.**

Keep an eye on your websites, APIs, servers, and DNS. Get alerted when something breaks, and keep your users informed with a status page.

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#-contributing)

[Features](#-features) •
[Quick Start](#-quick-start) •
[Configuration](#%EF%B8%8F-configuration) •
[API](#-api-reference) •
[Contributing](#-contributing)

</div>

---

## ✨ Features

### 📡 Monitoring
- **Multiple check types**: HTTP(S), TCP port, ICMP ping, and DNS
- **Keyword checks**: confirm that a response body contains the text you expect
- **Flexible intervals**: check every 1, 3, or 5 minutes, scheduled with APScheduler
- **Retries**: set how many consecutive failures count as "down" before anyone is alerted
- **Monitor groups**: nest monitors under a parent group whose status comes from its children
- **Uptime history**: uptime percentages, response-time charts, and per-check results

### 🚨 Alerting
- **Microsoft Teams and compatible webhooks**: alerts sent as Adaptive Cards
- **Email** over SMTP, Amazon SES, or Microsoft Entra (Graph API)
- **Recovery notifications** when a monitor comes back up
- **Test button** on every channel, so you can check the setup before you need it

### 🌐 Status Pages
- **Public status pages** served at `/status/{slug}`
- **Components** linked to your monitors
- **Incidents** with timeline updates
- **Scheduled maintenance** windows

### 👥 Team
- JWT authentication with a first-run setup flow
- Staff management with role-based access
- Audit logging

---

## 🧱 Tech Stack

| Layer      | Technology                                  |
|------------|---------------------------------------------|
| Backend    | Python 3.12, FastAPI, SQLAlchemy 2 (async)  |
| Database   | PostgreSQL 16 (asyncpg)                     |
| Cache      | Redis 7                                     |
| Scheduler  | APScheduler                                 |
| Frontend   | React 18, Vite, TanStack Query, React Router|
| Charts     | Recharts                                    |
| Auth       | JWT (python-jose and passlib/bcrypt)        |
| Deployment | Docker and Docker Compose                   |

---

## 🚀 Quick Start

### Prerequisites
- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/)

### Run with Docker (recommended)

```bash
# 1. Clone the repository
git clone https://github.com/om-ghimire/WatchTowerX.git
cd WatchTowerX

# 2. Configure the environment
cp .env.example .env
# Edit .env and set SECRET_KEY and REDIS_PASSWORD at minimum

# 3. Start everything
docker compose up --build
```

Then open:

| Service       | URL                          |
|---------------|------------------------------|
| Dashboard     | http://localhost:5173        |
| API docs      | http://localhost:8000/docs   |
| Health check  | http://localhost:8000/health |

On first launch, the dashboard asks you to create the initial admin account.

### Run locally (without Docker)

You need **Python 3.12+**, **Node.js 20+**, **PostgreSQL 16**, and **Redis 7** running locally.

```bash
# Backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

```bash
# Frontend (in a second terminal)
cd frontend
npm install
npm run dev
```

---

## ⚙️ Configuration

All settings are read from environment variables or `.env`. See [`.env.example`](.env.example).

| Variable                         | Description                         | Default                    |
|----------------------------------|-------------------------------------|----------------------------|
| `APP_NAME`                       | Application name                    | `WatchTowerX`              |
| `SECRET_KEY`                     | JWT signing key. **Change this!**   | `change-me-in-production`  |
| `ALGORITHM`                      | JWT algorithm                       | `HS256`                    |
| `ACCESS_TOKEN_EXPIRE_MINUTES`    | Token lifetime in minutes           | `60`                       |
| `DATABASE_URL`                   | Async PostgreSQL connection string  | `postgresql+asyncpg://...` |
| `REDIS_URL`                      | Redis connection string             | `redis://localhost:6379`   |
| `REDIS_PASSWORD`                 | Redis password                      | *(empty)*                  |
| `DEFAULT_CHECK_INTERVAL_MINUTES` | Default check interval              | `5`                        |

> **Email alerts** (SMTP, SES, or Microsoft Entra Graph) are configured per alert channel under **Settings** in the dashboard, not through global environment variables.
> For Microsoft Entra, the app registration needs the `Mail.Send` application permission with admin consent.

---

## 📁 Project Structure

```
WatchTowerX/
├── app/                      # FastAPI backend
│   ├── api/routes/           #   REST endpoints
│   ├── core/                 #   config, security, Redis client
│   ├── db/                   #   async SQLAlchemy session
│   ├── models/               #   ORM models
│   ├── schemas/              #   Pydantic schemas
│   └── services/             #   check engine, scheduler, alerts, email
├── frontend/                 # React + Vite dashboard
│   └── src/
│       ├── components/       #   UI, dashboard, and monitor components
│       ├── hooks/            #   custom React hooks
│       ├── lib/              #   API client, auth context, date helpers
│       └── pages/            #   Dashboard, Monitors, Incidents, Status Pages, Settings
├── tests/                    # Backend tests (pytest)
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```

---

## 📖 API Reference

Interactive OpenAPI docs are available at **`/docs`** (Swagger UI) and **`/redoc`** once the API is running.

<details>
<summary><b>Endpoint overview</b></summary>

| Area              | Method(s)                  | Endpoint                                               |
|-------------------|----------------------------|--------------------------------------------------------|
| Auth              | `GET`                      | `/api/auth/setup`                                      |
|                   | `POST`                     | `/api/auth/register`, `/api/auth/login`                |
|                   | `GET`                      | `/api/auth/me`                                         |
| Staff             | `GET` `POST`               | `/api/auth/staff`                                      |
|                   | `PATCH` `DELETE`           | `/api/auth/staff/{id}`                                 |
| Monitors          | `GET` `POST`               | `/api/monitors/`                                       |
|                   | `GET` `PATCH` `DELETE`     | `/api/monitors/{id}`                                   |
| Results           | `GET`                      | `/api/monitors/{id}/results`                           |
|                   | `GET`                      | `/api/monitors/{id}/stats?hours=24`                    |
| Groups            | `GET` `POST`               | `/api/groups`                                          |
|                   | `PATCH` `DELETE`           | `/api/groups/{id}`                                     |
|                   | `GET` `POST`               | `/api/groups/{id}/children`                            |
|                   | `PATCH`                    | `/api/groups/{id}/children/reorder`                    |
| Alert channels    | `GET` `POST`               | `/api/alert-channels/`                                 |
|                   | `PATCH` `DELETE`           | `/api/alert-channels/{id}`                             |
|                   | `POST`                     | `/api/alert-channels/{id}/test`                        |
| Status pages      | `GET` `POST`               | `/api/status-pages`                                    |
|                   | `PATCH` `DELETE`           | `/api/status-pages/{id}`                               |
| Components        | `GET` `POST`               | `/api/status-pages/{id}/components`                    |
| Incidents         | `GET` `POST`               | `/api/status-pages/{id}/incidents`                     |
|                   | `POST`                     | `/api/status-pages/{id}/incidents/{iid}/updates`       |
| Maintenance       | `GET` `POST`               | `/api/status-pages/{id}/maintenance`                   |
|                   | `POST`                     | `/api/status-pages/{id}/maintenance/{mid}/updates`     |
| Public            | `GET`                      | `/status/{slug}`, `/api/status/{slug}`                 |
| Health            | `GET`                      | `/health`                                              |

</details>

---

## 🧪 Running Tests

```bash
pip install -r requirements.txt
pytest
```

The tests use an in-memory SQLite database (`aiosqlite`), so you don't need PostgreSQL to run them.

---

## 🗺️ Roadmap

- [ ] Slack, Discord, and Telegram alert channels
- [ ] SSL certificate expiry monitoring
- [ ] Multi-region checks
- [ ] Alembic migrations
- [ ] Production Docker image for the frontend

Have an idea? [Open an issue](https://github.com/om-ghimire/WatchTowerX/issues).

---

## 🤝 Contributing

Contributions are welcome, from bug reports and feature ideas to pull requests.

1. **Fork** the repository
2. **Create a branch**: `git checkout -b feat/my-feature`
3. **Commit** using [Conventional Commits](https://www.conventionalcommits.org/): `git commit -m "feat: add my feature"`
4. **Push**: `git push origin feat/my-feature`
5. **Open a Pull Request**

Before you submit:
- Run `pytest` and make sure all tests pass
- Add tests for new backend behaviour
- Keep PRs focused, with one feature or fix per PR

---

## 🔒 Security

If you find a security vulnerability, please **do not** open a public issue. Report it privately through [GitHub Security Advisories](https://github.com/om-ghimire/WatchTowerX/security/advisories/new).

---

## 📄 License

WatchTowerX is released under the [MIT License](LICENSE).

---

<div align="center">

Made with ❤️ by [Om Ghimire](https://github.com/om-ghimire)

If WatchTowerX is useful to you, please consider giving it a ⭐

</div>
