# Phase I Technical Implementation Plan

**Version**: 1.0.0
**Status**: APPROVED
**Phase**: I of V
**Based on**: PHASE_I_SPECIFICATION.md v1.0.0 (APPROVED)
**Constitutional Compliance**: Verified against CONSTITUTION.md v1.0.0
**Approval Date**: 2026-01-03

---

## Plan Overview

This plan describes HOW the approved Phase I specification will be implemented. It defines the technical architecture, data structures, control flow, and implementation strategy for the in-memory Python console todo application.

**Critical Constraint**: This plan introduces ZERO new features beyond the approved specification.

---

## Constitutional Compliance Verification

### Article I Compliance: Spec-Driven Development
- ✓ Specification APPROVED before plan creation
- ✓ Plan will be approved before task breakdown
- ✓ No code will be written until tasks are defined

### Article II Compliance: Agent Behavior
- ✓ No feature invention - all features from approved spec
- ✓ No deviation from specification requirements
- ✓ Plan describes implementation, not new features

### Article III Compliance: Phase Governance
- ✓ Strictly Phase I scope only
- ✓ No Phase II+ concepts (no web, no API, no persistence)
- ✓ No future-phase dependencies

### Article IV Compliance: Technology Constraints
- ✓ Python only (no frameworks prohibited in Phase I)
- ✓ Standard library only
- ✓ No databases, no files, no external services

### Article V Compliance: Quality Principles
- ✓ Clean architecture with separation of concerns
- ✓ Stateless where applicable
- ✓ Clear layer boundaries

---

## 1. Application Structure

### 1.1 Module Architecture

The application will consist of **4 Python modules** with clear separation of concerns:

```
todo_app/
├── main.py              # Entry point and application orchestration
├── task.py              # Task data model and validation
├── task_manager.py      # Business logic and in-memory storage
└── cli.py               # User interface and I/O handling
```

### 1.2 Dependency Flow

```
main.py
  ├─> cli.py (for UI operations)
  └─> task_manager.py (for business operations)
        └─> task.py (for data modeling)

Rule: Dependencies flow downward only
- main.py depends on cli.py and task_manager.py
- task_manager.py depends on task.py
- cli.py is independent (only I/O)
- task.py is independent (only data)
```

### 1.3 Responsibility Boundaries

**main.py**:
- Application entry point
- Main event loop (menu → action → menu)
- Orchestrate CLI and TaskManager
- No business logic
- No UI logic

**task.py**:
- Task class definition with `__init__` method
- Field validation (description length, type checking)
- Data integrity enforcement
- No storage logic
- No UI logic

**task_manager.py**:
- In-memory task storage (Python list)
- CRUD operation implementations
- ID generation and management
- Task lookup and validation
- No UI logic

**cli.py**:
- Menu display
- User input collection and parsing
- Output formatting (tables, messages)
- Error message display
- No business logic
- No data validation (delegated to task_manager/task)

---

## 2. In-Memory Data Structures

### 2.1 Task Storage

**Primary Storage**: Python `list` of `Task` objects

```python
# In task_manager.py
tasks: list[Task] = []  # Module-level list, persists during runtime
```

**Rationale**:
- Simple, built-in data structure
- Ordered by insertion (matches spec: "oldest first")
- Direct index access for lookups
- No persistence (cleared on exit per spec)

### 2.2 Task Object Structure

**Class Definition** (in task.py):

```python
class Task:
    id: int                    # Unique identifier
    description: str           # Task text (1-500 chars)
    is_complete: bool          # Completion status
```

**Implementation Strategy**:
- Use `@dataclass` decorator for automatic `__init__`, `__repr__`, etc.
- Validate description in `__post_init__` method
- Raise `ValueError` for invalid descriptions (empty, too long)
- ID assigned by TaskManager, not by Task class

**Field Constraints** (from spec):
- `id`: Positive integer, unique, auto-assigned
- `description`: Non-empty string, 1-500 characters after trimming
- `is_complete`: Boolean, defaults to False

### 2.3 Data Access Pattern

**No direct list access from main.py or cli.py**:
- All task access goes through task_manager.py methods
- Encapsulation ensures data integrity
- TaskManager owns the list lifecycle

