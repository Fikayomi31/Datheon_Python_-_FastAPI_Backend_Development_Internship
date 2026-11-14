# Python + FastAPI Backend Development Internship

Three backend projects built during the **Python + FastAPI Backend Development Internship at Datheon**. Each one adds a layer on top of the last: a CRUD API, an authentication system, and a full multi-user API with database migrations.

| Project | What it is | Key skills |
|---|---|---|
| [BookStore_API](https://github.com/Fikayomi31/BookStore_API) | CRUD REST API for managing books | Pydantic validation, async SQLAlchemy, search and sorting, HTTP error handling |
| [User_auth](https://github.com/Fikayomi31/User_auth) | Registration, login and role-based access | Password hashing, JWT, dependency injection, RBAC |
| [Notes](https://github.com/Fikayomi31/Notes) | Multi-user notes API with per-user data | Authentication, ownership checks, Alembic migrations |

**Common stack:** Python, FastAPI, Pydantic, async SQLAlchemy, SQLite (aiosqlite), Uvicorn.


---

## 1. BookStore_API

A RESTful API for managing a collection of books.

**What it does**
- Create, read, update and delete books under `/api/books`.
- Validates input with Pydantic: title and author length, positive price, publication year range, and a fixed set of genres (a Python enum).
- Filters books by title or author with case-insensitive partial matching.
- Sorts by title, author, price or publication year, ascending or descending.
- Blocks duplicate books (same title and author, case-insensitive) with `409 Conflict`.
- Returns clear status codes: `201`, `204`, `404`, `409`, `422` and `500`, rolling back the transaction when a database write fails.

**What it taught:** designing request and response schemas, using async database sessions, and handling errors cleanly.

## 2. User_auth

An authentication and authorization API.

**What it does**
- `POST /api/auth/register` creates a user with a hashed password (bcrypt) and the default `USER` role.
- `POST /api/auth/login` verifies credentials and returns a signed JWT access token.
- `GET /api/auth/me` returns the logged-in user, resolved from the token.
- A reusable `require_role(...)` dependency protects routes by role, with example `/api/user-area` and `/api/admin-area` endpoints (`403 Forbidden` on a role mismatch).

**Worth knowing:** the login endpoint takes JSON (`email` and `password`), while Swagger UI's Authorize button sends form data (`username` and `password`), so Authorize returns `422`. Postman or a pasted bearer token works fine. Switching to `OAuth2PasswordRequestForm` would fix it.

**What it taught:** the full login flow, JWT handling, FastAPI dependency injection, and role-based access control.

## 3. Notes

A multi-user notes API that builds on the authentication work.

**What it does**
- Same register, login and `/me` flow as User_auth, with JWTs that expire after 30 minutes.
- Full CRUD for notes under `/api/notes`, always scoped to the logged-in user. A note owned by someone else returns `404`, so its existence is not revealed.
- Note titles must be unique per user (case-insensitive), otherwise `409 Conflict`.
- `PATCH` updates the title, the content, or both.
- `User` has a one-to-many relationship with `Note`, and deleting a user cascades to their notes.
- **Alembic** manages the database schema instead of `create_all()`.

**What it taught:** data ownership, relational modelling, and evolving a schema safely with migrations.

---

## Running a project

Each project is its own repository with its own README. In general:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
uvicorn app.main:app --reload    # then open http://127.0.0.1:8000/docs
```

Notes also needs a `SECRET_KEY` in `.env` and `alembic upgrade head` before the first run. User_auth needs a JWT secret in `.env`.


## Author
Fikayo Ogundijo