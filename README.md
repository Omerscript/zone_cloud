# 🌩️ ZONE | Cloud
This was created with developer rights: @IEI_T  It was created for unlimited storage space.  And you can save and retrieve files from Cloud Zone for free..

# ملاحظه:
اذا ما فهمت شي حول الشرح لاي ذكاء اصطناعي وراح يفهمك ويربط كود حقك بل سحابه ( cloud )


---

## 🔎 Project summary

ZONE | Cloud provides a tiny HTTP API for:

* Uploading a file (POST `/upload`) with an API key.
* Downloading a file (GET `/download/{filename}`).
* Listing available files (GET `/list`).
* Deleting a file (DELETE `/delete/{filename}`).
* Optional status endpoint (GET `/status`) for health checks.

Typical usage: a Telegram bot keeps a local `users.json` and pushes the file to ZONE | Cloud after updates, so multiple instances or developers can share the same data file.

---

## 📌 Features

* Automatic file upload from any Python script
* Minimal authentication via `API key`
* Works with `ngrok` for development or any public host for production
* File-level operations: upload / download / list / delete
* Example client code included (Python `requests`)
* Lightweight and easy to self-host

---

## 🚀 Requirements

* Python 3.9+
* `fastapi` + `uvicorn` (for the server)
* `requests` (client scripts)
* `telebot` (if integrating with a Telegram bot)
* `pyngrok` (optional; only for local tunnel convenience)

`requirements.txt` example:

```
fastapi
uvicorn[standard]
requests
pyngrok        # optional for local ngrok automation
python-telegram-bot  # or pyTelegramBotAPI (telebot)
telethon        # optional
```

---

# Quick start (developer)

## 1) Run the Cloud API (local)

Assuming you have a FastAPI server file `main.py`:

```bash
# install deps
pip install -r requirements.txt

# run server
uvicorn main:app --host 0.0.0.0 --port 8000
```

## 2) Expose it publicly (development) with ngrok

```bash
ngrok http 8000
```

ngrok will give you a public URL like:

```
https://supertutelary-soberly-ezra.ngrok-free.dev
```

Use that URL as your `CLOUD_API` in client scripts.

> Tip: you must `ngrok authtoken <your-token>` on your machine once to enable some features. The authtoken itself is persistent until you remove or reset it in your ngrok dashboard.

---

# API contract (endpoints)

> Replace `{CLOUD_API}` with your server URL (ngrok or production domain). Include the `key` param for endpoints that require it.

### POST `/upload`

Upload a file (multipart form).

Request:

* Form field `key` — your API key.
* File field `file` — the file content. Server will store it under the uploaded filename or a configured fixed name.

Example `curl`:

```bash
curl -X POST "{CLOUD_API}/upload" \
  -F "key=supersecret123" \
  -F "file=@users.json;filename=users.json"
```

Response:

```json
{ "filename": "users.json", "message_id": 123, "link": "https://t.me/..." }
```

*(Response fields depend on your server logic — tailor them to your needs.)*

---

### GET `/download/{filename}`

Download stored file.

Example:

```
GET {CLOUD_API}/download/users.json?key=supersecret123
```

Returns file content as `application/octet-stream` or proper content-type.

---

### GET `/list`

Return the list of stored filenames.

Example:

```
GET {CLOUD_API}/list?key=supersecret123
```

Response:

```json
["users.json", "tokens.json"]
```

---

### DELETE `/delete/{filename}`

Delete stored file (requires `key`).

```
DELETE {CLOUD_API}/delete/users.json?key=supersecret123
```

Response:

```json
{ "status": "deleted", "file": "users.json" }
```

---

### GET `/status` (recommended)

Simple health check to verify server availability.

```
GET {CLOUD_API}/status
```

Respond with `200 OK` and a JSON like `{ "status": "ok" }`.

---

# Example client (Python)

