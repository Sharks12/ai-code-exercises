# Code Understanding Journal: Codebase Exploration Challenge

[cite_start]**Course Module:** Using GenAI to Support Software Development   
[cite_start]**Student Name:** Seshoka Mailula [cite: 4]  
[cite_start]**Student Email:** smailula40@ai.wethinkcode.co.za [cite: 5]  
[cite_start]**Date:** May 2026 [cite: 6]  

---

## 🏛️ Exercise Part 1: Understanding Project Structure

### 1.1 Initial vs. Final Codebase Interpretation
* [cite_start]**Initial Assumption:** An initial surface inspection of the root directory files suggested a standard, multi-layered CLI utility layout[cite: 12]. [cite_start]The distinct absence of a `requirements.txt` profile verified that the runtime environment functions under a strict zero-dependency constraint[cite: 13].
* [cite_start]**Final Discovery:** Using targeted project structure prompts, this initial view shifted from assuming it was a basic script to recognizing a highly modular system design[cite: 14]. [cite_start]The application cleanly decouples human input presentation rules from underlying processing algorithms and state file storage layers[cite: 15].

### 1.2 Technology and Frameworks Infrastructure Stack
[cite_start]The application executes entirely via the built-in **Python Standard Library (v3.11+)**, relying on specialized core modules to manage tasks without external dependencies[cite: 17]:
* [cite_start]**`argparse`:** Acts as the CLI entry pipeline, validating user arguments and routing text commands[cite: 18].
* [cite_start]**`uuid`:** Ensures system record uniqueness by issuing random 128-bit cryptographic IDs (`uuid4`)[cite: 19].
* [cite_start]**`enum`:** Controls field value definitions to maintain strict structural data type checking across core states[cite: 20].
* [cite_start]**`datetime`:** Captures calendar markers to compute runtime overdue intervals and completion timestamps[cite: 21].

### 1.3 Structural Layers and Application Entry Point
[cite_start]The codebase boundaries split individual engineering responsibilities safely[cite: 22]:
* [cite_start]**`cli.py` (Primary Entry Point):** Runs the main block to ingest command flags and format output tables[cite: 23].
* [cite_start]**`models.py` (Data Layer):** Anchors entity objects and calculates primary date validations[cite: 24].
* [cite_start]**`task_manager.py` (Core Orchestration Service):** Connects human actions to file mutations[cite: 25].

### 1.4 Mathematical Formulation of Data Objects
[cite_start]A single entity instance $T$ is modeled as a structured data tuple[cite: 27]:  
$$T = (\text{id}, \text{title}, \text{description}, P, S, t_{\text{created}}, t_{\text{updated}}, t_{\text{due}}, t_{\text{completed}})$$

[cite_start]Categorical metrics for Priority ($P$) and Status ($S$) are bound to rigid sets[cite: 27]:  
* $P \in \{1 = \text{LOW}, 2 = \text{MEDIUM}, 3 = \text{HIGH}, 4 = \text{URGENT}\}$
* $S \in \{\text{"todo"}, \text{"in\_progress"}, \text{"review"}, \text{"done"}\}$

[cite_start]The state logic for a deadline breach ($O_T$) is evaluated dynamically[cite: 27, 28]:  
[cite_start]$$O_T = \text{True} \quad \text{if} \quad t_{\text{due}} < t_{\text{current}} \wedge S \neq \text{"done"}, \quad \text{else} \quad \text{False}$$ [cite: 29]

---

## 🔍 Exercise Part 2: Finding Feature Implementation

### 2.1 Initial Codebase Search Strategy
[cite_start]To identify existing file operations, a codebase-wide search pattern was simulated matching parameters: `open()`, `.json`, `save`, and `load`[cite: 31, 32]. [cite_start]This targeted text lookup isolated **`storage.py`** as the system's single source of disk file handling[cite: 32].

### 2.2 Data Serialization and Reusability
[cite_start]File system access is managed by the `TaskStorage` class operating on a local flat file named `tasks.json`[cite: 34]. [cite_start]Because JSON data arrays cannot handle specialized Python types natively, serialization is split into two custom utilities[cite: 35]:
* [cite_start]**`TaskEncoder`:** Converts variables (Enums and datetimes) into text records or ISO-8601 data configurations[cite: 36].
* **`TaskDecoder`:** Rebuilds plaintext streams back into valid runtime objects using a targeted object hook[cite: 37].

*The TaskEncoder formatting loop and `get_all_tasks()` query are fully reusable assets for any upcoming export tool additions[cite: 38].*

### 2.3 Architectural Hypothesis and Modification Plan
* [cite_start]**Hypothesis:** Any structural data export feature (e.g., Export to CSV) must sit inside `storage.py` to keep file interactions properly isolated[cite: 43]. [cite_start]Implementing this requires minimal adjustments across only two specific application components[cite: 44]:
  1. [cite_start]**`storage.py`:** To introduce the conversion logic class and host the file output streams[cite: 45].
  2. [cite_start]**`cli.py`:** To inject an additional subparser command switch allowing direct execution from the terminal[cite: 46].

