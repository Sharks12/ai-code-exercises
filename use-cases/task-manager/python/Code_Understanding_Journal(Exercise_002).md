# Code Understanding Journal
**Course Module:** Using GenAI to Support Software Development  
**Student Name:** Seshoka Mailula  
**Student Email:** smailula40@ai.wethinkcode.co.za  
**Date:** May 2026

---

##  Exercise Part 1: Understanding a Specific Feature (Task Creation & Updates)

### 1. Component Responsibility Matrix
The application logic handles task management commands from the terminal and stores data records locally without requiring an active external database engine. The system structure divides responsibilities across three decoupled layers:
* **`cli.py` (Presentation Layer):** Ingests raw text commands and flag arguments from the terminal using Python's native `argparse` library. It sanitizes basic user string input before routing execution variables downward and formats plain data objects into user-facing console text blocks using `format_task()`.
* **`models.py` (Domain Data Layer):** Acts as the foundational blueprint for core business entities. It houses the structural `Task` class definition and secures allowed baseline states using two explicit Enums: `TaskPriority` (integers 1–4) and `TaskStatus` (strings: "todo", "in_progress", "review", "done").
* **`storage.py` (Persistence Layer):** Owns total responsibility for file input/output operations. It manages an in-memory lookup collection (`self.tasks`) inside the `TaskStorage` service and handles safe reading and writing operations targeting the local flat-file named `tasks.json`.

### 2. Operational Execution Flows
* **Task Creation Path (`python cli.py create "Title" ...`):** `argparse` traps the creation command flags inside `cli.py`. The application routes variables to instantiate a new `Task` entity object in `models.py`. Upon instantiation, a unique 128-bit tracking ID token is automatically assigned via `str(uuid.uuid4())`, timeline parameters anchor to `datetime.now()`, and status defaults to `TaskStatus.TODO`. The object pointer hands off to `TaskStorage.add_task()`, updating the dictionary memory cache before flushing changes onto disk via a structural write channel.
* **Status Update Path (`python cli.py status <task_id> in_progress`):** `cli.py` captures the target identity variable along with the explicit keyword state argument. The application queries the cache lookup table in `storage.py`. If a valid ID match occurs, it drops into `models.py` and triggers the internal mutation method `task.update(**kwargs)`. This uses Python's structural reflection attribute `setattr()` to overwrite data variables dynamically and refreshes the chronological tracking parameter `self.updated_at = datetime.now()`.

### 3. Data Transformations & Caching Patterns
Data balances between volatile random-access memory (RAM) operations and non-volatile document file caches:
* **In-Memory Cache:** Active processes track assignments inside a native Python dictionary lookup table mapped as `{ task_id_string: TaskObjectInstance }` inside the `TaskStorage` class.
* **On-Disk Serialized State:** Data rests inside `tasks.json` as indented plaintext strings. The architecture bridges the object-to-text gap using a custom serialization pattern.

Because standard JSON pipelines throw a `TypeError` when encountering complex types like custom class Enums or native `datetime` data objects, the application deploys specialized encoding subclasses:
* **`TaskEncoder`:** Copies the internal object dictionary variables (`obj.__dict__.copy()`) and flattens abstract structures to basic JSON configurations. It converts Enum tracking instances down to their backing primitive representations (e.g., `TaskPriority.MEDIUM` $\rightarrow$ `2`) and strings dates into standardized string lines using `.isoformat()` (e.g., `"2026-05-21T14:15:00"`).
* **`TaskDecoder`:** Intercepts reading commands utilizing an explicit `object_hook`. When reading lines from `tasks.json`, it manually checks for structural task properties, reversing text configurations back into active Python instances using `TaskPriority(obj['priority'])` and `datetime.fromisoformat()`.

---

##  Exercise Part 2: Deepen Understanding (Prioritization System)

