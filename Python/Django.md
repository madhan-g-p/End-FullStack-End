# Django Patterns

## Overview
Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. It is known as the "batteries-included" framework because it comes with an ORM, Authentication, Admin Panel, and Templating engine out of the box.

## Recommended Project Structure
Unlike FastAPI, Django dictates its own structure. However, modern API-first Django applications (often using Django Rest Framework or Django Ninja) adapt it slightly.

```text
myproject/                 # Root wrapper
│
├── core/                  # Main Django app (settings, urls, asgi/wsgi)
│   ├── settings/
│   │   ├── base.py
│   │   ├── local.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py
│
├── users/                 # Dedicated app for Custom User Model & Auth
│   ├── models.py          # ALWAYS create a custom user model
│   ├── views.py
│   ├── urls.py
│   └── serializers.py     # If using DRF
│
├── blog/                  # Feature App
│   ├── models.py
│   ├── services.py        # Business logic (kept out of views)
│   ├── selectors.py       # Complex DB queries (kept out of views)
│   ├── views.py           # API Endpoints
│   └── urls.py
│
└── manage.py
```

## Best Practices

### 1. Always Use a Custom User Model
Even if you don't need it immediately, changing the user model mid-project is incredibly difficult.
```python
# users/models.py
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    # Add custom fields here
    pass

# core/settings/base.py
AUTH_USER_MODEL = 'users.CustomUser'
```

### 2. Services and Selectors (Business Logic)
Django Views (or ViewSets) can easily become bloated. Extract logic into `services.py` (write operations) and `selectors.py` (read operations).

```python
# blog/services.py
def create_post(*, title: str, content: str, author: User) -> Post:
    post = Post(title=title, content=content, author=author)
    post.full_clean()  # Validates model
    post.save()
    return post

# blog/views.py
from rest_framework.views import APIView
from rest_framework.response import Response

class PostCreateAPI(APIView):
    def post(self, request):
        serializer = PostSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        
        post = create_post(
            title=serializer.validated_data['title'],
            content=serializer.validated_data['content'],
            author=request.user
        )
        return Response(PostSerializer(post).data)
```

### 3. API Framework Choices
| Framework | Purpose | Best For |
| --- | --- | --- |
| `Django Rest Framework (DRF)` | Heavyweight, feature-rich API toolset | Large enterprise CRUD apps |
| `Django Ninja` | FastAPI-like routing and Pydantic validation | Modern APIs, async support, auto-docs |

*Recommendation*: If starting a new Django API today, **Django Ninja** offers a significantly better developer experience, utilizing Pydantic and type hints similarly to FastAPI.

### 4. Background Tasks
Django does not handle background tasks out of the box. Use **Celery** with Redis as a broker.

## Essential Packages
| Package | Purpose |
| --- | --- |
| `django-environ` | Environment variables parsing |
| `django-cors-headers` | CORS middleware |
| `djangorestframework` | API Building (Alternative to Ninja) |
| `django-ninja` | API Building (Alternative to DRF) |
| `celery` | Background tasks |
| `django-filter` | Advanced query filtering |
