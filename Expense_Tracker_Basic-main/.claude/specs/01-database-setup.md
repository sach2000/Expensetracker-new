# Step 01 — Database Setup

## 1. Overview

- Step 01 replaces the stub functions in `database/db.py` with a working SQLite implementation
- This is the foundational step — every subsequent feature depends on the database being ready
- Authentication (registration and login), the profile page, and all expense tracking routes cannot function until the tables exist and a connection can be opened
- This step also seeds one demo user and 8 sample expenses so the app has data to display immediately after startup

## 2. Depends on

- None — this is the first step with no prerequisites

## 3. Routes

- No new routes
- Existing placeholder routes in `app.py` remain unchanged

## 4. Database Schema

### Table A: `users`

| Column | SQLite Type | Constraints |
|---|---|---|
| `id` | `INTEGER` | PRIMARY KEY AUTOINCREMENT |
| `name` | `TEXT` | NOT NULL |
| `email` | `TEXT` | UNIQUE NOT NULL |
| `password_hash` | `TEXT` | NOT NULL |
| `created_at` | `TEXT` | NOT NULL, DEFAULT `CURRENT_TIMESTAMP` |

### Table B: `expenses`

| Column | SQLite Type | Constraints |
|---|---|---|
| `id` | `INTEGER` | PRIMARY KEY AUTOINCREMENT |
| `user_id` | `INTEGER` | NOT NULL, FOREIGN KEY → `users(id)` |
| `amount` | `REAL` | NOT NULL |
| `category` | `TEXT` | NOT NULL |
| `date` | `TEXT` | NOT NULL — must be stored as `YYYY-MM-DD` |
| `description` | `TEXT` | NOT NULL |
| `created_at` | `TEXT` | NOT NULL, DEFAULT `CURRENT_TIMESTAMP` |

## 5. Functions to Implement (`database/db.py`)

- `get_db()`
  - Opens `spendly.db` in the project root directory
  - Sets `row_factory = sqlite3.Row` so columns are accessible by name
  - Enables `PRAGMA foreign_keys = ON` on every connection
  - Returns the open connection — caller is responsible for closing it

- `init_db()`
  - Creates both tables using `CREATE TABLE IF NOT EXISTS`
  - Safe to call multiple times without error or data loss
  - Calls `get_db()` internally and closes the connection before returning

- `seed_db()`
  - Checks whether the `users` table already has any rows — if yes, returns early immediately
  - Otherwise inserts one demo user:
    - `name`: `Demo User`
    - `email`: `demo@spendly.com`
    - `password`: `demo123` hashed via `werkzeug.security.generate_password_hash`
  - Inserts 8 sample expenses linked to that demo user
  - Expenses must cover all 7 categories, with dates spread across the current month
  - Must be idempotent — safe to call on every app startup

## 6. Changes to `app.py`

- Import `get_db`, `init_db`, `seed_db` from `database.db`
- Call `init_db()` and then `seed_db()` inside an `app.app_context()` block at module startup
- The database must be fully initialised and seeded before any route is served

## 7. Files to Change

- `database/db.py` — replace stub function bodies with working implementation
- `app.py` — add imports and startup initialisation block

## 8. Files to Create

- None

## 9. Dependencies

- No new pip packages required
- Use `sqlite3` from the Python standard library
- Use `werkzeug.security` (`generate_password_hash`, `check_password_hash`) — already listed in `requirements.txt`

## 10. Categories (Fixed List)

- `Food`
- `Transport`
- `Bills`
- `Health`
- `Entertainment`
- `Shopping`
- `Other`

These are the only valid category values. Seed data must use only values from this list and must cover all seven.

## 11. Rules for Implementation

- No ORM, no SQLAlchemy — raw `sqlite3` only
- Parameterised queries only — never use string formatting or f-strings to inject values into SQL
- `PRAGMA foreign_keys = ON` must be set on every connection opened via `get_db()`
- `amount` must be stored as `REAL`, not `INTEGER`
- Passwords must be hashed with `generate_password_hash` from `werkzeug.security` — never store plaintext
- `seed_db()` must be idempotent — calling it multiple times must produce no duplicate rows
- All dates must be stored and handled as strings in `YYYY-MM-DD` format

## 12. Expected Behavior

- `get_db()`
  - Returns a `sqlite3.Connection` with `row_factory` set to `sqlite3.Row`
  - Foreign key enforcement is active on the returned connection
  - Can be called repeatedly to open multiple independent connections

- `init_db()`
  - Creates `users` and `expenses` tables if they do not already exist
  - Has no visible effect if both tables already exist
  - Does not drop or modify existing data

- `seed_db()`
  - Inserts demo user and 8 expenses on first call when the `users` table is empty
  - Returns immediately without any inserts on subsequent calls
  - After seeding, `users` has exactly 1 row and `expenses` has exactly 8 rows

- Database-level constraints must enforce:
  - `email` uniqueness on the `users` table
  - `user_id` referential integrity on the `expenses` table
  - `NOT NULL` on all columns marked as such

## 13. Error Handling Expectations

- Inserting a duplicate email into `users` must raise a `sqlite3.IntegrityError` due to the `UNIQUE` constraint
- Inserting an expense with a `user_id` that does not exist in `users` must raise a `sqlite3.IntegrityError` due to the foreign key constraint (requires `PRAGMA foreign_keys = ON`)
- Any malformed or invalid query must raise a clear `sqlite3.Error` — errors must not be silently swallowed during development

## 14. Definition of Done

- [ ] `spendly.db` is created in the project root on app startup
- [ ] Both `users` and `expenses` tables exist with the correct schema and all constraints
- [ ] The demo user `demo@spendly.com` exists with a properly hashed password
- [ ] 8 sample expenses exist, covering all 7 categories
- [ ] Running the app multiple times produces no duplicate seed rows
- [ ] The app starts without errors after a clean install
- [ ] Foreign key enforcement is active — inserting an expense with an invalid `user_id` fails
- [ ] All queries use parameterised SQL — no string formatting of values into query strings
