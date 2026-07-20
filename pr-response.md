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
**What conflicted:** The main branch refactored `Film.id` and related foreign keys from integers to UUID strings in `models.py`, while the feature branch added `WatchlistEntry` using the earlier model structure. The initial resolution retained the UUID refactor but dropped `WatchlistEntry`. The rebased app also attempted to import `watchlist_bp` from the empty `routes.watchlist` package instead of the module where the blueprint is defined.

**How I resolved it:** Restored `WatchlistEntry` using a UUID-compatible `film_id`, added its relationships to `User` and `Film`, and retained the uniqueness constraint for each user/film pair. I also moved the blueprint import into `create_app()` and imported it from `routes.watchlist.watchlist`, preserving the established application-factory pattern and avoiding a circular import.

**How I verified no conflict remains:** Confirmed that no Git conflict markers remain, all watchlist model and service imports resolve to their current locations, and the model's foreign-key type matches the UUID type used by `Film.id`. I also confirmed that the application registers the watchlist blueprint from the correct module. The full automated test suite should be run in an environment where `pytest` is available.

## PR Description
This PR adds watchlist support to CineLog. Users can save films they plan to watch and retrieve their saved films through the `/watchlist` endpoints. The implementation includes a `WatchlistEntry` model, watchlist service functions, Flask routes, film-existence validation, and duplicate-entry protection.

The service function follows the project's `verb_to_noun` naming convention as `add_to_watchlist()`. Duplicate additions raise `AlreadyInWatchlistError`, matching the collection service's error-handling pattern. Watchlist entries default to public because CineLog is a community film-tracking application, although the visibility choice should be clearly communicated and a private option should be exposed to users. For display order, recently added films would better reflect typical watchlist use than alphabetical ordering; changing the current query to `date_added` descending is the preferred follow-up.

The branch was rebased onto the UUID-based film model. `WatchlistEntry.film_id` therefore uses the same UUID string type as `Film.id`, and the watchlist blueprint is imported from its nested route module.

Manual verification steps:

1. Run `pytest tests/` and confirm the collection and watchlist tests pass.
2. Start the app with `python app.py`.
3. Send `POST /watchlist/<user_id>/add` with a valid `film_id` and confirm it returns `201` with the created entry.
4. Repeat the same request and confirm the duplicate is rejected.
5. Send the request with a nonexistent film ID and confirm it is handled as `FilmNotFoundError`.
6. Send `GET /watchlist/<user_id>` and confirm the saved film and watchlist metadata are returned.