---

## 3. Task Identification Strategy

### 3.1 ID Generation Algorithm

**Strategy**: Simple auto-increment counter

**Implementation**:
```python
# In task_manager.py
_next_id: int = 1  # Module-level counter, starts at 1

def _generate_id() -> int:
    global _next_id
    task_id = _next_id
    _next_id += 1
    return task_id
```

**Properties**:
- IDs start at 1 (user-friendly)
- Sequential and predictable
- Never reuse IDs (even after deletion)
- Simple and fast

### 3.2 ID Assignment Flow

1. User requests "Add Task"
2. CLI collects description
3. TaskManager.add_task(description) called
4. TaskManager generates ID via _generate_id()
5. TaskManager creates Task object with ID
6. TaskManager appends Task to list
7. TaskManager returns Task (or ID) to confirm

### 3.3 ID Lookup Strategy

**Method**: Linear search through list

```python
def get_task_by_id(task_id: int) -> Task | None:
    for task in tasks:
        if task.id == task_id:
            return task
    return None
```

**Rationale**:
- Simple implementation
- Adequate performance for up to 1000 tasks (spec requirement)
- No need for hash maps or indexes in Phase I

---

## 4. CLI Control Flow

### 4.1 Main Loop Structure

**Infinite loop until user selects "Exit"**:

```
START
  ↓
DISPLAY_MENU
  ↓
GET_USER_CHOICE
  ↓
VALIDATE_CHOICE
  ├─ Invalid → SHOW_ERROR → DISPLAY_MENU
  └─ Valid → DISPATCH_ACTION
               ↓
          EXECUTE_ACTION
               ↓
          DISPLAY_RESULT
               ↓
          WAIT_FOR_ENTER
               ↓
          DISPLAY_MENU (loop)
```

**Exit Condition**: User selects option 6 (Exit)

### 4.2 Menu Display Function

**Function**: `cli.display_menu()`

**Output**:
```
========================================
      Todo Application - Phase I
========================================

Main Menu:
1. Add Task
2. View All Tasks
3. Update Task
4. Delete Task
5. Mark Task Complete/Incomplete
6. Exit

Enter your choice (1-6):
```

**Implementation**:
- Print to stdout
- No return value
- Clear, consistent formatting

### 4.3 User Input Handling

**Input Collection Strategy**:
- Use `input()` built-in function
- Always read as string
- Parse/validate based on expected type
- Handle empty input gracefully

**Input Types**:

1. **Menu Choice** (1-6):
   - Read as string
   - Convert to int with try/except
   - Validate range (1-6)
   - Return error if invalid

2. **Task Description**:
   - Read as string
   - Strip whitespace
   - Validate length (1-500 chars)
   - Return error if empty or too long

3. **Task ID**:
   - Read as string
   - Convert to int with try/except
   - Validate existence via TaskManager
   - Return error if not found

4. **Yes/No Confirmation**:
   - Read as string
   - Convert to lowercase
   - Accept only 'y' or 'n'
   - Return error for other input

### 4.4 Action Dispatch

**Pattern**: Simple if/elif chain in main.py

```python
choice = cli.get_menu_choice()

if choice == 1:
    cli.handle_add_task(task_manager)
elif choice == 2:
    cli.handle_view_tasks(task_manager)
elif choice == 3:
    cli.handle_update_task(task_manager)
elif choice == 4:
    cli.handle_delete_task(task_manager)
elif choice == 5:
    cli.handle_toggle_complete(task_manager)
elif choice == 6:
    cli.display_exit_message()
    break
```

**No switch/case needed**: Simple if/elif is clear and sufficient

---

## 5. Separation of Responsibilities

### 5.1 Layer Responsibilities

**Presentation Layer (cli.py)**:
- Display information to user
- Collect user input
- Format output (tables, messages, prompts)
- Handle input parsing errors
- NO business logic
- NO data validation beyond type conversion

**Business Logic Layer (task_manager.py)**:
- Enforce business rules
- Manage task lifecycle (CRUD)
- Validate task data (via Task class)
- Maintain data integrity
- NO user interaction
- NO output formatting

**Data Layer (task.py)**:
- Define data structure
- Validate individual field constraints
- Ensure data integrity
- NO storage management
- NO business rules

