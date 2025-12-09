

#  Safe Code Executor

A secure Flask + Docker–based sandbox that runs **Python** and **JavaScript** code safely in isolated containers.

Safe Code Executor is a secure sandbox system that runs untrusted Python and JavaScript code inside isolated Docker containers. 
It enforces strict security controls—timeout, memory limits, no network, and read-only filesystem—to ensure safe execution. Includes a simple web UI and execution history.

This guide shows **exactly how to run the project**, with **step-by-step instructions**, **command examples**, and **expected output** for every step.

---

##  Project Structure

Safe_Code_Executor/
│
├── app/
│   ├── main.py              # Flask server (routes, validation, UI)
│   ├── executor.py          # Docker sandbox executor
│   ├── history.json         # Stores last 10 executions
│   └── templates/
│       └── index.html       # Web UI
│
├── docs/
│   └── SECURITY_NOTES.md    # Notes from Docker security tests
│
├── requirements.txt         # Python dependencies
└── README.md                # Documentation


###  What each component does

| Component          | Description |
|-------------------|-------------|
| **main.py**       | API `/run`, `/history`, `/ui`, validates input, saves history |
| **executor.py**   | Runs code safely inside Docker with limits (timeout, RAM, no network, read-only) |
| **index.html**    | UI (textarea + language selector + output box) |
| **history.json**  | Last 10 executions saved with timestamp |
| **SECURITY_NOTES.md** | Results of your safety experiments |


---

##  System Architecture

The system contains 5 layers that work together to securely run untrusted code:

### 1️ User Interface (UI)
- HTML page where the user writes code.
- Send `{ language, code }` to the backend.
- Displays the output or error.

### 2️ Flask API (Backend)
- Receives code from UI.
- Validates:
  - Supported language (`python` or `javascript`)
  - Code length ≤ **5000 characters**
- Saves the request to history.
- Calls the Docker executor.

### 3️ Secure Docker Sandbox
Runs the user’s code inside an isolated container with:
- `--network none` → no internet  
- `--read-only` → cannot write any files  
- `--memory=128m --memory-swap=128m` → memory limit  
- Timeout: **10 seconds**  
- Python container: **python:3.11-slim**  
- JS container: **node:18-slim**

### 4️ Executor Module
- Writes code to temporary file.
- Mounts file into container (`script.py:ro`).
- Executes code with Docker.
- Captures:
  - `stdout`  
  - `stderr`  
  - `exit_code`

### 5️ History System
- Stores latest 10 executions in `history.json`.
- Accessible via `/history` endpoint.


---

##  Architecture Diagram

Flow:
User → UI → Flask API → Docker Executor → Container → Output → History → UI

### Architecture Diagram  










##  Execution Flow (Step-by-Step)

1. User writes code in the UI.
2. Browser sends POST request to `/run`.
3. Flask validates code length + language.
4. Code written to temporary file.
5. Executor chooses correct Docker image (Python / Node.js).
6. Docker container runs with:
   - timeout
   - memory limit
   - no network
   - read-only filesystem
7. Container outputs stdout / stderr.
8. Flask formats response → returns to UI.
9. Execution stored in `history.json`.
10. User can view history via `/history`.







#  Prerequisites

* Python 3.8+
* Docker Desktop running
* WSL2 (recommended for Windows users)
* Git

---

#  Getting Started (Step-by-Step)



---

## 1️⃣ Clone the Project

**What we do:** Get the project on your machine.
**Command:**

```bash
git clone <your_repo_url>
cd Safe_Code_Executor
```

**Expected Output:**

```
Cloning into 'Safe_Code_Executor'...
```

---

## 2️⃣ Create & Activate Virtual Environment

**Command (WSL/Linux):**

```bash
python3 -m venv venv
source venv/bin/activate
```

**Expected:** Shell changes to:

```
(venv)
```

---

## 3️⃣ Install Dependencies

```bash
pip install -r requirements.txt
```

**Expected Output:**

```
Successfully installed flask ...
```

---

## 4️⃣ Start Docker Daemon

**Action:** Open Docker Desktop → wait for **Docker is running**.

**Optional verification:**

```bash
docker run hello-world
```

**Expected:**

```
Hello from Docker!
```

---

## 5️⃣ Start the Flask Server

```bash
python3 app/main.py
```

**Expected Output:**

