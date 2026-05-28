# Exercise: Understanding FastAPI Code Patterns

**Language:** Python
**Framework:** FastAPI
**Focus:** Dependency injection, middleware, repository pattern, and role-based access control

---

## Part 1: Analyzing Complex Code

### The Repository Pattern

The `Repository` class acts as an abstraction layer between the application logic and the database. Instead of writing raw database queries directly inside route handlers or service functions, all database access is centralised in repository classes. This means if the database changes — say from PostgreSQL to MongoDB — only the repository layer needs to change, not every route that touches data.

The `Generic[T]` type variable makes the repository reusable across different models. By writing `Repository[User]` or `Repository[Post]`, the same base class provides `get_by_id` and `list` methods for any model without duplicating code. The `TypeVar('T', bound=Base)` constraint ensures that only classes that inherit from `Base` (i.e., database models) can be used as the type parameter, catching type errors early.

```python
# T is a placeholder for any model class that inherits from Base
T = TypeVar('T', bound=Base)

class Repository(Generic[T]):
    def __init__(self, model: Type[T]):
        self.model = model

    async def get_by_id(self, db: AsyncSession, id: int) -> Optional[T]:
        result = await db.execute(select(self.model).where(self.model.id == id))
        return result.scalars().first()
```

**Why this matters:** Without the repository pattern, database queries get scattered across route handlers. With it, each model has one place where all its queries live, making the codebase far easier to test and maintain.

---

### Dependency Injection Layers

The application uses three nested layers of dependency injection:

| Layer | Dependency | What it provides |
| :--- | :--- | :--- |
| 1 — Infrastructure | `get_db()` | An async database session |
| 2 — Authentication | `get_current_user()` | The authenticated user object |
| 3 — Authorisation | `requires_role("admin")` | Role-based access enforcement |

Each layer depends on the one below it. `get_current_user` depends on both `oauth2_scheme` (for the token) and `get_db` (for the database session). The `requires_role` decorator then depends on `get_current_user`. FastAPI resolves this entire chain automatically before the route handler runs.

---

### Role-Based Access Control

```python
def requires_role(role: str):
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        async def wrapper(*args, current_user: User = Depends(get_current_user), **kwargs):
            if role == "admin" and not current_user.is_superuser:
                raise HTTPException(
                    status_code=status.HTTP_403_FORBIDDEN,
                    detail="Insufficient permissions"
                )
            return await func(*args, current_user=current_user, **kwargs)
        return wrapper
    return decorator
```

`requires_role` is a decorator factory — a function that returns a decorator. When you write `@requires_role("admin")`, Python calls `requires_role("admin")` first, which returns the `decorator` function, which then wraps the route handler. The `@wraps(func)` call preserves the original function's name and docstring so FastAPI can still generate correct documentation for it.

---

## Part 2: Tracing Execution Flow

### Request to `/admin/users/` — Step by Step

```
1. HTTP GET /admin/users/ arrives at the server

2. TimingMiddleware.__call__() fires
   └── records start_time
   └── calls call_next(request) to continue the chain

3. FastAPI resolves dependencies for list_users():
   │
   ├── get_db()
   │   └── opens an AsyncSession
   │   └── yields the session to the route
   │
   ├── oauth2_scheme
   │   └── extracts the Bearer token from the Authorization header
   │   └── raises HTTP 401 if no token is present
   │
   └── get_current_user(token, db)
       └── decodes the JWT token
       └── extracts username from payload["sub"]
       └── queries UserRepository.get_by_username(db, username)
       └── raises HTTP 401 if user not found
       └── returns the User object

4. requires_role("admin") wrapper runs
   └── checks current_user.is_superuser
   └── raises HTTP 403 if False
   └── proceeds if True

5. list_users() route handler executes
   └── calls UserRepository(User).list(db, skip, limit)
   └── returns list of User objects

6. FastAPI serialises the response using List[UserSchema]
   └── orm_mode = True allows reading from ORM objects directly

7. TimingMiddleware calculates elapsed time
   └── adds X-Process-Time header to the response

8. HTTP 200 response returned to client
```

---

## Part 3: Simplifying Complex Concepts

### The `lifespan` Context Manager — Plain English

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    print("Application startup")   # runs once when the server starts
    yield                          # application runs here, handling requests
    print("Application shutdown")  # runs once when the server stops
```

Think of it like a restaurant: the startup code is the kitchen prep before opening — connecting to databases, loading configuration, warming caches. The shutdown code is the closing routine — disconnecting from databases, flushing logs. Without `lifespan`, startup and shutdown logic had to be split into separate `@app.on_event("startup")` and `@app.on_event("shutdown")` decorators. The context manager keeps it in one place with a clear before/after structure.

---

### `TimingMiddleware` — Plain English

Middleware wraps every single request that passes through the application. The `TimingMiddleware` records the time before passing the request to the next handler, waits for the response to come back, calculates how long it took, and attaches that duration as a response header.

```python
class TimingMiddleware:
    async def __call__(self, request: Request, call_next: Callable) -> Response:
        start_time = datetime.utcnow()
        response = await call_next(request)  # all other processing happens here
        process_time = (datetime.utcnow() - start_time).total_seconds() * 1000
        response.headers["X-Process-Time"] = str(process_time)
        return response
