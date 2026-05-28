# Exercise: Documentation Navigation for FastAPI
 
**Language:** Python
**Framework:** FastAPI
**Goal:** Learn to navigate FastAPI documentation efficiently and translate concepts into working code
 
---
 
## Part 1: Documentation Summarization
 
### Personalized Reading Guide
 
**Recommended reading order for someone new to FastAPI:**
 
1. **Tutorial — User Guide: First Steps** — Get the app running and understand the basic structure before anything else.
2. **Path Parameters and Query Parameters** — These are the building blocks of every endpoint.
3. **Request Body with Pydantic Models** — This is where FastAPI becomes genuinely different from Flask; understanding Pydantic early saves a lot of confusion later.
4. **Dependency Injection (`Depends`)** — The most important architectural concept in FastAPI; learn this before building anything larger than a toy project.
5. **Security — OAuth2 with JWT** — Once dependencies make sense, authentication follows naturally from the same pattern.
6. **Background Tasks** — Simple but powerful; useful for almost every real application.
7. **Advanced: Middleware and CORS** — Needed before deploying anything to production.
---
 
### 5 Most Important Documentation Sections for Building a REST API Quickly
 
| Priority | Section | Why It Matters |
| :--- | :--- | :--- |
| 1 | Path & Query Parameters | Every endpoint uses these |
| 2 | Request Body (Pydantic) | Replaces all manual validation |
| 3 | Dependency Injection | Powers authentication, DB sessions, shared logic |
| 4 | Response Models | Controls what data is returned and in what shape |
| 5 | HTTP Exceptions | Consistent, structured error handling |
 
---
 
### Dependency Injection — Key Points Summary
 
- `Depends()` tells FastAPI to call a function and inject its return value as a parameter.
- Dependencies can depend on other dependencies, forming a chain that FastAPI resolves automatically.
- The same dependency function can be reused across many routes without any duplication.
- Dependencies are ideal for: authentication checks, database session management, shared query parameter logic, and permission validation.
- FastAPI caches dependency results within a single request by default, so a dependency is not called multiple times for the same request even if multiple routes use it.
---
 
## Part 2: Documentation Deep Dive
 
### Feature Chosen: Dependency Injection (`Depends`)
 
**What `Depends` is and when to use it:**
 
`Depends` is FastAPI's dependency injection mechanism. It accepts a callable (usually a function) and tells FastAPI to execute that callable before running the route handler, passing the result as a parameter. This is useful any time a route needs something that requires its own logic to produce — a verified user, a database session, a validated API key, or a set of common query parameters.
 
**When NOT to use `Depends`:**
 
- For simple, stateless logic that does not need to be reused — just write it inline.
- For logic that belongs in the Pydantic model itself (e.g., field-level validation).
- For global concerns like CORS — use middleware instead.
**Core patterns identified from the docs:**
 
- **Single dependency:** one function injected into one route.
- **Chained dependencies:** a dependency that itself uses `Depends` to call another dependency.
- **Class-based dependencies:** using a class with `__call__` for stateful dependencies (e.g., a query parameter bundle).
- **Global dependencies:** applied to an entire router or the whole app using `dependencies=[Depends(...)]`.
---
 
## Part 3: Concept to Code Translation
 
### Concept 1: Dependency Injection
 
```python
from fastapi import FastAPI, Depends, HTTPException, Header
from typing import Annotated
 
app = FastAPI()
 
# Step 1: Define a dependency function
async def verify_api_key(x_api_key: str = Header(...)):
    """Verify that the API key is valid."""
    if x_api_key != "valid_key_123":
        raise HTTPException(status_code=403, detail="Invalid API key")
    return x_api_key
 
# Step 2: Chain a second dependency on top of the first
async def get_current_user(api_key: str = Depends(verify_api_key)):
    """Get the current user based on the verified API key."""
    return {"user_id": 123, "name": "Demo User"}
 
# Step 3: Inject the final dependency into a route
@app.get("/profile/")
async def read_profile(user: dict = Depends(get_current_user)):
    """Get the current user's profile information."""
    return {
        "message": f"Hello, {user['name']}!",
        "profile_data": {
            "subscription": "premium",
            "joined_date": "2023-01-15"
        }
    }
```
 