**Orchestration Layer (main.py)**:
- Connect CLI and TaskManager
- Control application flow
- Handle exit conditions
- NO direct data access
- NO UI rendering

### 5.2 Communication Patterns

**main.py → cli.py**:
- Call UI functions (display_menu, handle_add_task, etc.)
- Pass task_manager as parameter to action handlers
- Receive user choices and input

**cli.py → task_manager.py**:
- Call business methods (add_task, get_all_tasks, etc.)
- Pass user input (descriptions, IDs)
- Receive results or errors

**task_manager.py → task.py**:
- Create Task instances
- Pass field values
- Catch validation errors (ValueError)

**Return Flow**:
- task.py raises ValueError for invalid data
- task_manager.py catches errors, returns None or raises
- cli.py displays error messages to user
- main.py continues loop

### 5.3 Error Handling Boundaries

**Where errors are raised**:
- task.py: ValueError for invalid field values
- task_manager.py: Custom exceptions or None returns for business rule violations

**Where errors are caught**:
- cli.py: Catches exceptions from task_manager, displays user-friendly messages
- main.py: No exception handling (cli handles all)

**Where errors are displayed**:
- cli.py ONLY: All user-facing error messages

---

## 6. Error Handling Strategy

### 6.1 Error Categories

**Category 1: Input Type Errors**
- Non-numeric menu choice
- Non-numeric task ID
- Examples: "abc" when integer expected

**Strategy**:
- Use try/except around int() conversions
- Display: "✗ Error: Invalid input. Please enter a number."
- Return to prompt or menu

**Category 2: Input Range Errors**
- Menu choice out of range (e.g., 9 when valid is 1-6)
- Examples: 0, 7, -1

**Strategy**:
- Check range after successful int conversion
- Display: "✗ Error: Invalid choice. Please enter a number between 1 and 6."
- Return to prompt or menu

**Category 3: Data Validation Errors**
- Empty task description
- Task description too long (>500 chars)
- Examples: "", "   ", 501-character string

**Strategy**:
- Task class raises ValueError in __post_init__
- CLI catches ValueError, displays message
- Display: "✗ Error: Task description cannot be empty."
- Display: "✗ Error: Task description cannot exceed 500 characters."
- Return to prompt or menu

**Category 4: Business Logic Errors**
- Task ID not found
- Examples: ID 999 when only IDs 1-5 exist

**Strategy**:
- TaskManager returns None for get_task_by_id()
- CLI checks for None, displays error
- Display: "✗ Error: Task not found. Please enter a valid task ID."
- Return to prompt or menu

**Category 5: Confirmation Errors**
- Invalid yes/no response
- Examples: "maybe", "sure", "1"

**Strategy**:
- Check input against allowed values ('y', 'n')
- Display: "✗ Error: Invalid input. Please enter 'y' for yes or 'n' for no."
- Re-prompt until valid input

### 6.2 Error Handling Functions

**In cli.py**:

```python
def get_menu_choice() -> int | None:
    """Get and validate menu choice from user."""
    try:
        choice = int(input("Enter your choice (1-6): "))
        if 1 <= choice <= 6:
            return choice
        else:
            print("✗ Error: Invalid choice. Please enter a number between 1 and 6.")
            return None
    except ValueError:
        print("✗ Error: Invalid input. Please enter a number.")
        return None

def get_task_id() -> int | None:
    """Get and validate task ID from user."""
    try:
        return int(input("Enter task ID: "))
    except ValueError:
        print("✗ Error: Invalid input. Please enter a numeric task ID.")
        return None

def get_confirmation() -> bool:
    """Get yes/no confirmation from user."""
    while True:
        response = input("Are you sure? (y/n): ").lower()
        if response == 'y':
            return True
        elif response == 'n':
            return False
        else:
            print("✗ Error: Invalid input. Please enter 'y' for yes or 'n' for no.")
```

### 6.3 Error Recovery Strategy

**Non-Destructive Errors**:
- Invalid input → re-prompt
- Task not found → return to menu
- Empty description → re-prompt

**Destructive Action Protection**:
- Delete task → require confirmation
- If user says 'n' → cancel operation, return to menu

