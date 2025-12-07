# 🌩️ ZONE | Cloud — Integration Guide (User-Based Edition)

**Developer Rights:** @IEI_T
**Purpose:** Connect your existing Python script or Telegram bot to the ready-to-use ZONE | Cloud with per-user authentication.

**Cloud URL:**

```text
https://supertutelary-soberly-ezra.ngrok-free.dev
```

> This cloud is already running. You do **not** need to create a bot or a server.
> You only connect your existing bot/script to store and retrieve files.
> Each user can upload files with a username and password, keeping their files private from others.

---

## 📌 Required Library

To link your script or bot to the cloud:

```bash
pip install requests
```

> Only `requests` is required.

---

## 🔗 API Endpoints

Use the following URL in your script:

```python
CLOUD_API = "https://supertutelary-soberly-ezra.ngrok-free.dev"
API_KEY = "CLOUDEZONE:512523524952740985RHIEWHRK3HK423O23IIO23U325"
username = "your username here"
password = "your password here"
```

| Endpoint    | Method | Description                                           |
| ----------- | ------ | ----------------------------------------------------- |
| `/upload`   | POST   | Upload a file **with username & password**            |
| `/download` | GET    | Download a file **by specifying username & password** |
| `/list`     | GET    | List all files for a **specific user**                |
| `/delete`   | DELETE | Delete a file for a **specific user**                 |
| `/status`   | GET    | Check cloud status                                    |

---

## 📝 How to Link Your Script or Bot

1. **Install `requests`**
   Your script or bot must have the `requests` library installed.

2. **Set Cloud URL and API Key**
   Assign the cloud URL and API key as variables in your script.

3. **Authenticate Your User**
   Each user must provide a `username` and `password` for every request.
   Files are stored per-user, so users cannot access others’ files.

4. **Upload Files to Cloud**
   Use a POST request to `/upload` with:

   * `username`
   * `password`
   * `key` (API_KEY)
   * `file`

5. **Download Files from Cloud**
   Use a GET request to `/download` with:

   * `filename`
   * `username`
   * `password`
   * `key` (API_KEY)

6. **List Stored Files for a User**
   Use a GET request to `/list` with:

   * `username`
   * `password`
   * `key` (API_KEY)

7. **Delete Files**
   Use a DELETE request to `/delete` with:

   * `filename`
   * `username`
   * `password`
   * `key` (API_KEY)

8. **Optional Health Check**
   Use a GET request to `/status` to check if the cloud is online.

---

## ⚡ Benefits of Connecting Your Bot/Script

* Works instantly with your existing Telegram bot or Python script.
* Each user has **private file storage** with username & password.
* No need to host your own server.
* Unlimited file storage.
* Multiple users can upload same file names safely (no collisions).
* Simple integration using only `requests`.

---

**DEV:** ZONE
**TELEGRAM USERNAME:** @IEI_T

---
