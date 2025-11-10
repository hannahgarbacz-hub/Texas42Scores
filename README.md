# Texas 42 Tournament Scorer — index.html

This project was developed with the assistance of an AI coding assistant. It is a single-file web application for scoring Texas 42 (dominoes) tournaments: create rosters, run deterministic round-robin schedules, record hands and games, and export results. The project is provided as open-source for non-commercial use; you are free to copy and modify it provided you give attribution to the original source and do not use it for commercial/profit purposes.

This README describes the structure of the single-page application contained in `index.html`.
# Texas 42 Tournament Scorer — index.html

This README describes the structure of the single-file SPA in `index.html`. It documents the UI views, important element IDs/classes, the embedded JavaScript API (the `TournamentScorer` class), data shapes, and quick usage notes.

## Purpose

`index.html` is a self-contained tournament scorer for Texas 42 (dominoes). It uses embedded CSS and JavaScript, stores tournament state in `localStorage`, and allows exporting results as CSV. No build or server is required — open the file in a modern browser.

## Top-level structure

- `<head>`: meta tags, page `<title>`, and a single embedded `<style>` block. The CSS defines theme variables, layout helpers (`.grid`, `.grid-2`), component styles (`.card`, `.stat-card`), responsive rules, and utility classes like `.view`, `.active`, and `.hidden`.
- `<body>`: contains the top navigation `.nav-bar`, a `.container` with multiple view sections, and a fixed `.action-buttons` bar at the bottom for global actions (clear/export).

Views are simple `<div id="..." class="view">` sections and are shown/hidden by toggling the `.active` class.

## Main views (IDs)

- `#setup` — Tournament setup (default view)
  - `#team-input` (textarea): paste or type team lines (format: "Player A & Player B" or "A and B").
  - `#num-rounds` (number): rounds (5–15).
  - Initialize button → `app.initializeTournament()`.

- `#leaderboard` — Leaderboard view
  - `#leaderboard-content` populated by `app.updateAllViews()` with `.leaderboard-entry` rows.
  - Columns now include: Rank, Team (Players), W-L, Marks Won, Marks Lost, Differential (Marks Won - Marks Lost), and Games Played. `updateAllViews()` computes marks won/lost by aggregating `game.marks` and uses wins, then differential, then marks won as tiebreakers when sorting the leaderboard.

- `#schedule` — Round schedule
  - `#round-select` (select), onchange → `app.displaySchedule()`.
  - `#schedule-content` shows matchups and status.

- `#table` — Table Scorekeeper (game play)
  - `#game-select` (select) to pick in-progress games; onchange → `app.loadGame()`.
  - `#new-game-form`: form to start a new game (`#first-bidder-select-new`, `#second-bidder-select-new`, `#new-game-round`, `#new-game-table`).
  - `#current-game-info`: shows team names (`#team1-name`, `#team2-name`), marks (`#team1-marks`, `#team2-marks`), next dealer (`#next-dealer`), and next bidder (`#next-bidder`).
  - `#hand-entry-card`: bidding UI. Important elements created/used here:
    - `#hand-number` — current hand index
    - `#sequential-bidding-section` — contains `#current-bidder-display`, `#bid-history-display`, `#current-bid-options` and `Record Bid` / `Cancel Hand` buttons.
    - A `Start New Hand` button (id `start-hand-btn`) is dynamically inserted by JS when a hand can be started. This is new/important: `checkForHandStart()` manages showing or hiding the start button.
  - `#bid-result-section` — shown after bidding to choose result (Made/Set) and complete the hand.
  - `#hand-history-card` with `#hand-history` lists completed hands and buttons for `Complete Game` and `Undo Last Hand`.

- `#stats` — Stats and anomaly detection
  - `#anomaly-content` — shows flagged anomalies.
  - `#stats-team-select` and `#team-stats-content` — show per-team statistics.

- Fixed bottom controls: `.action-buttons` (outside `.container`) with `Clear All Data` (`app.clearTournament()`) and `Export to CSV` (`app.exportCSV()`).

## Notable new/changed behaviors in the latest `index.html`

- The Table view now inserts a `Start New Hand` button when a hand can be started; `checkForHandStart()` shows/hides this button. This creates a two-step hand flow: first click "Start New Hand" to open the bidding UI, then use `Record Bid` / `Complete Hand` to finalize.
- There is a `Cancel Hand` button (calls `app.cancelHand()`) which discards current bids and restores the UI. `cancelHand()` performs a confirmation and hides bidding UI.
- `finalizeBidding()` and `completeHand()` now call `checkForHandStart()` at appropriate times to re-enable the start button for future hands.
- Bidding options include points (30–42), a special `84` entry (treated as 2 marks), and "N marks" entries ("2 marks", "3 marks", ...).