**No Data Loss**:
- All errors return to menu
- Task list remains unchanged on error
- No partial updates

---

## 7. Implementation Approach

### 7.1 Implementation Order

**Phase 1: Foundation**
1. Create project structure (directories)
2. Implement task.py (Task class with validation)
3. Write unit tests for Task class
4. Verify tests pass

**Phase 2: Business Logic**
5. Implement task_manager.py (CRUD operations)
6. Write unit tests for TaskManager
7. Verify tests pass

**Phase 3: User Interface**
8. Implement cli.py (menu, input/output functions)
9. Test CLI functions in isolation

**Phase 4: Integration**
10. Implement main.py (orchestration)
11. Write integration tests
12. Manual testing of complete flows

**Phase 5: Refinement**
13. Test all error cases
14. Verify all acceptance criteria
15. Write README.md
16. Final testing

**Rationale**: Bottom-up approach ensures each layer is solid before building the next

### 7.2 Testing Strategy

**Unit Tests**:
- test_task.py: Validate Task creation, validation logic
- test_task_manager.py: Validate CRUD operations, ID generation

**Integration Tests**:
- test_integration.py: Full user flows (add → view → update → delete → toggle)

**Manual Tests**:
- Run application and test each menu option
- Verify error messages for all error cases
- Verify empty list handling
- Verify confirmation prompts

**Coverage Goal**: ≥90% per specification

### 7.3 Development Constraints

**No Exploratory Coding**:
- Every function maps to specification requirement
- No "nice to have" features
- No refactoring beyond plan

**No External Dependencies**:
- Python 3.10+ standard library only
- No pip install required

**No Persistence**:
- No file writes
- No database connections
- All data in memory only

---

## 8. Detailed Module Specifications

### 8.1 task.py

**Purpose**: Define Task data model

**Classes**:
```python
@dataclass
class Task:
    id: int
    description: str
    is_complete: bool = False

    def __post_init__(self) -> None:
        """Validate task fields after initialization."""
        # Validate description
        if not self.description or not self.description.strip():
            raise ValueError("Task description cannot be empty")
        if len(self.description) > 500:
            raise ValueError("Task description cannot exceed 500 characters")

        # Trim description
        self.description = self.description.strip()
```

**Exports**: Task class

**Dependencies**: dataclasses (standard library)

### 8.2 task_manager.py

**Purpose**: Business logic and storage

**Module Variables**:
```python
tasks: list[Task] = []
_next_id: int = 1
```

**Functions**:
```python
def add_task(description: str) -> Task:
    """Add new task, return created task."""

def get_all_tasks() -> list[Task]:
    """Return all tasks."""

def get_task_by_id(task_id: int) -> Task | None:
    """Return task by ID or None if not found."""

def update_task(task_id: int, new_description: str) -> Task | None:
    """Update task description, return updated task or None."""

def delete_task(task_id: int) -> bool:
    """Delete task, return True if deleted, False if not found."""

def toggle_complete(task_id: int) -> Task | None:
    """Toggle task completion status, return updated task or None."""
```

**Exports**: All functions above

**Dependencies**: task.py

### 8.3 cli.py

**Purpose**: User interface

**Functions**:
```python
def display_menu() -> None:
    """Display main menu."""

def get_menu_choice() -> int | None:
    """Get and validate menu choice."""

def handle_add_task(task_manager) -> None:
    """Handle add task flow."""

def handle_view_tasks(task_manager) -> None:
    """Handle view tasks flow."""

def handle_update_task(task_manager) -> None:
    """Handle update task flow."""

def handle_delete_task(task_manager) -> None:
    """Handle delete task flow."""

def handle_toggle_complete(task_manager) -> None:
    """Handle toggle complete flow."""

def display_exit_message() -> None:
    """Display goodbye message."""

def format_task_table(tasks: list[Task]) -> str:
    """Format tasks as table string."""

def get_task_id() -> int | None:
    """Get and validate task ID."""

def get_confirmation() -> bool:
    """Get yes/no confirmation."""
```

**Exports**: All functions above

**Dependencies**: task_manager.py (indirectly via parameter)

### 8.4 main.py

**Purpose**: Application orchestration