### 1. Initial Understanding vs. Actual Discoveries
* **Initial Impression:** The prioritization system was assumed to treat task visibility rankings as static, hardcoded data parameters that remain stagnant unless a user manually updates the priority tier from the terminal interface.
* **Actual Discovery:** Code analysis inside `models.py` reveals that the codebase evaluates task properties dynamically relative to current application constraints. The application checks deadline thresholds relative to the live system calendar clock dynamically rather than reading static, frozen database values.

### 2. Guided Insights Uncovered
* **The Deadline Status Guard:** The application checks for overdue conditions inside the `is_overdue(self)` method by evaluating if `self.due_date < datetime.now()`. However, this evaluation is bound simultaneously to a status validation conditional check: `and self.status != TaskStatus.DONE`. This design ensures that once a task's lifecycle crosses into the completed phase, the system safely suppresses the overdue flag automatically to prevent skewed metrics.
* **Mathematical Sorting Strategies:** Defining the `TaskPriority` Enum using primitive scalar integers (`1, 2, 3, 4`) instead of arbitrary descriptive words gives the runtime engine a major mathematical sorting advantage. It allows data processing components to sort or rank active datasets using basic comparative operators ($4 > 1$), instantly floating urgent rows to the top of display lists.

### 3. Misconceptions Clarified
* **The Chronological State Misconception:** I initially hypothesized that if a task passed its calendar deadline date, it would always remain flagged as overdue. Examining the internal validation methods clarified that entity status overrides chronological rules—marking a task as `DONE` instantly shields it from overdue classification loops.

---

##  Exercise Part 3: Mapping Data Flow (Task Completion)

### 1. State Modifications
When an item crosses the completion threshold, three explicit attributes mutate inside the `Task` entity simultaneously within `models.py`:
* `self.status` switches lifecycle values: `TaskStatus.TODO` or `TaskStatus.IN_PROGRESS` $\rightarrow$ `TaskStatus.DONE`.
* `self.completed_at` replaces its default `None` initialization marker with an active execution timestamp: `datetime.now()`.
* `self.updated_at` synchronizes exactly with the completion clock to maintain lifecycle tracking integrity.

### 2. Vulnerabilities & Points of Failure

| Potential Failure Point | Architectural Impact | Current Handling Type |
| :--- | :--- | :--- |
| **Broad Exception Handlers** | If the local storage drive fills up completely or encounters strict permission errors, the system prints a basic console error message but fails silently inside the execution loop. The runtime continues executing under the false assumption that data was saved. | Silent Catch (Generic Exception) |
| **Stale Unhandled Value Errors** | If an external system or manual edit introduces an unvalidated or corrupted date string layout into `tasks.json`, the parsing layer inside `TaskDecoder` will completely break on the next startup sweep, raising an unhandled `ValueError`. | Unhandled Crash |

### 3. Persistence Methodology
The codebase secures data state settings using an instant, synchronous write-through policy. The orchestration layer triggers a callback to `self.save()` immediately following any data attribute modification. This opens the local storage path file using a native Python file context manager in total overwrite mode (`with open(self.storage_path, 'w') as f:`), purging old file blocks and dropping down the fresh, updated text records.

---

##  Exercise Part 4: Technical Reflection

### 1. Core Engineering Lesson Learned
Navigating an unfamiliar codebase without changing live lines highlighted the immense power of system isolation. Separating human input presentation constraints (`cli.py`), application blueprints (`models.py`), and storage streams (`storage.py`) ensures that changes made to one layer do not cause unintended breakages in other modules.

### 2. Prompt Strategy Evaluation
* **Prompt 1 (Feature Isolation):** Most effective for tracing structural file handoffs and discovering how input arguments flow down into file actions.
* **Prompt 2 (Guided Questions):** Most effective for forcing close source inspections. Acting as a pair programmer prevented me from scanning code blindly and guided me to spot hidden conditional business rules (like the status guard inside `is_overdue`).
* **Prompt 3 (Data Mapping):** Best for identifying structural single points of failure (like the broad, silent try-except blocks in the persistence module).
