# Spec: Login and Logout

## Overview
Implement session-based login and logout so registered users can authenticate with Spendly.
This step upgrades the existing `GET /login` stub by adding a `POST /login` route that
validates credentials, checks the hashed password, and writes the user's `id` and `name`
into the Flask session on success. It also implements the `GET /logout` stub to clear the
session and redirect to the landing page. With this step in place, all subsequent features
can gate access behind a logged-in session check.

## Depends on
- Step 1 — Database Setup (`users` table, `get_db()` must exist)
- Step 2 — Registration (`create_user()` in place; at least one user in the DB to log in with)

## Routes
- `GET /login` — renders the login form — public (already exists, needs POST wiring)
- `POST /login` — validates credentials, sets session, redirects to `/dashboard` or `/` — public
- `GET /logout` — clears session, redirects to `/` — public (currently a raw string stub)

## Database changes
No new tables or columns. Login reads from the existing `users` table:
- `email` TEXT UNIQUE NOT NULL
- `password_hash` TEXT NOT NULL

## Templates
- **Modify:** `templates/login.html`
  - Ensure the form has `method="POST"` and `action="{{ url_for('login') }}"`
  - Fields: `email` (email, required) and `password` (password, required)
  - Display flash messages for errors (wrong credentials, empty fields)
  - Show a success flash message if redirected here after registration
- **Modify:** `templates/base.html`
  - Add a conditional nav: if `session.user_id` is set, show a "Sign out" link
    (`url_for('logout')`); otherwise keep the existing "Sign in / Get started" links

## Files to change
- `app.py` — implement `POST /login` handler; implement `GET /logout` handler; import `session`, `check_password_hash` as needed
- `database/db.py` — add `get_user_by_email(email)` helper that returns a `sqlite3.Row` or `None`
- `templates/login.html` — add POST form and flash message rendering
- `templates/base.html` — add logged-in nav state using `session`

## Files to create
None.

## New dependencies
No new dependencies. `werkzeug.security.check_password_hash` is already available via the
existing `werkzeug` install.

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` via `get_db()` only
- Parameterised queries only — never f-strings or string concatenation in SQL
- Password verification with `werkzeug.security.check_password_hash` — never compare plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Use `url_for()` for every internal link and form action — never hardcode URLs
- Flash messages via `flask.flash` + `get_flashed_messages` — no custom session keys
- On successful login: set `session['user_id']` and `session['user_name']`, then redirect to `url_for('landing')`
- On failed login: re-render `login.html` with a single generic error ("Invalid email or password") — never reveal which field was wrong
- On logout: call `session.clear()` then redirect to `url_for('landing')`
- `get_user_by_email()` belongs in `database/db.py`, not inline in the route
- `app.secret_key` is already set in `app.py` — do not change it or generate a new one

## Definition of done
- [ ] `GET /login` renders the login form with email and password fields
- [ ] Submitting correct credentials sets the session and redirects to `/`
- [ ] The nav in `base.html` shows "Sign out" after a successful login
- [ ] Submitting wrong email or wrong password re-renders login with a generic error message
- [ ] Submitting with an empty field re-renders login with an error message
- [ ] `GET /logout` clears the session and redirects to `/`
- [ ] After logout the nav reverts to "Sign in / Get started"
- [ ] The demo user (`demo@spendly.com` / `demo123`) can log in successfully
- [ ] App starts without errors after all changes
