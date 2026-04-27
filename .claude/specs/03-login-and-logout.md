# Spec: Login and Logout

## Overview
This step completes the authentication cycle. `GET /login` and `login.html` already
exist as stubs; this step adds `POST /login` to look up the user by email, verify
the password with werkzeug, open a session, and redirect to `/profile`. It also
wires up the `GET /logout` stub to clear the session and redirect to the landing
page. Finally, `base.html`'s navbar is updated to show a personalised greeting and
a "Sign out" link when a session is active, and the public "Sign in" / "Get started"
links when no session exists.

## Depends on
- Step 01 — Database Setup (`get_db()`, `users` table must exist)
- Step 02 — Registration (`app.secret_key` set, session infrastructure in place,
  `create_user` pattern established)

## Routes
- `POST /login` — validates credentials, sets session, redirects to profile — public
- `GET /logout` — clears session, redirects to landing — logged-in (but safe for anyone)

## Database changes
No new tables or columns. A new read helper `get_user_by_email(email)` must be
added to `database/db.py`. It must:
- Query `users` by `email` using a parameterized query
- Return the full row as a `sqlite3.Row` object (so callers can access
  `user['id']`, `user['name']`, `user['password_hash']`)
- Return `None` if no matching row is found
- Open and close its own connection (same pattern as `create_user`)

## Templates
- **Modify:** `templates/login.html`
  - Line 20: change `action="/login"` → `action="{{ url_for('login') }}"` (hardcoded URL)
- **Modify:** `templates/base.html`
  - The `nav-links` div (lines 21-24) currently always shows "Sign in" and
    "Get started". Wrap it in a session check:
    - If `session.get('user_id')` is set: show `"Hi, {{ session['user_name'] }}"` (non-linked)
      and a `"Sign out"` link to `url_for('logout')`
    - Else: show the existing "Sign in" + "Get started" links
  - No new CSS classes required — reuse existing nav link styles

## Files to change
- `database/db.py` — add `get_user_by_email(email)` helper
- `app.py` — add `check_password_hash` import from werkzeug, implement `POST /login`,
  implement `GET /logout`
- `templates/login.html` — fix hardcoded action URL
- `templates/base.html` — add session-conditional nav links

## Files to create
None.

## New dependencies
No new pip packages. Uses:
- `werkzeug.security.check_password_hash` (already installed, not yet imported in `app.py`)
- `flask.session` (already imported in `app.py` from Step 02)

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterized queries only — never f-strings in SQL
- Passwords verified with `werkzeug.security.check_password_hash`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Login failure must use a **generic error message** `"Invalid email or password."` —
  never reveal whether the email exists or not (prevents user enumeration)
- `session.clear()` on logout — clears all keys at once, not just `user_id`
- After logout, redirect to `url_for('landing')` — not `/login`, so the user lands
  on the public home page
- On login failure: re-render `login.html` with `error=` — never use `abort()`
- Strip whitespace from `email` before lookup; do NOT strip `password`

## Definition of done
- [ ] Submitting valid email + correct password redirects to `/profile`
- [ ] `session['user_id']` and `session['user_name']` are set after successful login
- [ ] Submitting a valid email with the wrong password re-renders login with
  `"Invalid email or password."` — no redirect
- [ ] Submitting an email that does not exist re-renders login with
  `"Invalid email or password."` — same message as wrong password (no enumeration)
- [ ] Submitting with a blank email or blank password re-renders with
  `"All fields are required."`
- [ ] Visiting `/logout` clears the session and redirects to `/` (landing page)
- [ ] After logout, `session['user_id']` is no longer set
- [ ] The navbar shows `"Hi, <name>"` and `"Sign out"` when logged in
- [ ] The navbar shows `"Sign in"` and `"Get started"` when not logged in
- [ ] `login.html` line 20 uses `url_for('login')` not `"/login"`
- [ ] The demo user (`demo@spendly.com` / `demo123`) can log in successfully
- [ ] Running the app raises no Python exceptions during login or logout
