# Step 02 — User Registration

## Overview

- Step 02 upgrades the existing stub `GET /register` route into a fully functional form handling both `GET` and `POST`
- The form accepts four fields: `name`, `email`, `password`, `confirm_password`
- On success: flash a success message and redirect to `/login`
- This is the entry point for all authenticated features — every user account begins here

## Depends on

- Step 01 — Database setup (`users` table and `get_db()` exist)

## Routes

- `GET /register` — render the registration form — public
  - Already exists as a stub; upgrade it in place
- `POST /register` — process the submitted form, validate input, insert the new user, redirect to `/login` — public

## Database changes

- No new tables or columns
- The existing `users` table covers all requirements
- One new DB helper to add to `database/db.py`:
  - `create_user(name, email, password)`
    - Hashes `password` with `werkzeug.security.generate_password_hash`
    - Inserts a new row into `users` with `name`, `email`, and the resulting `password_hash`
    - Returns the new user's `id` (from `cursor.lastrowid`)
    - Raises `sqlite3.IntegrityError` if `email` is already taken (enforced by the `UNIQUE` constraint)

## Templates

- Modify `templates/register.html`:
  - Change the form `action` to `url_for('register')` and set `method="post"`
  - Add `name` attributes to all inputs: `name`, `email`, `password`, `confirm_password`
  - Add a block to display flashed error messages (e.g. `"Email already registered"`, `"Passwords do not match"`)
  - Keep all existing visual design unchanged — no layout or style changes

## Files to change

- `app.py` — upgrade `register()` to handle both `GET` and `POST`; add flash and redirect logic
- `database/db.py` — add `create_user()` helper
- `templates/register.html` — wire up form `action`/`method` and add flashed message display

## Files to create

- None

## New dependencies

- None — uses `werkzeug.security` (already installed) and Flask's built-in `flash`, `redirect`, `url_for`

## Rules for implementation

- No SQLAlchemy or ORMs — raw `sqlite3` only via `get_db()`
- Parameterised queries only — never use f-strings or string concatenation in SQL
- Hash passwords with `werkzeug.security.generate_password_hash` — never store plaintext
- `app.secret_key` must be set in `app.py` for `flash()` to work — use a hardcoded dev string for now
- Server-side validation must run in this exact order:
  1. All four fields are non-empty
  2. `password == confirm_password`
  3. Email is not already registered — catch `sqlite3.IntegrityError` from `create_user()`
- On any validation failure: re-render the registration form with a flashed error message — do not redirect
- On success: flash a success message and redirect to `url_for('login')`
- Use `abort(405)` if an unsupported HTTP method reaches the route
- All templates must extend `base.html`
- Use CSS variables — never hardcode hex values
- Use `url_for()` for every internal link — never hardcode URLs

## Definition of Done

- [ ] `GET /register` renders the registration form without errors
- [ ] Submitting with all valid fields creates a new user in `users` and redirects to `/login`
- [ ] Submitting with mismatched passwords re-renders the form with an error — no DB insert occurs
- [ ] Submitting with an already-registered email re-renders with `"Email already registered"`
- [ ] Submitting with any empty field re-renders with a validation error
- [ ] Password is stored as a hash — never plaintext — verifiable by inspecting `spendly.db`
- [ ] No duplicate user is created on repeated valid submissions with the same email