```python
import requests, json, os

CLOUD_API = "https://your-ngrok-url.ngrok-free.dev"
CLOUD_KEY = "supersecret123"
LOCAL_DB = "users.json"
CLOUD_FILENAME = "users.json"

def ensure_local():
    if not os.path.exists(LOCAL_DB):
        with open(LOCAL_DB, "w") as f:
            json.dump({}, f, indent=4)

def upload():
    ensure_local()
    with open(LOCAL_DB, "rb") as f:
        res = requests.post(
            f"{CLOUD_API}/upload",
            data={"key": CLOUD_KEY},
            files={"file": (CLOUD_FILENAME, f)}
        )
    print("upload response:", res.status_code, res.text)

def download():
    res = requests.get(f"{CLOUD_API}/download/{CLOUD_FILENAME}", params={"key": CLOUD_KEY})
    if res.status_code == 200:
        open(LOCAL_DB, "wb").write(res.content)
        print("Downloaded OK")
    else:
        print("Download failed:", res.status_code, res.text)

# Example:
# add a user then upload
ensure_local()
db = {}
with open(LOCAL_DB, "r") as f: db = json.load(f)
db["123"] = "example_user"
with open(LOCAL_DB, "w") as f: json.dump(db, f, indent=4)
upload()
```

---

# Example Telegram bot snippet (client side)

Use your bot to call upload after saving:

```python
import os, json, requests, telebot

BOT_TOKEN = "YOUR_BOT_TOKEN"
bot = telebot.TeleBot(BOT_TOKEN, parse_mode="HTML")

CLOUD_API = "https://your-ngrok-url.ngrok-free.dev"
CLOUD_KEY = "supersecret123"
LOCAL_DB = "users.json"

if not os.path.exists(LOCAL_DB):
    with open(LOCAL_DB, "w") as f:
        json.dump({}, f, indent=4)

def upload_to_cloud():
    try:
        with open(LOCAL_DB, "rb") as f:
            return requests.post(
                f"{CLOUD_API}/upload",
                data={"key": CLOUD_KEY},
                files={"file": ("users.json", f)}
            ).json()
    except Exception as e:
        return {"error": str(e)}

def save_user(user_id, username):
    with open(LOCAL_DB, "r") as f:
        db = json.load(f)
    db[str(user_id)] = username or "no_name"
    with open(LOCAL_DB, "w") as f:
        json.dump(db, f, indent=4)
    print("Uploading file to cloud...")
    print(upload_to_cloud())

@bot.message_handler(commands=["start"])
def start(m):
    save_user(m.from_user.id, m.from_user.username)
    bot.send_message(m.chat.id, "Welcome! Your ID saved.")

bot.infinity_polling()
```

---

# Security & hardening (recommended)

* Use a strong random `CLOUD_KEY` (not `supersecret123` in production).
* Serve via HTTPS in production (ngrok provides HTTPS; for self-host use TLS).
* Rate-limit endpoints (prevent abuse).
* Validate uploaded filenames: sanitize to prevent path traversal (`../` attacks).
* Limit file size (e.g. 50 MB) to avoid storage DOS.
* Keep logs for uploads/deletes with timestamps and IPs.
* Use CORS appropriately (only allow origins you need).
* For multi-user systems, migrate to per-user API keys or OAuth.

---

# Persistence & hosting considerations

* **ngrok**: fast for development, but public URL changes on restart (unless you pay for reserved domains). Use it for testing.
* **Production**: deploy on Render / Railway / DigitalOcean / AWS / GCP for stable URL and uptime.
* **Storage**: local filesystem is fine for small projects. For scale, use S3 or other object storage (and store metadata in DB).
* **Atomic writes**: write to `file.tmp` then `rename()` to avoid partial reads during upload.
* **Backups**: periodically copy stored files to a backup location or S3.

---

# Troubleshooting checklist

* `ModuleNotFoundError: pyngrok` → `pip install pyngrok`
* `ERR_NGROK_4018` (ngrok auth) → run `ngrok config add-authtoken <token>` or use `ngrok.set_auth_token(...)` in your automation.
* `ngrok URL changed` → update `CLOUD_API` in clients or use a config value/env var.
* Upload returns 403 → check `key` param matches server config.
* Server complains `on_event is deprecated` → migrate FastAPI startup to Lifespan handler (works but is only a warning).

