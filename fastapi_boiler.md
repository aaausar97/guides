# FastAPI + Tortoise ORM + Docker + Aerich Boilerplate Guide

## 1. Project Structure

```
project_root/
│   README.md
│   pyproject.toml
│   requirements.txt
│   .env
│
├── app/
│   ├── main.py
│   ├── aerich_config.py
│   ├── models/
│   ├── schemas/
│   ├── routes/
│   ├── methods/
│   └── tests/
│
├── environment/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── docker-compose.test.yml
│
└── migrations/
```

## 2. Environment & Dependencies

- Use [Poetry](https://python-poetry.org/) for dependency management.
- Add dependencies:
  ```sh
  poetry add fastapi uvicorn tortoise-orm passlib[bcrypt] python-jose
  poetry add --group dev pytest httpx asgi-lifespan aerich
  ```
- Create a `.env` file in the project root for DB and secret config:
  ```env
  POSTGRES_USER=youruser
  POSTGRES_PASSWORD=yourpassword
  POSTGRES_DB=yourdb
  SECRET_KEY=your_secret
  ```

## 3. Models, Schemas, and Routers

- **Models:** Define Tortoise ORM models in `app/models/` (e.g., `user.py`, `sales_lead.py`).
- **Schemas:** Define Pydantic schemas in `app/schemas/` for validation/serialization.
- **Routers:** Implement API endpoints in `app/routes/` and import them in `main.py`.
- **Methods:** Place business logic (e.g., authentication) in `app/methods/`.

## 4. Main App Setup

- In `app/main.py`, create the FastAPI app, include routers, and register Tortoise ORM:
  ```python
  from fastapi import FastAPI
  from tortoise.contrib.fastapi import register_tortoise
  import os
  from app.routes.auth import router as auth_router
  from app.routes.sales_lead import router as sales_lead_router

  app = FastAPI()
  app.include_router(auth_router)
  app.include_router(sales_lead_router)

  register_tortoise(
      app,
      db_url=f"postgres://{os.getenv('POSTGRES_USER')}:{os.getenv('POSTGRES_PASSWORD')}@db:5432/{os.getenv('POSTGRES_DB')}",
      modules={"models": ["app.models.user", "app.models.sales_lead"]},
      generate_schemas=True,
      add_exception_handlers=True,
  )
  ```

## 5. Dockerization

- **Dockerfile:** Build the FastAPI app image.
- **docker-compose.yml:** Define services for app, Postgres, and pgAdmin. Mount volumes for code and DB data.
- **docker-compose.test.yml:** For running tests in isolation.
- Build and run:
  ```sh
  docker compose -f environment/docker-compose.yml up --build
  ```

## 6. Aerich Migrations

- **Aerich config:** In `app/aerich_config.py`, define `TORTOISE_ORM` with all models and DB connection.
- **aerich.ini:** Should point to your config and migrations folder.
- **Initialize migrations (first time only):**
  ```sh
  docker compose -f environment/docker-compose.yml exec app poetry run aerich init-db
  ```
- **After model changes:**
  ```sh
  docker compose -f environment/docker-compose.yml exec app poetry run aerich migrate
  docker compose -f environment/docker-compose.yml exec app poetry run aerich upgrade
  ```

## 7. Testing

- Write tests in `app/tests/` using `pytest` and `httpx`.
- Run tests locally:
  ```sh
  pytest
  ```
- Or with Docker Compose:
  ```sh
  docker compose -f environment/docker-compose.test.yml up --build --abort-on-container-exit
  ```

## 8. Best Practices & Tips

- Use `.env` for all secrets and DB config.
- Keep Docker and deployment files in `environment/`.
- Use Aerich for safe, versioned migrations (never rely on `generate_schemas=True` in production).
- Document all workflows in your README or a boilerplate guide.
- Pin dependencies for reproducibility.
- Modularize code: keep models, schemas, routes, and business logic separate.

---

**Result:**  
You now have a robust, modular, and production-ready FastAPI project with authentication, migrations, Dockerized workflows, and a clean structure—ready for rapid development and deployment!

## 9. Example Dockerfiles

### FastAPI + Poetry Dockerfile

```Dockerfile
FROM python:3.13-slim

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

WORKDIR /code

RUN apt-get update && apt-get install -y build-essential libpq-dev && rm -rf /var/lib/apt/lists/*

RUN pip install --upgrade pip && pip install poetry

COPY pyproject.toml poetry.lock ./
RUN poetry config virtualenvs.create false && poetry install --no-interaction --no-ansi

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Test Runner Dockerfile (via docker-compose.test.yml)

You can use the same Dockerfile as above, but override the command in your `docker-compose.test.yml`:

```yaml
services:
  app:
    build:
      context: ..
      dockerfile: environment/Dockerfile
    command: ["pytest"]
    # ...other config...
```

---

## 10. Major Dependencies & Documentation

- [FastAPI](https://fastapi.tiangolo.com/) — Modern, fast (high-performance) web framework for building APIs with Python 3.7+.
- [Tortoise ORM](https://tortoise-orm.readthedocs.io/en/latest/) — Easy-to-use asyncio ORM inspired by Django.
- [Aerich](https://tortoise-orm.readthedocs.io/en/latest/migration.html) — Database migrations tool for Tortoise ORM.
- [Uvicorn](https://www.uvicorn.org/) — Lightning-fast ASGI server for Python web apps.
- [Poetry](https://python-poetry.org/docs/) — Python dependency management and packaging.
- [Passlib](https://passlib.readthedocs.io/en/stable/) — Secure password hashing library.
- [python-jose](https://python-jose.readthedocs.io/en/latest/) — JOSE implementation for JWT authentication.
- [pytest](https://docs.pytest.org/en/stable/) — Simple, powerful testing framework for Python.
- [httpx](https://www.python-httpx.org/) — Next generation HTTP client for Python (used for testing APIs).
- [asgi-lifespan](https://github.com/florimondmanca/asgi-lifespan) — ASGI Lifespan protocol utilities (for testing FastAPI startup/shutdown).
- [Docker](https://docs.docker.com/) — Containerization platform.
- [Docker Compose](https://docs.docker.com/compose/) — Tool for defining and running multi-container Docker applications.
- [PostgreSQL](https://www.postgresql.org/docs/) — Advanced open source relational database.
- [pgAdmin](https://www.pgadmin.org/docs/) — PostgreSQL database management tool (optional, for local dev).

