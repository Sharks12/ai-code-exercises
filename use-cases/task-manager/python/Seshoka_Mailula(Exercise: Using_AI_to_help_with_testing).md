# Testing Exercise: Task Manager Prioritization

## Part 1: Behavior Analysis & Test Planning
### 1.1 List of 5 Essential Test Cases
1. **Urgent & Overdue:** Verify that an URGENT task (60 points) that is overdue (+35) gets the correct high score (95).
2. **Completed Task:** Verify that a task marked `TaskStatus.DONE` receives a -50 penalty, regardless of other factors.
3. **No Due Date:** Verify that a task with `due_date = None` does not trigger date-based bonuses and only calculates base priority.
4. **Critical Tags:** Verify that tasks with multiple tags (e.g., ["blocker", "urgent"]) only receive the +8 bonus once.
5. **Recency Check:** Verify that a task updated less than 24 hours ago receives the +5 bonus.

### 1.2 Test Plan Document
| Priority | Test Case | Type | Expected Outcome |
| :--- | :--- | :--- | :--- |
| **P0** | Score calculation (All factors) | Unit | Accurate total sum |
| **P0** | Sorting Order | Integration | Tasks returned from high to low score |
| **P1** | Empty list handling | Unit | Returns empty list |
| **P1** | Date calculation logic | Unit | Correct score based on day differences |
| **P2** | Limit in top priority | Integration | Correctly truncates list to N items |

---

## Part 2: Improving Tests
### 2.1 Unit Test (Initial vs Improved)
* **Initial Concept:** `assert calculate_task_score(task) == 10` (Too fragile/implementation-focused).
* **Improved Test:** `assert calculate_task_score(high_priority_task) > calculate_task_score(low_priority_task)` (Checks the *behavior* of prioritization).

### 2.2 Due Date Test (Refined)
* **Goal:** Verify due date offsets.
* **Principle:** Use time-mocking to ensure the test is deterministic.
* **Test Case:** If `days_until_due` is exactly 0, assert score includes +20.

---

## Part 3: TDD Practice
### 3.1 New Feature: Current User Boost (+12)
1. **Failing Test:** `assert calculate_task_score(user_task) == original_score + 12`
2. **Implementation:** Add `if task.assigned_to == current_user: score += 12`.
3. **Refactor:** Verify that this boost works independently of other tags or date bonuses.

### 3.2 Bug Fix: Days Since Update
1. **Test to reproduce:** Create a task updated 2 hours ago. The current `.days` implementation returns `0`.
2. **Fix:** Update to use `total_seconds() / 86400` to capture partial days.
3. **Regression Test:** Add a test case for tasks updated exactly 23 hours ago vs 25 hours ago to ensure accuracy.

---

## Part 4: Integration Test
**Workflow Test:** 1. **Data:** Create 3 tasks: `[DONE_TASK, URGENT_TASK, LOW_TASK]`.
2. **Workflow:** - Pass list to `sort_tasks_by_importance`.
   - Verify `[0]` is `URGENT_TASK`.
   - Verify `get_top_priority_tasks(tasks, limit=1)` returns only `URGENT_TASK`.
3. **Assertion:** Verify index 0 is high score, index last is lowest score (or negative).

---

## Reflection
Through this exercise, I learned that testing is not about validating the code I wrote, but validating the *business rules* the code must follow. Using TDD forced me to think about the "edge cases" (like empty lists or future dates) before writing the implementation, which led to cleaner, more resilient code. Integration testing taught me that while individual functions might pass, the logic only succeeds when the components communicate correctly.
