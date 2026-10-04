# Handwritten Notes: What to Write

Copy these by hand onto paper (about 1 page per 2-3 bugs), photograph or scan the pages, and put the images in `handwritten/`.
Put your name and the date at the top of page 1. Use your own words wherever you can. They may ask you to explain each bug on the call.

Format for every bug: **Where / How I found it / Cause / Fix / Why it matters**

---

## BACKEND AND SQL

### Bug 1: SQL precedence (most important)
- **Where:** `TaskRepository.java` (search query). Same mistake in `db/queries/search_tasks.sql` and `db/oracle/task_search_package.sql`.
- **How I found it:** read the WHERE clause, and saw that the data has archived tasks which matched "api" but still appeared.
- **Cause:** `AND` runs before `OR`. The query was `archived=FALSE AND title LIKE ... OR description LIKE ... AND status...`. So it meant (not archived AND title match) OR (description match AND status).
- **Fix:** put brackets around the OR: `archived=FALSE AND (title LIKE OR description LIKE) AND (status filter)`.
- **Why:** archived tasks leaked into results, the status filter was ignored for title matches, and the total count was wrong.

### Bug 2: Thread.sleep in controller
- **Where:** `TaskController.java`.
- **How I found it:** read the controller. A sleep was hidden behind a comment saying "for logging".
- **Cause:** the code slept up to 1 second, longest for short queries.
- **Fix:** deleted the sleep.
- **Why:** every search was slow for no reason, and it made the frontend race condition (Bug 7) worse.

### Bug 3: Invalid status gave a 500 error
- **Where:** `TaskController.java`.
- **How I found it:** saw `TaskStatus.valueOf(status)` with no error handling.
- **Cause:** an unknown value like `status=foo` throws `IllegalArgumentException`, which becomes a server error.
- **Fix:** catch the exception and return 400 Bad Request with a message. Also accept lowercase.
- **Why:** bad input from the user is a client error (400), not a server crash (500).






---

## FRONTEND

### Bug 1: Race condition
- **Where:** `frontend/src/hooks/useTasks.js`.
- **How I found it:** saw the effect had no cleanup function.
- **Cause:** if two requests are sent, the slower old one can finish last and overwrite the newer results.
- **Fix:** `let ignore = false` in the effect, set to `true` in the cleanup. Responses return early if `ignore` is true.
- **Why:** the screen could show results for the wrong search.

### Bug 2: Stuck loading and stale error
- **Where:** `frontend/src/hooks/useTasks.js`.
- **How I found it:** noticed `setLoading(false)` was only in `.then`, and `error` was never reset.
- **Cause:** after a failed request, loading stayed true, so the user saw "Loading..." forever. An old error stayed after a later success.
- **Fix:** `setError(null)` at the start, and `setLoading(false)` inside `.finally(...)`.
- **Why:** the user should see the real error and recover when a request works again.

### Bug 3: Page not reset when filter changes
- **Where:** `frontend/src/App.jsx`.
- **How I found it:** thought through: go to page 3, then search for something with one page of results.
- **Cause:** `page` stayed at 3, so the app asked for page 3 of a 1-page result and showed an empty table ("Page 3 of 1").
- **Fix:** call `setPage(1)` when the status changes (`handleStatusChange`) and when the search changes.
- **Why:** a new filter should always start at the first page.

### Bug 4: No debounce on search
- **Where:** `frontend/src/App.jsx`.
- **How I found it:** saw the input was connected directly to the value that triggers requests.
- **Cause:** every keystroke sent a request. Typing "api" sent 3.
- **Fix:** keep the typed text in `searchInput`, and copy it to `query` after 300 ms using `setTimeout`, with `clearTimeout` in the cleanup.
- **Why:** fewer requests, less load, and fewer race conditions.

---


