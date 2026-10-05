# Summary of Changes
- **JPA & SQL Query Precedence:** Fast-fixed the `AND/OR` operator precedence bug in both `TaskRepository.java` and `task_search_package.sql` which was incorrectly causing the `archived = FALSE` condition to be bypassed when a task's description matched the search term.
- **Backend Performance:** Removed the intentional `Thread.sleep` in `TaskController.java` that was artificially delaying short search queries.
- **Frontend Race Conditions & Debouncing:** Introduced a debounce mechanism (300ms) with an abort flag (`ignore=true`) in `useTasks.js` to handle race conditions when rapidly typing in the search bar. This prevents old responses from overwriting new state. Error states are also correctly cleared now.
- **Pagination Bug:** Modifying the search query or status filter now correctly resets the pagination offset (`page` state) to 1 in `App.jsx`, preventing users from getting stuck in empty out-of-bounds pagination pages.

# What I Chose Not to Change and Why
- **CORS configuration in Backend**: `TaskController.java` has `@CrossOrigin(origins = "http://localhost:5173")`. While strictly not needed since the Vite development server proxies requests to the backend, it acts as a safeguard directly for the browser, so I left it in place to minimize unnecessary diffs.
- **Full AbortController implementation**: I used a simple flag `ignore = false` inside `useTasks` for cleanup rather than `AbortController`. It perfectly avoids state mutation on an unmounted component or canceled request and keeps the code lightweight without touching `fetchTasks`. 

# The Biggest Remaining Risk
The most critical remaining risk is an unindexed `LIKE` query over unstructured, growing text (`LOWER(title) LIKE %term% OR LOWER(description) LIKE %term%`). As the table scales, scanning wildcard characters iteratively via sequential scans degrades runtime performance massively. A robust full-text search engine or simply TSV vector indexes in Postgres should be adopted.

# Tools Used
Used Claude 3.5 Sonnet to quickly inspect files, navigate the source, and patch the simple logic errors directly in a single context window.
