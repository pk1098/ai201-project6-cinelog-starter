# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py`. I also updated the import and function call in `routes/watchlist/watchlist.py`.

**How I verified:** Searched the project for both function names. The new name appears in the service definition, route import, and route call, and no references to `save_to_watchlist` remain.

## Comment 2 — Deduplication
**What I did:** Added `AlreadyInWatchlistError` and updated `add_to_watchlist()` to query for an existing entry with the same `user_id` and `film_id`. The function now raises this exception instead of inserting a duplicate, following the pattern used by `add_to_collection()`.

**How I verified:** Compared the implementation with the collection service's deduplication flow and confirmed that the duplicate check runs before the new entry is added and committed. I could not run automated tests because the configured local Python executable was inaccessible.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` with `test_add_to_watchlist_nonexistent_film_raises`. It uses an isolated in-memory database and a sample-user fixture, then verifies that an unknown film ID raises `FilmNotFoundError`.

**How I verified:** Matched the fixture and assertion structure of `test_add_to_collection_nonexistent_film_raises` in `tests/test_collection.py`. I could not execute the test suite because the configured local Python executable was inaccessible.

## Comment 4 — Default visibility
**My position:** Watchlist entries should default to public because CineLog is designed as a community film-tracking application where users share film activity.

**Reasoning:** This makes discovery and recommendations easier without requiring users to enable sharing for every entry.

**Tradeoff acknowledged:** Public-by-default provides less privacy, so the UI should clearly communicate visibility and allow users to choose private visibility.

## Comment 5 — Sort order
**My position:** I would sort watchlist entries by `date_added` descending.

**Reasoning:** Users are most likely to return to recently saved movies.

**Engagement with reviewer's point:** Alphabetical sorting is predictable, but it does not reflect how users generally interact with a watchlist.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
