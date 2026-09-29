# Day-by-Day Mentoring Plan: Build a Real Python Project (FastAPI + MySQL)

**Mentee:** Fresh graduate. Knows Python basics, a little FastAPI and SQL. New to design, ER diagrams, project setup and authentication.
**Mentor:** Experienced full-stack developer, new to Python. This document includes the Python code and commands you need.
**Time:** 30–60 minutes per session, about 5 sessions a week, roughly 6 weeks.
**Focus:** One real project built end to end the way it is done at work. Python basics are not taught here; he learns those on his own.

---

## 1. Teaching Principles (So He Is Not Overwhelmed)

1. **One new concept per session.** If a session feels heavy, split it into two days.
2. **Explain why before how.** Example: "Why do we hash passwords?" comes before any code.
3. **I do → We do → You do.** You demo a small piece, build the next piece together, and he builds the rest as homework.
4. **He types, you guide.** Don't take over the keyboard, even if it's slow.
5. **End every session with "explain it back to me".** Two minutes in his own words, as if in an interview.
6. **Keep homework small, 30–60 minutes.** Review it at the start of the next session.
7. **Mistakes are the lesson.** Let him hit errors, read the stack trace, and search for the answer. Help him if he has been stuck for more than 45 minutes.
8. **Keep it working at all times.** Every session ends with a running app and a commit.

### Session Format (Use Every Day)
| Time | Activity |
|---|---|
| 5 min | Review homework / questions |
| 5–10 min | Concept: why and what (whiteboard or drawing) |
| 20–35 min | Build together (he types) |
| 5 min | He explains it back, and you assign homework |

---

## 2. Python Cheat-Sheet for the Mentor (Mapped to What You Know)

| Python / FastAPI | Equivalent you may know |
|---|---|
| `pip` + `requirements.txt` | `npm` + `package.json` / Maven `pom.xml` |
| `venv` (virtual environment) | Project-local `node_modules` |
| FastAPI | Express.js / Spring Boot / ASP.NET Web API |
| Pydantic models (schemas) | DTOs + validation (Joi / Zod / `@Valid`) |
| SQLAlchemy ORM | Hibernate / TypeORM / Sequelize / Entity Framework |
| Alembic | Flyway / Liquibase / Knex / EF migrations |
| `Depends()` | Dependency injection / middleware per route |
| `uvicorn` / `fastapi dev` | `nodemon` / embedded Tomcat |
| `pytest` | Jest / JUnit |
| `/docs` (Swagger UI, auto-generated) | Swagger / Springdoc |
| `.env` + `pydantic-settings` | `dotenv` / `application.properties` |

---

## 3. Project Ideas (Pick One, Keep It Simple)

| Idea | Why it's good | Complexity |
|---|---|---|
| **TaskFlow – Personal Task Manager** (recommended) | Everyone understands tasks, and it covers auth, CRUD, ownership and filters | Low |
| Personal Expense Tracker | Categories, monthly totals (GROUP BY), charts later | Low–Medium |
| Library Book Borrowing | Many-to-many (users ↔ books), due dates, fines | Medium |
| Event Registration | Capacity limits, registrations, admin role | Medium |

**This plan uses TaskFlow.** Next projects: Expense Tracker, then Library (see the Fresher Training Plan).

### TaskFlow Scope (Version 1 Only, No More)
- A user can register and log in
- A logged-in user can create, view, edit and delete **their own** tasks
- Tasks have a title, description and status (`todo`, `in_progress`, `done`)
- Filter by status, search by title, pagination
- Simple web UI: login page and tasks page

**Out of scope for v1:** teams, file uploads, email, roles, Docker. Tell him: *"Real projects ship small versions first."*

---

## 4. Day 0 – Mentor Prep (Before First Session)