```

A useful analogy: `call_next` is like handing the baton to the next runner in a relay race. Middleware runs before passing it on, and again after getting it back.

---

### JWT Authentication — Plain English for a Junior Developer

1. **User logs in** — sends username and password to `/token`.
2. **Server verifies** — checks the credentials against the database.
3. **Server issues a token** — encodes the username into a JWT signed with a secret key and returns it.
4. **User stores the token** — typically in memory or local storage.
5. **User makes a protected request** — sends the token in the `Authorization: Bearer {token}` header.
6. **Server decodes the token** — verifies the signature and extracts the username.
7. **Server loads the user** — fetches the full user record from the database using the username.
8. **Request proceeds** — the user object is injected into the route handler.

The key insight is that the server never stores the token — it just signs it. Any token that can be verified with the secret key is trusted.

---

## Part 4: Implementing a User Action Logging System

### New Model

```python
# app/models/audit_log.py
from sqlalchemy import Column, Integer, String, DateTime
from datetime import datetime
from .base import Base

class AuditLog(Base):
    __tablename__ = "audit_logs"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, nullable=False)
    username = Column(String, nullable=False)
    action = Column(String, nullable=False)       # e.g. "login", "list_users"
    endpoint = Column(String, nullable=False)     # e.g. "/token", "/admin/users/"
    timestamp = Column(DateTime, default=datetime.utcnow)
    details = Column(String, nullable=True)
```

### Audit Log Repository

```python
# app/repositories/audit_log.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.future import select
from typing import List
from ..models.audit_log import AuditLog

class AuditLogRepository:
    async def create(
        self,
        db: AsyncSession,
        user_id: int,
        username: str,
        action: str,
        endpoint: str,
        details: str = None
    ) -> AuditLog:
        log = AuditLog(
            user_id=user_id,
            username=username,
            action=action,
            endpoint=endpoint,
            details=details
        )
        db.add(log)
        await db.commit()
        await db.refresh(log)
        return log

    async def list_by_user(self, db: AsyncSession, user_id: int) -> List[AuditLog]:
        result = await db.execute(
            select(AuditLog).where(AuditLog.user_id == user_id).order_by(AuditLog.timestamp.desc())
        )
        return result.scalars().all()
```

### Logging Dependency

```python
# app/utils/audit.py
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession
from ..repositories.audit_log import AuditLogRepository
from ..utils.db import get_db

async def log_user_action(
    action: str,
    endpoint: str,
    user_id: int,
    username: str,
    db: AsyncSession = Depends(get_db),
    details: str = None
):
    """Reusable dependency for logging any user action."""
    repo = AuditLogRepository()
    await repo.create(
        db=db,
        user_id=user_id,
        username=username,
        action=action,
        endpoint=endpoint,
        details=details
    )
```

### Updated Login Route with Logging

```python
@app.post("/token")
async def login(
    username: str,
    password: str,
    db: AsyncSession = Depends(get_db)
):
    user_repo = UserRepository(User)
    user_service = UserService(user_repo)
    user = await user_service.authenticate_user(db, username, password)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # Log the successful login
    audit_repo = AuditLogRepository()
    await audit_repo.create(
        db=db,
        user_id=user.id,
        username=user.username,
        action="login",
        endpoint="/token"
    )

    access_token = user_service.create_access_token(
        data={"sub": user.username},
        expires_delta=timedelta(minutes=30)
    )
    return {"access_token": access_token, "token_type": "bearer"}
```

### Updated Admin Route with Logging

```python
@app.get("/admin/users/", response_model=List[UserSchema])
@requires_role("admin")
async def list_users(
    skip: int = 0,
    limit: int = 10,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    # Log the admin action
    audit_repo = AuditLogRepository()
    await audit_repo.create(
        db=db,
        user_id=current_user.id,
        username=current_user.username,
        action="list_users",
        endpoint="/admin/users/",
        details=f"skip={skip}, limit={limit}"
    )

    user_repo = UserRepository(User)
    users = await user_repo.list(db, skip=skip, limit=limit)
    return users
```

---

## Reflection Questions

**How did implementing the logging feature help you understand the architecture?**

Adding the logging system forced me to trace where the user object was available, where the database session was accessible, and where in the dependency chain I could inject the new behaviour without disrupting existing code. I realised that the repository pattern made the new feature straightforward — I just created a new `AuditLogRepository` following the same pattern as `UserRepository`, and it slotted in without any friction.

**Which design patterns were most useful in the original code?**

The repository pattern was the most practically useful because it gave me a clear template to follow when adding the `AuditLogRepository`. Dependency injection was architecturally the most important — without it, passing the database session and current user into the logging logic would have required either globals or significant refactoring.

**How would you explain the repository pattern and dependency injection to a colleague?**

The repository pattern is like having a dedicated librarian for each type of data. Instead of everyone going directly to the shelves (the database), they ask the librarian (the repository), who knows exactly where everything is and how to find it. Dependency injection is like a catering service for your functions — instead of each function going to fetch its own ingredients (database sessions, user objects, config values), FastAPI delivers them automatically before the function even starts running.

**How did tracing the execution flow help you decide where to add the logging code?**

Tracing the flow showed me exactly where the authenticated user object became available — inside the `get_current_user` dependency — and where the database session was open. This told me that the logging code needed to run after authentication but before the route handler returned its response. Without tracing the flow first, I might have tried to add logging in the middleware layer where the user identity is not yet available.
