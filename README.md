
# 🌩️ ZONE | Cloud — Integration Guide

**Developer Rights:** @IEI_T  
**Purpose:** Connect your existing Python script or Telegram bot to the ready-to-use ZONE | Cloud.

**Cloud URL:**
If an error occurs, it may have changed. Please contact the developer.
```

[https://supertutelary-soberly-ezra.ngrok-free.dev](https://supertutelary-soberly-ezra.ngrok-free.dev)

````

> This cloud is already running. You do **not** need to create a bot or a server.  
> You only connect your existing bot/script to store and retrieve files.

---

## 📌 Required Library

To link your script or bot to the cloud:

```bash
pip install requests
````

> Only `requests` is required. No need for FastAPI, Uvicorn, or ngrok.

---

## 🔗 API Endpoints

Use the following URL in your script:

```python
CLOUD_API = "https://supertutelary-soberly-ezra.ngrok-free.dev"
API_KEY = "supersecret123"
```

| Endpoint               | Method | Description           |
| ---------------------- | ------ | --------------------- |
| `/upload`              | POST   | Upload a file         |
| `/download/{filename}` | GET    | Download a file       |
| `/list`                | GET    | List all stored files |
| `/delete/{filename}`   | DELETE | Delete a file         |
| `/status`              | GET    | Check cloud status    |

---

## 📝 How to Link Your Script or Bot

1. **Install `requests`**
   Your script or bot must have the `requests` library installed.

2. **Set Cloud URL and API Key**
   Assign the cloud URL and API key as variables in your script.

3. **Upload Files to Cloud**
   Use a POST request to `/upload` with your API key and the file you want to store.

4. **Download Files from Cloud**
   Use a GET request to `/download/{filename}` with your API key to fetch files.

5. **List Stored Files**
   Use a GET request to `/list` with your API key to see all files currently in the cloud.

6. **Delete Files**
   Use a DELETE request to `/delete/{filename}` with your API key to remove a file.

7. **Optional Health Check**
   Use a GET request to `/status` to check if the cloud is online.

---

## ⚡ Benefits of Connecting Your Bot/Script

* Works instantly with your existing Telegram bot or Python script.
* No need to host your own server.
* Unlimited file storage.
* Multiple scripts/bots can share the same cloud files.
* Simple integration using only `requests`.



DEV : ZONE

TELEGRAM USERNAME : @IEI_T