**Functions**:
```python
def main() -> None:
    """Main application loop."""
    while True:
        cli.display_menu()
        choice = cli.get_menu_choice()

        if choice is None:
            continue

        if choice == 1:
            cli.handle_add_task(task_manager)
        elif choice == 2:
            cli.handle_view_tasks(task_manager)
        elif choice == 3:
            cli.handle_update_task(task_manager)
        elif choice == 4:
            cli.handle_delete_task(task_manager)
        elif choice == 5:
            cli.handle_toggle_complete(task_manager)
        elif choice == 6:
            cli.display_exit_message()
            break

if __name__ == "__main__":
    main()
```

**Exports**: None (entry point)

**Dependencies**: cli.py, task_manager.py

---

## 9. Non-Functional Implementation Details

### 9.1 Performance Considerations

**Target**: All operations < 100ms (per spec)

**Strategy**:
- Linear search acceptable for ≤1000 tasks
- No premature optimization
- Simple algorithms sufficient

### 9.2 Code Quality Standards

**Type Hints**:
- Required on all function signatures
- Use `list[Task]`, `int | None`, etc.
- Python 3.10+ syntax

**Docstrings**:
- Required on all public functions and classes
- Format: Brief one-line summary
- Include parameter and return types if not obvious

**PEP 8 Compliance**:
- 4 spaces for indentation
- 2 blank lines between top-level definitions
- Max line length: 100 characters
- snake_case for functions and variables

### 9.3 Output Formatting

**Success Messages**:
- Start with ✓ symbol
- Green color if terminal supports it (optional)
- Example: "✓ Task added successfully!"

**Error Messages**:
- Start with ✗ symbol
- Red color if terminal supports it (optional)
- Example: "✗ Error: Task not found."

**Tables**:
- Use simple ASCII borders (|, -)
- Align columns with fixed widths
- Example from spec Flow 2

---

## 10. Acceptance Criteria Mapping

**Specification → Plan Mapping**:

| Spec Requirement | Plan Section |
|-----------------|--------------|
| US-1: Add Task | §8.2 add_task(), §8.3 handle_add_task() |
| US-2: View Tasks | §8.2 get_all_tasks(), §8.3 handle_view_tasks() |
| US-3: Update Task | §8.2 update_task(), §8.3 handle_update_task() |
| US-4: Delete Task | §8.2 delete_task(), §8.3 handle_delete_task() |
| US-5: Toggle Complete | §8.2 toggle_complete(), §8.3 handle_toggle_complete() |
| Task Data Model | §8.1 Task class |
| CLI Menu Flow | §4.1 Main Loop Structure |
| Error Handling | §6 Error Handling Strategy |
| In-Memory Storage | §2.1 Task Storage |
| ID Generation | §3.1 ID Generation Algorithm |

**All specification requirements mapped**: ✓

---

## 11. Risks and Mitigations

### Risk 1: Invalid Input Crashes Application
**Mitigation**: Comprehensive try/except blocks in CLI layer

### Risk 2: ID Collision After Many Adds/Deletes
**Mitigation**: Never reuse IDs (counter never decrements)

### Risk 3: Poor UX with Long Task Lists
**Mitigation**: Specification limits scope to 1000 tasks; linear display acceptable

### Risk 4: Test Coverage Below 90%
**Mitigation**: Write tests alongside implementation; verify coverage before completion

---

## 12. Plan Approval Checklist

- [ ] All specification requirements addressed
- [ ] No new features introduced
- [ ] Constitutional compliance verified
- [ ] Clear module boundaries defined
- [ ] Error handling strategy complete
- [ ] Testing strategy defined
- [ ] Implementation order specified
- [ ] All constraints respected (no DB, no files, etc.)

---

## Approval

**Plan Status**: APPROVED
**Approval Date**: 2026-01-03
**Approved By**: Project Authority
**Specification Compliance**: VERIFIED
**Constitutional Violations**: NONE
**New Features Introduced**: NONE

---

**This plan is now APPROVED and ready for task breakdown.**

---

## Version History

| Version | Date       | Author | Changes                    |
|---------|------------|--------|----------------------------|
| 1.0.0   | 2026-01-03 | System | Initial implementation plan |
