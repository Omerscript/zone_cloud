# 🌩️ ZONE | Cloud

**Developer Rights:** @IEI_T  
**Purpose:** Unlimited cloud storage for your scripts. Save and retrieve files from ZONE | Cloud easily.  

**Storage:** Unlimited  

**Note:**  
If you don’t understand any part of this guide, any AI can help you link your code to the cloud automatically.

---

## 🔎 Project Overview

**ZONE | Cloud** provides a lightweight HTTP API for:

* Uploading files (`POST /upload`) using an API key.  
* Downloading files (`GET /download/{filename}`).  
* Listing all stored files (`GET /list`).  
* Deleting files (`DELETE /delete/{filename}`).  
* Optional health check endpoint (`GET /status`).  

**Use case:**  
A Telegram bot or Python script can store local data (like `users.json`) and automatically sync it to ZONE | Cloud, allowing multiple developers or instances to share the same data file.

---

## 📌 Key Features

* Automatic file upload from any Python script  
* Minimal authentication via `API key`  
* Works with `ngrok` for testing or any public host in production  
* Full file-level operations: upload, download, list, delete  
* Lightweight and self-hostable  
* Example client code in Python included  

---

## 🚀 Requirements

* Python 3.9+  
* `fastapi` + `uvicorn` (for the server)  
* `requests` (for client scripts)  
* `telebot` (optional, for Telegram bot integration)  
* `pyngrok` (optional, for local tunnels)  

`requirements.txt` example:

```txt
fastapi
uvicorn[standard]
requests
pyngrok        # optional for ngrok automation
python-telegram-bot  # or pyTelegramBotAPI
telethon        # optional
````

---

## Quick Start (Developer)

### 1) Run the Cloud API locally

```bash
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000
```

### 2) Expose it publicly (for development) using ngrok

```bash
ngrok http 8000
```

ngrok will give you a public URL like:

```
https://your-ngrok-url.ngrok-free.dev
```

Use this URL as `CLOUD_API` in your scripts.

> **Tip:** Run `ngrok authtoken <your-token>` once on your machine. The token persists until removed.

---

## API Endpoints

> Replace `{CLOUD_API}` with your server URL. Include the `key` parameter for secured endpoints.

### POST `/upload`

Upload a file via `multipart/form-data`.

**Request fields:**

* `key` — your API key
* `file` — the file content

**Example (curl):**

```bash
curl -X POST "{CLOUD_API}/upload" \
  -F "key=supersecret123" \
  -F "file=@users.json;filename=users.json"
```

**Response:**

```json
{ "filename": "users.json", "message_id": 123, "link": "https://t.me/..." }
```

---

### GET `/download/{filename}`

Download a stored file:

```
GET {CLOUD_API}/download/users.json?key=supersecret123
```

Returns the file content.

---

### GET `/list`

List all stored filenames:

```
GET {CLOUD_API}/list?key=supersecret123
```

**Response:**

```json
["users.json", "tokens.json"]
```

---

### DELETE `/delete/{filename}`

Delete a stored file:

```
DELETE {CLOUD_API}/delete/users.json?key=supersecret123
```

**Response:**

```json
{ "status": "deleted", "file": "users.json" }
```

---

### GET `/status` (optional)

Health check endpoint:

```
GET {CLOUD_API}/status
```

**Response:**

```json
{ "status": "ok" }
```

---

## Example Python Client

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
    print("Upload response:", res.status_code, res.text)

def download():
    res = requests.get(f"{CLOUD_API}/download/{CLOUD_FILENAME}", params={"key": CLOUD_KEY})
    if res.status_code == 200:
        open(LOCAL_DB, "wb").write(res.content)
        print("Downloaded successfully")
    else:
        print("Download failed:", res.status_code, res.text)

# Example usage:
ensure_local()
db = {}
with open(LOCAL_DB, "r") as f:
    db = json.load(f)
db["123"] = "example_user"
with open(LOCAL_DB, "w") as f:
    json.dump(db, f, indent=4)
upload()
```

---

## Security & Hardening Tips

* Use a strong random `CLOUD_KEY` in production
* Serve via HTTPS
* Rate-limit endpoints
* Sanitize filenames to prevent path traversal
* Limit file sizes (e.g., 50MB)
* Keep logs of uploads/deletes with timestamps and IPs

---

## Hosting Notes

* **ngrok:** fast for dev, URL changes on restart
* **Production:** deploy on Render, Railway, DigitalOcean, AWS, or GCP
* **Storage:** local filesystem is fine for small projects; for scale, use S3
* **Backups:** periodically backup files to another location or S3

---

## Cloud Flow Diagram (Textual)

```
┌─────────────┐        ┌─────────────┐
│ Local Script│ ─────> │ ZONE | Cloud│
└───────┬─────┘        └───────┬─────┘
        │                      │
        │ upload/download       │ store files
        ▼                      ▼
    users.json <─> cloud_files/
        │                      │
        └─────> Telegram Bot ──┘
```

---

## Suggested Repo Structure

```
ZONE-cloud/
│
├── README.md                 # This file
├── main.py                   # FastAPI server
├── example_client.py         # Client + bot examples
├── requirements.txt
├── .env.example              # Example env variables
└── README-images/            # Optional screenshots/diagram
```




---

dev : ZONE 

TELEGRAM USERNAME : @IEI_T
