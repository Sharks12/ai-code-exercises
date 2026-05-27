# Exercise: Getting Started with FastAPI
 
**Language:** Python
**Framework:** FastAPI
**Goal:** Go from zero knowledge to building a working API with proper structure, validation, and error handling
 
---
 
## Part 1: Understanding FastAPI Fundamentals
 
### Notes from AI Research
 
**What is FastAPI?**
 
FastAPI is a modern, high-performance Python web framework for building APIs. It is built on top of Starlette for the web layer and Pydantic for data validation. Unlike Flask, which is minimal and requires third-party libraries for almost everything, and Django, which is a full-stack framework with an ORM and templating engine, FastAPI sits in the middle — lightweight like Flask but with built-in data validation, automatic documentation, and async support out of the box.
 
**Key Advantages over Flask and Django:**
 
| Feature | Flask | Django | FastAPI |
| :--- | :--- | :--- | :--- |
| Async support | Limited | Limited | Native |
| Auto documentation | No | No | Yes (Swagger + ReDoc) |
| Data validation | Manual | Forms-based | Built-in via Pydantic |
| Performance | Moderate | Moderate | Very high |
| Learning curve | Low | High | Low-Medium |
 
**Core Concepts Glossary:**
 
- **Path operation** — A function that handles a specific HTTP method and URL path (e.g., `@app.get("/items")`).
- **Path parameter** — A variable embedded in the URL path, e.g., `/items/{item_id}`.
- **Query parameter** — An optional parameter passed after `?` in the URL, e.g., `/search?q=laptop`.
- **Pydantic model** — A Python class that defines and validates the shape of request/response data.
- **Request body** — Data sent in the body of a POST/PUT request, validated by a Pydantic model.
- **Uvicorn** — An ASGI server used to run FastAPI applications locally.
- **Swagger UI** — Auto-generated interactive API documentation available at `/docs`.
- **Async/await** — Python syntax for writing non-blocking, concurrent code — FastAPI supports this natively.
---
 
## Part 2: Creating My First API
 
### Hello World Implementation
 
```python
# main.py
from fastapi import FastAPI
 
# Create a FastAPI instance — the core of the application
app = FastAPI(
    title="My First FastAPI App",
    description="A simple API built with FastAPI",
    version="0.1.0"
)
 
# Root endpoint — handles GET requests to "/"
@app.get("/")
async def root():
    """Returns a simple greeting message."""
    return {"message": "Hello World from FastAPI!"}
 
# Path parameter endpoint — item_id is captured from the URL
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    """Returns information about a specific item by its ID."""
    return {"item_id": item_id, "message": f"You requested item {item_id}"}
 
# Query parameter endpoint — q, skip, and limit are optional
@app.get("/search/")
async def search_items(q: str = None, skip: int = 0, limit: int = 10):
    """Searches for items based on optional query parameters."""
    return {
        "query": q,
        "skip": skip,
        "limit": limit,
        "message": f"Searching for '{q}' (skipping {skip}, limiting to {limit})"
    }
 
# To run:
# pip install fastapi uvicorn
# uvicorn main:app --reload
#
# Endpoints:
# http://127.0.0.1:8000/          — root
# http://127.0.0.1:8000/items/42  — item by ID
# http://127.0.0.1:8000/search?q=test — search
# http://127.0.0.1:8000/docs      — interactive documentation
```
 
### What I observed at `/docs`
 
Visiting `http://127.0.0.1:8000/docs` showed a fully interactive Swagger UI with all three endpoints listed, their parameters documented, and a "Try it out" button for each. This was the most impressive part — zero extra work required to get professional API documentation.
 
---
 
## Part 3: Enhancing the API
 
### Project Structure
 
```
my_fastapi_app/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── item.py
│   ├── routes/
│   │   ├── __init__.py
│   │   └── items.py
│   └── utils/
│       ├── __init__.py
│       └── exceptions.py
└── requirements.txt
```
 
### Pydantic Models (`app/models/item.py`)
 
```python
from pydantic import BaseModel, Field
from typing import Optional, List
 
class ItemBase(BaseModel):
    name: str = Field(..., min_length=1, max_length=100, description="The name of the item")
    description: Optional[str] = Field(None, max_length=1000)
    price: float = Field(..., gt=0, description="Price must be greater than zero")
    tags: List[str] = Field(default=[])
 
class ItemCreate(ItemBase):
    pass
 
class ItemResponse(ItemBase):
    id: int
 
    class Config:
        schema_extra = {
            "example": {
                "id": 1,
                "name": "Laptop",
                "description": "Powerful development machine",
                "price": 1299.99,
                "tags": ["electronics", "computers"]
            }
        }
```
 
### Custom Exception Handlers (`app/utils/exceptions.py`)
 
```python
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
 
class ItemNotFoundError(Exception):
    def __init__(self, item_id: int):
        self.item_id = item_id
        self.message = f"Item with ID {item_id} not found"
        super().__init__(self.message)
 
def add_exception_handlers(app: FastAPI) -> None:
    @app.exception_handler(ItemNotFoundError)
    async def item_not_found_handler(request: Request, exc: ItemNotFoundError):
        return JSONResponse(
            status_code=status.HTTP_404_NOT_FOUND,
            content={"detail": exc.message}
        )
 
    @app.exception_handler(RequestValidationError)
    async def validation_exception_handler(request: Request, exc: RequestValidationError):
        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            content={"detail": "Validation error", "errors": exc.errors()}
        )
```
 