```
Using history file at: /.../app/history.json
Running on http://0.0.0.0:5000
```

---

#  API Testing (Step-by-Step)

---

## 6️⃣ Health Check

```bash
curl http://127.0.0.1:5000/
```

**Expected Output:**

```
Safe Code Executor API is running!
```

---

## 7️⃣ Run Python Code

```bash
curl -X POST http://127.0.0.1:5000/run \
 -H "Content-Type: application/json" \
 -d '{"language":"python","code":"print(2+2)"}'
```

**Expected:**

```json
{"output":"4\n"}
```

---

## 8️⃣ Run JavaScript Code

```bash
curl -X POST http://127.0.0.1:5000/run \
 -H "Content-Type: application/json" \
 -d '{"language":"javascript","code":"console.log(2+2)"}'
```

**Expected:**

```json
{"output":"4\n"}
```

---

#  Security Test Cases (with Expected Results)

These demonstrate the sandbox protection.

---

## 9️⃣ Infinite Loop Prevention (Timeout)

```bash
curl -X POST http://127.0.0.1:5000/run \
 -H "Content-Type: application/json" \
 -d '{"language":"python","code":"while True: pass"}'
```

**Expected:**

```json
{"error":"Execution timed out after 10 seconds.","exit_code":-2}
```

---

## 🔟 Memory Limit (OOM Kill)

```bash
curl -X POST http://127.0.0.1:5000/run \
 -H "Content-Type: application/json" \
 -d '{"language":"python","code":"x = \"a\" * 1000000000"}'
```

**Expected:**

```json
{"error":"","exit_code":137}
```

---

## 1️⃣1️⃣ Network Block

```bash
curl -X POST http://127.0.0.1:5000/run \
 -H "Content-Type: application/json" \
 -d '{"language":"python","code":"import urllib.request; urllib.request.urlopen(\"https://google.com\")"}'
```

**Expected:**
DNS/connection error in stderr (container has no network).

---

## 1️⃣2️⃣ File System Protection (Read-Only)

```bash
curl -X POST http://127.0.0.1:5000/run \
 -H "Content-Type: application/json" \
 -d '{"language":"python","code":"open(\"/tmp/hack.txt\",\"w\").write(\"hello\")"}'
```

**Expected:**

```
OSError: [Errno 30] Read-only file system
```

---

## 1️⃣3️⃣ Reading Container Files (Allowed)

```bash
curl -X POST http://127.0.0.1:5000/run \
 -H "Content-Type: application/json" \
 -d '{"language":"python","code":"print(open(\"/etc/passwd\").read())"}'
```

**Expected:** Contents of container’s `/etc/passwd`.

---

## 1️⃣4️⃣ Code Length Limit (5000 chars)

**What we do:** Submit a very long code string.
**Expected:**

```json
{"error":"Code too long. Maximum allowed length is 5000 characters."}
```

---

#  Web UI

Open the UI in browser:

```
http://127.0.0.1:5000/ui
```

### Features:

✔ Text area for code
✔ Language dropdown
✔ Run button
✔ Clear button
✔ Output console

---

#  Execution History

Check last 10 runs:

```bash
curl http://127.0.0.1:5000/history
```

**Expected:**

```json
[
  {
    "language": "python",
    "code": "print(2+2)",
    "output": "4\n",
    "error": "",
    "exit_code": 0,
    "time": "2025-02-12 21:00:00"
  }
]
```

---

#  Security Features Implemented

* **Isolated Docker container** per execution
* **Timeout** (10 seconds)
* **Memory limit** (128 MB)
* **Network disabled** (`--network none`)
* **Read-only filesystem** (`--read-only`)
* **Mounted file read-only** (`script.py:ro`)
* **Code length restricted** to 5000 characters
* **History log** (last 10 executions)

---

#  What I Learned

From this project:

###  Docker Security

* How to run untrusted code safely
* Memory limits, network blocking, read-only root FS
* How containers isolate file systems

###  Safe Code Execution Principles

* Why sandboxing is mandatory
* How infinite loops, memory bombs, file writes, and network calls can be dangerous
* How to restrict and monitor executions

###  Backend API Skills

* Building `/run` and `/history` endpoints
* Handling errors safely
* Returning proper JSON responses

###  Frontend Integration

* Simple UI with textarea + output console
* Making a user-friendly testing interface

---


