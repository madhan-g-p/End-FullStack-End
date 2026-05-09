# FastAPI Patterns

## Overview
FastAPI is a modern, fast (high-performance), web framework for building APIs with Python based on standard Python type hints.

## Recommended Project Structure
```text
src/
│
├── api/                  # Routers and endpoints
│   ├── v1/
│   └── dependencies.py   # FastAPI dependency injections
├── core/                 # App configs, security, constants
│   ├── config.py
│   └── security.py
├── models/               # Database models (SQLAlchemy/SQLModel)
├── schemas/              # Pydantic models (DTOs / Validation)
├── crud/                 # Database operations (Repositories)
├── services/             # Business logic
├── db/                   # Database connection and session
├── worker/               # Background jobs (Celery)
└── main.py               # Application entry point
```

| Layer | Responsibility |
| --- | --- |
| api | Define endpoints, HTTP methods, route grouping |
| schemas | Pydantic models for request/response validation |
| crud | Reusable database queries (Create, Read, Update, Delete) |
| services | Core business logic, keeping routers clean |
| models | ORM definitions mapping to database tables |
| core | JWT setup, CORS, environment variables, settings |

## Essential Packages
| Package | Purpose |
| --- | --- |
| `fastapi` | Core framework |
| `uvicorn` | ASGI server |
| `pydantic` | Data validation and settings management |
| `pydantic-settings` | Environment variable management |
| `sqlalchemy` or `sqlmodel` | Database ORM |
| `alembic` | Database migrations |
| `python-jose` | JWT handling |
| `passlib[bcrypt]` | Password hashing |
| `slowapi` | Rate limiting |

## Validation (Pydantic)
FastAPI relies entirely on Pydantic. It automatically generates JSON Schemas and Swagger UI documentation.

Example Schema:
```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    name: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=8)
```

Usage in Router:
```python
from fastapi import APIRouter

router = APIRouter()

@router.post("/users/")
async def create_user(user_in: UserCreate):
    # FastAPI automatically validates user_in. 
    # If invalid, it returns a 422 Unprocessable Entity automatically!
    return {"name": user_in.name, "email": user_in.email}
```

## Error Handling
Never throw raw 500 exceptions to users. Use custom exception handlers.

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse

app = FastAPI()

class AppError(Exception):
    def __init__(self, message: str, status_code: int = 500):
        self.message = message
        self.status_code = status_code

@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError):
    return JSONResponse(
        status_code=exc.status_code,
        content={"success": False, "error": exc.message},
    )

# Triggering it:
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 0:
        raise AppError(message="Item zero is not allowed", status_code=400)
    return {"item_id": item_id}
```

## Security Best Practices
Middleware and Security tools are vital.

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

# 1. CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 2. Compression
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

## Dependency Injection (DI)
FastAPI has a powerful DI system, useful for Auth, DB sessions, etc.

```python
from fastapi import Depends, HTTPException
from sqlalchemy.orm import Session

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

def get_current_user(token: str = Depends(oauth2_scheme), db: Session = Depends(get_db)):
    user = verify_token(token, db)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

@router.get("/me")
def read_current_user(current_user: User = Depends(get_current_user)):
    return current_user
```

## Setting up FastAPI Server from Scratch (with `uv`)

```bash
uv init
uv add fastapi "uvicorn[standard]" pydantic pydantic-settings
uv run uvicorn main:app --reload
```

## Sample Server Init File
```python
# src/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from api.v1.api import api_router
from core.config import settings

def create_app() -> FastAPI:
    app = FastAPI(
        title=settings.PROJECT_NAME,
        openapi_url=f"{settings.API_V1_STR}/openapi.json"
    )

    # Security Middlewares
    app.add_middleware(
        CORSMiddleware,
        allow_origins=[str(origin) for origin in settings.BACKEND_CORS_ORIGINS],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

    # Routes
    app.include_router(api_router, prefix=settings.API_V1_STR)

    return app

app = create_app()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```