### 2.4 Four-Phase Practical Implementation Roadmap
* **Phase 1 (Data Transformation):** Write a custom text-formatter method within `storage.py` to output formatted rows[cite: 47].
* [cite_start]**Phase 2 (File I/O Stream):** Use native Python context handles inside try-except blocks to output files securely[cite: 48].
* [cite_start]**Phase 3 (CLI Presentation):** Update argparse rules in `cli.py` to map the user input to the new endpoint method[cite: 49].
* **Phase 4 (Automated QA):** Introduce verification assertions inside the `tests/` folder to guarantee formatting stability[cite: 50].

---

## 🔄 Exercise Part 3: Understanding Domain Model

### 3.1 Domain Model Extraction and Fields
The core system relies on three interconnected entities: the parent tracking object (`Task`), task priority rankings (`TaskPriority`), and lifecycle status values (`TaskStatus`)[cite: 53]. The `Task` functions as an encapsulation capsule governing internal properties, state properties, and real-time updates[cite: 54].

### 3.2 Core Business Logic Scoring Matrix
The system runs dynamic prioritization lists through a specialized service inside **`task_priority.py`**[cite: 56]. [cite_start]This script applies a precise weighted algorithm to determine active task ranking values[cite: 57]:
* [cite_start]**Base Weights:** Scale up with assignment severity, multiplying the baseline enum values up to a max of **+60** for Urgent tasks[cite: 58].
* **Timing Changes:** Add additional values: **+35** for overdue tasks, and **+20** for tasks with deadlines due on the same day[cite: 59].
* [cite_start]**Context Shortcuts:** Having the tags `"blocker"`, `"critical"`, or `"urgent"` instantly boosts task priority scores by **+8**[cite: 60].
* [cite_start]**State Mitigations:** Lower visibility safely: shifting to `REVIEW` reduces the score by **-15**, and completion (`DONE`) applies a heavy **-50** drop to move finished items to the bottom of visibility lists[cite: 61].

### 3.3 Application Glossary of Domain Terms
* **Aggregate Root:** A principal domain object (the `Task` class) that binds attributes together and manages internal modifications[cite: 63].
* [cite_start]**Domain Service:** A standalone script layer (`task_priority.py`) containing complex calculations that operate on entities but don't belong inside their structural properties[cite: 64].
* **Value Object:** Descriptive metadata parameters (`Status` and `Priority`) that define properties without possessing separate tracking IDs[cite: 65].

---

## 🧠 Exercise Part 4: Practical Application

### 4.1 Business Rule Assignment
> *"Tasks that are overdue for more than 7 days should be automatically marked as abandoned unless they are marked as high priority."* [cite: 68]

### 4.2 Target Implementation Blueprint
To add this automated rule cleanly without disturbing current codebase layers, the feature follows this execution route[cite: 70]:
1. In `models.py`, add a new operational state choice directly into the `TaskStatus` Enum: `ABANDONED = "abandoned"`[cite: 71].
2. In `task_manager.py`, introduce a cleanup sweep function called `process_abandoned_tasks(self)` applying logic[cite: 72]:
   * [cite_start]**If** $S \neq \text{"done"}$ [cite: 73]
   * [cite_start]**and** $P \neq \text{HIGH}$ [cite: 74]
   * **and** $(t_{\text{current}} - t_{\text{due}}) > 7\text{ days}$ $\rightarrow$ $S = \text{"abandoned"}$ [cite: 75]
3. Following status alterations, the system calls `self.storage.save()` to write changes back to `tasks.json`[cite: 76].

### 4.3 Pre-Code Strategic Team Questions
* [cite_start]**Execution Context Location:** Should this sweep routine run as an automated background process or trigger synchronously every time a user requests a task display command like `list`? [cite: 81]
* [cite_start]**Exemption Edge Cases:** Does protection from abandonment extend to the `URGENT` tier as well, or is it strictly limited to the literal `HIGH` value constraint? [cite: 82]

### 4.4 Professional Reflection and Codebase Strategy
[cite_start]Leveraging structured AI prompts provided immediate clarity for breaking down difficult, abstract blocks like the prioritization calculations inside `task_priority.py`[cite: 84]. [cite_start]However, some parts remain open for investigation; the structural data sorting rules within `task_list_merge.py` are not yet fully understood[cite: 85]. 

[cite_start]My ongoing strategy for navigating unfamiliar systems will involve mapping operational data traces from input layers down to file operations, and executing deliberate code mutation tests within the `tests/` sandbox to verify system failure behaviors[cite: 86].