### Item Routes (`app/routes/items.py`)
 
```python
from fastapi import APIRouter, Path, Query, status
from typing import List, Optional
from ..models.item import ItemCreate, ItemResponse
from ..utils.exceptions import ItemNotFoundError
 
router = APIRouter(prefix="/items", tags=["items"])
fake_items_db = {}
item_counter = 0
 
@router.post("/", response_model=ItemResponse, status_code=status.HTTP_201_CREATED)
async def create_item(item: ItemCreate):
    global item_counter
    item_counter += 1
    new_item = {**item.dict(), "id": item_counter}
    fake_items_db[item_counter] = new_item
    return new_item
 
@router.get("/{item_id}", response_model=ItemResponse)
async def read_item(item_id: int = Path(..., gt=0)):
    if item_id not in fake_items_db:
        raise ItemNotFoundError(item_id)
    return fake_items_db[item_id]
 
@router.get("/", response_model=List[ItemResponse])
async def list_items(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100),
    tag: Optional[str] = Query(None)
):
    items = list(fake_items_db.values())
    if tag:
        items = [i for i in items if tag in i.get("tags", [])]
    return items[skip:skip + limit]
```
 
---
 
## Part 4: Exercise Challenge — To-Do List API
 
### Pydantic Models
 
```python
# app/models/todo.py
from pydantic import BaseModel, Field
from typing import Optional
from datetime import date
 
class TodoBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    due_date: Optional[date] = None
 
class TodoCreate(TodoBase):
    pass
 
class TodoResponse(TodoBase):
    id: int
    completed: bool = False
 
    class Config:
        schema_extra = {
            "example": {
                "id": 1,
                "title": "Submit assignment",
                "description": "Upload to UNISA portal",
                "due_date": "2026-06-01",
                "completed": False
            }
        }
```
 
### To-Do Routes
 
```python
# app/routes/todos.py
from fastapi import APIRouter, Path, Query, status
from typing import List, Optional
from ..models.todo import TodoCreate, TodoResponse
 
router = APIRouter(prefix="/todos", tags=["todos"])
todos_db = {}
todo_counter = 0
 
@router.post("/", response_model=TodoResponse, status_code=status.HTTP_201_CREATED)
async def create_todo(todo: TodoCreate):
    """Create a new to-do item."""
    global todo_counter
    todo_counter += 1
    new_todo = {**todo.dict(), "id": todo_counter, "completed": False}
    todos_db[todo_counter] = new_todo
    return new_todo
 
@router.get("/", response_model=List[TodoResponse])
async def list_todos(
    status_filter: Optional[str] = Query(None, description="Filter by 'completed' or 'pending'")
):
    """List all to-do items with optional status filtering."""
    items = list(todos_db.values())
    if status_filter == "completed":
        items = [i for i in items if i["completed"]]
    elif status_filter == "pending":
        items = [i for i in items if not i["completed"]]
    return items
 
@router.patch("/{todo_id}/complete", response_model=TodoResponse)
async def complete_todo(todo_id: int = Path(..., gt=0)):
    """Mark a to-do item as completed."""
    if todo_id not in todos_db:
        from fastapi import HTTPException
        raise HTTPException(status_code=404, detail=f"To-do {todo_id} not found")
    todos_db[todo_id]["completed"] = True
    return todos_db[todo_id]
 
@router.delete("/{todo_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_todo(todo_id: int = Path(..., gt=0)):
    """Delete a to-do item."""
    if todo_id not in todos_db:
        from fastapi import HTTPException
        raise HTTPException(status_code=404, detail=f"To-do {todo_id} not found")
    del todos_db[todo_id]
```
 
### Main App
 
```python
# app/main.py
from fastapi import FastAPI
from .routes import todos
from .utils.exceptions import add_exception_handlers
 
app = FastAPI(
    title="To-Do List API",
    description="A simple to-do manager built with FastAPI",
    version="1.0.0"
)
 
app.include_router(todos.router)
add_exception_handlers(app)
 
@app.get("/", tags=["root"])
async def root():
    return {"message": "To-Do API is running", "docs": "/docs"}
 
# To run: uvicorn app.main:app --reload
```
 
---
 
## Reflection
 
**Which AI prompting strategies were most effective?**
 
Starting with conceptual questions before asking for code made the biggest difference. Understanding *why* Pydantic exists before seeing it in action meant I was not just copying syntax blindly — I understood what each `Field(...)` constraint was doing and why it mattered.
 
**What surprised me about FastAPI?**
 
The automatic `/docs` page genuinely surprised me. I expected to spend time writing API documentation separately, but FastAPI generates a fully interactive Swagger UI from the type hints and docstrings already in the code. That felt like getting something for free.
 
**How did my Python knowledge help?**
 
Knowing Python's type hints already made the Pydantic models feel natural. The `Optional[str]` and `List[str]` syntax was not new — I just had not used it for validation before. This made the learning curve noticeably shorter.
 
**What would I do differently next time?**
 
I would set up a real database connection earlier using SQLAlchemy or SQLModel instead of a fake in-memory dictionary. The in-memory approach was fine for learning, but it gave me a false sense of completion since nothing actually persists.
 
**What gaps remain?**
 
I still need to explore authentication with JWT tokens, connecting to a real database, and deploying a FastAPI application to a cloud environment. These will be my focus in the next session.
