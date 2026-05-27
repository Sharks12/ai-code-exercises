# Exercise: Contextual Learning with FastAPI

**Language:** Python
**Framework:** FastAPI
**Prior Knowledge:** Flask (intermediate), basic Django

---

## Part 1: Framework Comparison

### Personal Translation Table

| Concept | Flask | Django | FastAPI |
| :--- | :--- | :--- | :--- |
| App instance | `Flask(__name__)` | `django.setup()` | `FastAPI()` |
| Route definition | `@app.route("/path")` | `urlpatterns = [path(...)]` | `@app.get("/path")` |
| Route grouping | `Blueprint` | `include_router` in urls.py | `APIRouter` |
| Request data | `request.get_json()` | `request.POST` / serializers | Pydantic model as parameter |
| Middleware | `@app.before_request` | `MIDDLEWARE` in settings.py | `app.add_middleware(...)` |
| Dependency injection | Manual / Flask globals | Django's ORM / forms | `Depends()` |
| Auto documentation | None (needs Flask-Swagger) | None (needs drf-yasg) | Built-in at `/docs` |
| Async support | Limited (via extensions) | Limited | Native `async/await` |
| Data validation | Manual or WTForms | Django Forms / DRF serializers | Pydantic (built-in) |

### Key Observations

The biggest mental shift coming from Flask is that **FastAPI removes most of the boilerplate I used to write manually**. In Flask, I would manually call `request.get_json()`, check for missing fields, and return error responses myself. In FastAPI, I just define a Pydantic model as a function parameter and all of that happens automatically. Flask's `Blueprint` maps almost directly to FastAPI's `APIRouter` — same idea of grouping related routes, just slightly different syntax.

---

## Part 2: Understanding FastAPI's Design Choices

### Design Philosophy Summary

FastAPI was built around one core idea: **use what Python already gives you, and make it do more work for you**. Rather than inventing a new validation system, it adopted Pydantic, which already had a strong community and leveraged Python's native type hints. Rather than requiring developers to write API documentation manually, it generates it automatically from the same type annotations used for validation. This means you write the code once and get validation, documentation, and editor autocomplete all at the same time.

**Why Pydantic instead of custom validation?**

Pydantic already solved data validation cleanly using Python type hints. Building a custom system would have duplicated effort and split the ecosystem. By adopting Pydantic, FastAPI inherits its entire feature set — nested models, custom validators, JSON schema generation — for free.

**Why automatic documentation?**

In Flask and Django REST Framework, keeping documentation in sync with actual code is a constant maintenance burden. FastAPI eliminates this entirely by deriving the OpenAPI spec directly from the code itself. If the code changes, the docs change automatically.

**Why type hints so extensively?**

Type hints serve triple duty in FastAPI: they enable editor autocompletion, they drive Pydantic validation at runtime, and they generate the OpenAPI schema for documentation. Using them extensively is not just a style choice — it is what makes the entire framework function.

**Why async-first?**

Flask was designed when Python's async support was immature. FastAPI was built after `async/await` became a stable Python feature, so it treats async as the default rather than an afterthought. This matters especially for APIs that make many I/O calls (database queries, external API requests) where async provides significant performance gains without extra complexity.

---

## Part 3: Applied Contextual Learning — JWT Authentication

### Implementation