---
 
### Concept 2: Pydantic Models for Request and Response Validation
 
```python
from pydantic import BaseModel, Field, EmailStr
from typing import Optional
 
# Request model — includes password for input
class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=8)
    full_name: Optional[str] = None
 
    class Config:
        schema_extra = {
            "example": {
                "username": "johndoe",
                "email": "john@example.com",
                "password": "securepass123",
                "full_name": "John Doe"
            }
        }
 
# Response model — excludes password from output
class UserResponse(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None
    id: int
 
@app.post("/users/", response_model=UserResponse)
async def create_user(user: UserCreate):
    """Create a new user. FastAPI filters the response using UserResponse."""
    created_user = user.dict()
    created_user["id"] = 1001
    return created_user
    # Note: even though created_user contains 'password',
    # FastAPI strips it because it is not in UserResponse
```
 
---
 
### Concept 3: Background Tasks
 
```python
from fastapi import BackgroundTasks
 
def send_welcome_email(user_email: str):
    """Simulate sending a welcome email after registration."""
    print(f"Sending welcome email to {user_email}...")
    # In production: await email_client.send(...)
 
@app.post("/register/")
async def register_user(user: UserCreate, background_tasks: BackgroundTasks):
    """Register a user and send a welcome email in the background."""
    new_user = {
        "id": 1002,
        "username": user.username,
        "email": user.email,
        "full_name": user.full_name
    }
    # Response is returned immediately; email sends after
    background_tasks.add_task(send_welcome_email, user.email)
    return {
        "message": "Registered successfully. Welcome email will be sent shortly.",
        "user": new_user
    }
```
 
---
 
### Concept 4: Path, Query, Header and Cookie Parameters
 
```python
from fastapi import Path, Query, Header, Cookie
 
@app.get("/items/{item_id}")
async def read_item(
    item_id: int = Path(..., title="Item ID", gt=0),
    q: Optional[str] = Query(None, min_length=3, max_length=50),
    limit: int = Query(10, ge=1, le=100),
    user_agent: Optional[str] = Header(None),
    session: Optional[str] = Cookie(None)
):
    """Demonstrates all four parameter sources in a single endpoint."""
    return {
        "item_id": item_id,
        "query_param": q,
        "limit": limit,
        "user_agent": user_agent,
        "session": session
    }
```
 
---
 
### Concept 5: Exception Handling
 
```python
from fastapi import HTTPException, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
 
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    """Custom handler — returns a clearer validation error response."""
    return JSONResponse(
        status_code=422,
        content={
            "detail": "Validation error in the request data",
            "errors": exc.errors(),
            "body": exc.body
        }
    )
 
@app.get("/products/{product_id}")
async def get_product(product_id: int):
    """Demonstrates structured HTTP error responses."""
    if product_id == 404:
        raise HTTPException(
            status_code=404,
            detail=f"Product with ID {product_id} not found",
            headers={"X-Error-Code": "PRODUCT_NOT_FOUND"},
        )
    elif product_id < 1:
        raise HTTPException(status_code=400, detail="Product ID must be a positive integer")
    return {"id": product_id, "name": "Sample Product", "price": 29.99}
```
 
---
 
## Part 4: Comprehensive Documentation Challenge — Blog API
 
### Application Structure
 
```
blog_api/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   └── comment.py
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── posts.py
│   │   └── comments.py
│   └── utils/
│       ├── __init__.py
│       └── auth.py
└── requirements.txt
```
 
### Pydantic Models
 
