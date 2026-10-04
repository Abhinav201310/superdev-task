# Handwritten Explanations

Please place clear photos / scans of your handwritten notes in this folder before pushing your submission.

### Guide for what to write on paper:

For each bug found and fixed:
1. **Location**: File path, line number, layer (SQL, Backend, Frontend).
2. **Discovery**: How it was discovered (e.g. searching "api" or "decommission" returned archived tasks, inspecting query logs, profiling network requests).
3. **Root Cause**: What caused the defect (e.g. `AND` precedence over `OR` in SQL WHERE clause; `Thread.sleep` artificially blocking request threads; React async effect race conditions; pagination state desync).
4. **Fix & Rationale**: What was modified and why this minimal, high-impact patch was chosen.

---

### Suggested Breakdown for Handwritten Sheets:

- **Sheet 1: Bug 1 - SQL Operator Precedence in Search**
  - *Location*: `TaskRepository.java`, `db/queries/search_tasks.sql`, `db/oracle/task_search_package.sql`
  - *Root cause*: `AND` binds tighter than `OR`. Condition evaluated as `(archived = FALSE AND title LIKE :term) OR (desc LIKE :term AND status = :status)`.
  - *Fix*: Wrap `(LOWER(title) LIKE :term OR LOWER(description) LIKE :term)` in parentheses.

- **Sheet 2: Bug 2 - Artificial Thread Sleep / Query Delay**
  - *Location*: `backend/.../TaskController.java`
  - *Root cause*: `Thread.sleep(complexityScore * 100L)` added arbitrary 1000ms delay for short queries.
  - *Fix*: Remove `Thread.sleep` block.

- **Sheet 3: Bug 3 - Async Race Conditions & Error Reset in React Hook**
  - *Location*: `frontend/src/hooks/useTasks.js`
  - *Root cause*: Out-of-order fetch responses overwriting current state; stale error state not cleared on new search.
  - *Fix*: Add `ignore` cancellation flag in `useEffect` cleanup and clear error on request trigger.

- **Sheet 4: Improvement - Reset Page on Search / Filter Change**
  - *Location*: `frontend/src/App.jsx`
  - *Root cause*: Changing filter or query while on page > 1 leads to empty / invalid page states.
  - *Fix*: Call `setPage(1)` inside `onChange` handlers for SearchBar and StatusFilter.
