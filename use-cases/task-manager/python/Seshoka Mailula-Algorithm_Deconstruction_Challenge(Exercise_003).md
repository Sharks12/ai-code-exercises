# Code Understanding Journal: Algorithm Deconstruction Challenge

**Course Module:** Using GenAI to Support Software Development  
**Student Name:** Seshoka Mailula  
**Student Email:** smailula40@ai.wethinkcode.co.za  
**Date:** May 2026  

---

## 🏛️ Exercise Part 1: Selected Algorithm Identification
* **Chosen Algorithm:** Algorithm 1: Task Priority Sorting Algorithm (`task_priority.py`)
* **Core Functionality:** This module computes a dynamic, scalar integer importance score for individual task entity records by executing conditional checks across five distinct dimensions: core prioritization severity tiers, deadline parameters, current lifecycle status values, specific operational text keyword flags, and recent modification tracking timestamps.

---

## 🔍 Exercise Part 2: Structural Step-by-Step Breakdown

The calculation pipeline runs down a clear sequence where modifiers continuously pile on top of an initially derived priority base value:

### 1. Base Priority Score Instantiation
The system initializes an explicit mapping dictionary tracking integer multipliers against specific `TaskPriority` Enum keys:
* `LOW` $\rightarrow$ $1 \times 10 = 10 \text{ points}$
* `MEDIUM` $\rightarrow$ $2 \times 10 = 20 \text{ points}$
* `HIGH` $\rightarrow$ $4 \times 10 = 40 \text{ points}$
* `URGENT` $\rightarrow$ $6 \times 10 = 60 \text{ points}$

### 2. Temporal/Due Date Evaluation Block
If the task context possesses an active calendar date pointer (`task.due_date`), it isolates the delta days using integer boundaries relative to the host execution clock (`task.due_date - datetime.now().days`):
* $\Delta\text{ days} < 0$ (Overdue tasks): Adds $+35 \text{ points}$
* $\Delta\text{ days} = 0$ (Due today): Adds $+20 \text{ points}$
* $\Delta\text{ days} \le 2$ (Due within 48 hours): Adds $+15 \text{ points}$
* $\Delta\text{ days} \le 7$ (Due within a rolling week): Adds $+10 \text{ points}$

### 3. Lifecycle Status Mitigation Layer
Active tasks degrade visibility priority scores automatically when transitioning toward completion:
* `task.status == TaskStatus.DONE`: Deducts $-50 \text{ points}$
* `task.status == TaskStatus.REVIEW`: Deducts $-15 \text{ points}$

### 4. Semantic Metadata/Tag Scanning Check
The code evaluates the task string array using a structural `any()` generator list. If a task contains any keyword string matching `"blocker"`, `"critical"`, or `"urgent"`, it triggers a static importance boost:
* Priority Match Detected $\rightarrow$ Adds $+8 \text{ points}$

### 5. Recency Optimization Check
The execution path calculates processing delays since the last write update (`datetime.now() - task.updated_at`). If modifications were logged within a rolling 24-hour cycle ($\Delta\text{ days} < 1$), it applies a fresh modification bump:
* Recently Modified $\rightarrow$ Adds $+5 \text{ points}$

---

## 🔄 Exercise Part 3: Operational Control Flow Diagram

This architectural visualization traces how an unranked task maps through conditional pipelines to yield a final sorted collection array:
