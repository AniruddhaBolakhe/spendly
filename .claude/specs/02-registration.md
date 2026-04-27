# Spec: Registration

## Overview
This step wires up the registration form so new users can create an account.
The `GET /register` route and `register.html` template already exist; this step
adds `POST /register` to validate the submitted data, insert the new user into
the `users` table with a hashed password, open a session, and redirect to the
dashboard (currently `/profile`). It also sets the Flask secret key required
for session management.

## Depends on
- Step 01 — Database Setup (`get_db()`, `init_db()`, `users` table must exist)

## Routes
- `POST /register` — processes the registration form, creates user, starts session — public

## Database changes
No new tables or columns. The `users` table added in Step 01 is sufficient:
```
users(id, name, email, password_hash, created_at)
```
A new helper `create_user(name, email, password)` must be added to `database/db.py`.
It must:
- Hash the password with `werkzeug.security.generate_password_hash`
- Insert the row with a parameterized query
- Return the new user's `id`
- Raise `sqlite3.IntegrityError` naturally when the email is already taken (let
  the route catch it — do not suppress the error here)

## Templates
- **Modify:** `templates/register.html`
  - Change `action="/register"` → `action="{{ url_for('register') }}"` (never hardcode URLs)
  - The `{% if error %}` block is already present — no other template changes needed

## Files to change
- `app.py` — add `POST` method to `@app.route("/register")`, set `app.secret_key`,
  import `session`, `redirect` from flask, import `create_user` from `database.db`
- `database/db.py` — add `create_user(name, email, password)` helper
- `templates/register.html` — fix hardcoded `action` URL

## Files to create
None.

## New dependencies
No new pip packages. Uses:
- `werkzeug.security.generate_password_hash` (already installed)
- `flask.session`, `flask.redirect`, `flask.url_for` (already installed)
- `sqlite3` (stdlib)

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterized queries only — never f-strings in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `app.secret_key` must be set before session use; use a hard-coded dev string
  for now (e.g. `"dev-secret-change-me"`) — note in a comment that it must be
  an env var in production
- After successful registration, store `user_id` and `user_name` in `session`
  and redirect to `url_for('profile')` (the existing stub is fine for now)
- On failure (duplicate email, short password, missing fields) re-render
  `register.html` with an `error=` variable — do not use `abort()`
- Password minimum length: 8 characters — validate server-side, not just via HTML `required`
- Strip whitespace from `name` and `email` before storing

## Definition of done
- [ ] Submitting the form with valid, unique data creates a row in `users` with a hashed password (not plaintext)
- [ ] After successful registration the browser is redirected to `/profile`
- [ ] `session['user_id']` and `session['user_name']` are set after redirect
- [ ] Submitting a duplicate email re-renders the register page with the error message "An account with that email already exists."
- [ ] Submitting a password shorter than 8 characters re-renders with "Password must be at least 8 characters."
- [ ] Submitting with a blank name or email re-renders with "All fields are required."
- [ ] The `action` attribute in `register.html` uses `url_for('register')` not a hardcoded string
- [ ] Running the app and registering a new user does not raise any Python exceptions
