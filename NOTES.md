# Notes & Findings

## Summary of Changes

1. **Fixed SQL Operator Precedence in Search Queries** (`TaskRepository.java`, `db/queries/search_tasks.sql`, `db/oracle/task_search_package.sql`):
   - Grouped `LOWER(title) LIKE :term OR LOWER(description) LIKE :term` with parentheses before applying the `AND archived = FALSE` and `AND (:status IS NULL OR status = :status)` conditions.
   - Fixed data corruption where archived records and non-matching status records leaked into search results whenever a keyword matched `title`.

2. **Eliminated Artificial Request Latency & Controller Sleep** (`TaskController.java`):
   - Removed artificial `Thread.sleep(queryWeight)` delay (up to 1,000ms on short queries) which degraded search response times and tied up servlet request threads.

3. **Resolved Race Conditions & Error State Cleanup** (`frontend/src/hooks/useTasks.js`):
   - Added cancellation/ignore flag in `useEffect` cleanup to guarantee responses arriving out-of-order do not overwrite latest search results.
   - Cleared stale `error` state upon new requests and set `loading` state to false properly on catch.

4. **Added Page Reset on Filter / Search Change** (`frontend/src/App.jsx`):
   - Reset current pagination `page` back to `1` whenever query search or status filter is modified, preventing empty / out-of-bound pages.

---

## What Was Not Changed & Why

- **Database-level Pagination (Spring Data `Pageable`)**: The endpoint currently fetches matching records into memory and slices via `.subList()`. For larger datasets, migrating `TaskRepository` to use `org.springframework.data.domain.Pageable` is recommended. However, to preserve existing repository and query signatures and keep the diff small and focused within the exercise scope, the current memory slicing was maintained.
- **Debounced Input Hook**: Considered debouncing keystrokes in `SearchBar.jsx`, but with the backend delay removed, the local in-memory queries execute instantly (<5ms) and the `useTasks` cleanup flag handles in-flight request interleaving cleanly without adding third-party dependencies.
- **Oracle PL/SQL Execution**: The package in `db/oracle/` is a reference artifact that does not execute in the local H2 setup; the matching boolean precedence fix was applied directly to maintain consistency with the Java implementation.

---

## Biggest Remaining Risks

1. **In-Memory Slicing vs. DB-Level Paging**: As the task dataset scales into tens or hundreds of thousands of rows, `searchTasks` fetching the entire unpaginated result set into memory will cause heap pressure and high query latency.
2. **LIKE '%term%' Wildcard Performance**: Leading-wildcard searches cannot utilize standard B-tree indexes, resulting in full table scans. Full-text search or inverted indexing would be necessary for production scale.
3. **Task Status Validation / 500 on Unknown Enum**: `TaskStatus.valueOf(...)` will throw an unhandled `IllegalArgumentException` (HTTP 500) if an invalid status query parameter is passed instead of returning a clean HTTP 400 Bad Request.

---

## Tools & AI Usage

- **Antigravity AI Assistant**: Used to inspect the full-stack architecture, isolate SQL boolean operator precedence defects across Java, SQL, and Oracle reference layers, identify the synthetic thread sleep latency, and implement clean React async cleanup semantics.
