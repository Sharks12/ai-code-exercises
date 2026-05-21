# Code Understanding Journal: Codebase Exploration Challenge

**Course Module:** Using GenAI to Support Software Development  
**Student Name:** Seshoka Mailula  
**Student Email:** smailula40@ai.wethinkcode.co.za  
**Date:** May 2026  

---

## 🏛️ Exercise Part 1: Understanding Project Structure

### 1.1 Initial vs. Final Codebase Interpretation
* **Initial Assumption:** An initial surface inspection of the root directory files suggested a standard, multi-layered CLI utility layout. The distinct absence of a `requirements.txt` profile verified that the runtime environment functions under a strict zero-dependency constraint.
* **Final Discovery:** Using targeted project structure prompts, this initial view shifted from assuming it was a basic script to recognizing a highly modular system design. The application cleanly decouples human input presentation rules from underlying processing algorithms and state file storage layers.

### 1.2 Technology and Frameworks Infrastructure Stack
The application executes entirely via the built-in **Python Standard Library (v3.11+)**, relying on specialized core modules to manage tasks without external dependencies:
* **`argparse`:** Acts as the CLI entry pipeline, validating user arguments and routing text commands.
* **`uuid`:** Ensures system record uniqueness by issuing random 128-bit cryptographic IDs (`uuid4`).
* **`enum`:** Controls field value definitions to maintain strict structural data type checking across core states.
* **`datetime`:** Captures calendar markers to compute runtime overdue intervals and completion timestamps.

### 1.3 Structural Layers and Application Entry Point
The codebase boundaries split individual engineering responsibilities safely:
* **`cli.py` (Primary Entry Point):** Runs the main block to ingest command flags and format output tables.
* **`models.py` (Data Layer):** Anchors entity objects and calculates primary date validations.
* **`task_manager.py` (Core Orchestration Service):** Connects human actions to file mutations.

### 1.4 Mathematical Formulation of Data Objects
A single entity instance $T$ is modeled as a structured data tuple:  
$$T = (\text{id}, \text{title}, \text{description}, P, S, t_{\text{created}}, t_{\text{updated}}, t_{\text{due}}, t_{\text{completed}})$$

Categorical metrics for Priority ($P$) and Status ($S$) are bound to rigid sets:  
* $P \in \{1 = \text{LOW}, 2 = \text{MEDIUM}, 3 = \text{HIGH}, 4 = \text{URGENT}\}$
* $S \in \{\text{"todo"}, \text{"in\_progress"}, \text{"review"}, \text{"done"}\}$

The state logic for a deadline breach ($O_T$) is evaluated dynamically:  
$$O_T = \text{True} \quad \text{if} \quad t_{\text{due}} < t_{\text{current}} \wedge S \neq \text{"done"}, \quad \text{else} \quad \text{False}$$

---

## 🔍 Exercise Part 2: Finding Feature Implementation

### 2.1 Initial Codebase Search Strategy
To identify existing file operations, a codebase-wide search pattern was simulated matching parameters: `open()`, `.json`, `save`, and `load`. This targeted text lookup isolated `storage.py` as the system's single source of disk file handling.

### 2.2 Data Serialization and Reusability
File system access is managed by the `TaskStorage` class operating on a local flat file named `tasks.json`. Because JSON data arrays cannot handle specialized Python types natively, serialization is split into two custom utilities:
* **`TaskEncoder`:** Converts variables (Enums and datetimes) into text records or ISO-8601 data configurations.
* **`TaskDecoder`:** Rebuilds plaintext streams back into valid runtime objects using a targeted object hook.

*The TaskEncoder formatting loop and `get_all_tasks()` query are fully reusable assets for any upcoming export tool additions.*

### 2.3 Architectural Hypothesis and Modification Plan
* **Hypothesis:** Any structural data export feature (e.g., Export to CSV) must sit inside `storage.py` to keep file interactions properly isolated. Implementing this requires minimal adjustments across only two specific application components:
  1. **`storage.py`:** To introduce the conversion logic class and host the file output streams.
  2. **`cli.py`:** To inject an additional subparser command switch allowing direct execution from the terminal.