---

# Deployment recommendations

* Put `CLOUD_API` and `CLOUD_KEY` in environment variables (never commit keys).
* Use a `.env` file with `python-dotenv` for local development.
* Add a small `status` endpoint your bots can ping before attempting upload.
* Add a `max_content_length` check in FastAPI to avoid huge uploads.

---

# File structure suggestion for repo

```
ZONE-cloud/
│
├── README.md
├── main.py                 # FastAPI Cloud API server
├── example_client.py       # Simple client + bot examples
├── requirements.txt
├── .env.example            # example env var file
└── README-images/          # optional screenshots/diagram images
```

---

# Deep implementation & behavior details (25 precise points)

1. **Authentication Mode** — Single API key per server instance stored in configuration; server checks `data["key"]` on every upload/delete/download call. Consider per-user keys for multi-tenant.

2. **Filename handling** — Accept `multipart` filename from client, sanitize to allow only `[a-zA-Z0-9._-]` and a maximum length (e.g., 128 chars); reject others.

3. **Atomic file writes** — Server should write to `/<temp>/<filename>.tmp` then `os.replace()` to final path to avoid partial-file reads.

4. **Concurrent upload handling** — Use file locks or write-to-temp per-upload to avoid race conditions with parallel uploads.

5. **Max upload size enforcement** — Enforce a request body size limit (e.g., NGROK/uvicorn or FastAPI middleware) to avoid memory overload.

6. **Timeouts** — Configure request and client timeouts on both server and client (e.g., `requests.post(..., timeout=10)`).

7. **Health check endpoint** — Implement `/status` returning `{status: ok, uptime: N}` for bots to check before upload.

8. **Response semantics** — Return structured JSON `{ok: true, filename: ..., meta: {...}}` or `{ok:false, error:"..."}` to make client parsing deterministic.

9. **Atomic DB store** — If you keep a metadata DB (json mapping) update it atomically, or prefer a small sqlite store instead of JSON for concurrency.

10. **Rate limiting** — Add simple per-IP or per-key rate-limiting to prevent abuse.

11. **CORS** — Default to permissive in development (`allow_origins=["*"]`), lock down in production to allowed origins.

12. **Logging** — Log uploads, IP addresses, filenames, API key used, and timestamps. Use rotating logs to avoid disk exhaustion.

13. **Error handling** — Catch all exceptions at endpoint level and return helpful 4xx/5xx with message and code.

14. **File metadata** — Save length, upload time, and optionally uploader ID in a metadata JSON for each file.

15. **Delete semantics** — When delete is called, physically remove the file and optionally move it to a `trash/` folder for soft-delete.

16. **Download permission** — `GET /download/{filename}?key=...` should check key; consider signed expiring download URLs if exposing to third-parties.

17. **HTTPS in production** — Ensure TLS termination either at host or via reverse proxy (nginx, Cloudflare), not plain HTTP.

18. **Authtoken for ngrok automation** — If you automate ngrok from code, call `ngrok.set_auth_token("...")` once per environment.

19. **Session lifecycle** — For services like Telethon that connect on server startup, use FastAPI lifespan events (recommended) instead of deprecated `@app.on_event("startup")`.

20. **Testing** — Unit tests for endpoints: upload success, upload bad key, download missing file, delete missing file, list empty vs populated.

21. **CI/CD** — Add a GitHub Actions workflow that runs tests and lints before deploy; optionally auto-deploy to Render or Railway.

22. **Storage backend switch** — Abstract file storage behind an interface so you can switch from local FS to S3/GCS without changing API.

23. **Backups** — Implement nightly backups to remote storage and keep a retention policy (e.g., 30 days).

24. **Monitoring & Alerts** — Integrate an uptime monitor (UptimeRobot) to ping `/status` and alert if 5xx or not reachable.

