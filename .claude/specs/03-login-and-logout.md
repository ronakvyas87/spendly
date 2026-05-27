# Spec: Login and Logout

## Overview

Allow registered users to sign in to their Spendly account using email and password, and sign out when done. On successful login, the user's `id` and `name` are stored in the Flask session and they are redirected to `/profile` (the next step's landing page). Invalid credentials re-render the login form with an error. Logout clears the session and redirects to the landing page. This step also updates `base.html` so the navbar reflects auth state — showing "Sign in / Get started" to guests and "My account / Sign out" to logged-in users.

---

## Depends on

- Step 1 — Database Setup (`users` table, `get_db()`)
- Step 2 — Registration (users must exist to log in)

---

## Routes

- `GET  /login`  — render the login form — public
- `POST /login`  — verify credentials, set session, redirect — public
- `GET  /logout` — clear session, redirect to `/` — logged-in (no hard guard needed yet)

---

## Database changes

No database changes. Reads from the existing `users` table using email lookup.

---

## Templates

- **Modify:** `templates/login.html`
  - Already has `{% if error %}` block and posts to `POST /login`
  - Add `value="{{ email or '' }}"` to the email input to preserve it on error
- **Modify:** `templates/base.html`
  - Update `nav-links` to conditionally show:
    - **Guest:** "Sign in" + "Get started" (current behaviour)
    - **Logged in:** user's name (non-clickable or links to `/profile`) + "Sign out" link to `/logout`
  - Use `session` variable (available in Jinja2 automatically when `secret_key` is set)

---

## Files to change

- `app.py` — expand `GET /login` stub to `GET`+`POST`; implement `GET /logout`; add `session` to Flask imports
- `database/db.py` — add `get_user_by_email(email)` helper
- `templates/login.html` — add `value` attribute to email input
- `templates/base.html` — conditional navbar based on `session`

---

## Files to create

None.

---

## New dependencies

No new pip packages. Uses:
- `werkzeug.security.check_password_hash` (already installed)
- `flask.session` (already installed)

---

## Rules for implementation

- No SQLAlchemy or ORMs
- Parameterised queries only — never string formatting in SQL
- Password verified with `werkzeug.security.check_password_hash`
- Session stores only `user_id` (int) and `user_name` (str) — never the password or hash
- Use a single generic error for failed login: `"Invalid email or password."` — do not reveal which field was wrong
- `logout` clears the entire session with `session.clear()`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`

---

## Definition of done

- [ ] `GET /login` renders the form with no errors
- [ ] Submitting correct email + password sets the session and redirects to `/profile`
- [ ] Submitting wrong password re-renders the form with `"Invalid email or password."` — email field preserved
- [ ] Submitting an email that doesn't exist re-renders with the same generic error
- [ ] `GET /logout` clears the session and redirects to `/`
- [ ] After logout, visiting `/logout` again redirects cleanly (session is already empty — no crash)
- [ ] Logged-in navbar shows the user's name and a "Sign out" link
- [ ] Guest navbar shows "Sign in" and "Get started" (unchanged from current)
- [ ] App starts without errors after changes