Set up both machines:
- [ ] **Python 3.12+** (`brew install python@3.12` on Mac, python.org on Windows, and tick "Add to PATH")
- [ ] **VS Code** + extensions: Python, Pylance
- [ ] **MySQL 8** + **MySQL Workbench** (or DBeaver)
- [ ] **Git** + a GitHub account for him
- [ ] **Postman** (optional, since Swagger `/docs` is enough to start)
- [ ] Browser tool for diagrams: [dbdiagram.io](https://dbdiagram.io) or [draw.io](https://app.diagrams.net)

---

# Week 1 – Think Before Coding (Requirements & Design)

**Goal:** He learns that real projects start with understanding and design, not code.
**Very little coding this week.**

### Day 1 – How Real Projects Work + Project Kickoff
- **Teach:** The lifecycle: requirements → design → build → test → deploy → maintain. Show him a real Jira/GitHub board if possible.
- **Do together:** Explain TaskFlow scope (Section 3). Write 2 user stories together:
  ```
  As a user, I want to register with my email and password so that I can have my own account.
  As a user, I want to see only my tasks so that my data stays private.
  ```
- **Homework:** Write 5 more user stories covering the rest of the scope.
- **Done when:** He can explain what TaskFlow does in 3 sentences.

### Day 2 – Sketch the Screens (Wireframes)
- **Teach:** Thinking from the user's view helps discover the data and APIs you need.
- **Do together:** Draw on paper or Excalidraw: Register page, Login page, Tasks page (list + filters + add/edit form).
- **Ask him:** "What data does each screen show or send?" He lists fields.
- **Homework:** Clean up the wireframes and list every field per screen.

### Day 3 – ER Diagram, Part 1: Entities & Attributes
- **Teach (whiteboard):**
  - **Entity** = a thing we store (User, Task) → becomes a **table**
  - **Attribute** = a property (email, title) → becomes a **column**
  - **Primary Key (PK)** = unique id for each row
  - Data types: `INT`, `VARCHAR(n)`, `TEXT`, `DATETIME`, `BOOLEAN`
- **Do together:** From the Day 2 field list, identify the entities `users` and `tasks` and their columns.
- **Homework:** Pick sensible data types and lengths for each column.

### Day 4 – ER Diagram, Part 2: Relationships & Foreign Keys
- **Teach:**
  - One-to-many: *one user has many tasks* → `tasks.user_id` is a **Foreign Key (FK)** to `users.id`
  - Mention one-to-one and many-to-many briefly (just awareness)
  - `UNIQUE` (email), `NOT NULL`, default values, `created_at`
- **Do together:** Draw it on dbdiagram.io:
  ```
  Table users {
    id int [pk, increment]
    name varchar(100) [not null]
    email varchar(255) [not null, unique]
    password_hash varchar(255) [not null]
    created_at datetime [default: `now()`]
  }

  Table tasks {
    id int [pk, increment]
    title varchar(200) [not null]
    description varchar(1000)
    status varchar(20) [not null, default: 'todo']
    user_id int [not null, ref: > users.id]
    created_at datetime [default: `now()`]
  }
  ```
- **Homework:** In MySQL Workbench, create a **scratch** database `practice_db`, create these two tables by hand with SQL, insert 2 users and 5 tasks, and write a JOIN to show each task with its owner's name.
- **Note:** Real tables will be created by migrations later. This is SQL practice only.

### Day 5 – API Design (the Contract)
- **Teach:** REST = resources + HTTP methods. Status codes 200, 201, 204, 400, 401, 404, 422. Version the API (`/api/v1`).
- **Do together:** Fill this table (he proposes, you correct):

| Method | Endpoint | Purpose | Auth? | Success |
|---|---|---|---|---|
| GET | `/health` | App is running | No | 200 |
| POST | `/api/v1/auth/register` | Create account | No | 201 |
| POST | `/api/v1/auth/login` | Get token | No | 200 |
| GET | `/api/v1/auth/me` | Current user info | Yes | 200 |
| GET | `/api/v1/tasks?status=&search=&skip=&limit=` | List my tasks | Yes | 200 |
| POST | `/api/v1/tasks` | Create task | Yes | 201 |
| GET | `/api/v1/tasks/{id}` | Get one task | Yes | 200 |
| PATCH | `/api/v1/tasks/{id}` | Update task | Yes | 200 |
| DELETE | `/api/v1/tasks/{id}` | Delete task | Yes | 204 |

- **Homework:** Write a sample JSON request/response for `POST /tasks` and `POST /auth/register`.
- **Week 1 output:** User stories, wireframes, ER diagram, API table. Save them all in a `docs/` folder. These are great to show in interviews.

---

# Week 2 – Project Setup (the Foundation)

**Goal:** A clean, professional project skeleton connected to MySQL.

### Day 6 – Git, GitHub & Virtual Environment
- **Teach:** Why each project needs its own `venv` (isolated packages). Why Git from day 1.
- **Do together:**
  ```bash
  mkdir taskflow && cd taskflow
  git init
  python3 -m venv .venv
  source .venv/bin/activate          # Windows: .venv\Scripts\activate
  pip install "fastapi[standard]"
  pip freeze > requirements.txt
  ```
  Create `.gitignore`:
  ```
  .venv/
  __pycache__/
  .env
  test.db
  ```
  Create `app/__init__.py` (empty) and `app/main.py`:
  ```python
  from fastapi import FastAPI

  app = FastAPI(title="TaskFlow API")


  @app.get("/health")
  def health():
      return {"status": "ok"}
  ```
  Run: `fastapi dev app/main.py` → open http://localhost:8000/docs
- **Homework:** Create a GitHub repo, push the code, and write a 5-line README (what it is, how to run).
- **Done when:** `/docs` opens and `/health` returns `{"status": "ok"}`.

### Day 7 – Folder Structure & Routers
- **Teach:** Why not put everything in one file? Separation of concerns, like controllers/services/repositories.
- **Target structure** (build it gradually; don't create everything today):
  ```
  taskflow/
  ├── app/
  │   ├── __init__.py
  │   ├── main.py            # creates app, adds routers
  │   ├── config.py          # settings from .env
  │   ├── database.py        # DB connection + session
  │   ├── models.py          # SQLAlchemy tables
  │   ├── schemas.py         # Pydantic request/response
  │   ├── security.py        # hashing + JWT
  │   ├── dependencies.py    # get_current_user
  │   ├── routers/
  │   │   ├── __init__.py
  │   │   ├── auth.py
  │   │   └── tasks.py
  │   └── services/
  │       ├── __init__.py
  │       └── task_service.py
  ├── alembic/               # migrations (Day 10)
  ├── frontend/              # HTML/JS (Week 5)
  ├── tests/                 # pytest (Week 6)
  ├── docs/                  # Week 1 design files
  ├── .env                   # secrets (never commit)
  ├── .env.example           # template (commit this)
  ├── .gitignore
  ├── requirements.txt
  └── README.md
  ```
- **Do together:** Create `app/routers/tasks.py` with one dummy endpoint and include it in `main.py`:
  ```python
  # app/routers/tasks.py
  from fastapi import APIRouter

  router = APIRouter(prefix="/api/v1/tasks", tags=["tasks"])


  @router.get("")
  def list_tasks():
      return []
  ```
  ```python
  # app/main.py (add)
  from app.routers import tasks

  app.include_router(tasks.router)
  ```
- **Teach Git workflow:** Create a branch `feature/project-structure`, commit, push, **open a Pull Request**, and you review and merge it. **Use this flow for every feature from now on.**
- **Homework:** Read the FastAPI docs sections "First Steps" and "Path Parameters".

### Day 8 – Configuration & Secrets (.env)
- **Teach:** Never hardcode passwords or secrets in code, and never commit `.env`.
- **Do together:**
  ```bash
  pip install sqlalchemy pymysql cryptography alembic pydantic-settings
  pip freeze > requirements.txt
  ```
  Create the MySQL database and user (in Workbench):
  ```sql
  CREATE DATABASE taskflow;
  CREATE USER 'taskflow_user'@'localhost' IDENTIFIED BY 'ChangeThisPassword1';
  GRANT ALL PRIVILEGES ON taskflow.* TO 'taskflow_user'@'localhost';
  ```
  `.env` (not committed):
  ```
  DATABASE_URL=mysql+pymysql://taskflow_user:ChangeThisPassword1@localhost:3306/taskflow
  JWT_SECRET=replace-with-output-of-openssl-rand-hex-32
  ```
  `.env.example` (committed, same keys with fake values).
  ```python
  # app/config.py
  from pydantic_settings import BaseSettings, SettingsConfigDict


  class Settings(BaseSettings):
      database_url: str
      jwt_secret: str
      jwt_algorithm: str = "HS256"
      access_token_expire_minutes: int = 60

      model_config = SettingsConfigDict(env_file=".env")


  settings = Settings()
  ```
- **Tip:** Avoid `%` and `@` in the DB password (they break the URL / Alembic config).
- **Homework:** Generate a real secret with `openssl rand -hex 32` and put it in `.env`.

### Day 9 – Connect to the Database (SQLAlchemy)
- **Teach:** What an ORM is (you know Hibernate/TypeORM, so compare). What a DB session is (one unit of work per request).
- **Do together:**
  ```python
  # app/database.py
  from sqlalchemy import create_engine
  from sqlalchemy.orm import DeclarativeBase, sessionmaker

  from app.config import settings

  engine = create_engine(settings.database_url, pool_pre_ping=True)
  SessionLocal = sessionmaker(bind=engine, autoflush=False)


  class Base(DeclarativeBase):
      pass


  def get_db():
      db = SessionLocal()
      try:
          yield db
      finally:
          db.close()
  ```
- **Explain `yield`:** Code before `yield` runs before the request, and code after it runs when the request ends. It works like try/finally middleware.
- **Homework:** Read the SQLAlchemy "ORM Quick Start" page (just skim).

### Day 10 – Models + Alembic Migrations
- **Teach:** Models = tables in code. Migrations = version control for the DB schema (like Flyway). Never change production tables by hand.
- **Do together:**
  ```python
  # app/models.py
  from datetime import datetime

  from sqlalchemy import DateTime, ForeignKey, String, func
  from sqlalchemy.orm import Mapped, mapped_column, relationship

  from app.database import Base


  class User(Base):
      __tablename__ = "users"

      id: Mapped[int] = mapped_column(primary_key=True)
      name: Mapped[str] = mapped_column(String(100))
      email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
      password_hash: Mapped[str] = mapped_column(String(255))
      created_at: Mapped[datetime] = mapped_column(DateTime, server_default=func.now())

      tasks: Mapped[list["Task"]] = relationship(back_populates="owner")


  class Task(Base):
      __tablename__ = "tasks"

      id: Mapped[int] = mapped_column(primary_key=True)
      title: Mapped[str] = mapped_column(String(200))
      description: Mapped[str | None] = mapped_column(String(1000))
      status: Mapped[str] = mapped_column(String(20), default="todo")
      user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), index=True)
      created_at: Mapped[datetime] = mapped_column(DateTime, server_default=func.now())

      owner: Mapped["User"] = relationship(back_populates="tasks")
  ```
  Set up Alembic:
  ```bash
  alembic init alembic
  ```
  Edit `alembic/env.py`. Find `target_metadata = None` and replace it with:
  ```python
  from app.config import settings
  from app.database import Base
  from app import models  # noqa: F401  (loads models so Alembic can see them)

  config.set_main_option("sqlalchemy.url", settings.database_url)
  target_metadata = Base.metadata
  ```
  Then:
  ```bash
  alembic revision --autogenerate -m "create users and tasks"
  alembic upgrade head
  ```
- **Show him:** The generated migration file, and the tables now visible in Workbench. **Compare with his Day 4 ER diagram.**
- **Homework:** Run `alembic downgrade -1` then `alembic upgrade head` and explain what happened.
- **Week 2 output:** Running app, connected DB, tables created by migration, all on GitHub via PRs.

---

# Week 3 – Build the Features (CRUD)

**Goal:** Working task APIs. **No login yet.** We use a temporary fixed user so he learns one thing at a time.

### Day 11 – Schemas (Request/Response Validation)
- **Teach:** Models = DB shape. Schemas = API shape. Why separate them? We never return `password_hash` to the client.
- **Do together:**
  ```python
  # app/schemas.py
  from datetime import datetime
  from typing import Literal

  from pydantic import BaseModel, ConfigDict, EmailStr, Field

  TaskStatus = Literal["todo", "in_progress", "done"]


  class TaskCreate(BaseModel):
      title: str = Field(min_length=1, max_length=200)
      description: str | None = Field(default=None, max_length=1000)


  class TaskUpdate(BaseModel):
      title: str | None = Field(default=None, min_length=1, max_length=200)
      description: str | None = Field(default=None, max_length=1000)
      status: TaskStatus | None = None


  class TaskOut(BaseModel):
      model_config = ConfigDict(from_attributes=True)

      id: int
      title: str
      description: str | None
      status: str
      created_at: datetime
  ```
- **Homework:** Add `UserCreate` (name, email, password min 8 / max 64), `UserOut` (id, name, email, no password), and `Token` (`access_token`, `token_type = "bearer"`).

### Day 12 – Create & List Tasks
- **Setup:** Insert one test user manually in Workbench:
  ```sql
  INSERT INTO users (name, email, password_hash) VALUES ('Test User', 'test@example.com', 'temp');
  ```
- **Do together:**
  ```python
  # app/routers/tasks.py
  from fastapi import APIRouter, Depends, HTTPException
  from sqlalchemy import select
  from sqlalchemy.orm import Session

  from app import models, schemas
  from app.database import get_db

  router = APIRouter(prefix="/api/v1/tasks", tags=["tasks"])

  CURRENT_USER_ID = 1  # TEMP: replaced by real login in Week 4


  @router.post("", response_model=schemas.TaskOut, status_code=201)
  def create_task(data: schemas.TaskCreate, db: Session = Depends(get_db)):
      task = models.Task(**data.model_dump(), user_id=CURRENT_USER_ID)
      db.add(task)
      db.commit()
      db.refresh(task)
      return task


  @router.get("", response_model=list[schemas.TaskOut])
  def list_tasks(db: Session = Depends(get_db)):
      stmt = select(models.Task).where(models.Task.user_id == CURRENT_USER_ID)
      return db.scalars(stmt).all()
  ```
- **Test in `/docs`:** Create tasks, list them, and send an empty title to see the **422** validation error.
- **Homework:** Create 10 tasks via Swagger and check them in Workbench.

### Day 13 – Get, Update, Delete
- **Do together:** He writes these with your guidance:
  ```python
  @router.get("/{task_id}", response_model=schemas.TaskOut)
  def get_task(task_id: int, db: Session = Depends(get_db)):
      task = db.get(models.Task, task_id)
      if task is None:
          raise HTTPException(status_code=404, detail="Task not found")
      return task


  @router.patch("/{task_id}", response_model=schemas.TaskOut)
  def update_task(task_id: int, data: schemas.TaskUpdate, db: Session = Depends(get_db)):
      task = db.get(models.Task, task_id)
      if task is None:
          raise HTTPException(status_code=404, detail="Task not found")
      for field, value in data.model_dump(exclude_unset=True).items():
          setattr(task, field, value)
      db.commit()
      db.refresh(task)
      return task


  @router.delete("/{task_id}", status_code=204)
  def delete_task(task_id: int, db: Session = Depends(get_db)):
      task = db.get(models.Task, task_id)
      if task is None:
          raise HTTPException(status_code=404, detail="Task not found")
      db.delete(task)
      db.commit()
  ```
- **Teach:** PUT vs PATCH, and why 404 and 204.
- **Note for mentor:** This code has a **security bug on purpose** (no ownership check). He finds it on Day 20. Don't point it out now.
- **Homework:** Test all 5 endpoints in Postman and save them as a Postman collection in `docs/`.

### Day 14 – Refactor: Service Layer
- **Teach:** Routers handle HTTP. Services handle business logic and DB. This makes code testable and reusable. **Refactoring is a normal part of real work.**
- **Do together:** Move DB logic into `app/services/task_service.py`:
  ```python
  # app/services/task_service.py
  from sqlalchemy import select
  from sqlalchemy.orm import Session

  from app import models, schemas


  def create_task(db: Session, user_id: int, data: schemas.TaskCreate) -> models.Task:
      task = models.Task(**data.model_dump(), user_id=user_id)
      db.add(task)
      db.commit()
      db.refresh(task)
      return task


  def list_tasks(db: Session, user_id: int) -> list[models.Task]:
      stmt = select(models.Task).where(models.Task.user_id == user_id)
      return list(db.scalars(stmt).all())
  ```
  The router now just calls `task_service.create_task(db, CURRENT_USER_ID, data)`.
- **Homework:** Move get/update/delete into the service too. Everything must still work.

### Day 15 – Filters, Search & Pagination
- **Teach:** Never return 10,000 rows at once. Query parameters are for filtering.
- **Do together:**
  ```python
  # app/services/task_service.py
  def list_tasks(db, user_id, status=None, search=None, skip=0, limit=10):
      stmt = select(models.Task).where(models.Task.user_id == user_id)
      if status:
          stmt = stmt.where(models.Task.status == status)
      if search:
          stmt = stmt.where(models.Task.title.contains(search))
      stmt = stmt.order_by(models.Task.id.desc()).offset(skip).limit(limit)
      return list(db.scalars(stmt).all())
  ```
  ```python
  # app/routers/tasks.py
  from fastapi import Query

  @router.get("", response_model=list[schemas.TaskOut])
  def list_tasks(
      status: schemas.TaskStatus | None = None,
      search: str | None = None,
      skip: int = Query(0, ge=0),
      limit: int = Query(10, ge=1, le=100),
      db: Session = Depends(get_db),
  ):
      return task_service.list_tasks(db, CURRENT_USER_ID, status, search, skip, limit)
  ```
- **Teach:** The ORM builds parameterized SQL, which prevents **SQL injection**. Never build SQL with f-strings.
- **Homework:** Try `?status=done&search=report&limit=5` and explain the SQL it generates (set `echo=True` in `create_engine` temporarily to see it).
- **Week 3 output:** Full CRUD with validation, filtering and pagination, in a clean structure.

---

# Week 4 – Authentication (Step by Step)

**Goal:** Real login. Split into small pieces because this is the hardest topic for freshers.

### Day 16 – Concepts Only (No Code)
- **Teach on a whiteboard:**
  - **Authentication** (who are you?) vs **Authorization** (what can you access?)
  - **Hashing vs encryption:** passwords are *hashed* (one-way) with **bcrypt**, never stored as plain text
  - **JWT:** a signed token the server gives after login. The client sends it in every request: `Authorization: Bearer <token>`
  - Draw the flow:

```mermaid
sequenceDiagram
    participant U as Browser
    participant A as FastAPI
    participant D as MySQL
    U->>A: POST /auth/register (name, email, password)
    A->>D: save user with password_hash
    U->>A: POST /auth/login (email, password)
    A->>D: find user, verify hash
    A-->>U: JWT access_token
    U->>A: GET /tasks (Authorization: Bearer token)
    A->>A: verify token, get user_id
    A->>D: select tasks where user_id = ?
    A-->>U: user's tasks
```

- **Homework:** Paste a sample JWT into [jwt.io](https://jwt.io) and see that the payload is **readable, not secret**. Ask him: *"So what must never go inside a JWT?"*

### Day 17 – Register (Password Hashing)
- **Do together:**
  ```bash
  pip install bcrypt pyjwt
  pip freeze > requirements.txt
  ```
  ```python
  # app/security.py
  from datetime import datetime, timedelta, timezone

  import bcrypt
  import jwt

  from app.config import settings


  def hash_password(password: str) -> str:
      return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()


  def verify_password(password: str, password_hash: str) -> bool:
      return bcrypt.checkpw(password.encode(), password_hash.encode())


  def create_access_token(user_id: int) -> str:
      expire = datetime.now(timezone.utc) + timedelta(minutes=settings.access_token_expire_minutes)
      payload = {"sub": str(user_id), "exp": expire}
      return jwt.encode(payload, settings.jwt_secret, algorithm=settings.jwt_algorithm)
  ```
  ```python
  # app/routers/auth.py
  from fastapi import APIRouter, Depends, HTTPException
  from sqlalchemy import select
  from sqlalchemy.orm import Session

  from app import models, schemas
  from app.database import get_db
  from app.security import hash_password

  router = APIRouter(prefix="/api/v1/auth", tags=["auth"])


  @router.post("/register", response_model=schemas.UserOut, status_code=201)
  def register(data: schemas.UserCreate, db: Session = Depends(get_db)):
      existing = db.scalar(select(models.User).where(models.User.email == data.email))
      if existing:
          raise HTTPException(status_code=400, detail="Email already registered")
      user = models.User(name=data.name, email=data.email, password_hash=hash_password(data.password))
      db.add(user)
      db.commit()
      db.refresh(user)
      return user
  ```
  Add `app.include_router(auth.router)` in `main.py`.
- **Show him:** The hashed password in Workbench. Register the same password twice and see that the hashes differ (salt).
- **Homework:** Try registering the same email twice and check that you get 400, not 500.

### Day 18 – Login (Issue JWT)
- **Do together:**
  ```python
  # app/routers/auth.py (add)
  from fastapi.security import OAuth2PasswordRequestForm

  from app.security import create_access_token, verify_password


  @router.post("/login", response_model=schemas.Token)
  def login(form: OAuth2PasswordRequestForm = Depends(), db: Session = Depends(get_db)):
      user = db.scalar(select(models.User).where(models.User.email == form.username))
      if user is None or not verify_password(form.password, user.password_hash):
          raise HTTPException(status_code=401, detail="Incorrect email or password")
      return {"access_token": create_access_token(user.id), "token_type": "bearer"}
  ```
- **Explain:** The OAuth2 form uses the field name `username`, so we put the email there. This makes the **Authorize** button in `/docs` work.
- **Teach:** Why the same error for wrong email and wrong password? So attackers can't discover which emails exist.
- **Homework:** Log in via `/docs`, copy the token, and decode it on jwt.io.

### Day 19 – Protect Routes (Current User)
- **Do together:**
  ```python
  # app/dependencies.py
  import jwt
  from fastapi import Depends, HTTPException
  from fastapi.security import OAuth2PasswordBearer
  from sqlalchemy.orm import Session

  from app import models
  from app.config import settings
  from app.database import get_db

  oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")


  def get_current_user(token: str = Depends(oauth2_scheme), db: Session = Depends(get_db)) -> models.User:
      credentials_error = HTTPException(
          status_code=401, detail="Invalid or expired token", headers={"WWW-Authenticate": "Bearer"}
      )
      try:
          payload = jwt.decode(token, settings.jwt_secret, algorithms=[settings.jwt_algorithm])
          user_id = int(payload["sub"])
      except (jwt.InvalidTokenError, KeyError, ValueError):
          raise credentials_error
      user = db.get(models.User, user_id)
      if user is None:
          raise credentials_error
      return user
  ```
  In `tasks.py`: **delete `CURRENT_USER_ID`** and add `current_user: models.User = Depends(get_current_user)` to every endpoint, using `current_user.id`.
  Add `GET /api/v1/auth/me` returning `current_user` (response_model `UserOut`).
- **Test:** Call `/tasks` without a token (401), then click **Authorize** in `/docs`, log in, and call again (200).
- **Homework:** Delete the temporary SQL test user and its tasks, then register real users via the API.

### Day 20 – Bug Hunt: Authorization
- **Real-world scenario:** Tell him: *"QA reported that a user can see someone else's task."*
- **Let him find it:** Register User A and User B. A creates a task (id 5). Log in as B and call `GET /tasks/5`. **B can see it.**
- **Fix together:** In the service, fetch a task **only if it belongs to the user**:
  ```python
  def get_owned_task(db: Session, user_id: int, task_id: int) -> models.Task | None:
      stmt = select(models.Task).where(models.Task.id == task_id, models.Task.user_id == user_id)
      return db.scalar(stmt)
  ```
  Use it in get, update and delete. Return **404** (not 403) so others can't even learn the task exists.
- **Teach:** This is **IDOR / Broken Access Control**, the #1 risk in the OWASP Top 10. It's a great interview story.
- **Week 4 output:** Secure register/login with JWT; users can only access their own data.

---

# Week 5 – Simple UI (Show It Working)

**Goal:** A basic web page using his APIs. Keep it plain HTML + JavaScript (no build tools) so the focus stays on API integration. React can come in the next project.

### Day 21 – CORS + Login Page
- **Teach:** CORS: why the browser blocks calls from a different origin (port 5500 → 8000).
- **Do together:**
  ```python
  # app/main.py (add)
  from fastapi.middleware.cors import CORSMiddleware

  app.add_middleware(
      CORSMiddleware,
      allow_origins=["http://localhost:5500", "http://127.0.0.1:5500"],
      allow_methods=["*"],
      allow_headers=["*"],
  )
  ```
  `frontend/login.html` with an email/password form, plus `frontend/app.js`:
  ```javascript
  const API = "http://localhost:8000/api/v1";

  async function login(event) {
    event.preventDefault();
    const body = new URLSearchParams({
      username: document.getElementById("email").value,
      password: document.getElementById("password").value,
    });
    const res = await fetch(`${API}/auth/login`, { method: "POST", body });
    if (!res.ok) {
      document.getElementById("error").textContent = "Invalid email or password";
      return;
    }
    const data = await res.json();
    localStorage.setItem("token", data.access_token);
    window.location.href = "tasks.html";
  }
  ```
  Serve the UI: `cd frontend && python3 -m http.server 5500`
- **Homework:** Build `register.html` the same way (JSON body with `Content-Type: application/json`).

### Day 22 – Tasks List Page
- **Do together:** `tasks.html` loads tasks with the token:
  ```javascript
  async function api(path, options = {}) {
    const res = await fetch(`${API}${path}`, {
      ...options,
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${localStorage.getItem("token")}`,
        ...options.headers,
      },
    });
    if (res.status === 401) {
      localStorage.removeItem("token");
      window.location.href = "login.html";
      return null;
    }
    return res;
  }

  async function loadTasks() {
    const res = await api("/tasks");
    if (!res) return;
    const tasks = await res.json();
    const list = document.getElementById("task-list");
    list.replaceChildren();
    for (const task of tasks) {
      const li = document.createElement("li");
      li.textContent = `${task.title} [${task.status}]`;  // textContent, not innerHTML, prevents XSS
      list.appendChild(li);
    }
  }
  ```
- **Teach:** Why `textContent` instead of `innerHTML` (XSS). Mention that `localStorage` is fine for learning, but real apps often use httpOnly cookies.
- **Homework:** Add a Logout button that clears the token.

### Day 23 – Create, Edit, Delete from UI
- **Do together:** Add a form to create a task (`POST`), a status dropdown per task (`PATCH`), and a delete button (`DELETE`), then reload the list.
- **Homework:** Finish whatever is left from the session.

### Day 24 – Filters, Pagination & Error Handling in UI
- **Do together:** Status filter dropdown, search box, Next/Previous buttons (`skip`/`limit`). Show loading text and friendly error messages.
- **Homework:** Test the full flow as a new user: register → login → create → edit → filter → delete → logout.
- **Week 5 output:** A working full-stack app he can demo.

---

# Week 6 – Quality, Change Requests & Demo

**Goal:** Professional finishing touches like tests, docs, a requirement change, and presenting the project.

### Day 25 – Testing Basics (pytest)
- **Teach:** Why tests matter: confidence to change code. Use a separate test database so real data is never touched.
- **Do together:**
  ```bash
  pip install pytest
  pip freeze > requirements.txt
  ```
  Create an empty `tests/__init__.py`, then:
  ```python
  # tests/conftest.py
  import pytest
  from fastapi.testclient import TestClient
  from sqlalchemy import create_engine
  from sqlalchemy.orm import sessionmaker

  from app.database import Base, get_db
  from app.main import app

  engine = create_engine("sqlite:///./test.db", connect_args={"check_same_thread": False})
  TestingSession = sessionmaker(bind=engine)


  @pytest.fixture
  def client():
      Base.metadata.create_all(engine)

      def override_get_db():
          db = TestingSession()
          try:
              yield db
          finally:
              db.close()

      app.dependency_overrides[get_db] = override_get_db
      yield TestClient(app)
      app.dependency_overrides.clear()
      Base.metadata.drop_all(engine)
  ```
  ```python
  # tests/test_auth.py
  def test_register_and_login(client):
      res = client.post(
          "/api/v1/auth/register",
          json={"name": "Asha", "email": "asha@example.com", "password": "secret123"},
      )
      assert res.status_code == 201

      res = client.post("/api/v1/auth/login", data={"username": "asha@example.com", "password": "secret123"})
      assert res.status_code == 200
      assert "access_token" in res.json()


  def test_duplicate_email_returns_400(client):
      body = {"name": "Asha", "email": "asha@example.com", "password": "secret123"}
      client.post("/api/v1/auth/register", json=body)
      res = client.post("/api/v1/auth/register", json=body)
      assert res.status_code == 400
  ```
  Run: `pytest -v`
- **Homework:** Add a test for wrong password (401).

### Day 26 – Test the Tasks API
- **Do together:** Write a helper that registers, logs in and returns auth headers. Test create, list, update and delete.
- **Most important test:** User B cannot access User A's task (404). This protects the Day 20 fix forever.
- **Homework:** Reach at least 10 passing tests.

### Day 27 – Change Request (Real-World Twist)
- **Scenario:** *"Product owner: users want **priority** (low/medium/high) and a **due date** on tasks."*
- **He does, with guidance:**
  1. Update the ER diagram in `docs/`
  2. Add columns to the model (`priority` with `server_default="medium"`, `due_date` nullable)
  3. `alembic revision --autogenerate -m "add priority and due date"` → **read the generated file** → `alembic upgrade head`
  4. Check that **existing tasks still work** (that's why a default is needed)
  5. Update schemas, the filter (by priority), the UI and the tests
- **Teach:** Requirements always change, and migrations let you change the DB safely without losing data.
- **Homework:** Finish and raise a PR.

### Day 28 – README & Code Cleanup
- **Do together:** Review the whole codebase: unused code, naming, consistent errors.
- **README must have:** Project description, features, tech stack, screenshots, ER diagram, setup steps (venv, `.env`, `alembic upgrade head`, run), API endpoints table, how to run tests.
- **Homework:** Record a 2–3 min screen demo video (optional but great for LinkedIn).

### Day 29 – Demo Day
- **He presents to you as if you were the interviewer (15 min):**
  1. What problem it solves (1 min)
  2. Live demo (5 min)
  3. ER diagram and API design (3 min)
  4. How auth works (3 min)
  5. One bug he found and fixed (Day 20 IDOR) and one change request (Day 27) (3 min)
- **You ask interview questions:** "Why FastAPI?", "Why hash, not encrypt?", "What's in a JWT?", "Why a service layer?", "What is a migration?", "How do you prevent SQL injection?"

### Day 30 – Retrospective & Next Steps
- **Discuss:** What was hardest? What would he do differently? What did he learn?
- **Write resume bullets together**, for example:
  - *Built TaskFlow, a full-stack task manager using FastAPI, MySQL, SQLAlchemy and JavaScript, with JWT authentication and bcrypt password hashing.*
  - *Designed the DB schema (ER diagram) and managed schema changes with Alembic migrations; fixed a broken access control (IDOR) vulnerability and covered it with automated tests.*
- **Pin the repo on GitHub and post on LinkedIn.**

---

## After TaskFlow: What Comes Next

Repeat the same process with **less hand-holding** each time:

| Project | New concepts added | Your role |
|---|---|---|
| **2. Expense Tracker** | Categories (more tables), monthly reports with GROUP BY, charts, **React** frontend, roles (admin/user) | Give requirements only; he designs, and you review the design and PRs |
| **3. Library / Event System** | Many-to-many, transactions, Docker, deployment (Render/Railway), GitHub Actions CI | Act as Product Owner + reviewer; introduce a "production bug" drill |

See **Fresher-Training-Plan.md** for the detailed version of these projects and the self-learning topics.

---

## Progress Tracker

| Week | Topic | Status | Notes |
|---|---|---|---|
| 1 | Requirements, wireframes, ER diagram, API design | ☐ | |
| 2 | Git, venv, structure, config, DB, models, migrations | ☐ | |
| 3 | Schemas, CRUD, service layer, filters, pagination | ☐ | |
| 4 | Auth concepts, register, login, protected routes, IDOR fix | ☐ | |
| 5 | CORS, login UI, tasks UI, CRUD UI, filters UI | ☐ | |
| 6 | Tests, change request, README, demo, retro | ☐ | |

**If he falls behind:** Slow down rather than skipping topics. Repeat a day, or split it into two. Understanding matters more than speed.