Club Night (Roster & Games)
 - Roster size: Club Night rosters can now contain up to 25 players. The roster textarea (`#club-players-input`) shows a live counter (`#club-player-count`) updated on input and when a roster is loaded. If the roster exceeds 25 players the Save button (`#save-club-roster-btn`) is disabled and saving is blocked.
 - Roster storage: rosters are persisted in localStorage under the key `clubRosters` and are stored keyed by a generated unique id (so saving a roster with the same display name will not overwrite an existing roster). When a duplicate display name is detected the app appends a numeric suffix to make the displayed name unique (for example "My Club (2)").
 - Club game storage: recorded club games are saved under `clubGames_<rosterId>` where `<rosterId>` is the id assigned when the roster is saved. Each club game object follows the shape: `{ id, team1: [p1,p2], team2: [p3,p4], marks: { team1: n, team2: m }, timestamp }`.
 - UI elements (Club Night): `#club-name`, `#club-players-input`, `#club-player-count`, `#club-permanent`, `#save-club-roster-btn`, `#club-roster-select`, `#club-team1-p1`, `#club-team1-p2`, `#club-team2-p1`, `#club-team2-p2`, `#club-team1-marks`, `#club-team2-marks`, and `#club-games` (history list).
 - Actions: `Save Roster` (persist roster and load any saved club games), `Load saved roster` (populates the players and counter), `Record Club Game` (validate 4 unique players and save the game), and `Clear Club Games` (confirmation then remove saved games for the loaded roster).
 - Actions: `Save Roster` (persist roster and load any saved club games), `Load saved roster` (populates the players and counter), `Record Club Game` (validate 4 unique players and save the game), and `Clear Club Games` (confirmation then remove saved games for the loaded roster).
 - Management: the roster selector has `Rename` and `Delete` controls to rename a saved roster (name uniqueness is enforced) or delete a roster and its saved club games from localStorage.
 - Integration note: club games are stored separately from the tournament model by default. If you want club games to contribute to tournament/team/player statistics, I can add an import or automatic merge step.

# Texas 42 Tournament Scorer — index.html

This README documents the single-file SPA in `index.html`. It covers the UI views, element IDs/classes the JavaScript API (`TournamentScorer`), data shapes, and the recent features added: deterministic round-robin scheduling (single/double), an explicit rounds confirmation dialog, and per-player statistics.

## Purpose

`index.html` is a self-contained tournament scorer for Texas 42 (dominoes). It uses embedded CSS and JavaScript, stores tournament state in `localStorage`, and enables exporting results as CSV. No build or server is required — open the file in a modern browser.

## Top-level structure

- `<head>`: meta tags, page `<title>`, and a single embedded `<style>` block. The CSS defines theme variables, layout helpers (`.grid`, `.grid-2`), component styles (`.card`, `.stat-card`), responsive rules, and utility classes like `.view`, `.active`, and `.hidden`.
- `<body>`: contains the top navigation `.nav-bar`, a `.container` with multiple view sections, and a fixed `.action-buttons` bar at the bottom for global actions (clear/export).

Views are simple `<div id="..." class="view">` sections and are shown/hidden by toggling the `.active` class.

## Main views (IDs)

- `#setup` — Tournament setup (default view)
  - `#team-input` (textarea): paste or type team lines (format: "Player A & Player B" or "A and B").
  - `#num-rounds` (number): rounds (5–15).
  - `#round-robin-type` (select): choose `single` or `double` round-robin. Single means every pair meets once (N-1 rounds). Double means each pair meets twice (2*(N-1) rounds).
  - Initialize button → `app.initializeTournament()`.

- `#leaderboard` — Leaderboard view
  - `#leaderboard-content` populated by `app.updateAllViews()` with `.leaderboard-entry` rows.

- `#schedule` — Round schedule
  - `#round-select` (select), onchange → `app.displaySchedule()`.
  - `#schedule-content` shows matchups and status.

- `#table` — Table Scorekeeper (game play)
  - `#game-select` (select) to pick in-progress games; onchange → `app.loadGame()`.
  - `#new-game-form`: form to start a new game (`#first-bidder-select-new`, `#second-bidder-select-new`, `#new-game-round`, `#new-game-table`).
  - `#current-game-info`: shows team names (`#team1-name`, `#team2-name`), marks (`#team1-marks`, `#team2-marks`), next dealer (`#next-dealer`), and next bidder (`#next-bidder`).
  - `#hand-entry-card`: bidding UI. Important elements created/used here:
    - `#hand-number` — current hand index
    - `#sequential-bidding-section` — contains `#current-bidder-display`, `#bid-history-display`, `#current-bid-options` and `Record Bid` / `Cancel Hand` buttons.
    - A `Start New Hand` button (id `start-hand-btn`) is dynamically inserted by JS when a hand can be started. `checkForHandStart()` manages showing/hiding it.
  - `#bid-result-section` — shown after bidding to choose result (Made/Set) and complete the hand.
  - `#hand-history-card` with `#hand-history` lists completed hands and buttons for `Complete Game` and `Undo Last Hand`.

- `#stats` — Stats and anomaly detection
  - `#anomaly-content` — shows flagged anomalies.
  - `#stats-team-select` and `#team-stats-content` — show per-team statistics.
  - `#stats-player-select` and `#player-stats-content` — show per-player (individual) statistics.

- Fixed bottom controls: `.action-buttons` (outside `.container`) with `Clear All Data` (`app.clearTournament()`) and `Export to CSV` (`app.exportCSV()`).

## Quick suggestions (optional follow-ups)

- Extract inline CSS and JS into `styles.css` and `app.js` to improve maintainability and enable caching.
- Add a small seed script (e.g., `seed-tournament.js`) that populates `localStorage` with a sample `tournament` for manual QA.
- Disambiguate players with unique IDs (teamId+playerIndex) to support duplicate names and more reliable player-level stats.

---

If you'd like, I can now:
- extract the JS/CSS into separate files and update `index.html`,
- create a small seed script that populates a sample tournament in `localStorage` for testing,
- or add a JSON example of the `tournament` object to the README.

Tell me which follow-up you'd like and I will implement it.