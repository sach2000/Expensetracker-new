# Step 03 — Login and Logout

## Overview

- This step converts the `/login` stub into a functional `POST` handler that:
  - Verifies submitted credentials against the `users` table
  - Stores the authenticated user's `id` in `session["user_id"]`
  - Redirects to the landing page (until a dashboard route exists)
- Also implements `/logout`, which clears the session and redirects to `/`
- After this step the app can distinguish logged-in users from guests
- This is a prerequisite for all expense features — protected routes depend on `session["user_id"]` being set

## Depends on

- Step 01 — Database Setup (`users` table must exist)
- Step 02 — Registration (`create_user()` and password hashing must be in place; at least one user must exist to log in against)

## Routes

- `GET /login` — render the login form — public
- `POST /login` — validate credentials, set session, redirect — public
- `GET /logout` — clear session, redirect to `/` — public (no login required to call)

## Database changes

- No database changes
- The `users` table from Step 01 already stores `email` and `password_hash`
- One new DB helper to add to `database/db.py`:
  - `get_user_by_email(email)`
    - Queries the `users` table by `email`
    - Returns the matching row as a `sqlite3.Row` or `None` if not found
    - Must live in `database/db.py` — not inlined into the route handler

## Templates

- Modify `templates/login.html`:
  - Add a `POST` form with `email` and `password` fields
  - Set `form action` to `url_for('login')` with `method="post"`
  - Add a block to display flashed error messages
  - Add a link to `/register` for new users
  - Keep all existing visual design unchanged

## Files to change

- `app.py` — implement `login()` as a `GET`+`POST` handler; implement `logout()`
- `database/db.py` — add `get_user_by_email()` helper
- `templates/login.html` — add `POST` form and flashed message display

## Files to create

- None

## New dependencies

- None — `werkzeug.security.check_password_hash` is already available

## Rules for implementation

- No SQLAlchemy or ORMs — use raw `sqlite3` via `get_db()`
- Parameterised queries only — never use f-strings or string concatenation in SQL
- Verify passwords with `werkzeug.security.check_password_hash`
- The session key for the authenticated user must be `session["user_id"]` storing an `int`
- Use `flask.session` — do not roll a custom session mechanism
- On failed login show a single generic flash error: `"Invalid email or password."` — do not reveal which field was wrong
- After successful login redirect to `url_for("landing")` until a dashboard route exists
- `logout()` must call `session.clear()` then redirect to `url_for("landing")`
- All templates must extend `base.html`
- Use CSS variables — never hardcode hex values
- Use `url_for()` for every internal link — never hardcode paths

## Definition of Done

- [ ] `GET /login` renders the login form with `email` and `password` fields
- [ ] Submitting with valid credentials (`demo@spendly.com` / `demo123`) sets `session["user_id"]` and redirects to `/`
- [ ] Submitting with a wrong password shows `"Invalid email or password."` flash and stays on the login page
- [ ] Submitting with an unregistered email shows the same generic error flash
- [ ] `GET /logout` clears the session and redirects to `/`
- [ ] After logout, `session["user_id"]` is no longer present
- [ ] The `/logout` route no longer returns the raw stub string