```python
# app/main.py
from datetime import datetime, timedelta
from typing import Optional

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# Security configuration
SECRET_KEY = "your-secret-key-for-jwt"  # Use proper secret management in production
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Simulated user database
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",  # "secret"
        "disabled": False,
    }
}

# Pydantic models
class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None

class User(BaseModel):
    username: str
    email: Optional[str] = None
    full_name: Optional[str] = None
    disabled: Optional[bool] = None

class UserInDB(User):
    hashed_password: str

# Security utilities
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI(title="FastAPI JWT Auth Demo")

# Password utilities
def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password):
    return pwd_context.hash(password)

# User utilities
def get_user(db, username: str):
    if username in db:
        return UserInDB(**db[username])
    return None

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user or not verify_password(password, user.hashed_password):
        return False
    return user

# Token creation
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

# Dependency: get current user from token
async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

# Dependency: ensure user is active
async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

# Routes
@app.post("/token", response_model=Token)
async def login_for_access_token(form_data: OAuth2PasswordRequestForm = Depends()):
    """OAuth2 compatible token login — returns a JWT for future requests."""
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token = create_access_token(
        data={"sub": user.username},
        expires_delta=timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me/", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    """Returns the currently authenticated user's profile."""
    return current_user

@app.get("/users/me/items/")
async def read_own_items(current_user: User = Depends(get_current_active_user)):
    """Returns items belonging to the current user."""
    return [{"item_id": "Foo", "owner": current_user.username}]

# To use:
# pip install fastapi uvicorn python-jose[cryptography] passlib[bcrypt]
# uvicorn main:app --reload
# POST /token with form data: username=johndoe, password=secret
# Add header: Authorization: Bearer {token} to access protected routes
```

### Reflection on Authentication

**How does FastAPI's approach compare to other frameworks?**

In Flask, I would typically use Flask-Login or Flask-JWT-Extended as separate extensions, each with their own configuration patterns. In Django, authentication is built into the framework but tightly coupled to the session and ORM system. FastAPI's approach is different in that authentication is implemented as a **dependency** — a reusable function injected wherever it is needed. This means I can protect any endpoint simply by adding `Depends(get_current_active_user)` to its parameters, without any decorators or middleware configuration.

**What advantages does dependency injection provide for authentication?**

The `Depends()` system makes authentication composable and testable. I can swap out `get_current_user` for a test version during unit testing without touching the route handlers at all. In Flask, mocking authentication usually required patching globals or using test clients with session manipulation, which was messy. FastAPI's approach keeps each concern cleanly separated.

**How does type hinting make security clearer?**

Because the current user is typed as `User`, my editor immediately knows what fields are available on the object inside a protected route. There is no guessing whether `current_user.email` exists — the type system confirms it. This also means if the `User` model changes, the type checker catches every place that would break.

**What patterns from other frameworks are visible here?**

The overall flow — receive credentials, verify against a user store, issue a token, validate the token on subsequent requests — is the same pattern used in Django REST Framework's JWT implementation and Flask-JWT-Extended. FastAPI did not reinvent the security model; it just made the wiring cleaner and more Pythonic through dependency injection and type hints.

---

## Part 4: Mental Model Translation

### From Django/Flask to FastAPI

| Django / Flask Concept | FastAPI Equivalent | Key Difference |
| :--- | :--- | :--- |
| Flask `Blueprint` / Django `urls.py` | `APIRouter` | Same grouping idea; FastAPI uses `include_router()` |
| Django `Middleware` | `app.add_middleware()` | Similar placement; FastAPI uses Starlette middleware |
| Django `Serializer` / Flask manual validation | Pydantic `BaseModel` | Validation is automatic via type hints, not manual |
| Django `View` / Flask route function | Path operation function | FastAPI functions return dicts directly, not `Response` objects |
| Flask `g` / Django `request.user` | `Depends(get_current_user)` | Injected as a parameter, not a global |
| Django `settings.py` | Environment variables + Pydantic `BaseSettings` | Configuration is typed and validated |
| DRF `@permission_classes` | `Depends()` on route parameter | Dependencies are composable and reusable |

### Updated Mental Model

My previous mental model of web frameworks was built around **request/response cycles with middleware stacks** — a request comes in, passes through middleware layers, hits a view, and a response goes back out. FastAPI fits this model but adds a layer I had not thought about before: **dependency resolution**. Before the route function even runs, FastAPI resolves a tree of dependencies, injecting validated data, authenticated users, and database connections as function parameters. This feels closer to how dependency injection works in typed languages like Java or C# than to how Python frameworks traditionally operate, and it makes large applications significantly easier to organise and test.
