# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py`. Searched the entire codebase for `save_to_watchlist` (project-wide grep, not just editor search) to confirm every call site was caught — found exactly one, in `routes/watchlist/watchlist.py`, and updated it to call `add_to_watchlist`.
**How I verified:** Ran `grep -rn "save_to_watchlist" .` from the repo root after the rename — zero matches remained, confirming no stale references. Also ran the full test suite (`pytest tests/ -v`) to confirm nothing broke from the rename.

## Comment 2 — Deduplication
**What I did:** Added a duplicate check to `add_to_watchlist()` in `services/watchlist_service.py`, following the exact pattern used in `add_to_collection()` (`services/collection_service.py`): query for an existing `WatchlistEntry` matching `user_id` and `film_id` before creating a new one, and raise if found. Since no watchlist-specific exception existed yet, I defined a new `AlreadyInWatchlistError` class in `watchlist_service.py` (mirroring `AlreadyInCollectionError`), rather than reusing the collection's exception — a duplicate watchlist entry is a different condition from a duplicate collection entry, and reusing the wrong exception would produce a misleading error message ("already in this user's collection" when the user was actually adding to their watchlist).
**How I verified:** Initially made two mistakes while implementing this — queried `CollectionEntry` instead of `WatchlistEntry` (so the check silently checked the wrong table), and raised the wrong exception class. Ran `pytest tests/ -v` after each fix; caught the `CollectionEntry`/`WatchlistEntry` mix-up because the existing collection tests broke on import, which surfaced the bug before it reached watchlist-specific tests. Manually confirmed the fix by tracing that the query now filters on `WatchlistEntry.user_id` and `WatchlistEntry.film_id`, matching the collection service's pattern one-for-one.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` and wrote `test_add_to_watchlist_nonexistent_film_raises`, modeled directly on `test_add_to_collection_nonexistent_film_raises` from `tests/test_collection.py`. Reused the same `app` and `sample_user` fixture structure (redeclared locally since the project has no shared `conftest.py`), used the same fake-UUID pattern (`"00000000-0000-0000-0000-000000000000"`) for a film_id guaranteed not to exist, and asserted `FilmNotFoundError` is raised via `pytest.raises`, same as the collection version.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` to confirm the new test passes in isolation, then `pytest tests/ -v` to confirm the full suite (including the pre-existing collection tests) still passes with no regressions.


## Comment 4 — Default visibility

**My position:**
I think `public=True` should remain the default.

**Reasoning:**
CineLog is designed as a social platform where users can share and discover movies with friends and the community. A public watchlist allows others to see what someone plans to watch, making it easier to give recommendations, discuss upcoming films, or even plan to watch movies together. Since many users do not change default settings, making watchlists private by default would likely reduce these social interactions and make the feature less useful.

**Tradeoff acknowledged:**
I recognize that a watchlist can reveal a user's future viewing intentions, which may feel more personal than sharing movies they have already watched. Some users may not want others to see certain films on their watchlist. However, I believe the social benefits outweigh this concern because CineLog's primary purpose is community engagement. Users who prefer more privacy can still change their watchlist visibility after creating it.

## Comment 5 — Sort order

**My position:**
I think the watchlist should be sorted by `date_added` (newest first).

**Reasoning:**
Sorting by date added makes it easier for users to find movies they recently discovered and were excited enough to save. It also creates consistency with the collection feature, which already displays entries by `date_added` in descending order. Using the same sorting behavior across both features makes the application more predictable and easier to use.

**Engagement with reviewer's point:**
While alphabetical order is useful for browsing a large list, I think date-added is more practical for a watchlist because users often decide what to watch based on what they have recently discovered or added. The newest entries usually represent their current interests, so showing those first better supports the purpose of a watchlist.


## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->


