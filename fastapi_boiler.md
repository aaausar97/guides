# FastAPI + Tortoise ORM + Docker + Aerich Boilerplate Guide

## 1. Project Initialization

- **Create project structure:**
  ```
  app/
    models/
    schemas/
    routes/
    methods/
    tests/
  environment/
  ```

- **Initialize Poetry and add dependencies:**
  ```sh
  poetry init
  poetry add fastapi uvicorn tortoise-orm passlib[bcrypt] python-jose
  poetry add --group dev pytest httpx asgi-lifespan aerich
  ```

## 2. Application Code

- **Define models** in `app/models/` (e.g., `user.py`, `sales_lead.py`).
- **Create Pydantic schemas** in `app/schemas/`.
- **Write API routes** in `app/routes/`.
- **Add business logic** in `app/methods/`.
- **Register routers and Tortoise ORM** in `app/main.py`.

## 3. Dockerization

- **Create `environment/Dockerfile`** for the FastAPI app.
- **Create `environment/docker-compose.yml`** for app, Postgres, and pgAdmin.
- **Add `.env`** in the project root for environment variables.

## 4. Tortoise ORM & Aerich Setup

- **Configure Tortoise in `main.py`:**
  ```python
  register_tortoise(
      app,
      db_url=...,
      modules={"models": ["app.models.user", "app.models.sales_lead"]},
      generate_schemas=True,
      add_exception_handlers=True,
  )
  ```

- **Create `app/aerich_config.py`** with a `TORTOISE_ORM` dict for Aerich.

- **Initialize Aerich:**
  ```sh
  docker compose -f environment/docker-compose.yml up -d
  docker compose -f environment/docker-compose.yml exec app poetry run aerich init-db
  ```

- **After model changes:**
  ```sh
  docker compose -f environment/docker-compose.yml exec app poetry run aerich migrate
  docker compose -f environment/docker-compose.yml exec app poetry run aerich upgrade
  ```

## 5. Testing

- **Write tests** in `app/tests/` using `pytest` and `httpx`.
- **Run tests locally:**
  ```sh
  pytest
  ```
- **Or with Docker Compose:**
  ```sh
  docker compose -f environment/docker-compose.test.yml up --build --abort-on-container-exit
  ```

## 6. Best Practices

- **Use environment variables** for secrets and DB config.
- **Keep Docker and deployment files in `environment/`.**
- **Use Aerich for safe, versioned migrations.**
- **Document all workflows in your README.**
- **Pin dependencies for reproducibility.**

---

**Result:**  
You now have a robust, modular, and production-ready FastAPI project with authentication, migrations, Dockerized workflows, and a clean structure—ready for rapid development and deployment!