25. **Migration path to DB** — If data grows beyond JSON, move to a small DB (SQLite → PostgreSQL). For user files, store in S3 with DB metadata referencing keys.

---

# Example `main.py` (minimal FastAPI server)

```python
from fastapi import FastAPI, File, UploadFile, Form, HTTPException
from fastapi.responses import FileResponse, JSONResponse
import os, json, shutil

API_KEY = "supersecret123"
DB_DIR = "cloud_files"
os.makedirs(DB_DIR, exist_ok=True)

app = FastAPI()

@app.get("/status")
async def status():
    return {"status": "ok"}

@app.post("/upload")
async def upload(key: str = Form(...), file: UploadFile = File(...)):
    if key != API_KEY:
        raise HTTPException(403, "API key incorrect")
    filename = os.path.basename(file.filename)
    safe_name = "".join(ch for ch in filename if ch.isalnum() or ch in "._-")[:128]
    tmp_path = os.path.join(DB_DIR, safe_name + ".tmp")
    final_path = os.path.join(DB_DIR, safe_name)
    with open(tmp_path, "wb") as f:
        shutil.copyfileobj(file.file, f)
    os.replace(tmp_path, final_path)
    return {"ok": True, "filename": safe_name}

@app.get("/download/{filename}")
async def download(filename: str, key: str):
    if key != API_KEY:
        raise HTTPException(403, "API key incorrect")
    path = os.path.join(DB_DIR, filename)
    if not os.path.exists(path):
        raise HTTPException(404, "File not found")
    return FileResponse(path, filename=filename)

@app.get("/list")
async def list_files(key: str):
    if key != API_KEY:
        raise HTTPException(403, "API key incorrect")
    return [f for f in os.listdir(DB_DIR) if os.path.isfile(os.path.join(DB_DIR, f))]

@app.delete("/delete/{filename}")
async def delete(filename: str, key: str):
    if key != API_KEY:
        raise HTTPException(403, "API key incorrect")
    path = os.path.join(DB_DIR, filename)
    if not os.path.exists(path):
        raise HTTPException(404, "File not found")
    os.remove(path)
    return {"status": "deleted", "file": filename}
```

---

# Environment & configuration best practices

* Use environment variables: `CLOUD_KEY`, `PORT`, `DB_DIR`, `API_DEBUG=false`.
* Example `.env`:

```
CLOUD_KEY=replace_with_secure_random_string
PORT=8000
DB_DIR=cloud_files
```

* In Python, load with `os.getenv("CLOUD_KEY")` or `python-dotenv`.

---

# README extras (for GitHub)

Add these to the repo:

* `README.md` (this file)
* `main.py` (server)
* `example_client.py` / `example_bot.py`
* `requirements.txt`
* `.env.example` (do **not** commit secrets)
* `CONTRIBUTING.md` (how to contribute)
* `LICENSE` (MIT recommended for open source)

---

# Troubleshooting quick answers

* **Upload returns 403**: you forgot `key` or it is wrong.
* **ngrok `ERR_NGROK_4018`**: ngrok needs verified account + authtoken.
* **URL changed**: update `CLOUD_API` in your bot client or use reserved domain on paid ngrok.
* **Partial/corrupt download**: ensure server used atomic writes (tmp + rename).
* **Concurrent overwrite**: consider locking or versioned filenames.

---

# Contributing & license

If you want to open-source the project:

* Add `CONTRIBUTING.md` with guidelines (PR, issue template).
* Use `MIT` or `Apache-2.0`.
* Add tests and a CI workflow.

---

# Final words

This README is meant to be both a **user guide** and a **developer guide** — you can paste it directly as `README.md` in the `ZONE | cloud` repository.
If you want, I can:

* Generate a polished Markdown file ready to paste.
* Produce a PNG diagram of the flow and include it in the repo.
* Create `main.py`, `example_bot.py`, and a GitHub Actions workflow (CI) ready to push.

Tell me which extras you want (diagram, code files, CI), and I’ll generate them now.
