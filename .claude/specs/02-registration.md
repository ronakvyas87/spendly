# Spec: Registration

## Overview

Allow new users to create a Spendly account by submitting their name, email, and password via the `/register` form. On success the user is redirected to the login page. On failure (duplicate email, short password, missing fields) the form re-renders with a clear error message. This step wires the existing `register.html` template to a real `POST` handler and establishes the user-creation pattern that the login step will mirror.

---

## Depends on

- Step 1 — Database Setup (`users` table and `get_db()` must exist)

---

## Routes

- `GET  /register` — render the registration form — public
- `POST /register` — validate input and create user, redirect to `/login` on success — public

---

## Database changes

No new tables or columns. Uses the existing `users` table:

| Column | Used here |
|---|---|
| name | stored as-is |
| email | stored lowercase; must be unique |
| password_hash | hashed with `werkzeug` |
| created_at | auto-set by SQLite default |

---

## Templates

- **Modify:** `templates/register.html`
  - Form already exists and posts to `POST /register`
  - `{% if error %}` block already present — no structural changes needed
  - Pass back `name` and `email` field values on validation failure so the user doesn't re-type them

---

## Files to change

- `app.py` — split the existing `GET /register` stub into `GET` + `POST` methods; add validation logic and DB insert; add `secret_key` to the Flask app config (needed for `flash` if used, and safe practice)
- `database/db.py` — add `create_user(name, email, password)` helper function

---

## Files to create

None.

---

## New dependencies

No new pip packages. Uses:
- `werkzeug.security.generate_password_hash` (already installed)
- `sqlite3` (stdlib)
- `flask.request`, `flask.redirect`, `flask.url_for` (already installed)

---

## Rules for implementation

- No SQLAlchemy or ORMs
- Parameterised queries only — never string formatting in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash`
- Email stored and compared in lowercase (`email.strip().lower()`)
- Validate server-side: name non-empty, valid email format, password ≥ 8 characters
- On duplicate email: catch `sqlite3.IntegrityError`, re-render form with error `"An account with that email already exists."`
- On success: `redirect(url_for('login'))`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `app.secret_key` must be set (use `os.urandom(24)` or a fixed dev string)

---

## Definition of done

- [ ] `GET /register` renders the form with no errors
- [ ] Submitting valid name, email, password creates a row in `users` with a hashed password
- [ ] After successful registration, browser is redirected to `/login`
- [ ] Submitting a duplicate email re-renders the form with an error message (no crash)
- [ ] Submitting a password shorter than 8 characters re-renders the form with an error message
- [ ] Submitting with an empty name or email re-renders the form with an error message
- [ ] Previously typed name and email are preserved in the form on validation failure
- [ ] Password is never stored in plaintext — confirm via `sqlite3 spendly.db "SELECT password_hash FROM users LIMIT 1;"`
- [ ] App starts without errors after changes
