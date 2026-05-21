# Code Understanding Journal: Codebase Exploration Challenge
**Course Module:** AI Software Engineering Track  
**Student Name:** Seshoka Mailula  
**Student Email:** smailula40@ai.wethinkcode.co.za  
**Date:** May 2026

---

## Exercise Part 1: Understanding a Specific Feature (Task Creation & Status Updates)

### 1. Main Components Involved
* **`cli.py` (Presentation Layer):** Ingests raw terminal input flags and text from the user using Python's built-in `argparse` library.
* **`models.py` (Domain Data Layer):** Defines the blueprints for the application's states using `TaskStatus` and `TaskPriority` Enums, and structures the core fields inside the `Task` initialization blueprint.
* **`storage.py` (Persistence Layer):** Manages data storage functions, reading from and writing records to the local `tasks.json` file on disk.

### 2. Detailed Execution Flows
* **Task Creation Execution Flow:** 1. The user inputs a terminal command (e.g., `python cli.py create "Title"`).  
  2. `cli.py` extracts the text strings and passes them to the creation handler.  
  3. A new `Task` instance is instantiated inside `models.py`, which automatically assigns a unique string identifier via `uuid.uuid4()`.  
  4. The object is sent to `storage.py`, where a custom `TaskEncoder` converts the data into plain text lines before updating `tasks.json`.
* **Status Update Execution Flow:** 1. The user runs an update command pointing to a specific ID and target state.  
  2. The system checks the task ID against the stored cache database in `storage.py`.  
  3. The properties are altered using the entity's inner `.update()` method or specifically through `.mark_as_done()` inside `models.py`.  
  4. If marked as completed, `status` switches to `DONE`, and the application automatically locks in the current clock time for `completed_at` and `updated_at`.

### 3. Core Design Patterns Discovered
The application utilizes a clean **Data Mapper/Serialization Pattern**. Because plain JSON files cannot naturally interpret complex Python data structures like dates or custom Enums, the code creates a dedicated `TaskEncoder` and `TaskDecoder` loop to safely translate data formats back and forth during disk saves.

---

## Exercise Part 2: Deepen Understanding Through Guided Questions (Prioritization System)

### 1. Initial Understanding vs. Final Discoveries
* **Initial Assumption:** Task priority was originally assumed to be a fixed metadata flag that stays static once chosen during task creation.
* **Final Discovery:** The implementation runs a dynamic algorithmic service. Priority assignments are converted into base numbers which are continuously scaled using time metrics and textual content shortcuts.

### 2. Key Insights Uncovered
The task sorting rules apply specific balance values to calculate an active importance score:
* Baseline weight values are set by priority tiers, reaching a maximum value of $+60$ points for Urgent records.
* Passing a task's deadline adds an immediate $+35$ point escalation penalty, while tasks due today receive a $+20$ bump.
* Context tags offer processing shortcuts; matching key phrases like `"blocker"` or `"critical"` adds an extra $+8$ point boost.
* Moving a task to `REVIEW` reduces its visibility weight by $-15$, and finishing an item completely applies a heavy $-50$ point drop to clear it from active display screens.

---

## Exercise Part 3: Mapping Data Flow (Task Completion Lifecycle)

### 1. State Mutations During Completion
When a task is successfully checked off, multiple variables shift state inside `models.py` simultaneously:
* `status` transitions cleanly from `TaskStatus.TODO` or `TaskStatus.IN_PROGRESS` $\rightarrow$ `TaskStatus.DONE`.
* `completed_at` captures the exact current execution timestamp (`datetime.now()`).
* `updated_at` matches the completion timestamp to preserve chronological tracking consistency.

### 2. Potential Points of Failure
* **Date Parsing Errors:** If a task's internal due date string lacks proper format sanitization, comparing timestamps against active clock cycles can trigger unhandled runtime failures.
* **File Input/Output Collisions:** If a user closes the command terminal abruptly while the `TaskEncoder` is halfway through executing file writing sweeps on `tasks.json`, the flat-file structure could become corrupted.

---

## Exercise Part 4: Reflection and Presentation Notes (3-5 Minute Summary)

### 1. High-Level Architecture Overview
The application follows a modular **Layered Architecture** driven by strict separation of concerns. By routing requests through an interface shell (`cli.py`), organizing changes via a domain framework (`models.py`), and isolating file input/output workflows (`storage.py`), the codebase functions as an enterprise-grade utility entirely within the Python Standard Library.

### 2. Most Challenging Aspect and AI Value
The most complicated element to parse initially was tracking how separate file boundaries passed state adjustments down to storage without a traditional relational database framework. Utilizing structured AI data-flow prompts mapped out the step-by-step pipeline clearly, helping visualize exactly how complex objects flatten into JSON arrays and safely rebuild upon rebooting the console.
