# Documentation Module: Final Submission
**Student Name:** Seshoka Mailula  
**Student Email:** smailula40@ai.wethinkcode.co.za  
**Date:** May 2026  

---

## Part 1: Code Documentation (Task Manager)
*Ref: calculate_task_score function*

### Final Code
```python
def calculate_task_score(task) -> int:
    """
    Calculates a dynamic importance metric score for a given task entity.
    """
    priority_weights = {TaskPriority.LOW: 1, TaskPriority.MEDIUM: 2, TaskPriority.HIGH: 4, TaskPriority.URGENT: 6}
    score = priority_weights.get(task.priority, 0) * 10
    
    if task.due_date:
        days_until_due = (task.due_date - datetime.now()).days
        if days_until_due < 0: score += 35
        elif days_until_due == 0: score += 20
        elif days_until_due <= 2: score += 15
        elif days_until_due <= 7: score += 10

    if task.status == TaskStatus.DONE: score -= 50
    elif task.status == TaskStatus.REVIEW: score -= 15

    if any(tag in ["blocker", "critical", "urgent"] for tag in task.tags): score += 8

    ## Part 2: API Documentation (User Registration)

    openapi: 3.0.0
paths:
  /api/users/register:
    post:
      summary: Register a new user
      responses:
        '201': {description: "Created"}
        '400': {description: "Bad Request"}        


   ## Part 3: Project README & User Guide
Ref: Task Manager System

##  Part 3: Project README & User Guide

### Project README
**Description:** A Python-based engine that sorts tasks by dynamic importance.

**Installation:** 1. Clone the repository.
2. Ensure Python 3.8+ is installed.
3. Execute the scripts directly from the source directory.

**Contributing:** 1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/name`).
3. Commit your changes and push to your branch.
4. Open a Pull Request on GitHub.

**License:** MIT License.

---

### Developer Usage Guide
* **Authentication:** None (Public endpoint).
* **Request Format:** Submit a `POST` request with the header `Content-Type: application/json`.
* **Troubleshooting:** * **409 Errors:** Verify username/email uniqueness in the database.
    * **400 Errors:** Validate the input payload against the required schema (username, email, password).

---

##  Part 4: Final Exercise Reflection
* **Challenge Assessment:** The most difficult aspect was intent extraction; AI needs specific business context (like status penalties) to explain *why* code exists, rather than just *what* it does.
* **Prompt Strategy:** Context-heavy prompts—specifically defining `Task` object attributes and data structures—were critical to preventing the AI from hallucinating requirements.
* **Format Effectiveness:** OpenAPI is superior for collaborative/API-focused teams as it allows for automated schema validation and client code generation, whereas Markdown remains the gold standard for human-readable project guides.
* **Workflow Integration:** I plan to automate documentation generation within the CI/CD pipeline, ensuring that technical docs remain synced with the codebase whenever the endpoint logic evolves.
    days_since_update = (datetime.now() - task.updated_at).days

    
    if days_since_update < 1: score += 5
    return score
