# Exercise: Learning a New Programming Language with AI

**Source Language:** Python (intermediate)
**Target Language:** Python (advanced/idiomatic)
**Goal:** Write cleaner, more Pythonic code using advanced features like decorators, generators, and context managers

---

## Part 1: Learning Journey Plan

### My Learning Goals

1. Understand and apply idiomatic Python patterns such as list comprehensions, generators, and decorators.
2. Learn how to structure larger Python projects using modules, classes, and proper error handling.
3. Build a small data processing CLI tool that demonstrates these advanced concepts in practice.

### AI-Generated Learning Plan (Refined)

**Phase 1: Python Foundations Review**
- Prerequisites: Basic syntax, variables, loops, functions
- Steps:
  1. Revisit built-in data structures (lists, dicts, sets, tuples)
  2. Understand mutability vs immutability
  3. Practice writing functions with default and keyword arguments
  4. Review exception handling with `try-except-finally`
- Verification: Rewrite a basic script using only built-ins, no external libraries

**Phase 2: Idiomatic Python**
- Prerequisites: Completion of Phase 1
- Steps:
  1. Replace `for i in range(len(...))` loops with direct iteration
  2. Write list, dict, and set comprehensions
  3. Use `with` statements for file and resource management
  4. Apply `enumerate()` and `zip()` in place of manual indexing
- Verification: Transform a non-idiomatic script into clean Pythonic code

**Phase 3: Advanced Features**
- Prerequisites: Completion of Phase 2
- Steps:
  1. Understand and write decorators
  2. Use generators and `yield` for memory-efficient iteration
  3. Explore `functools` (e.g., `@lru_cache`, `partial`)
  4. Work with `dataclasses` for clean data modelling
- Verification: Build a decorator that logs execution time for any function

**Phase 4: Project and Best Practices**
- Prerequisites: Completion of Phase 3
- Steps:
  1. Structure a project with modules and packages
  2. Write unit tests using `unittest` or `pytest`
  3. Follow PEP 8 and use a linter (e.g., `flake8`)
  4. Build and document a small CLI project end to end
- Verification: Complete mini-project with tests and a README

---

## Part 2: Four-Step Prompting Strategy

**Topic Chosen:** Decorators

### Step 1: Conceptual Understanding

**Prompt submitted to AI:**
> I am proficient in basic Python and want to understand decorators deeply. Before writing code, what are the key ideas behind decorators? What problems do they solve? What mental models should I develop, and what are common misconceptions beginners have?

**Key takeaways from AI response:**
- A decorator is simply a function that receives another function as an argument and returns a new function.
- They solve the problem of adding repetitive behaviour (logging, timing, access control) without modifying the original function.
- The mental model shift: think of `@decorator` as syntactic sugar for `func = decorator(func)`.
- Common misconception: beginners think decorators change the function permanently — they do not; they wrap it.

---

### Step 2: Step-by-Step Breakdown

**Prompt submitted to AI:**
> I want to understand decorators in Python. Could you break down how they work, how the `@` syntax maps to function wrapping, the key structures involved, and common patterns — without writing complex code yet?

**Key takeaways:**
- The `wrapper` inner function is what gets called in place of the original.
- `*args` and `**kwargs` allow the wrapper to accept any arguments the original function expects.
- `functools.wraps` should be used to preserve the original function's name and docstring.

---

### Step 3: Guided Implementation

**Prompt submitted to AI:**
> I am ready to implement my first decorator in Python. Could you guide me through creating a simple execution logger that prints the function name before it runs? Please explain each part of the syntax.

**AI-guided example:**

```python
import functools

def log_execution(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Executing: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_execution
def greet(name):
    print(f"Hello, {name}!")

greet("Seshoka")
```

---

### Step 4: Understanding Verification

**My own implementation:**

```python
import functools

def log_execution(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Running function: {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished: {func.__name__}")
        return result
    return wrapper

@log_execution
def add_numbers(a, b):
    return a + b

print(add_numbers(3, 7))
```

**AI feedback summary:**
- Correctly used `@functools.wraps` to preserve function metadata.
- Capturing and returning `result` before printing "Finished" is the right approach.
- Suggested next step: build a timing decorator using `time.perf_counter()`.

---

## Part 3: Advanced Prompting Techniques

### Technique 1: Using Context Effectively

**Prompt:**
> I am learning Python decorators coming from writing plain functions. Could you explain how decorators relate to the concept of higher-order functions that I already understand in Python?

**What I learned:**
Decorators are just higher-order functions with clean syntax. Since I already understood passing functions as arguments, connecting that to the `@` syntax made the concept click immediately.

---

### Technique 2: Promoting Deep Understanding

**Prompt:**
> I have implemented this timing decorator. What are the performance implications? What alternative approaches could I take? How would this change if I needed to support class methods as well?

```python
import time
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper
```

**What I learned:**
- `perf_counter()` is more precise than `time.time()` for short operations.
- For class methods, the decorator needs to handle `self` correctly — `*args` already covers this.
- At scale, consider using a proper profiling tool like `cProfile` instead of manual timers.

---

## Part 4: Mini-Project

**Project:** A command-line text statistics tool that reads a `.txt` file and reports word count, line count, and the five most common words.

### Project Plan

| Component | Description |
| :--- | :--- |
| `read_file(path)` | Reads and returns file contents using a context manager |
| `calculate_stats(text)` | Returns word count, line count, and top 5 words |
| `display_results(stats)` | Prints a formatted report to the terminal |
| `@timer` decorator | Wraps `calculate_stats` to log execution time |

### Implementation

```python
import time
import functools
from collections import Counter

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"[{func.__name__}] completed in {end - start:.4f}s")
        return result
    return wrapper

def read_file(path):
    with open(path, 'r', encoding='utf-8') as f:
        return f.read()

@timer
def calculate_stats(text):
    words = text.lower().split()
    lines = text.splitlines()
    top_words = Counter(words).most_common(5)
    return {
        'word_count': len(words),
        'line_count': len(lines),
        'top_words': top_words
    }

def display_results(stats):
    print(f"\nWord Count : {stats['word_count']}")
    print(f"Line Count : {stats['line_count']}")
    print(f"\nTop 5 Words:")
    for word, count in stats['top_words']:
        print(f"  {word}: {count}")

if __name__ == "__main__":
    text = read_file("sample.txt")
    stats = calculate_stats(text)
    display_results(stats)
```

---

## Reflection Questions

**Which prompting strategies were most effective for your learning style?**

The step-by-step breakdown was the most useful for me. Getting the concept explained in plain language before seeing any code meant I was not just copying syntax — I actually understood what each line was doing before I wrote it.

**What surprised you about Python that was not immediately obvious?**

I was surprised by how much `functools.wraps` matters. Without it, the decorated function loses its original name and docstring, which makes debugging significantly harder. It seemed like a small detail but turned out to be essential.

**How did your existing Python knowledge help or hinder learning?**

It helped enormously. Because I already understood functions as first-class objects, the leap to passing functions as arguments felt natural. The only hindrance was that I initially overthought decorators, assuming they were more complex than they are.

**What would you do differently in your next learning session?**

I would implement each concept immediately after the conceptual explanation rather than waiting until Step 3. Writing code sooner would have surfaced my misunderstandings earlier.

**What gaps remain in your understanding?**

I want to explore class-based decorators and decorators that accept their own arguments (decorator factories). Those patterns are slightly more involved and will be my focus in the next session.
