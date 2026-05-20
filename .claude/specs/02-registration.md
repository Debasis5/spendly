# Spec: Registration

## Overview
Implement user registration so new visitors can create a Spendly account. This step
upgrades the existing `GET /register` stub (which already renders `register.html`) by
adding a `POST /register` route that validates form input, hashes the password, inserts
a new row into the `users` table, and redirects to the login page on success and showing the user a success message. This isthe entry point to all authenticated features and must be complete before login (Step 3)
can be built.

## Depends on
- Step 1 — Database Setup (`users` table and `get_db()` must exist)

## Routes
- `GET /register` — renders the registration form — public (already exists, no change needed)
- `POST /register` — processes form submission, inserts user, redirects to `/login` — public

## Database changes
No new tables or columns. The existing `users` table has all required fields:
- `name` TEXT NOT NULL
- `email` TEXT UNIQUE NOT NULL
- `password_hash` TEXT NOT NULL
- `created_at` TEXT DEFAULT (datetime('now'))

## Templates
- **Modify:** `templates/register.html`
  - Ensure the form has `method="POST"` and `action="{{ url_for('register') }}"`
  - Add a `name` field (text, required)
  - Add an `email` field (email, required)
  - Add a `password` field (password, required)
  - Display flash messages for errors (duplicate email, missing fields) and success

## Files to change
- `app.py` — add `POST /register` route handler and required imports
- `database/db.py` — add `create_user(name, email, password_hash)` helper
- `templates/register.html` — add POST form and flash message rendering

## Files to create
None.

## New dependencies
No new dependencies.

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` via `get_db()` only
- Parameterised queries only — never f-strings or string concatenation in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash` before insert
- Duplicate email must be caught and shown as a user-friendly flash error, not a 500
- All templates extend `base.html`
- Use CSS variables — never hardcode hex values
- Use `url_for()` for every internal link and form action — never hardcode URLs
- Flash messages via `flask.flash` + `flask.get_flashed_messages` — no custom session keys
- After successful registration redirect to `url_for('login')` using `redirect()`
- DB logic (insert, duplicate check) belongs in `database/db.py`, not in the route
- `app.py` must import `request`, `redirect`, `flash`, `session` from `flask` as needed
- The route must handle the `sqlite3.IntegrityError` raised by a duplicate email

## Definition of done
- [ ] `GET /register` renders the form with name, email, and password fields
- [ ] Submitting the form with valid unique data inserts a new user and redirects to `/login`
- [ ] The inserted password is stored as a hash, never plaintext
- [ ] Submitting with a duplicate email re-renders the form with a visible error message
- [ ] Submitting with any empty field re-renders the form with a visible error message
- [ ] Registering a second user with a different email succeeds independently
- [ ] App starts without errors after changes
