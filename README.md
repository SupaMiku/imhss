# IMHSS – Intelligent Mental Health Support System

Web app with accounts, AI support chat, mood tracking, PHQ-9 / GAD-7 screening, education resources and a staff portal.

**Stack:** HTML/CSS/JavaScript (frontend in `static/`) · Python FastAPI (`main.py`) · SQLAlchemy · SQLite database (single file `imhss.db`) · Anthropic API for the AI chat · bcrypt + JWT for auth.

## Run locally
```bash
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
export SECRET_KEY=any-long-random-string
export ANTHROPIC_API_KEY=sk-ant-...                    # optional; without it the chat uses simple built-in replies
uvicorn main:app --reload                              # open http://localhost:8000
```
Windows (PowerShell): set `$env:SECRET_KEY` and `$env:ANTHROPIC_API_KEY` instead of using `export`.
Create the first admin interactively with `python create_admin.py admin@example.com`; it prompts for the password without echoing it. Admins can create user, staff, and admin accounts from the administrator dashboard. Users can also register for their own accounts; all roles sign in with their own email and password through the same form. Each role has a separate dashboard.

For direct SQLite inserts, `role` must be `user`, `staff`, or `admin`. You may enter the chosen password directly in `pw_hash`; on the first successful login, the app replaces it with a bcrypt hash. This is a convenience for local setup, so do not leave plain-text passwords in the database longer than needed. To insert a bcrypt hash instead, generate one locally with:
```bash
python -c "import bcrypt,getpass; print(bcrypt.hashpw(getpass.getpass('Password: ').encode(), bcrypt.gensalt()).decode())"
```
Then insert the row (replace the sample email, role, and hash):
```sql
INSERT INTO users (email, pw_hash, role, created)
VALUES ('staff@example.com', '<bcrypt-hash>', 'staff', CURRENT_TIMESTAMP);
```
All roles use the same Log in form. The account role determines which dashboard opens; public registration always creates a `user`.

## Put it on GitHub
```bash
git init && git add . && git commit -m "IMHSS initial version"
git branch -M main
git remote add origin https://github.com/<your-username>/imhss.git   # create the empty repo on github.com first
git push -u origin main
```
Never commit `.env` or API keys (`.gitignore` already excludes `.env`).

## Deploy on Render (with SQLite)
SQLite stores everything in one file, so the host must keep that file between restarts. On Render this needs a **persistent disk**, which requires a paid web service plan (check Render’s current prices). Without a disk, your data is erased every time the app redeploys.
1. Push the project to GitHub (steps above).
2. On render.com choose **New → Blueprint** and connect your repo. `render.yaml` creates the web service and a 1 GB disk mounted at `/var/data`; the database file is saved at `/var/data/imhss.db`.
3. Set `ANTHROPIC_API_KEY` (AI chat), if desired. `SECRET_KEY` is generated automatically.
4. Create the initial admin in an environment connected to the deployed database by running `python create_admin.py admin@example.com`.
5. Open the URL Render gives you. Users, staff, and admins all log in through the same **Log in** form.
Other hosts with persistent volumes (for example Railway or Fly.io) also work: mount a volume and set `DB_PATH` to a file on it.
Keep only one running instance, since SQLite is a single-file database. To back up, download a copy of `imhss.db` regularly.

The database tables are in `schema.sql`; the app also creates them automatically on first start.

## Notes and next steps
- Tables are created automatically on startup (users, moods, screenings, resources). For schema changes later, add Alembic.
- Self-service password recovery is unavailable now that security questions are removed. To reset a password, update that account’s `pw_hash` directly in SQLite; a plain-text value is converted to bcrypt after a successful login.
- Staff and admins can add, edit, and delete education resources from the dashboard.
- Staff see only counts and a “follow up” flag, never mood notes, screening answers or chats. Chats are not stored.
- Before real users: add HTTPS-only (Render does this), login rate limiting, a secure password-reset flow, and review privacy rules (e.g. the Philippines Data Privacy Act) with your adviser.
- Crisis numbers (911, NCMH 1553, Hopeline) come from public sources; verify before launch.
- PHQ-9 / GAD-7 are screening tools, not diagnoses.
# imhss