```python
# app/models/user.py
from pydantic import BaseModel, EmailStr, Field
from typing import Optional
 
class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=8)
 
class UserResponse(BaseModel):
    id: int
    username: str
    email: EmailStr
 
class Token(BaseModel):
    access_token: str
    token_type: str
 
 
# app/models/post.py
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime
 
class PostCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    content: str = Field(..., min_length=1)
    tags: list[str] = Field(default=[])
 
class PostResponse(BaseModel):
    id: int
    title: str
    content: str
    tags: list[str]
    author_id: int
    created_at: datetime
 
 
# app/models/comment.py
from pydantic import BaseModel, Field
from datetime import datetime
 
class CommentCreate(BaseModel):
    content: str = Field(..., min_length=1, max_length=1000)
 
class CommentResponse(BaseModel):
    id: int
    content: str
    author_id: int
    post_id: int
    created_at: datetime
```
 
### Authentication Utilities
 
```python
# app/utils/auth.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
 
SECRET_KEY = "blog-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 60
 
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")
 
def hash_password(password: str) -> str:
    return pwd_context.hash(password)
 
def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)
 
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
 
async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: int = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    return {"id": user_id}
```
 
### Auth Routes
 
```python
# app/routes/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from datetime import timedelta
from ..models.user import UserCreate, UserResponse, Token
from ..utils.auth import hash_password, verify_password, create_access_token, ACCESS_TOKEN_EXPIRE_MINUTES
 
router = APIRouter(prefix="/auth", tags=["auth"])
 
# Simulated user store
users_db = {}
user_counter = 0
 
@router.post("/register", response_model=UserResponse, status_code=201)
async def register(user: UserCreate):
    global user_counter
    if any(u["username"] == user.username for u in users_db.values()):
        raise HTTPException(status_code=400, detail="Username already taken")
    user_counter += 1
    users_db[user_counter] = {
        "id": user_counter,
        "username": user.username,
        "email": user.email,
        "hashed_password": hash_password(user.password)
    }
    return users_db[user_counter]
 
@router.post("/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = next((u for u in users_db.values() if u["username"] == form_data.username), None)
    if not user or not verify_password(form_data.password, user["hashed_password"]):
        raise HTTPException(status_code=401, detail="Incorrect username or password")
    token = create_access_token(
        data={"sub": user["id"]},
        expires_delta=timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    return {"access_token": token, "token_type": "bearer"}
```
 
### Blog Post Routes
 
```python
# app/routes/posts.py
from fastapi import APIRouter, Depends, HTTPException, Query, status
from typing import List, Optional
from datetime import datetime
from ..models.post import PostCreate, PostResponse
from ..utils.auth import get_current_user
 
router = APIRouter(prefix="/posts", tags=["posts"])
posts_db = {}
post_counter = 0
 
@router.post("/", response_model=PostResponse, status_code=201)
async def create_post(post: PostCreate, current_user: dict = Depends(get_current_user)):
    global post_counter
    post_counter += 1
    new_post = {
        **post.dict(),
        "id": post_counter,
        "author_id": current_user["id"],
        "created_at": datetime.utcnow()
    }
    posts_db[post_counter] = new_post
    return new_post
 
@router.get("/", response_model=List[PostResponse])
async def list_posts(
    search: Optional[str] = Query(None, description="Search in title or content"),
    tag: Optional[str] = Query(None, description="Filter by tag"),
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100)
):
    posts = list(posts_db.values())
    if search:
        posts = [p for p in posts if search.lower() in p["title"].lower()
                 or search.lower() in p["content"].lower()]
    if tag:
        posts = [p for p in posts if tag in p["tags"]]
    return posts[skip:skip + limit]
 
@router.get("/{post_id}", response_model=PostResponse)
async def get_post(post_id: int):
    if post_id not in posts_db:
        raise HTTPException(status_code=404, detail=f"Post {post_id} not found")
    return posts_db[post_id]
 
@router.put("/{post_id}", response_model=PostResponse)
async def update_post(post_id: int, post: PostCreate, current_user: dict = Depends(get_current_user)):
    if post_id not in posts_db:
        raise HTTPException(status_code=404, detail=f"Post {post_id} not found")
    if posts_db[post_id]["author_id"] != current_user["id"]:
        raise HTTPException(status_code=403, detail="Not authorised to edit this post")
    posts_db[post_id].update(post.dict())
    return posts_db[post_id]
 
@router.delete("/{post_id}", status_code=204)
async def delete_post(post_id: int, current_user: dict = Depends(get_current_user)):
    if post_id not in posts_db:
        raise HTTPException(status_code=404, detail=f"Post {post_id} not found")
    if posts_db[post_id]["author_id"] != current_user["id"]:
        raise HTTPException(status_code=403, detail="Not authorised to delete this post")
    del posts_db[post_id]
```
 
### Comment Routes
 
```python
# app/routes/comments.py
from fastapi import APIRouter, Depends, HTTPException, status
from typing import List
from datetime import datetime
from ..models.comment import CommentCreate, CommentResponse
from ..utils.auth import get_current_user
from .posts import posts_db
 
router = APIRouter(prefix="/posts/{post_id}/comments", tags=["comments"])
comments_db = {}
comment_counter = 0
 
@router.post("/", response_model=CommentResponse, status_code=201)
async def add_comment(post_id: int, comment: CommentCreate, current_user: dict = Depends(get_current_user)):
    if post_id not in posts_db:
        raise HTTPException(status_code=404, detail=f"Post {post_id} not found")
    global comment_counter
    comment_counter += 1
    new_comment = {
        **comment.dict(),
        "id": comment_counter,
        "post_id": post_id,
        "author_id": current_user["id"],
        "created_at": datetime.utcnow()
    }
    comments_db[comment_counter] = new_comment
    return new_comment
 
@router.get("/", response_model=List[CommentResponse])
async def list_comments(post_id: int):
    if post_id not in posts_db:
        raise HTTPException(status_code=404, detail=f"Post {post_id} not found")
    return [c for c in comments_db.values() if c["post_id"] == post_id]
```
 
### Main App
 
```python
# app/main.py
from fastapi import FastAPI
from .routes import auth, posts, comments
 
app = FastAPI(
    title="Blog API",
    description="A RESTful blog API built with FastAPI",
    version="1.0.0"
)
 
app.include_router(auth.router)
app.include_router(posts.router)
app.include_router(comments.router)
 
@app.get("/", tags=["root"])
async def root():
    return {"message": "Blog API is running", "docs": "/docs"}
 
# To run: uvicorn app.main:app --reload
# To install: pip install fastapi uvicorn python-jose[cryptography] passlib[bcrypt] pydantic[email]
```
 
---
 
## Reflection
 
**How did navigating the documentation help structure the learning?**
 
Starting with the reading roadmap meant I knew which concepts to tackle first. Without that structure I would have jumped straight to authentication before understanding how `Depends` works, which would have made the JWT implementation confusing. Reading in order — parameters, then Pydantic, then dependencies, then security — meant each concept built naturally on the previous one.
 
**What was the most valuable documentation section?**
 
The dependency injection section was the single most useful page. Once I understood that `Depends` is just a function that gets called automatically before the route runs, everything else — authentication, shared query parameters, database sessions — clicked into place. It is the concept that makes FastAPI architecturally different from Flask.
 
**How did translating concepts to code change my understanding?**
 
Writing the background tasks example made me realise that `BackgroundTasks` is itself injected as a dependency, just like `Depends`. The same mental model applies everywhere in FastAPI, which made learning new features much faster — I just had to ask "how does this get injected?" rather than learning a completely new pattern each time.
 
**What would I approach differently next time?**
 
I would test each endpoint in the `/docs` Swagger UI immediately after writing it rather than writing the full application first. Catching a parameter type error early saves significantly more time than discovering it after five functions have been built on top of it.
 
**What gaps remain?**
 
I want to explore FastAPI's integration with SQLAlchemy for real database persistence, WebSockets for real-time features, and the `lifespan` context manager for startup and shutdown events. These are the areas where the in-memory approach used here starts to show its limits.

