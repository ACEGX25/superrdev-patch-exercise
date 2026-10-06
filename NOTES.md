# NOTES

## Summary of changes
1. **SQL grouping bug** (JPA @Query, db/queries/search_tasks.sql, Oracle package x2): missing parentheses let archived and wrong-status rows through and inflated the total. Grouped the title/description match.
2. **Artificial Thread.sleep** in TaskController: any query under 10 characters slept up to 1s, tying up request threads. Removed it.
3. **page/pageSize validation**: page=0 caused a negative subList index (500). Clamped page to 1-10000 and pageSize to 1-100, which also avoids int overflow.
4. **Invalid status**: ?status=foo returned 500. Now returns 400 with a JSON error.
5. **useTasks.js**: loading never reset after an error, so the UI hung on "Loading".
6. **App.jsx**: page now resets to 1 when search or status changes.

## What I chose not to change
- Service layer, Lombok, hardcoded CORS: design preferences, and this is a patch, not a rewrite.
- Search debounce / request cancellation: real race condition, but ran out of time.
- TaskTable null-status check: column is NOT NULL, low risk.

## Biggest remaining risk
Pagination is done in memory: the controller loads every matching task, then slices with subList. This slows down as the table grows. It should be pushed into the database (Pageable, or the ROWNUM approach in the Oracle package).

## Tools and AI used
Used Antigravity to scan the codebase read-only and Claude to quiz me on each finding. I reproduced a few of them myself before fixing eg. found the SQL bug by reading the query, and the page=0 crash myself. The key stategy I used was to use two AI agents, antigravity found out the bugs, I tested myself with claude on the same, later when either of them showed a roadmap or path to fix things I held it up against one another, giving optimum ways to find undiscovered bugs or pointing out mistakes in given approaches.
