# Code Understanding Journal: Algorithm Deconstruction Challenge

**Course Module:** Using GenAI to Support Software Development  
**Student Name:** Seshoka Mailula  
**Student Email:** smailula40@ai.wethinkcode.co.za  
**Date:** May 2026  

---

## 🏛️ Exercise Part 1: Selected Algorithm Identification

- **Chosen Algorithm:** Algorithm 1: Task Priority Sorting Algorithm (`task_priority.py`)
- **Core Functionality:** This module computes a dynamic, scalar integer importance score for individual task entity records by executing conditional checks across five distinct dimensions: core prioritization severity tiers, deadline parameters, current lifecycle status values, specific operational text keyword flags, and recent modification tracking timestamps.

---

## 🔍 Exercise Part 2: Structural Step-by-Step Breakdown

The calculation pipeline runs down a clear sequence where modifiers continuously pile on top of an initially derived priority base value:

### 1. Base Priority Score Instantiation

The system initializes an explicit mapping dictionary tracking integer multipliers against specific `TaskPriority` Enum keys:

| Priority | Calculation | Points |
|----------|-------------|--------|
| `LOW` | 1 × 10 | 10 pts |
| `MEDIUM` | 2 × 10 | 20 pts |
| `HIGH` | 4 × 10 | 40 pts |
| `URGENT` | 6 × 10 | 60 pts |

### 2. Temporal/Due Date Evaluation Block

If the task context possesses an active calendar date pointer (`task.due_date`), it isolates the delta days using integer boundaries relative to the host execution clock (`task.due_date - datetime.now().days`):

| Condition | Points |
|-----------|--------|
| Δ days < 0 (Overdue) | +35 pts |
| Δ days = 0 (Due today) | +20 pts |
| Δ days ≤ 2 (Within 48 hours) | +15 pts |
| Δ days ≤ 7 (Within a week) | +10 pts |

### 3. Lifecycle Status Mitigation Layer

Active tasks degrade visibility priority scores automatically when transitioning toward completion:

| Status | Points |
|--------|--------|
| `TaskStatus.DONE` | −50 pts |
| `TaskStatus.REVIEW` | −15 pts |

### 4. Semantic Metadata/Tag Scanning Check

The code evaluates the task string array using a structural `any()` generator list. If a task contains any keyword string matching `"blocker"`, `"critical"`, or `"urgent"`, it triggers a static importance boost:

- Priority Match Detected → **+8 pts**

### 5. Recency Optimization Check

The execution path calculates processing delays since the last write update (`datetime.now() - task.updated_at`). If modifications were logged within a rolling 24-hour cycle (Δ days < 1), it applies a fresh modification bump:

- Recently Modified → **+5 pts**

---

## 🔄 Exercise Part 3: Operational Control Flow Diagram

```
[ Input Task Object Stream ]
│
▼
[ Step 1: Priority Base ] ──────► Matches Enum (1, 2, 4, 6) x 10 Base Points
│
▼
< Check: Has Due Date? >
├── YES ──────────────────────► Evaluate Days Delta (Adds +10 to +35 Points)
└── NO  ──────────────────────► Pass through unmodified
│
▼
< Check: Status Level? >
├── TaskStatus.DONE ──────────► Deduct -50 Points
├── TaskStatus.REVIEW ────────► Deduct -15 Points
└── TaskStatus.TODO/IN_PROG ──► Pass through unmodified
│
▼
< Check: Target Tag Match? >
├── ["blocker", "critical", "urgent"] ──► Adds +8 Points
└── No Match ────────────────────────────► Pass through unmodified
│
▼
< Check: Updated < 24 Hours? >
├── YES ──────────────────────► Adds +5 Points
└── NO  ──────────────────────► Return calculated integer score
│
▼
[ Step 2: sort_tasks_by_importance ] ──► Builds (Score, Task) Tuples
│
▼
[ Step 3: Sorted Array List Output ] ──► Arranges rows in descending order using sorted()
```

---

## 🧠 Exercise Part 4: Technical Insights & Refactoring Roadmaps

### 4.1 Foundational Discoveries & Code Issues

- **The Negative Bounds Deficiency:** While the completion penalty applies a heavy −50 points reduction to clear finished items from active views, a `LOW` priority task (10 points) with no due date will drop down into an absolute negative mathematical score (−40). Sorting negative data types is functional, but it causes significant reporting anomalies if metrics modules assume that the minimum floor boundary remains clamped to zero.

- **Algorithmic Redundancy Vulnerability:** The orchestration utility function `sort_tasks_by_importance` generates temporary Python tuples inside a list comprehension to sort elements. While this avoids mutating the source array, invoking `calculate_task_score` repeatedly during every comparative pass scales performance complexity layout behaviors unreliably if array profiles scale to large datasets.

### 4.2 Modular Refactoring Strategy

To safely normalize boundaries and resolve performance degradation risks, the implementation can be optimized with an internal floor normalization check and custom sorting cache hooks:

```python
def calculate_task_score_optimized(task):
    # Core calculations run identical baseline values...
    score = base_calculated_value
    
    # Enforce strict normalization floor to prevent metric errors
    return max(0, score)

def sort_tasks_efficiently(tasks):
    # Uses native Python Timsort cached mapping to run scoring exactly once per task object
    return sorted(tasks, key=calculate_task_score_optimized, reverse=True)
```

---

## 📝 Reflection Responses

**1. How did the AI's explanation change your understanding of the algorithm?**

Initially, I assumed priority sorting was a static value assignment evaluated strictly through the core `TaskPriority` parameters selected by a user in the terminal. The AI deconstruction highlighted that priority is actually a fluid, composite calculations matrix where metadata fields, calendar times, and state transitions continuously shift visibility layers.

**2. What aspects were still difficult to understand after AI explanation?**

The timing boundary calculations initially looked prone to edge-case errors. Calculating `days_until_due` using `.days` on standard datetime differentials can trim hours depending on the local machine's calendar clock settings. It was tough to parse if a task due at 08:00 AM tomorrow would be handled as an active 1-day step or collapse down to a 0-day step if processed later in the evening.

**3. How would you explain this algorithm to another junior developer?**

"Think of this algorithm like a security guard deciding who gets to stand at the front of a line. Everyone starts with a base point level depending on the color of their badge (Priority Enum). If you're late for an appointment, the guard gives you +35 points to push you forward immediately. If your issue is explicitly tagged as a 'blocker', you get another +8 point push. But if you're already finished with your meeting (Status is DONE), the guard knocks you back by -50 points so you drop straight to the back of the line."

**4. Did you test this understanding against AI?**

Yes, I verified the edge-case profiles by feeding mock dataset attributes to the AI. I tested a scenario involving a completed URGENT task vs an overdue LOW priority task to evaluate if the completion penalty successfully pushes critical items below standard incomplete assignments.

**5. How might you improve the algorithm based on your understanding?**

I would implement three specific upgrades:
1. Apply a strict mathematical clamp using `max(0, score)` to prevent negative outputs.
2. Refactor the sorting loops to use an isolated key-lambda configuration to guarantee single-pass O(N) execution scaling.
3. Migrate raw tag array comparisons into a dedicated hash set container configuration to avoid slower linear string scanning routines.
