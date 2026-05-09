# Common Patterns in Python Backend

## Auth Strategy

Just like Node.js, Python supports all major auth strategies.

### JWT Strategy (Stateless)
Ideal for FastAPI or Django APIs serving mobile apps or SPAs.

**Packages:** `python-jose`, `PyJWT`

*FastAPI Example (`python-jose`):*
```python
from datetime import datetime, timedelta
from jose import JWTError, jwt

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 15

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def verify_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload.get("sub")
    except JWTError:
        return None
```

### Session Strategy (Stateful)
In Django, session management is built-in and secure by default. In FastAPI, you need external libraries like `fastapi-sessions` or custom Redis integration.

**Packages:** `redis`, `fastapi-sessions` (FastAPI), Built-in (Django)

*Django Session Magic:*
```python
# Django handles this automatically when you use login()
from django.contrib.auth import authenticate, login

def login_view(request):
    user = authenticate(username=username, password=password)
    if user is not None:
        login(request, user) # Creates session in DB, sets cookie
```

## Database ORMs

Python has the most powerful ORMs in the backend ecosystem.

### SQL Database Packages
| ORM | Performance | Flexibility | Type Safety | Learning Curve | Best For |
| --- | --- | --- | --- | --- | --- |
| **SQLAlchemy** | High | Very High | Excellent | Steep | Complex queries, Enterprise FastAPI apps |
| **SQLModel** | High | High | Excellent | Medium | FastAPI projects (combines Pydantic & SQLAlchemy) |
| **Django ORM** | Medium | High | Moderate | Easy | Rapid development, built-in Django apps |
| **Tortoise ORM** | High | Medium | Good | Medium | Async-first projects, similar to Django ORM |

*Recommendation*: For FastAPI, use **SQLModel** or **SQLAlchemy 2.0**. For Django, stick to **Django ORM**.

### NoSQL Database Packages
| Package | Database | Sync/Async |
| --- | --- | --- |
| `PyMongo` | MongoDB | Sync |
| `Motor` | MongoDB | Async (Best for FastAPI) |

*FastAPI Motor Example:*
```python
import motor.motor_asyncio

client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.my_database

async def create_user(user_data: dict):
    result = await db.users.insert_one(user_data)
    return result.inserted_id
```

## Background Jobs and Queues

Python is excellent at background processing.
| Package | Usage | Complexity | Best For |
| --- | --- | --- | --- |
| **Celery** | Redis/RabbitMQ queue | High | Enterprise, complex workflows, cron jobs |
| **RQ (Redis Queue)**| Redis queue | Low | Simple background tasks |
| **APScheduler** | In-memory/DB | Low | Simple scheduled jobs within the same process |

*Recommendation*: Start with **RQ** for simple queues. Move to **Celery** when workflows become complex.

## Testing Stack
Python's testing ecosystem is incredibly robust.

| Package | Usage |
| --- | --- |
| `pytest` | Test runner (Absolute standard, avoids `unittest` boilerplate) |
| `pytest-asyncio` | For testing async FastAPI endpoints |
| `httpx` | Async HTTP client for API testing |
| `factory_boy` | Generating fake test data and models |

*FastAPI Test Example:*
```python
import pytest
from httpx import AsyncClient
from main import app

@pytest.mark.asyncio
async def test_read_main():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```
