

This is the file everyone sees first.
A great README makes your project look professional and interview-ready.


```markdown
#  Safe Code Executor (Python + Flask + Docker Sandbox)

A secure web-based Python code execution sandbox built using **Flask** and **Docker**.

User code runs *safely* inside a locked-down Docker container with:

- CPU timeout  
- Memory limits  
- No internet  
- Read-only filesystem  
- Read-only mounted script  
- Code size limit  
- Clean JSON API  
- Simple Web UI  

This project demonstrates **how to execute untrusted code safely**, a skill used in online judges, cloud sandboxes, interview platforms, etc.

---

#  Features

###  Safe Docker Execution
User code runs inside:

```

python:3.11-slim

```

with:

- `--memory=128m`  
- `--memory-swap=128m`
- `--network none`  
- `--read-only`  
- read-only volume mount  
- subprocess `timeout=10`  

###  Security Protections

| Protection | Status |
|-----------|--------|
| Infinite Loop Timeout | ✔ Blocks after 10s |
| Memory Bomb Limit | ✔ Container killed |
| Internet Disabled | ✔ DNS fails |
| Filesystem Read-only | ✔ Write attempts fail |
| Code Length Limit | ✔ max 5000 chars |
| No Host Access | ✔ container isolated |

###  Simple Web UI
Runs code directly from browser  
(`/ui` endpoint).

---

#  Project Structure

```

safe-code-executor/
│
├── app/
│   ├── main.py              # Flask API routes
│   ├── executor.py          # Docker execution sandbox
│   └── templates/
│       └── index.html       # Web UI
│
├── docs/
│   └── SECURITY_NOTES.md    # Security experiments & learnings
│
├── tests/                   # (manual tests)
│
├── requirements.txt
└── README.md

````

---

#  Installation & Setup

### 1. Clone the repository
```bash
git clone <your_repo_url>
cd safe-code-executor
````

### 2. Create virtual environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Start Flask server

```bash
python3 app/main.py
```

### 5. Open UI

Visit:

```
http://127.0.0.1:5000/ui
```

---

#  API Usage

### **POST** `/run`

**Request:**

```json
{
  "code": "print(2 + 2)"
}
```

**Response:**

```json
{
  "output": "4\n"
}
```

Errors return:

```json
{
  "error": "Execution timed out after 10 seconds.",
  "exit_code": -2
}
```

---

#  Security Learnings

See full notes:
`docs/SECURITY_NOTES.md`

You will find results of:

* `/etc/passwd` test
* Memory bomb test
* Infinite loop
* Write attack test
* Network isolation
* Docker filesystem isolation

Great for interviews & resume.

---

#  What I Learned

* How to run Python code inside Docker
* How to isolate untrusted code using Docker flags
* How to use timeouts, memory caps, and network isolation
* Importance of read-only filesystems for security
* How online judges (HackerRank, LeetCode) execute user code
* How to expose a clean API + simple UI for code execution

---





✅ Push to GitHub  
✅ Write commit messages  
✅ Add screenshots to README  
Just say the word!
```