### 2.4 Four-Phase Practical Implementation Roadmap
* **Phase 1 (Data Transformation):** Write a custom text-formatter method within `storage.py` to output formatted rows.
* **Phase 2 (File I/O Stream):** Use native Python context handles inside try-except blocks to output files securely.
* **Phase 3 (CLI Presentation):** Update argparse rules in `cli.py` to map the user input to the new endpoint method.
* **Phase 4 (Automated QA):** Introduce verification assertions inside the `tests/` folder to guarantee formatting stability.

---

## 🔄 Exercise Part 3: Understanding Domain Model

### 3.1 Domain Model Extraction and Fields
The core system relies on three interconnected entities: the parent tracking object (`Task`), task priority rankings (`TaskPriority`), and lifecycle status values (`TaskStatus`). The `Task` functions as an encapsulation capsule governing internal properties, state properties, and real-time updates.

### 3.2 Core Business Logic Scoring Matrix
The system runs dynamic prioritization lists through a specialized service inside `task_priority.py`. This script applies a precise weighted algorithm to determine active task ranking values:
* **Base Weights:** Scale up with assignment severity, multiplying the baseline enum values up to a max of **+60** for Urgent tasks.
* **Timing Changes:** Add additional values: **+35** for overdue tasks, and **+20** for tasks with deadlines due on the same day.
* **Context Shortcuts:** Having the tags `"blocker"`, `"critical"`, or `"urgent"` instantly boosts task priority scores by **+8**.
* **State Mitigations:** Lower visibility safely: shifting to `REVIEW` reduces the score by **-15**, and completion (`DONE`) applies a heavy **-50** drop to move finished items to the bottom of visibility lists.

### 3.3 Application Glossary of Domain Terms
* **Aggregate Root:** A principal domain object (the `Task` class) that binds attributes together and manages internal modifications.
* **Domain Service:** A standalone script layer (`task_priority.py`) containing complex calculations that operate on entities but don't belong inside their structural properties.
* **Value Object:** Descriptive metadata parameters (`Status` and `Priority`) that define properties without possessing separate tracking IDs.

---

## 🧠 Exercise Part 4: Practical Application

### 4.1 Business Rule Assignment
> *"Tasks that are overdue for more than 7 days should be automatically marked as abandoned unless they are marked as high priority."*

### 4.2 Target Implementation Blueprint
To add this automated rule cleanly without disturbing current codebase layers, the feature follows this execution route:
1. In `models.py`, add a new operational state choice directly into the `TaskStatus` Enum: `ABANDONED = "abandoned"`.
2. In `task_manager.py`, introduce a cleanup sweep function called `process_abandoned_tasks(self)` applying logic:
   * **If** $S \neq \text{"done"}$
   * **and** $P \neq \text{HIGH}$
   * **and** $(t_{\text{current}} - t_{\text{due}}) > 7\text{ days}$ $\rightarrow$ $S = \text{"abandoned"}$
3. Following status alterations, the system calls `self.storage.save()` to write changes back to `tasks.json`.

### 4.3 Pre-Code Strategic Team Questions
* **Execution Context Location:** Should this sweep routine run as an automated background process or trigger synchronously every time a user requests a task display command like `list`?
* **Exemption Edge Cases:** Does protection from abandonment extend to the `URGENT` tier as well, or is it strictly limited to the literal `HIGH` value constraint?

### 4.4 Professional Reflection and Codebase Strategy
Leveraging structured AI prompts provided immediate clarity for breaking down difficult, abstract blocks like the prioritization calculations inside `task_priority.py`. However, some parts remain open for investigation; the structural data sorting rules within `task_list_merge.py` are not yet fully understood. 

My ongoing strategy for navigating unfamiliar systems will involve mapping operational data traces from input layers down to file operations, and executing deliberate code mutation tests within the `tests/` sandbox to verify system failure behaviors.
