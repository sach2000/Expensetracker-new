# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Spendly** — a personal expense tracker web app. Stack: Flask 3.1.3 + SQLite (stdlib) + Jinja2. No ORM.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the app (port 5001, debug mode)
python app.py

# Run all tests
pytest

# Run a single test file
pytest tests/test_db.py
```

## Architecture

### Entry point
`app.py` registers all routes and, once the database layer is wired up, calls `init_db()` and `seed_db()` inside an `app.app_context()` block on startup so the DB is ready before the first request.

### Database layer (`database/db.py`)
Three functions, all using raw `sqlite3` (no SQLAlchemy):
- `get_db()` — opens `spendly.db` at the project root, sets `row_factory = sqlite3.Row`, enables `PRAGMA foreign_keys = ON`, returns connection
- `init_db()` — `CREATE TABLE IF NOT EXISTS` for `users` and `expenses`; safe to call repeatedly
- `seed_db()` — idempotent; inserts one demo user (`demo@spendly.com` / `demo123`) and 8 sample expenses only if the table is empty

### Schema
| Table | Key columns |
|-------|------------|
| `users` | `id` PK, `name`, `email` UNIQUE NOT NULL, `password_hash`, `created_at` |
| `expenses` | `id` PK, `user_id` FK→users, `amount` REAL, `category`, `date` (YYYY-MM-DD), `description`, `created_at` |

Fixed category list: `Food`, `Transport`, `Bills`, `Health`, `Entertainment`, `Shopping`, `Other`.

### Templates
`templates/base.html` is the Jinja2 base template; all pages extend it. Google Fonts (DM Serif Display, DM Sans) are loaded from CDN.

### Styling
CSS custom properties are defined in `static/css/style.css` (`:root`). Landing-page styles live in `static/css/landing.css`.

## Implementation rules
- Parameterized queries only — never string-format SQL
- Passwords hashed with `werkzeug.security.generate_password_hash` / `check_password_hash`
- `amount` stored as `REAL`, dates always `YYYY-MM-DD`
- `PRAGMA foreign_keys = ON` on every connection
